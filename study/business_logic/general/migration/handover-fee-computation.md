# Handover — Fee Computation (Additive Billing Engine)

**Cross-file dependencies — read these before deep-diving here:**
- Override/waiver mechanics (what happens when the computed number gets changed) → `handover-pricing-integrity-overrides.md`
- How a package suppresses `therapy_fee_in_paise` and links via `package_id` → `handover-packages.md`
- The session/encounter gap (architectural context for why the four-bucket design exists) → `handover-index.md` standing note

**Full-detail reference:** `README.md` §2 (Fee Restructuring), §5 (Schema/Architecture Notes), §6 (Configurability Architecture), §9 (Fee-Scope Configurability, Specialty Services, and Package Derivation)

---

## Visit Types — Schema Reality

`visits.visit_type CHECK (visit_type IN ('CONSULTATION', 'MACHINE_ONLY'))` already exists and is the real division, confirmed against the actual schema. This lives on `visits` — not on `clinics`, not on `patients`. The same patient can have `MACHINE_ONLY` one day and `CONSULTATION` the next; nothing above the individual visit row ever locks the billing model. The flexibility to mix billing models per patient/per visit is a side effect of where that column sits, not a deliberate feature that needs building.

**`MACHINE_ONLY`:** itemized, per-machine billing. Every machine/modality used (IFT, TENS, ultrasound, infrared, wax, others) is a `services` row; its per-clinic price lives in `clinic_service_prices`; the visit's charge is the sum of whatever `visit_services` rows exist for that visit. Already fully built — only seed data needed.

**CONSULTATION visits:** non-itemized. The rest of this file is about this type.

**Related — `clinics.default_visit_type`:** add as a simple pre-fill convenience for the visit-creation form. No enforcement needed (and the lock concept was explicitly set aside — add as a single RLS policy only when a specific clinic actually asks for it).

---

## The Three Visit States

Three states, confirmed by Om directly (not inferred). All three are specific to CONSULTATION visits.

| State | Trigger | Fee shape |
|---|---|---|
| **First-time** | First-ever touchpoint for this complaint | ₹350 exam-fee once (shared) + ₹200/300 per complaint if therapy taken that day |
| **Follow-up** | Normal subsequent visit | ₹200/300 per complaint being treated, no flat add-on |
| **Gap-return** | 10 consecutive missed days mid-course | ₹200/300 per complaint + ₹150 lapse penalty once (shared) |

**Confirmed:** the 150 lapse penalty is shared once across however many complaints reactivate together that day, same logic as the 350 exam fee being shared. A patient with two lapsed complaints: 200+200+150=550 (Om's own number, confirmed directly).

**Additive rule design (three rules, not branches):**
1. Always add the per-complaint therapy rate for every complaint treated.
2. If this is the first-ever billing touchpoint: add 350 once.
3. If this is the first touchpoint after a gap: add 150 once.

Rules 2 and 3 are mutually exclusive on any given visit. Neither depends on complaint count. Every future combination falls out of these same three rules automatically — no new branch needed.

**`consultation_type` column:** currently `CHECK (consultation_type IN ('FIRST', 'SUBSEQUENT'))` — no gap-return value exists yet. Widening this CHECK to add `'GAP_RETURN'` is a safe migration (existing rows already satisfy a more permissive rule). Must happen before any trigger can record which of the three states applied.

---

## The Four-Bucket Money-Column Redesign

**Why:** the old `consultation_fee_in_paise` is a single opaque bucket. Same-day split encounters (exam at 8am, therapy at 10am, same complaint, same day — genuinely two visit rows) need to coordinate whether the flat fee has already fired today on *another row*. A trigger can't do this without knowing which specific flat fee it's looking for. Splitting into named columns makes each fee independently checkable.

**The four columns replacing the old two:**

| Column | What it represents | Sharing rule |
|---|---|---|
| `exam_fee_in_paise` | The 350 first-touchpoint fee | At most once per patient per day across all visit rows |
| `lapse_penalty_in_paise` | The 150 gap-return add-on | At most once per patient per day across all visit rows |
| `therapy_fee_in_paise` | The per-complaint therapy rate (200 or 300 by tier) | No sharing — applies in full to every encounter |
| `services_total_in_paise` | Itemized add-ons (specialty services, machines) | Unchanged from current |

`grand_total_in_paise` becomes a four-term generated sum instead of two.

**The BEFORE INSERT trigger (same-day dedup logic):** before assigning `exam_fee_in_paise` or `lapse_penalty_in_paise` on a new visit, check: does a visit row already exist today for this same `patient_id` with that specific column already nonzero? Lookup scope: `patient_id + date` — NOT `patient_id + complaint_course_id + date`. The reason: the penalty covers whichever complaint reactivates first that day, and a second complaint treated later the same day doesn't pay it again even though it's a different complaint course. Same-day dedup is patient-scoped, not complaint-scoped.

**Race condition caveat:** the sequential trace (visit 1 charges the fee, visit 2 sees it and charges 0) only holds if the two inserts happen sequentially — which is the natural case for a receptionist submitting one at a time. If visit creation is ever built as genuinely concurrent parallel requests, both triggers could see "nothing yet" before either commits, and both charge. Not an active risk today; worth being aware of if that changes.

**Related constraint (enforce once the columns exist):** at most one therapy-charging visit per complaint per day (exam-only and therapy-charging visits aren't restricted against each other):
```sql
CREATE UNIQUE INDEX ON visits(complaint_course_id, date)
WHERE therapy_fee_in_paise > 0;
```
This is a hard rule, not a warning — if there's ever a genuine clinical reason for two therapy touches on the same complaint same day, it needs the same admin-override escape hatch as everything else, not a silent failure.

---

## Consultation-Fee Scope Configurability

**Problem:** this clinic charges one flat exam fee shared across however many complaints are examined same-day (`PER_VISIT_DAY`). A different business might charge per complaint instead (`PER_COMPLAINT`).

**Solution:** `clinics.consultation_fee_scope text CHECK (consultation_fee_scope IN ('PER_VISIT_DAY', 'PER_COMPLAINT')) DEFAULT 'PER_VISIT_DAY'`

The same setting governs the lapse penalty — same category of fee, same sharing question.

**Why `PER_COMPLAINT` is actually simpler to implement:** it only needs "has this complaint ever had its own first-touchpoint fee charged before?" — information sitting entirely on that one complaint course, no cross-complaint coordination. `PER_VISIT_DAY` additionally needs the same-day sibling-complaint check layered on top. The harder logic only turns on when the setting demands it.

**Possible future extension (not built now):** this could need to vary by tier as well as by clinic (Regular vs. Rehab might reasonably want different scopes at the same clinic) — if that's ever real, it belongs on the `clinic_tier_rates` table (see below), not the flat `clinics` column.

**Cleaner decoupling option:** a dedicated `clinic_fee_scopes(clinic_id, fee_type CHECK IN ('EXAM','LAPSE_PENALTY'), scope CHECK IN ('PER_VISIT_DAY','PER_COMPLAINT'))` table keeps scope narrowly self-contained and lets you choose the amount-representation (dedicated column, or services-row approach below) completely independently.

---

## Examination Fee as a Service (Validated Idea)

Modeling the examination fee as a `services` catalog row (price via `clinic_service_prices`, `0` for a clinic that doesn't charge it) reuses the same architecture already proven for machines and specialty services. A clinic that charges `0` just never shows it as a selectable item.

**Important:** this answers "how much does this clinic charge for an exam" (amount). It does NOT answer "does this charge share across same-day complaints or apply per complaint" (scope). Those are two independent axes — neither idea eliminates the need for the other. The `clinic_fee_scopes` table approach above keeps them independent cleanly.

---

## Open Tension — Joining Fee vs. Per-Complaint Exam Fee

**Needs Om's direct answer before the exam-fee trigger can be built.**

Currently confirmed and logged in `README.md`: a patient who was treated for a shoulder in January and comes back in June with a brand-new, unrelated ankle complaint pays a *fresh* 350 exam fee for the ankle — a new complaint gets its own exam fee, even for long-standing patients.

Om separately floated a "joining fee" concept — charged once, ever, per patient — that would NOT re-charge that returning patient in June. These are two meaningfully different behaviors:

- **Per-complaint exam fee** (current logged rule): new complaint = fresh exam fee, always.
- **Once-ever joining fee**: a returning patient with a new complaint pays nothing extra.

Om needs to confirm which behavior holds going forward. This cannot be inferred from the conversation — it is a genuine design fork.

---

## Rehab Tier — `clinic_tier_rates` Architecture

**Confirmed:** rehab-category complaints (fracture, post-surgery, post-op, neuro/paralysis) have genuinely separate rate tables covering both daily visit pricing and package pricing. "Everything changes" for a rehab complaint — Om's own words.

**Exam fee tier-awareness:** the ₹350 exam fee is currently confirmed tier-independent (flat 350 regardless of complaint category). Decision: build it tier-aware anyway, as a column on `clinic_tier_rates`, with both tiers set to 350 for now. Costs nothing structurally since that table exists regardless for the daily/package rates; avoids a migration later if the numbers diverge.

**`clinic_tier_rates` shape:**
```
clinic_id        -- FK to clinics; locality-scoping falls out naturally (same pattern as clinic_service_prices)
tier             -- text CHECK IN ('REGULAR','REHAB') or FK to a tier lookup table (see below)
daily_rate_in_paise
package_rate_in_paise
exam_fee_in_paise    -- both rows = 35000 for now; here to avoid a migration later
```

**Tier representation on `complaint_courses`:** two options, not decided yet:
- A `CHECK`-constrained enum column — cheap, matches `services.category` precedent, but adding a third tier later means a migration.
- A small lookup table FK'd from `complaint_courses` — no migration needed to add a third tier, matches the `clinic_service_prices` pattern better.

Given tiers are explicitly "his business, he decides," and given the whole point of this architecture is avoiding rework later, the lookup-table version leans more future-proof. Worth a deliberate written decision, not an inference.

**On env files for rates:** explicitly rejected. An env file is read once at process start — a rate change requires a redeploy, not an `UPDATE`. It also can't be joined against in SQL, so a trigger computing a fee from it would need to mirror the values into the database anyway, making the env file a redundant second source of truth. Per-clinic, queryable, admin-editable rates are a database's job, not an env file's.

---

## Specialty Services Pricing

**Specialty services (dry needling, cupping, laser therapy, ISTM, others — full list TBD via screenshot)** sit outside the flat-fee bundling of CONSULTATION visits entirely, regardless of visit type. Each has its own separate price chart via `clinic_service_prices`.

**Multi-complaint representation:** a single cupping application covering neck+back+shoulder in one sitting has no clean home under the current schema (`visits.complaint_course_id` is singular). Three options considered:

1. A fully separate `special_treatment_events` table with a junction to whichever complaint courses it covered. Matches the real event as one row; costs a second place billing-assembly logic must check.
2. **Recommended:** Extend `visit_services` — keep the application attached to whichever visit it naturally belongs to, add a small junction (`visit_service_complaint_links`) recording which *other* complaint courses that same application also covered. Reuses the existing anchor; billing-assembly logic that already reads `visit_services` picks these up automatically.
3. Bolting multi-complaint awareness directly onto `visit_services` rows without a junction — stretches an existing column's meaning, set aside.

**Pricing mode** (flat-once vs. per-complaint): belongs on the *service itself* (or a clinic override via `clinic_service_prices`), not a single clinic-wide switch — different specialty services could plausibly want different multi-complaint pricing philosophies at the same clinic.

**Package exclusion hook:** `services.category CHECK (category IN ('STANDARD','PREMIUM'))` already exists. Seed specialty services as `'PREMIUM'`; package-coverage logic simply filters `WHERE category != 'PREMIUM'`. This is an existing hook, not a new mechanism.
