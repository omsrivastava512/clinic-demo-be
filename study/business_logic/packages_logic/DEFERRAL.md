## PKGDEFER-01 — Phase 1: packages/core-engine touchpoint map

Read everything before mapping. `supabase_migration.md` was already sitting in the project files — v11, matches what every handover doc assumes as baseline (no `visits.package_id`, no `'Cancelled'` on `packages.status`, `duration_days` still `NOT NULL` — all consistent). No need to fetch it separately.

`specialty-decision-log.md` and `package_decision_log.md` were uploaded but not inlined, so I read both off disk before doing anything else. Worth flagging plainly: they move meaningfully past the five handover docs on several points that matter for this exact evaluation — not contradictions of intent, but later resolutions the handover docs don't yet reflect. Two concrete examples: `handover-packages.md`'s Priority 2 has the trigger *incrementing* `attended_days`; `package_decision_log.md` confirms that was reconsidered — it's a read-time `COUNT(*)`, not trigger-maintained, specifically because a live count self-corrects and a maintained counter doesn't. And `handover-packages.md`'s package-exclusion note ("`WHERE category != 'PREMIUM'`") is corrected in both `handover_specialty_context_for_packages.md` and `specialty-decision-log.md` Entry 1 — exclusion is automatic (different money bucket), not a filter. Full closures/waivers/mode-config system, the `CALENDAR_BASED`/`VISIT_BASED` naming split, the lazy two-touchpoint status flip — all of that is now confirmed in `package_decision_log.md`'s own "Resolved" section, which is where I'm anchoring the map below rather than the older handover text.

The diagram above is the skeleton — four touchpoints running through the middle, one flagged. Details below.

### 1. `package_id` FK + lookup trigger — and the build order that governs everything else

`visits.package_id` (nullable) doesn't exist in v11. Confirmed to be added. The `BEFORE INSERT` trigger, whenever `therapy_fee_in_paise` would be nonzero, does a single-row lookup — `packages WHERE linked_complaint_id = complaint_course_id AND status = 'Active'` — sets `package_id`, zeroes `therapy_fee_in_paise`. Because `attended_days` is now read-time, that's the trigger's *entire* job: one lookup, one column write, all on `visits` itself. No cross-table increment. That's a narrower footprint than the original handover doc described.

`specialty-decision-log.md` Entry 9 gives the authoritative build order, reasoned from the same alphabetical-trigger-ordering bug already hit once (Iteration 5, invoices): **four-bucket columns → `package_id` trigger + partial unique index → override columns → `session_id`**. The partial unique index (`idx_one_active_package_per_complaint`) has to exist *before* the trigger deploys — Postgres `SELECT INTO` doesn't error on multiple matches, it silently picks one. This ordering is the direct answer to "what does core need to expose before packages can plug in cleanly."

### 2. `therapy_fee_in_paise` — the actual coupling point into the four-bucket redesign

The trigger touches exactly one core money column. Never `services_total_in_paise`. That's architecturally narrow — specialty/machine charges live in a bucket the package trigger never reaches, which is *why* package exclusion of specialty services is automatic rather than a filter (Touchpoint 2 and the correction in item above are the same fact). `exam_fee_in_paise` and `lapse_penalty_in_paise` are explicitly unaffected by package status — flat charges regardless of an active package. Worth treating as an invariant to protect whenever this gets built.

### 3. Attendance & lifecycle — the real size of the module, and one open contradiction

This is not a thin hook. It's most of what "Packages" actually is: two new tables (`clinic_closures`, `package_day_waivers`), a three-layer mode config (`clinics.default_package_mode`, `patients.default_package_mode_hint`, `packages.package_mode` with a conditional CHECK mirroring `chk_consultation_type`), `visits_promised` alongside a newly-nullable `duration_days`, `recurring_closed_weekdays` generalizing `exclude_sundays`, `closure_type` on closures, the cancel/refund override columns, `'Cancelled'` added to `packages.status`, and two new unique constraints. Lazy status flip at two touchpoints (visit-link attempt, detail-view open) — no cron, consistent with the rest of the schema's philosophy.

The flagged item: `package_decision_log.md` §3(a) confirms a hard constraint —
```sql
unique (complaint_course_id, date)
```
— reasoning that it "formalizes something the fee-restructure design already implies." But `handover-fee-computation.md` explicitly describes the opposite as a real, designed-for case: *"exam at 8am, therapy at 10am, same complaint, same day — genuinely two visit rows."* And `handover-index.md`'s own standing note already names this exact tension unresolved: *"'one visit per complaint per day' as a hard rule conflicts with confirmed real usage... the right boundary is session, not day."* So one track confirmed a constraint that a different, already-written note says breaks a confirmed scenario. This isn't something I'm resolving here — it needs an explicit reconciliation before it becomes schema, independent of whatever happens with the Packages timing decision, since the fee-computation four-bucket work is presumably shipping regardless.

### 4. Invoice line items / package purchase

This is being driven by the Specialty track, not gated on the Packages decision — `invoice_line_items` (superseding the old Option A/B/C indecision) is likely getting built either way. That's favorable for blast radius: the ₹0 package-consumption line-item slot would exist as schema surface whether or not Packages ships now. Package *consumption* = ₹0 line item tied to `package_id`. Package *purchase* (the ₹5,000 sale event) is a separate, still-open concern needing its own branch in a proposed mutual-exclusion CHECK (`visit_id` / `treatment_event_id` / `package_purchase_id`). Ghost-invoice prevention is a deferred constraint trigger (`AFTER INSERT ... DEFERRABLE INITIALLY DEFERRED`) checking line-item existence and sum-match at commit — chosen specifically to match the schema's all-trigger idiom over an RPC. Confirmed correction: line items don't carry `complaint_course_id` directly — attribution derives from source, since one specialty event can legitimately cover multiple complaints.

### Cross-cutting: the `patient_clinic_access` second entry point

Not one of the four, but load-bearing for the deferral decision. `patient_clinic_access` is provisioned *only* through `process_new_complaint_course()`. Two independent tracks hit the identical gap: Specialty's "shallow patient / service-seeker" (`specialty-decision-log.md` Entry 10 — someone treated with no complaint course, invisible under staff RLS as currently designed) and Packages' "walk-in buys a package before a complaint course exists" (`handover_specialty_context_for_packages.md` §4). Since Specialty is apparently moving forward, whatever second-provisioning mechanism it builds for its own case is a real candidate for reuse when Packages' upfront-purchase case eventually needs the same thing — worth keeping in view for Phase 2 rather than treating as two separate future problems.

### The scope question this whole evaluation actually hinges on

"Defer the Packages module" splits into two different blast-radius problems: **(a)** `packages` stays exactly as-is in v11 — dormant, untouched, and Phase 2 evaluates the cost of adding all four touchpoints later; or **(b)** some subset of the cheap, foundational pieces (the nullable `visits.package_id` column, the partial unique index, maybe the mode-config CHECK shape) gets pre-built now as inert hooks specifically to shrink that later cost. Which of these "defer" means changes what gets evaluated in Phase 2 — this is exactly what the Structural / Safety-net / Performance classification per touchpoint is for, not a single verdict on "Packages" as one unit.

### Readiness

Full context: v11 baseline, all five handover docs, both decision logs, and the existing project logs (`DECISIONS.md`, `GAPS.md`, `LOGS.md`) cross-referenced against each other rather than trusted individually. Two things came out of that cross-referencing that aren't resolved by mapping alone — the split-encounter/one-visit-per-day conflict, and the (a)/(b) scope question above. Both are more naturally entries in `package_decision_log.md`'s Open section than something to solve here — happy to draft that addendum text if useful once Phase 2 gets to them.

Ready for the detailed evaluation whenever you want to move to it — either straight into Phase 2, or resolving the two flags first if you'd rather clear them before scoring anything.

---

## PKGDEFER-02 — Phase 2: blast radius, hooks & recommendation

One dependency before the four questions: several answers below hinge on how the split-encounter conflict from Phase 1 resolves. I'm answering under the assumption that the per-complaint-per-day constraint should ship **scoped to therapy-charging visits only** (matching `handover-fee-computation.md`'s partial-index version), not the blanket version `package_decision_log.md` marks as resolved — reasoning is in Question 2. That's a recommendation, not a fact; it needs Om's sign-off since it's a real clinical-workflow question, not a technical one. Everything else below holds regardless of which way that resolves.

### 1. Exact blast radius — `visits`, `invoices`, `complaint_courses`

**`visits`** — two items, very different weight:
- `package_id uuid references packages(id)`, nullable: pure `ADD COLUMN`, no backfill, no behavior change for any existing row. Cheapest possible migration.
- The `derive_package_id_and_apply_coverage` trigger: this is where the real cost lives, and it's not a schema cost — it's an *ordering* cost. Pay-Per-Visit already ships a `BEFORE INSERT` trigger on `visits` that computes `exam_fee_in_paise`/`lapse_penalty_in_paise`/`therapy_fee_in_paise` via the same-day dedup logic. The package trigger has to run **after** that one — it zeroes `therapy_fee_in_paise`, which has to already hold its correct computed value first, or it's zeroing a default instead of a real number. Postgres fires same-event `BEFORE` triggers alphabetically unless you control for it. This schema has already been bitten by exactly this failure mode once — the `invoices_clinic_id_guard`/`invoices_generate_number` alphabetical-ordering bug in Iteration 5. If the fee-computation trigger isn't authored with this in mind, "add packages later" stops being an additive change and becomes an edit to already-shipped, already-trusted billing logic. That's the real blast radius on `visits` — not the column, the ordering contract.
- The per-complaint-per-day constraint is the third item and it's genuinely structural, covered in Question 2.

**`invoices`** — none. Everything Packages needs routes through `invoice_line_items`, a new table Specialty is driving regardless of the Packages timeline. `invoices.amount_in_paise` and `invoices.visit_id` don't need to know a package exists.

**`complaint_courses`** — no *Packages-specific* blast radius, but a shared-dependency trap worth naming here and expanding in Question 4: the tier representation (`clinic_tier_rates` + however `complaint_courses` references it — CHECK-enum vs. lookup table, still explicitly undecided) is being chosen for Pay-Per-Visit's `therapy_fee_in_paise` needs, but `README.md` §3.3 confirms Packages needs the identical tier reference for `package_rate_in_paise`. Get this representation right once and both consumers are served for free. Get it wrong optimizing only for Pay-Per-Visit's narrower needs, and it's a migration against a table that by then carries real clinical history on every row.

### 2. Pluggability hooks — what ships with Pay-Per-Visit today

| Hook | Lives on | Cost today | Ship now? |
|---|---|---|---|
| `package_id uuid references packages(id)`, nullable | `visits` | Zero — stays NULL until Packages activates | Yes |
| `idx_one_active_package_per_complaint` partial unique index | `packages` | Near-zero — rejects a state that's already invalid regardless of Packages | Yes |
| Per-complaint-per-day uniqueness, **scoped to `WHERE therapy_fee_in_paise > 0`** | `visits` | Real, one-time, but non-deferrable — see below | Yes |
| `package_purchase_id uuid references packages(id)`, folded into a 3-way (not 2-way) mutual-exclusion `CHECK` from day one | `invoice_line_items` | Zero — stays NULL until Packages activates | Yes |
| Fee-computation trigger authored as **one function** with an internal staging comment, not two competing triggers | `visits` (authoring discipline, no DDL) | Zero extra migration cost | Yes |
| `'Cancelled'` status value, cancel/refund override columns | `packages` | Cheap anytime — text+CHECK, the schema's own established pattern | No |
| `clinic_closures`, `package_day_waivers` | new tables | Isolated, no FK from any Pay-Per-Visit table | No |
| Mode config (`package_mode`, `default_package_mode`, patient hint) | `packages`, `clinics`, `patients` | Isolated | No |

Concrete DDL for the four items I'd ship now, in migration order:

```sql
-- visits.package_id — dormant hook, zero cost, no trigger attached yet
alter table visits add column package_id uuid references packages(id);

-- packages — enforce "at most one active package per complaint" now,
-- while packages presumably has few or zero rows. Retrofitting this after
-- months of real (possibly-already-duplicate) package data is a data-
-- cleanup project, not a migration.
create unique index idx_one_active_package_per_complaint
  on packages (linked_complaint_id)
  where status = 'Active';

-- visits — per-complaint-per-day uniqueness, scoped to therapy-charging
-- visits only. This keeps the confirmed exam-then-therapy split-day
-- pattern legal (two rows, only one has therapy_fee_in_paise > 0) while
-- still giving the package trigger an unambiguous single row to zero.
create unique index uq_one_therapy_visit_per_complaint_per_day
  on visits (complaint_course_id, date)
  where therapy_fee_in_paise > 0;

-- invoice_line_items — third source type present from day one, dormant
alter table invoice_line_items add column package_purchase_id uuid references packages(id);
alter table invoice_line_items add constraint chk_line_item_source check (
  (visit_id is not null and treatment_event_id is null and package_purchase_id is null)
  or (visit_id is null and treatment_event_id is not null and package_purchase_id is null)
  or (visit_id is null and treatment_event_id is null and package_purchase_id is not null)
);
```

Why the third `visits` constraint deliberately overrides what `package_decision_log.md` marks "Resolved": that entry justifies the blanket `unique(complaint_course_id, date)` as *"formalizes something the fee-restructure design already implies."* But `handover-fee-computation.md` says the opposite in plain language — *"exam at 8am, therapy at 10am, same complaint, same day — genuinely two visit rows"* — and `handover-index.md`'s own standing note independently names this exact tension as unresolved. The blanket version was apparently confirmed without cross-referencing the fee-computation doc's split-encounter language — precisely the "documents disagree, check the source" failure mode this project has hit before. The partial version is the strictly safer default: it can be tightened to blanket later if real data cooperates, but starting blanket forecloses the split-encounter case the moment Pay-Per-Visit goes live, and *that's* the one still needing Om's word, not the SQL shape.

One thing already working in your favor here: `clinic_tier_rates`'s proposed shape (`clinic_id, tier, daily_rate_in_paise, package_rate_in_paise, exam_fee_in_paise`) already carries a `package_rate_in_paise` slot per `handover-fee-computation.md` — that hook is already designed, not something this evaluation is inventing.

### 3. Downstream invoicing compatibility

Two findings, one reassuring, one worth flagging precisely.

**Reassuring:** if `package_purchase_id` ships now (Question 2), deferring package purchases has *zero* impact on `invoice_line_items`' shape — the column just sits NULL. Even if it didn't ship now, the retrofit cost is genuinely low: `ADD COLUMN` + widen the `CHECK`, and since every pre-existing row has NULL in the new column, a correctly-written 3-way exclusion validates against existing data with no cleanup. The real reason to add it now isn't migration cost — it's avoiding the `CHECK` getting authored narrowly (hardcoded 2-way) in the first place, which is a discipline question at write time, not a risk question at migration time. The ghost-invoice-prevention deferred trigger (sum-of-line-items = `invoices.amount_in_paise`, checked at commit) is already source-agnostic — it doesn't care whether a line item came from a visit, a treatment event, or a package purchase, so it needs nothing extra either.

**Worth flagging:** the "package coverage = ₹0 line item" framing in `handover_specialty_context_for_packages.md` §3 is a simplification that only holds for the common case. A package zeroes the *therapy component* of a visit, not the whole row. If the same visit is also the day's first-touchpoint or gap-return visit, `exam_fee_in_paise` or `lapse_penalty_in_paise` still applies on that row — so its line item shows ₹350 or ₹150, not ₹0, even though the package correctly covered the therapy piece. This isn't a schema gap (the amount is still correctly `visits_with_effective_charge`'s per-row total either way) — it's a reporting/UI expectation worth setting now, before "why does my covered visit still show a charge" becomes a support question later.

Also favorable: the "shallow patient with no `complaint_course_id`" attribution gap that a package purchase would eventually hit is the *same* gap Specialty's shallow-patient case (`specialty-decision-log.md` Entry 10) already forces a solution for, independent of Packages. Whatever `invoice_line_items` attribution path gets built for a complaint-course-free specialty treatment is very likely reusable for a complaint-course-free package sale later — deferring Packages doesn't strand this problem, Specialty is already going to solve the general version of it.

### 4. Recommendation

Yes — with a sharper boundary than a flat yes. **Defer the build, not the decisions.** The attendance/lifecycle surface (closures, waivers, mode config, cancel/refund overrides) is genuinely large, genuinely isolated from Pay-Per-Visit's tables, and — per `package_decision_log.md`'s own open-items list (final naming, whether the patient hint binds, the Hold-pending tradeoff, expired-at-write-time behavior) — genuinely not fully designed yet. Building it now means building against decisions that are still explicitly open. Deferring that whole surface costs nothing, because nothing in Pay-Per-Visit's own tables depends on it existing.

What deferring does *not* buy you is a pass on the handful of items above — because those aren't Packages decisions wearing a Packages label, they're Pay-Per-Visit's own schema decisions that happen to have a second future consumer. Two hidden traps specifically, since that's what you asked for:

1. **Fee dedup — the ordering trap, not a column trap.** The real risk isn't a missing hook, it's the fee-computation trigger getting authored as a "just ship it" quick implementation without the single-function/staged-internals discipline. If that happens, adding package coverage later isn't an additive migration — it's an edit to production billing logic that's been live and trusted for months, in a schema that has already once shipped a same-event-trigger-ordering bug for exactly this reason. This doesn't show up in any migration-file diff review; it only shows up when someone actually tries to add the second trigger.
2. **Tier rates — the shared-dependency trap.** Choosing the tier representation on `complaint_courses` purely for Pay-Per-Visit's convenience (the enum-CHECK column is marginally less work right now) risks a harder migration later if Packages' rate model ever needs the lookup table's extensibility — against a table that will have real clinical rows attached by then, not an empty one.

Neither trap is fatal, and neither argues against deferring the module. They argue for treating "ship Pay-Per-Visit first" as still requiring these four specific choices to be made with both consumers in mind, not as license to make them unilaterally in Pay-Per-Visit's narrower interest and revisit later. That's a small, bounded list — one column, one index, one constraint scope (pending Om), one CHECK shape, and one authoring convention. Everything else genuinely waits.

## PKGDEFER-03 — No: minimal manual package-revenue design

Straight answer: no, Phase 2 doesn't cover this, and no, it wasn't the minimal version. Here's specifically why, then the actual design.

### Why Phase 2 missed it

Every hook I proposed in Phase 2 — `visits.package_id`, the partial unique index, `invoice_line_items.package_purchase_id` — was framed as *prep for the eventual automated trigger*. Question 2's own title was "what ships today so the derivation trigger causes zero breaking migrations tomorrow." That's a real question with a real answer, but it quietly assumes the endgame is always "wire up automation eventually." It never asked what you're asking now: what does revenue capture look like if the automation *never* ships, or ships much later than the money does. Phase 2 also bundled attendance and revenue together under "the attendance/lifecycle surface" and recommended deferring the whole thing as one unit — which buried exactly the split you're making now. Attendance and revenue are not the same problem, and only one of them is optional.

### The actual distinction

**Revenue is not optional.** ₹6000 changed hands. It has to land in whatever query eventually says "how much did we make this month," or that query is wrong — either undercounted (the sale never shows up anywhere) or overcounted (each subsequent visit still bills the normal per-visit rate on top of money already collected). Either failure is a real accounting error, not a cosmetic gap.

**Attendance is optional, by your own call.** Which specific day, how many sessions are left, whether Tuesday was a closure — none of that touches a rupee figure. It's bookkeeping about scheduling, not about money. That's the entire `clinic_closures`/`package_day_waivers`/mode-config/read-time-day-count surface from Phase 1, and it stays exactly as deferred as it already was.

### Tier 0 — works today, zero schema change

`packages` already exists in v11 with `amount_paid_in_paise`. That column *is* the ₹6000. A receptionist creates a `packages` row at sale — patient, clinic, `linked_complaint_id`, `amount_paid_in_paise = 600000` — and creates an `invoices` row for that same amount with `visit_id = NULL`, which is already a legal state (`visit_id` is nullable, no unique constraint ties it to exactly one visit). That invoice is the ₹6000 in revenue, today, with the current schema, no migration at all.

This is the honest floor. It's also fragile: nothing distinguishes that invoice as *package* revenue from any other visit-less invoice later, and nothing stops a receptionist from also charging the normal per-visit fee on a covered session by accident.

### Tier 1 — the actual recommendation: two small additions

```sql
-- Tags an invoice as (at least partly) a package sale. Doesn't wait on
-- invoice_line_items — that table doesn't exist yet and is a separate,
-- larger migration Specialty is driving on its own timeline. This works
-- against invoices as it stands in v11 today.
alter table invoices add column package_id uuid references packages(id);

-- Traceability: which visit actually drew against which package.
-- Nullable — most visits have none.
alter table visits add column package_id uuid references packages(id);

-- The actual safety net. Not a lookup, not a derivation — just refuses to
-- let a package-linked visit keep a nonzero therapy charge, whether a
-- receptionist forgets to zero it or something else tries to write both.
alter table visits add constraint chk_package_therapy_fee_zeroed
  check (package_id is null or therapy_fee_in_paise = 0);
```

Three ADD COLUMN/CONSTRAINT statements, no backfill, no trigger, no cross-table lookup. This is what actually earns the word "minimal."

**On how `therapy_fee_in_paise` gets zeroed — a real judgment call, not a mechanical one.** The established principle from Phase 1 is that fee columns are backend-derived and the frontend never writes them directly — that's what the override columns (`final_amount_in_paise`, `override_reason`, `override_by`, `override_status`) exist to protect. Routing package coverage through that same mechanism would reuse existing infrastructure, but it's the wrong shape for this: that mechanism carries admin-approval semantics (`override_status` moving to `'approved'` implies someone signed off on a deviation), and "this session is covered by a package the patient already paid for" isn't a discretionary waiver — it's routine, and gating it on an approval step is friction you didn't ask for. I'd let the frontend set `therapy_fee_in_paise` directly, but *only* when `package_id` is also set — and the CHECK constraint above is what makes that safe rather than a trust violation: the only value that will ever pass is 0. The frontend isn't being trusted with an arbitrary number, it's being allowed exactly one legal value, enforced at the database. `exam_fee_in_paise` and `lapse_penalty_in_paise` stay computed exactly as they already are on every other visit — the exception is scoped to one column, nothing else changes.

**The workflow, concretely:**
1. Sale: create `packages` row (`amount_paid_in_paise = 600000`), create `invoices` row (`package_id` set, `visit_id` NULL, `amount_in_paise = 600000`). Revenue recorded, done.
2. Covered visit: receptionist logs the visit normally. `exam_fee_in_paise`/`lapse_penalty_in_paise` compute as usual. They set `package_id` on the visit and `therapy_fee_in_paise = 0`. The CHECK constraint rejects the insert if those two disagree.

Total revenue = `SUM(invoices.amount_in_paise)`, and the ₹6000 shows up exactly once, at sale, never again at any subsequent covered visit.

### What this doesn't protect against — said plainly, not glossed over

The CHECK catches a *partial* mistake: someone sets `package_id` but forgets to zero the fee. It does **not** catch a *complete* one: a receptionist who doesn't realize a package exists at all, and just logs the visit normally, package_id left NULL. That gap only closes with an actual lookup — which is the automation you're explicitly not building right now. The cheap mitigation that doesn't require a trigger: a read-only query in the UI at visit-logging time — "does an active package exist for this complaint?" — surfaced as a prompt, not enforced by the database. Pure convenience, zero schema, zero automation, and it's the same category of risk this schema already accepts elsewhere (`complaint_name` free text isn't validated against `complaint_catalog_id` either — the app is trusted to keep denormalized fields honest, same pattern here).

If that residual gap ever turns into a real, observed problem — staff genuinely losing track, real revenue leaking — that's the actual trigger for building the derivation logic from Phase 2. Not before. This design is a deliberate, reversible starting point, not a permanent ceiling.

### Bottom line

Tier 0 (zero schema) works but leaves revenue integrity entirely on trust. Tier 1 (three lines of DDL, no trigger, no attendance system) is the minimal version that actually protects the number you said matters — ₹6000 collected, not ₹300/day re-billed on top of it — without touching anything from the deferred attendance surface. That's the recommendation.