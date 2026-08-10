# Business Rules Log — Client Conversations (Clinic Owner)

**Log covers:** Aug 8 evening + Aug 9 early morning conversations with the clinic owner, captured via voice recording and summarized through Meta AI.

**Purpose:** a running record of new business requirements surfacing directly from the client, separate from the schema-module gaps logs (`module-N-gaps-log.md`), which track bugs in the existing migration rather than requirements that aren't in the schema at all yet. Entries here graduate into the schema study plan / migration once a design decision actually gets made and written down.

**Reliability note, per Om:** these were captured mid-discussion — the owner and Om were still working several of these out together in real time ("is it this, or is it that?"). Treat figures especially as provisional until Om confirms them. Where the source material contradicts itself, that's flagged explicitly below rather than silently resolved one way or the other.

---

## 1. Package Management Features

**Hold.** Admin/owner-only action. The record should not carry an explicit "on hold" flag or duration — attendance shows as separate date-range segments instead (e.g. "first 6 days: Jul 1–6," "remaining 4 days: Aug 9–12"), and the gap between segments is how a hold reads in the history, implicitly. Current `packages` columns (`day_log jsonb`, `duration_days`, `expiry_date`, `attended_days`/`missed_days`) don't represent multiple discontinuous active segments today — this needs its own design pass, not solved here.

**Cancel + Refund.** Admin/owner-only, same restriction as Hold. Agreed shape: refund = package total − (days used × some per-day rate), and this same formula is meant to cover both an early "emergency, can't continue" cancellation and a package that's mostly used with some missed days along the way. Which rate applies to "days used" is NOT settled — see the open contradiction in Section 3.1.

**Flag / Red Zone.** Manual only — Om was explicit the app should never auto-flag; a staff/admin member flags a patient and types a reason (disputes, abusive behavior, "has a lawyer, be careful" were the examples given). Needs to be visually unmissable on the patient's profile and needs flagged patients surfaced near the top of some list, so staff don't miss it years later. Stated purpose is closer to institutional memory/character-reference than clinical alerting. Open design fork: extend `patient_alerts.type` (currently `'ALLERGY'|'FALL_RISK'|'DNR'|'OTHER'`) to cover this, or give it its own `patient_flags` table mirroring the same owner_id-derivation pattern. The alerts table today reads as clinical; mixing "penicillin allergy" and "disputed a bill" into one list and one UI treatment is a real question, not just a schema one.

---

## 2. Fee Restructuring

**Visit types.** Already in the schema as-is — `visits.visit_type CHECK (visit_type in ('CONSULTATION', 'MACHINE_ONLY'))`. Confirmed by Om's description as the real division, not a new concept to add. The two types bill completely differently.

**MACHINE_ONLY visits.** Itemized, per-machine billing. Every machine used (IFT, TENS, ultrasound, infrared, wax therapy, others) has its own price in a catalog; a visit's charge is the sum of whichever machines were actually used, like a cart. Maps directly onto the existing `services` + `visit_services` + `clinic_service_prices` architecture — confirmed by Om directly, not just an inference.

**Doctor-directed visits — still needs a settled name.** Om flagged "consultation therapy" as a placeholder, not final. This is the non-itemized visit type: the clinician has full discretion over which machines a complaint gets, all covered by one flat fee rather than priced per machine. A small set of specialty services sit outside this flat-fee bundling entirely, regardless of visit type — dry needling, cupping therapy, and laser therapy were named as examples, each with its own separate price chart. A screenshot is coming to clarify the full list.

*Old model (doctor-directed visits):* first-time consultation = ₹300 flat, bundling that day's exam and therapy into one number, clinician's choice of machines included. Subsequent/follow-up visits = ₹200 flat, same bundling logic. The specialty carve-outs sat outside this in both cases.

*New model (doctor-directed visits):* first-time consultation = ₹350, now exam-only, no therapy included. If the patient also takes therapy that same day — the default; most patients do unless they explicitly decline — therapy bills separately, **per body part / per complaint**, at ₹200 each, clinician's choice of machines still included within that flat per-complaint rate (not itemized by machine). Two complaints treated same day = 350 (once, shared) + 200 + 200 = ₹750. Subsequent/follow-up visits stay ₹200 per complaint being treated that day — unchanged from the old model; only the first-time bundling changed. Specialty carve-outs are unchanged in both models.

**Gap-return policy (10 consecutive missed days).** Specific to this clinic — Om flagged it himself as unusual and isn't sure how to make it scalable to other clinics later. If a patient misses sessions for 10 *consecutive* days mid-course (a lapse inside an ongoing complaint, not "10 days since ever first visiting"), the next visit on return is charged a fresh ₹350 — but as a courtesy, the doctor folds that day's therapy into the 350 rather than charging a separate 200 on top, unlike a true first-time patient. A third state, confirmed directly rather than inferred: first-time (350 exam-only + separate 200 if therapy taken), normal follow-up (200 flat), gap-return (350 flat, therapy included). Two pieces still open, in Om's own words, not resolved:
- If a patient has two active complaints and both lapse past 10 days, does the reset ₹350 apply once (shared, like a normal consultation) or per complaint (350+350)? Not yet asked of the owner.
- The reset charge is waivable at admin discretion — loyalty, urgency, a serious personal circumstance. Needs to work as a manual override on top of whatever computes the default charge, not as a rule a trigger rigidly enforces with no escape hatch. Same shape as the refund flow's "system shows the calculated number, human enters the final figure" pattern from the original package conversation — worth building both the same way rather than as two unrelated mechanisms.

**Rehab tier — confirmed, not inferred.** Rehab-category complaints (fracture, post-surgery, post-op, neuro/paralysis) have a genuinely separate rate table covering both daily visit pricing and package pricing — "everything changes" for a rehab complaint, per Om directly. Whether the ₹350 exam fee itself varies by tier, or stays flat regardless of complaint category, hasn't been stated either way — **flagged as open**, not assumed in either direction.

**Old two-column mapping, confirmed.** `clinics.consultation_fee_first_in_paise` / `consultation_fee_subsequent_in_paise` (300/200) is confirmed as exactly the old bundled model. Under the new rule that becomes: a first-time exam-only fee (350) plus a separate per-complaint therapy fee (200 regular-tier default) that used to be folded into the 300 and no longer is. Om explicitly said not to fixate on the exact rupee figures from here on — they're likely to keep moving before this is final with the owner. The stable part to design around is the shape: decoupled exam fee, per-complaint therapy fee, tier-dependent therapy/package rates, a third gap-return state, and itemized machine-only billing as a fully separate visit path.

---

## 3. Contradictions in the Source Material — Need Om's Confirmation, Not an Inference

**3.1 — RESOLVED: refund deducts at the regular rate.**
Om confirmed directly, overriding the transcript: early cancellation deducts used days at the **regular, non-discounted rate** — not the package's per-day rate. The transcript's ₹810 worked example (which had deducted at the discounted ₹270/day rate) is confirmed incorrect; treat it as the AI summary catching an unsettled point mid-conversation rather than the final rule. The "discount forfeited on early exit" rule text was the accurate read all along.

**3.2 — Superseded by direct explanation.**
The original flag here was about the transcript's confusing "old rule" phrasing for gap-returns. Om has since walked through the actual mechanics directly — see the Gap-Return Policy entry in Section 2 — so the transcript's internal wording issue no longer matters; the real rule is documented from the source now, not inferred from an unreliable summary.

**3.3 — CONFIRMED: packages are tier-aware.**
Om confirmed directly: rehab-category complaints carry separate pricing for both daily visits and packages. This was a speculative cross-reference in the original entry; it's settled now. Packages need the same tier reference `complaint_courses` gets.

---

## 4. Configuration / Business Logic Module — Direct Answer

No — this has never come up before, under this name or any equivalent concept, anywhere in `backend-schema-iterations-corrected.md`, `supabase_migration.md`, `codebase-context.md`, or either schema study plan. This conversation is the first time it's been discussed.

Closest existing prior art, and why each falls short:
- `clinics.config jsonb` — exists, but its own comment scopes it to "rarely accessed, low-frequency reads" boolean feature flags (`inventoryEnabled`, `appointmentsEnabled`). Pricing gets read on every billing calculation — the opposite case — so this was never meant to hold what's being discussed now.
- `clinics.consultation_fee_first_in_paise` / `consultation_fee_subsequent_in_paise` — the one real piece of prior art for "the owner sets his own numbers per clinic," but two fixed columns representing exactly the old bundled model. No room for decoupled consultation/therapy, multiple tiers, or per-complaint summed billing — this pair is obsolete under the new rules, not extendable. (Small aside: the seed defaults on these two columns are ₹300/₹200, not the ₹350 figure discussed now — a placeholder mismatch worth updating regardless of the bigger restructuring.)
- `clinic_service_prices` — a genuine per-clinic override table with base-price fallback, but scoped to the `services` catalog specifically, not consultation fees or tier rates. The *pattern* (base value + per-clinic override + COALESCE) is worth reusing — see Section 5.
- `services.category CHECK (category in ('STANDARD','PREMIUM'))` — a fixed two-value enum tier, structurally similar in shape to what Regular/Rehab tiering would need, though it doesn't carry its own rate.

Today, "configurable" means two hardcoded numbers per clinic. Everything in Section 2 is new schema territory, not an extension of something partially built.

---

## 5. Schema / Architecture Notes — Brainstorm, Not a Decision

*Nothing here is settled. Treat it the way the invoice/session Option A/B/C decision gets treated — written up and decided deliberately, not inferred from a chat reply.*

**Machines are basically already solved.** Seed each machine/modality as a `services` row, let `clinic_service_prices` handle any per-clinic price differences, and "which machines were used in this visit" is exactly what `visit_services` already represents. No new junction table needed — just seed data and a clinic-level flag for whether that clinic bills this way.

**The therapy tier fee might belong in the same place, not a new `visits` column.** Rather than adding a dedicated `visits.therapy_fee_in_paise`, the base session itself could be a `services` row too — "Regular Physio Session" at 200, "Rehab Physio Session" at 300 — auto-attached to `visit_services` based on the complaint's tier, alongside whatever optional add-ons get logged normally. Reuses the catalog + override architecture that's already built, rather than inventing a third money bucket on `visits` next to `consultation_fee_in_paise` and `services_total_in_paise`. A new column would also work, just duplicates a pattern that already exists elsewhere.

**Complaint tiers need a representation:** either a CHECK-constrained enum on `complaint_courses` (cheap, matches the `services.category` precedent, but a new tier later means a migration) or a small lookup table FK'd from `complaint_courses` (closer to `clinic_service_prices`'s shape, no migration needed to add a third tier). Given tiers are explicitly "his business, he decides," and given the point of this whole conversation is avoiding rework later, the lookup-table version leans more future-proof — but this is exactly the kind of call worth making in writing, not deciding here.

**The shared ₹350 is the same open problem as the invoice/session gap already in the M1 log.** `invoices.visit_id` being a singular FK, and the schema having no session/encounter concept, was already flagged before this conversation — Option A (one invoice per visit, frontend batches), B (a junction table), C (session as frontend-only, never persisted, leaning direction at the time). The rule that consultation is shared across every complaint billed the same day is the *same* "what counts as one session" question, arriving from the billing side instead of the invoicing side — and it makes Option C harder to hold onto as-is, since something (a trigger, most likely) now needs to know "is this visit the first billing touchpoint of today's session" to decide whether ₹350 applies, which is exactly the kind of state Option C says shouldn't be persisted anywhere. Worth reopening that decision with this new information before committing to either the tier design or the visit-fee design above — they both hang off the same unresolved question.

**Packages likely need the tier link too,** per 3.3 — if that cross-reference is real, wherever package pricing ends up living needs the same tier reference `complaint_courses` gets.

**Not addressed here:** Hold's date-segment representation. Not enough detail yet on how `day_log` actually gets populated and read to responsibly propose a fix in the same pass as everything else — feels like its own focused design conversation once the rest of this settles.

---

## 6. Configurability Architecture — Coupled vs. Decoupled

*Still brainstorm-stage, same caveat as Section 5 — write this up and decide deliberately when it's time, don't treat anything here as settled.*

**The constraint, as Om set it.** No configuration UI for the MVP. Every rate and rule stays a fixed, seed-level value, editable only through direct database access. The open question isn't whether to build configurability now — it's how to shape the schema so a future config module touches as few tables as possible when it does get built.

**The actual test, not a blanket rule either way:** does this specific value vary along more than one dimension at once? A value that's genuinely 1:1 with its natural owner row belongs as a plain column on that row — no table needed, no over-engineering. A value that varies along two or more dimensions simultaneously (by clinic *and* by tier, say) can't be represented as a column without the column count growing every time a new tier or a new clinic-specific case shows up — exactly the maintenance burden this whole conversation is trying to avoid later.

**Applying that test to what's actually on the table:**
- Machine prices, and the specialty always-separate services (dry needling, cupping, laser): vary by clinic *and* by service — two dimensions. Already solved. `services` (base catalog) + `clinic_service_prices` (per-clinic override, `COALESCE`-falling-back to base) is exactly this pattern, already built, already proven elsewhere in the schema. Just seed more rows — no new structure.
- The ₹350 exam fee, if it turns out tier-independent (still open — see Section 2): stays a plain column on `clinics`, same shape as `consultation_fee_first_in_paise` today. One dimension (clinic only) → a column is correct, not under-engineered.
- Therapy day-rate and package rate: vary by tier for certain (200 vs 300, confirmed), and possibly by clinic too if the locality idea below ever gets used. Two dimensions in waiting — same shape as the services problem, different subject. This is the one place worth a small dedicated table — something like `clinic_tier_rates` (clinic_id, tier, daily_rate_in_paise, package_rate_in_paise) — rather than a growing set of columns on `clinics` (`regular_daily_rate`, `rehab_daily_rate`, `regular_package_rate`, `rehab_package_rate`, and a fresh pair every time a third tier shows up).

**On locality:** not actually a new problem — the same `clinic_id`-scoping pattern already used everywhere else in this schema (`clinic_service_prices`, the consultation-fee columns, the whole RLS model). Whatever ends up holding the tier-rate matrix just needs a `clinic_id` column and it's already locality-aware, same as `clinic_service_prices` already is. Nothing new to invent.

**On the env-file idea:** worth steering away from, specifically. An env file is read once at process start — a receptionist's screen and a database trigger computing a bill wouldn't see a change until a redeploy or restart, not a simple `UPDATE`. It also can't be joined against in SQL, so a trigger trying to look up "this complaint's tier rate at this clinic" would have no way to read it directly — the values would need to be mirrored into the database anyway to be usable, making the env file a redundant second source of truth rather than a simplification. Env files are the right tool for deployment secrets and values that are genuinely the same for every user of the whole app; per-clinic, queryable, dashboard-editable business rates are a database's job.

**On "map each number to its specific table":** right some of the time, not universally. Correct for the genuinely 1:1 values — services, and the flat clinic-level fee if it stays tier-independent — and it's already what the schema does today for those. It runs into trouble specifically where a value depends on more than one thing at once, which is exactly the tier situation. A small table scoped to that one multi-dimensional case isn't the same thing as "a separate table with all the numbers" in the way a single sprawling generic settings table would be — it's one narrowly-scoped table for one specific kind of variation, much cheaper to reason about later than either a pile of columns or one giant key-value blob.