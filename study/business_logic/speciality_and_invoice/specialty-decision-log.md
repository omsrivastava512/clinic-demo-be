# Specialty Treatments / Session Architecture — Decision & Gap Log

**ChatID:** SPECIALTY. This file is maintained across the SPECIALTY chat and is meant to survive it — if this chat is lost or forgotten, this file is what carries the thread forward into a new one.

## Purpose & Lineage

This log is a **child of `business-rules-log.md`** (the business rules log — referred to as `README.md` inside the handover files, since that's the filename the handover docs were written against; both names point at the same document). Everything already settled in `business-rules-log.md` remains the authoritative prior record and is not restated here.

This file exists to capture only what is **NEW, CHANGED, or CORRECTED** relative to that parent document, arising from conversation in the SPECIALTY chat. It does not re-document anything already settled there — if a topic hasn't moved since `business-rules-log.md` was last updated, it has no entry here.

Every entry that changes or corrects something names the exact section of `business-rules-log.md` (and any handover file) that needs updating once the entry is folded back into the permanent record. Until that happens, this log — not memory of the chat — is the current state of truth for anything listed below.

**Maintenance rule:** don't duplicate what's settled. If a future entry confirms something already in `business-rules-log.md` unchanged, it doesn't get an entry here.

## Open items at a glance

1. **CORRECTION** — Package-exclusion of specialty treatments is structural (different money bucket), not a category filter as currently described.
2. **OPEN** — Specialty-treatment table design (embedded vs. standalone) and whether a session table gets built. Not confirmed.
3. **CORRECTION** — A specialty-treatment event should require at least one linked complaint by default, not zero.
4. **OPEN** — Junction table vs. array column for linking treatment events to complaints. Leaning junction, not confirmed.
5. **OPEN** — How a session-anchored invoice preserves per-complaint billing granularity. Proposed: `invoice_line_items`. Not confirmed.
6. **OPEN** — Specialty-treatment logging needs no new frontend screen. Belongs in a future `WORKFLOW.md`, not `business-rules-log.md`.
7. **OPEN** — Session boundary leaning day-level (patient+clinic+date), not sitting-level. `sessions` needs `clinic_id`; `treatment_events` needs fuller columns.
8. **OPEN** — Invoice boundary decoupled from session boundary. "1 vs 2 invoices per day" still Om's call — no real workflow exists yet to observe.
9. **OPEN** — Recommended build order for the four proposals touching `visits`, to avoid repeating the historical trigger-ordering bug.
10. **OPEN** — Shallow-patient/service-seeker archetype: no separate table needed, but `patient_clinic_access` provisioning has a real gap for anyone who never gets a complaint course.
11. **OPEN** — Body-part linking without a complaint course: proposed polymorphic link (`complaint_course_id` OR `catalog_region`, CHECK-enforced, reusing `complaint_catalog.region`).
12. **OPEN** — Specialty-treatment frontend UX: one shared component (inline chip-expansion), two entry points (Procedure Logger + standalone Daily Ledger flow). Refines Entry 6.
13. **OPEN** — Ghost-invoice prevention: recommended a deferred constraint trigger checking line-item existence and sum-match at commit time, over a stored-procedure/RPC alternative.
14. **CORRECTION** — `invoice_line_items` should not carry its own `complaint_course_id` column; a treatment-event-sourced line item can legitimately cover several complaints at once, which a single column can't represent. Attribution is always derived from the source instead.
15. **OPEN** — Session boundary revised: submission-batch (Om's proposal) now recommended over day-level. Supersedes Entry 7's lean, not a correction — a better option was proposed, not an error found.
16. **OPEN** — Package purchases and retail products added as valid `invoice_line_items` sources; `chk_line_item_source` expands via `num_nonnulls()`. New `product_sales`/`product_sale_items` tables. `invoices.patient_id` needs to become nullable for anonymous retail sales.
17. **OPEN** — Pay Later: invoice created at checkout regardless of payment, `payment_status = 'Pending'` (already a valid, unused CHECK value in the real schema). `daily_ledger`'s hardcoded `'Complete'` status is now factually wrong and needs to change.
18. **OPEN** — Split payment modes: recommended three dedicated columns (`cash_amount_in_paise`/`online_amount_in_paise`/`card_amount_in_paise`, matching `clinics.payment_methods_accepted`) over JSONB or free text. A `payment_lines` child table noted as a future upgrade path, not needed now.
19. **OPEN** — `treatment_event_links`' XOR stays exactly as designed; multi-target coverage (e.g. cupping touching a complaint and an incidental region) uses two rows, not a relaxed constraint. New `is_primary` flag proposed.
20. **CORRECTION** — Entry 10 solved the write-side provisioning gap for shallow patients but missed a read-side one: a frontend re-fetch between patient creation and the first clinic-scoped record could hit the staff RLS gap. New `patients`-insert trigger proposed to close it at the root, for both archetypes.
21. **OPEN** — Double-counting resolved: report a safe-to-sum per-complaint number (jointly-linked treatments excluded or evenly split) separately from a deliberately double-counting "full-touch" number — never let the two share a label or get chain-summed together.
22. **OPEN** — Frontend UX revised again: a button opening a modal (Approach D) now recommended over Entry 12's always-visible inline section (Approach C). Supersedes, not a correction — same pattern as Entry 15.

---

## Log Entries

### 1. Package-exclusion mechanism — CORRECTION

- **Source:** SPECIALTY-03
- **Old position:** `business-rules-log.md` §9 (echoed in `handover-packages.md`'s "Specialty Services — Package Exclusion" note) describes package coverage excluding specialty treatments via a filter — `WHERE category != 'PREMIUM'` on `visit_services`.
- **New position:** The proposed package-matching trigger (also specified in §9 and `handover-packages.md` Priority 2) only ever reads and writes `visits.therapy_fee_in_paise`. It never touches `visit_services` or `services.category`. Specialty charges land in `services_total_in_paise`, a column the trigger never reaches — so the exclusion is already automatic as a side effect of which bucket the money sits in, not something achieved by a filter. No behavior changes; only the explanation of the mechanism was wrong.
- **Status:** CORRECTION
- **Where it lands:** `business-rules-log.md` §9 (the "existing hook" paragraph); `handover-packages.md`'s "Specialty Services — Package Exclusion" section.

### 2. Specialty-treatment table design + session architecture — OPEN

- **Source:** SPECIALTY-03
- **Old position:** `business-rules-log.md` §9 lists three options for specialty multi-complaint representation (standalone table / extend `visit_services` + junction / bolt directly onto `visit_services`), recommending the junction extension. `handover-index.md`'s Standing Cross-Cutting Note logs the session/encounter gap as unresolved and "worth a dedicated decision conversation."
- **New position:** Full comparison produced: embedding specialty in `visit_services` (+ junction) vs. a standalone `special_treatment_events` table, plus a session-table cost/benefit analysis against three lighter alternatives (specialty events linked by patient+date only; invoice-as-grouping-entity instead of a session table; duplicated per-complaint rows with a shared group id). Key finding: a session table would **not** replace the existing `patient_id + date` fee-dedup scoping used for the exam-fee/lapse-penalty logic — that check needs to be coarser than any one session and stays exactly as designed, independent of whatever session concept gets built. Recommendation (not confirmed): build a session table scoped narrowly to specialty-event anchoring and invoice assembly, and reopen the old "Option C" (session as frontend-only, never persisted) leaning, since a frontend-only concept can't be the target of a foreign key from a specialty event.
- **Status:** OPEN
- **Where it lands (once decided):** `business-rules-log.md` §9 (likely a new subsection); `handover-fee-computation.md`'s specialty-services section; `handover-packages.md`; `handover-index.md`'s standing note.

### 3. Specialty events requiring a linked complaint — CORRECTION

- **Source:** SPECIALTY-04 (correcting SPECIALTY-03)
- **Old position:** SPECIALTY-03 stated a specialty-treatment event with zero linked complaints is an acceptable default, justified by an invented non-clinical-wellness hypothetical. This was never written into `business-rules-log.md` — it only existed in this chat.
- **New position:** No confirmed business case exists anywhere in the log for a complaint-free specialty treatment — every real example given (cupping covering neck+back+shoulder) is complaint-linked. Corrected default: require at least one linked complaint per event, enforced via a deferred constraint trigger or, more practically, at the API layer (never exposing raw insert access to the junction/parent tables). A genuinely complaint-free case, if one is ever real, should be an explicit flagged exception with a reason — not silent absence, which is indistinguishable from a receptionist simply forgetting to link one.
- **Status:** CORRECTION
- **Where it lands:** `business-rules-log.md` §9, when the specialty-treatment design (Entry 2) is formalized there.

### 4. Junction table vs. array column — OPEN

- **Source:** SPECIALTY-04
- **Old position:** None — new comparison, not previously discussed in `business-rules-log.md` or any handover file.
- **New position:** Junction table recommended over an array column (`complaint_course_ids uuid[]`) for linking a treatment event to multiple complaints. Wins on enforceable referential integrity (Postgres has no native foreign-key constraint on individual array elements) and on query/join composability for billing reports; also stays consistent with every existing guard-trigger pattern in this schema. Genuine tradeoff conceded: an array makes "must have at least one entry" a trivial single-table `CHECK`, while a junction table needs a deferred constraint trigger or API-layer enforcement for the same guarantee (see Entry 3). Not a final decision.
- **Status:** OPEN
- **Where it lands (once decided):** `business-rules-log.md` §9; `handover-fee-computation.md`'s specialty-services section.

### 5. Invoice granularity under session-based invoicing — OPEN

- **Source:** SPECIALTY-04
- **Old position:** `business-rules-log.md` §5 logs the invoice/session gap as three options — A (one invoice per visit, frontend batches), B (a junction table between invoices and visits), C (session as frontend-only, never persisted — the leaning at the time). No mechanism was ever specified for how a session-anchored invoice would preserve per-visit/per-complaint billing breakdown.
- **New position:** Proposed `invoice_line_items` — a real child table of `invoices`, effectively a fleshed-out Option B — where each line item points back to its source visit or specialty-treatment event and carries its own `complaint_course_id`, with its amount read from the already-proposed `visits_with_effective_charge` view (`handover-pricing-integrity-overrides.md`). This also resolves a question that view's own doc left explicitly open under "Downstream blast radius" — that an override on a visit doesn't currently flow through to an invoice. Supporting evidence: the existing frontend `InvoiceBuilder` component (mock data, not yet wired) already expects an `items` array shaped exactly this way (`complaintId`/`complaintName`/`cost` per item), and the Procedure Logger screen's own copy already says "Current Session Bill," not "visit bill" — both point independently toward this being the intended target shape. Also clarified along the way: `invoices.visit_id` was never schema-enforced 1:1 (nullable, no unique constraint) — the per-complaint breakdown people remember came from having many invoice rows, not from any structure inside one invoice. Not confirmed by Om — a proposed resolution, not a decision.
- **Status:** OPEN
- **Where it lands (once decided):** `business-rules-log.md` §5 (the Option A/B/C entry); `handover-pricing-integrity-overrides.md`'s "Downstream blast radius" note (currently lists this as unresolved); `handover-index.md`'s standing cross-cutting note.

### 6. Specialty-treatment frontend workflow — OPEN

- **Source:** SPECIALTY-04
- **Old position:** None — not previously discussed anywhere; this is frontend-workflow content, a different domain from business rules.
- **New position:** No new workflow screen or route is needed for specialty-treatment logging, regardless of which backend design (Entry 2) wins. The existing Procedure Logger screen gets a modest addition — a complaint multi-select shown when a specialty/premium service is selected — rather than a rebuild. The backend table decision and the frontend capture form are separable; the same form works either way.
- **Status:** OPEN
- **Where it lands:** Does **not** belong in `business-rules-log.md` (business-rule domain, not workflow). Belongs in the `WORKFLOW.md` deliverable requested separately (attached doc, not yet produced) once that gets built.

### 7. Session boundary and shape — OPEN (refines Entry 2)

- **Source:** SPECIALTY-06
- **Old position:** Entry 2 recommended building a session table but left where the boundary sits, and the table's exact columns, unresolved.
- **New position:** Leaning toward day-level granularity (`patient_id + clinic_id + date`) over sitting-level (a time-window heuristic), because day-level can be inferred entirely server-side via a trivial date-equality check — no explicit "start new session" UI action, no API change to how visits get submitted, no window-heuristic edge cases. Confirmed this doesn't conflict with the existing `patient_id + date` fee-dedup logic (Entry 2), which stays exactly as designed either way. Cost: day-level session is a looser, more administrative grouping than "one continuous encounter" — worth naming it accordingly (e.g. `day_group`) rather than `sessions`, so the name doesn't overpromise clinical cohesion it doesn't have. Correction to the SPECIALTY-04 diagram: `SESSIONS` must carry `clinic_id`, not just `patient_id` + `date` — a session is location-specific (like `visits`/`packages`), not patient-global (like `patient_alerts`, which uses `owner_id`, not `clinic_id` as originally miscited) — without it, a patient visiting two branches the same day would incorrectly merge into one session. `TREATMENT_EVENTS` also needs `service_id`, `clinic_id`, `date`, and `clinician_id` — specified in SPECIALTY-03's prose but dropped from the SPECIALTY-04 diagram; corrected in the SPECIALTY-06 diagram. Not confirmed by Om.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` §9; `handover-fee-computation.md`'s specialty section; `handover-index.md`'s standing note.

### 8. Invoice boundary decoupled from session — OPEN (refines Entry 5)

- **Source:** SPECIALTY-06
- **Old position:** Entry 5 proposed `invoice_line_items` with invoices implicitly grouped by session (the SPECIALTY-04 diagram gave `INVOICES` a `session_id` column).
- **New position:** Invoice boundary doesn't need to equal session boundary. Whatever session ends up meaning (Entry 7), invoicing can follow its own boundary — whatever the receptionist has queued up at the moment "Create Invoice" is clicked, optionally including an earlier same-day uninvoiced visit if they choose to consolidate. This sidesteps needing to resolve session granularity before answering "one invoice per day vs. one per sitting," which stays a separate, still-open business question — Om's call once the clinic has an actual workflow to observe; no schema rule blocks either answer (`invoices` has no constraint limiting how many can exist per patient per day). Diagram corrected: `INVOICES` no longer carries `session_id`; only its line items (via `visits`/`treatment_events`) carry it indirectly.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` §5; `handover-pricing-integrity-overrides.md`'s "downstream blast radius" note.

### 9. Build sequencing for the four proposals touching `visits` — OPEN

- **Source:** SPECIALTY-06
- **Old position:** None — new content.
- **New position:** Four separate proposals add columns and `BEFORE INSERT`/`UPDATE` trigger logic to `visits` — the four-bucket money redesign, `visits.package_id`, the override/waiver columns, and `session_id`. Recommended build order, to avoid repeating the class of bug already logged in `backend-schema-iterations.md` Iteration 5 (same-event triggers firing in alphabetical order by accident, not by design): four-bucket columns first (foundational — everything else reads `therapy_fee_in_paise`/`grand_total_in_paise`), then `package_id` (its trigger zeroes a four-bucket column, so must run after that logic exists and is correct — see the SPECIALTY-07 trigger, which also depends on the partial unique index from Entry-adjacent `handover-packages.md` Priority 3 existing first), then the override columns (overriding a total is only testable once that total is reliably correct), then `session_id` last (lowest interaction risk with the money logic). Wherever more than one of these end up as separate trigger functions on the same INSERT event, they should be consolidated into as few functions as practical, with ordering explicit in code rather than left to alphabetical accident.
- **Status:** OPEN
- **Where it lands:** Not a `business-rules-log.md` matter — implementation planning. Worth a note in `supabase_migration.md` or a future migration-planning doc when build starts (per `schema-study-plan-v2.md` Module 9), not the business rules log.

### 10. Shallow-patient / service-seeker archetype — OPEN

- **Source:** SPECIALTY-07
- **Old position:** None — this archetype (someone with no complaint course, who may never get one) was not previously discussed anywhere in `business-rules-log.md` or any handover file.
- **New position:** Validated — `patients` does not need a separate table or a stored flag. Strongest argument: conversion. If a shallow patient later becomes a real patient, one table means "becoming real" is just their first `complaint_courses` row being created, with zero migration or identity-merge risk. `referral_mode`/`referral_doctor_info` already fit the externally-prescribed sub-case naturally. A derived view (`NOT EXISTS complaint_courses` for this patient, `security_invoker = true`) recommended for reporting/UI convenience over a stored flag, consistent with the existing preference for derived-over-stored (same reasoning as `final_amount_in_paise` staying nullable). Real gap found, unsolved by any of this: `patient_clinic_access` is currently only auto-provisioned from `complaint_courses` (`process_new_complaint_course()`); a shallow patient who never gets a complaint course would never get a `patient_clinic_access` row and would be invisible under the staff RLS policy as currently designed. Same class of problem as the intake deadlock already solved once in Iterations 6–7 of `backend-schema-iterations.md`, resurfacing for a new reason. Whatever records a shallow patient's treatment (Entry 11) needs to become a second valid provisioning entry point for `patient_clinic_access`, alongside `complaint_courses`.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` — new subsection, likely near §9; the comment on `process_new_complaint_course()` in `supabase_migration.md` would need updating, since it currently states complaint_courses is "the correct and only place" to provision access — no longer true under this design.

### 11. Recording body part without a complaint course — OPEN

- **Source:** SPECIALTY-07
- **Old position:** `business-rules-log.md` §9 and this log's Entries 2/7 propose a link table between a treatment event and `complaint_courses`, always required (Entry 3: at least one link required) — no accommodation for a target that isn't a complaint course at all.
- **New position:** Proposed a polymorphic link — each row points to exactly one of two targets, `CHECK`-enforced: a real `complaint_course_id` (active-patient case, full diagnostic specificity) OR a `catalog_region` value reusing `complaint_catalog.region`'s existing 7-value enum (Spine/Shoulder/Knee/Hip/Elbow/Ankle/Neuro) for the service-seeker case. Region-level (not a specific `complaint_catalog_id`) is deliberate: a specific catalog entry represents a diagnosis, and recording a shallow patient against one would claim an assessment that never happened — region only claims location, which is honest for someone who wasn't assessed. An optional free-text `note` alongside (for an external prescription's exact wording) is additive context only, never the primary queryable field — mirroring how `override_reason` already sits alongside structured columns rather than replacing them. Considered and rejected: a wholly separate body-part taxonomy decoupled from `complaint_catalog` (duplicates the region concept with no real gain — the strongest case for it, laterality, is a pre-existing gap affecting regular complaints too, not something specific to this case); free text as the primary field (the exact problem `complaint_catalog_id` was added to fix once already, per `backend-schema-iterations.md` Iteration 2). Connects to Entry 10: whichever table ends up being `treatment_events` needs its insert trigger to also upsert `patient_clinic_access`, mirroring `process_new_complaint_course()`.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` §9; `handover-fee-computation.md`'s specialty-services section.

### 12. Specialty-treatment frontend workflow — OPEN (refines Entry 6)

- **Source:** SPECIALTY-07
- **Old position:** Entry 6 concluded no new screen/route was needed, just a modest addition to the Procedure Logger — specifically a complaint multi-select shown when a specialty service is selected. Rejected by Om: a button opening a separate selection window conflicts with the screen's existing chip-tap interaction paradigm.
- **New position:** One specialty-logging component with inline chip-expansion (matching the existing tap paradigm, no modal/button), mounted at two entry points — inside the Procedure Logger as its own section, not nested under any one complaint (avoiding the original Approach A's header/body inversion problem), for the active-patient case; and as a separate, lighter "Log Specialty Treatment" flow from the Daily Ledger, skipping complaint selection entirely, for the service-seeker case. The chip row's source list changes by context — today's selected complaints for an active patient, body-region chips (Entry 11) for a service seeker — but the tap interaction itself stays identical either way.
- **Status:** OPEN
- **Where it lands:** Not `business-rules-log.md` (workflow domain) — belongs in the future `WORKFLOW.md` deliverable, same as Entry 6.

### 13. Ghost-invoice prevention mechanism — OPEN

- **Source:** SPECIALTY-08
- **Old position:** None — the `invoice_line_items` proposal (Entry 5/8) never specified what stops an invoice from being created with zero line items, or with a stored total that disagrees with the sum of its line items.
- **New position:** Recommended a deferred constraint trigger (`CONSTRAINT TRIGGER ... AFTER INSERT ... DEFERRABLE INITIALLY DEFERRED`) on `invoices`, checking at transaction-commit time that at least one `invoice_line_items` row exists and that its sum matches `invoices.amount_in_paise` — rolling back the whole transaction, ghost invoice included, if either check fails. Chosen over a `SECURITY DEFINER` stored-procedure/RPC alternative (`create_invoice_with_items(...)`, blocking direct table writes) specifically for consistency with this schema's existing idiom — every other guard in this project is a trigger reacting to a write, never a restricted RPC clients must call instead. Also established: `invoice_line_items` should be treated as write-once — created together with their parent invoice, never edited afterward, mirroring the no-lifecycle MVP philosophy already applied to `visits` — so this check only ever needs to hold at creation, not be re-verified on an ongoing basis. Postgres specifics confirmed: CHECK constraints cannot contain subqueries (hard rejection at DDL time, not just discouraged), and `CONSTRAINT TRIGGER`s only support `AFTER`, never `BEFORE`.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` §5 (alongside the invoice_line_items proposal, once formalized); a future migration-planning doc per Entry 9.

### 14. `invoice_line_items` should not store `complaint_course_id` directly — CORRECTION

- **Source:** SPECIALTY-08
- **Old position:** The SPECIALTY-04/06 diagrams, and Entry 5's description, gave `LINE_ITEMS` its own `complaint_course_id` column.
- **New position:** Corrected, found while working a concrete example. A treatment-event-sourced line item can legitimately cover more than one complaint at once (e.g. one cupping application, ₹500 flat, linked to two complaint courses via `treatment_event_links` — Entry 11) — a single `complaint_course_id` column can't represent that without arbitrarily picking one or leaving it null, both wrong. Fix: `invoice_line_items` carries no complaint attribution of its own at all — it's always derived from the source, a direct lookup through `visits.complaint_course_id` for a visit-sourced item, or through `treatment_event_links` (possibly multiple rows) for a treatment-event-sourced one. Consequence, genuinely unresolved: querying "lifetime revenue for complaint X" is unambiguous for visit-sourced line items, but for a jointly-linked treatment event, counting its full amount toward every complaint it covers double-counts if summed chain-wide, while excluding it understates what was actually done for any one of them. Which convention to use is a real reporting decision, not something inferable from the schema — Om's call.
- **Status:** CORRECTION
- **Where it lands:** `business-rules-log.md` §9 / §5; `handover-fee-computation.md`'s specialty-services section.

### 15. Session boundary revised to submission-batch — OPEN (supersedes Entry 7's lean)

- **Source:** SPECIALTY-09
- **Old position:** Entry 7 leaned day-level (`patient_id + clinic_id + date`) specifically because it needed zero API changes and avoided the fuzziness of a time-window heuristic.
- **New position:** Om proposed session = one atomic receptionist submission batch (the Amazon-cart analogy) — a genuinely better option, not a correction to Entry 7, since day-level was the right answer among the options on the table at the time. Batch-level captures the real distinction day-level can't (two sittings hours apart on the same day are genuinely different encounters) with none of a time-window heuristic's fuzziness, since "was this one request or two" is never ambiguous. Real cost, unlike day-level: not derivable from data any row already carries, so it structurally requires new API surface — a nullable `session_id` on `visits`/`treatment_events` inserts, omitted on the first insert of a new checkout (server creates a session, derives `clinic_id` from that row's own already-validated `clinic_id`, returns the new id), echoed back by the frontend on subsequent inserts within the same UI pass. Never trust a client-*invented* session_id — only ever echo one the server just issued, validated on arrival against `patient_id`/`clinic_id` matching. `sessions` no longer needs a `date` column at all under this model — `created_at` already tells you which day, so it's derived rather than duplicated.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` §9; `handover-fee-computation.md`; `handover-index.md`'s standing note.

### 16. Package purchases and retail products as `invoice_line_items` sources — OPEN

- **Source:** SPECIALTY-09
- **Old position:** `chk_line_item_source` (Entry 14) covered two sources, `visit_id`/`treatment_event_id`. Package purchases and retail products weren't discussed anywhere.
- **New position:** Both added as valid sources rather than living outside the invoice hierarchy — keeping them outside would just reproduce the ghost-invoice/integrity problem (Entry 13) a second time, for a parallel pathway. Packages: no new table needed — `packages.amount_paid_in_paise` already exists in the real schema; the row's creation moment already is the purchase event, so `invoice_line_items.package_id` references `packages` directly. Retail products: genuinely different shape (need quantity, which no existing source has), and a single retail transaction can bundle several distinct products — mirrors the existing `visits` → `visit_services` granularity rule (one line item per billable *event*, not per itemized component), so a new `product_sales` (parent, one row = one line item) + `product_sale_items` (child, itemized with quantity) pair is proposed, reusing `services`/`clinic_service_prices` for the catalog/pricing layer. `chk_line_item_source` expands to four possible sources via Postgres's `num_nonnulls(...) = 1` rather than hand-written nested OR branches. Concrete conflict found against the real schema: `invoices.patient_id` is `not null` today, which blocks a genuinely anonymous retail sale (no clinical relationship, shouldn't need even a shallow-patient record) — recommended fix: make it nullable, a low-risk migration, while every clinical source keeps requiring a real patient.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` — new subsection; `handover-packages.md` (package purchase as an invoicing event); `supabase_migration.md`'s `invoices.patient_id` constraint.

### 17. Pay Later / payment status lifecycle — OPEN

- **Source:** SPECIALTY-09
- **Old position:** MVP assumption logged throughout this project: "a visit row existing = session complete and paid," single-touch, `invoices.payment_status` defaulting to `'Paid'` always.
- **New position:** Invoice gets created at checkout regardless of payment — `payment_status = 'Pending'` when the receptionist chooses Pay Later, `'Paid'` when cash/UPI is collected immediately. No "draft" concept needed on `invoices` at all — nothing gets a row until "Create Invoice" is clicked and the deferred trigger (Entry 13) can enforce real, final line items; before that, whatever's being assembled is just frontend state. Good finding on cross-check: `'Pending'`/`'Overdue'` are already valid values in the real `payment_status` CHECK constraint today, with a comment noting they're "kept for future credit/package billing" — this feature needs zero schema change to that column, only to what the app does with values already sitting there unused. `check_invoice_line_items_integrity` (Entry 13) needs no change — line-item correctness and payment collection are orthogonal questions. Outstanding balance: `SUM(amount_in_paise) WHERE payment_status IN ('Pending','Overdue')`, cheap specifically because the total is already guaranteed correct at creation (Entry 13). `'Overdue'` recommended as read-time derived (`Pending AND date < threshold`) rather than a cron-maintained stored state, consistent with the derived-over-stored preference used elsewhere; the actual threshold is a real open business question (Om mentioned "settles weekly" as the real pattern), not resolved here. Blast radius found: `daily_ledger`'s real definition hardcodes `'Complete' as status` unconditionally — that premise is now false the moment a visit can be done but unpaid; the view needs to actually reflect real payment status, not assert it. Also recommended: `paid_by`/`paid_at` audit columns on `invoices`, matching the audit-trail pattern already used for `override_by`. Naming collision flagged (not a conflict, just worth knowing): `visits.override_status = 'pending'` (a pricing-approval concept) and `invoices.payment_status = 'Pending'` (a cash-collection concept) are unrelated despite the shared word.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` — new subsection; `daily_ledger`'s definition in `supabase_migration.md` needs updating; `handover-pricing-integrity-overrides.md` (audit-column precedent, naming-collision note).

### 18. Split / hybrid payment modes — OPEN

- **Source:** SPECIALTY-09
- **Old position:** None — not previously discussed. `invoices.payment_mode` exists today as a single, unconstrained free-text column.
- **New position:** Recommended three dedicated nullable-by-default columns — `cash_amount_in_paise`, `online_amount_in_paise`, `card_amount_in_paise` — over a JSONB `payment_breakdown` column (loses insert-time type/shape safety this fixed, small use case doesn't need) or free text (repeats the exact anti-pattern already corrected once for `complaint_catalog`). Checked against the real schema: three columns, not two, matching `clinics.payment_methods_accepted`'s existing `array['CASH','UPI','CARD']` rather than the cash/online split as originally framed. CHECK constraint made conditional on payment status (`payment_status != 'Paid' OR sum-of-three = amount_in_paise`), synthesizing directly with Entry 17 — a Pending invoice legitimately has all three at zero, nothing collected yet. `payment_mode` recommended to become derived from the breakdown rather than independently settable. A `payment_lines` child table (one row per instrument, arbitrary N-way splits, zero-migration new modes) considered and explicitly not recommended yet — Om's own instinct that a junction table is more than this specific problem currently needs was judged correct, not overridden; noted as the upgrade path if a genuine 3+-way split or a new payment method ever becomes real.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` — new subsection; `supabase_migration.md`'s `invoices` table definition.

### 19. `treatment_event_links` XOR confirmed, not relaxed — OPEN

- **Source:** SPECIALTY-10
- **Old position:** Entry 11 established the XOR (`complaint_course_id` OR `catalog_region`, never both) on a single link row.
- **New position:** Confirmed correct as-is, after direct challenge. The scenario that seemed to require relaxing it — one treatment (cupping) covering a real complaint plus an incidental, non-diagnosed body region — is fully representable as two link rows against the same `treatment_event_id`, each individually satisfying the existing XOR; no schema change needed. Relaxing it anyway would cost the "exactly one of two clean shapes" property this schema reuses three times over (`chk_referral_doctor_info`, the pricing-override CHECK, this table), for no scenario that actually needs it. New addition proposed: `treatment_event_links.is_primary boolean not null default false`, letting a receptionist mark which target (if any) was the actual clinical reason versus incidental — directly useful for Entry 21's reporting question.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` §9; `handover-fee-computation.md`'s specialty-services section.

### 20. Shallow-patient intake gap — CORRECTION (completes Entry 10)

- **Source:** SPECIALTY-10
- **Old position:** Entry 10 proposed `treatment_events`' insert trigger as a second provisioning point for `patient_clinic_access`, solving the write-side circular dependency (mirroring `process_new_complaint_course()`).
- **New position:** That solves the write side but leaves a read-side gap Entry 10 didn't catch: between "patient row created" and "first clinic-scoped record inserted," a frontend re-fetch of the patient (e.g. a list/search query) would hit the staff RLS policy ("visible if `patient_clinic_access` has a row at this clinic") and genuinely fail to show a patient who was just created — the same shape of bug as the Iteration 6/7 deadlock, manifesting as a read-visibility gap instead of a write deadlock. Frontend discipline (carry the id forward from the create response, never re-search) avoids it, matching how the existing registration flow already works per the workflow doc — but shouldn't be the only safeguard, consistent with this schema's own established distrust of relying on client behavior alone. New trigger proposed: `patients_provision_initial_access`, `AFTER INSERT` on `patients`, deriving the registering clinic from `auth.uid()`'s own profile (never client input) and upserting `patient_clinic_access` immediately — closing the gap at the root for both shallow patients and ordinary new patients alike. Doesn't replace `process_new_complaint_course()`'s provisioning, which still handles the separate cross-branch case.
- **Status:** CORRECTION
- **Where it lands:** `business-rules-log.md` — the Entry 10 subsection, once written; `supabase_migration.md`'s `patients` table (a third trigger, `AFTER INSERT`, alongside `patients_updated_at` and `patients_owner_is_admin`).

### 21. Jointly-linked specialty revenue — OPEN (resolves Entry 14's open question)

- **Source:** SPECIALTY-10
- **Old position:** Entry 14 flagged that a treatment event linked to multiple complaints creates a genuine reporting ambiguity — count its full amount toward every complaint (double-counts chain-wide) or exclude it (undercounts any one complaint) — left as "Om's call," not resolved.
- **New position:** Resolved by separating two different questions that were being conflated. Chain-wide total revenue has exactly one correct number — computed directly from invoices/line-items, once each, completely independent of any per-complaint breakdown, never in doubt. Per-complaint revenue is where the real ambiguity lives, and the fix is to report two clearly labeled, never-conflated numbers rather than forcing one convention to serve both purposes: a safe-to-sum figure (jointly-linked treatments excluded from the per-complaint breakdown and shown on their own separate line, or evenly split across the complaints they touched — exclusion preferred as more honest than an arbitrary-looking split) that reconciles exactly to real revenue, and a separate full-touch figure (deliberately counts the full amount toward every complaint touched) that must never be chain-summed or shown beside the first without a clear label. Entry 19's `is_primary` flag gives a cleaner default than blunt exclude-or-split, if the therapist actually knows which target was primary.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` §9; `handover-fee-computation.md`'s specialty-services section (reporting conventions).

### 22. Specialty-treatment frontend UX revised again — OPEN (supersedes Entry 12)

- **Source:** SPECIALTY-10
- **Old position:** Entry 12 recommended one shared component with inline chip-expansion, mounted as its own always-visible section inside the Procedure Logger (Approach C) plus a standalone Daily Ledger entry point for service seekers.
- **New position:** Om proposed a genuinely better option, not a correction to Entry 12 — a single `( + Add Specialty Treatment )` button, positioned outside the tappable procedure list entirely, opening a modal reused identically from both the Procedure Logger (active-patient case) and directly from the Daily Ledger (service-seeker case). Confirmed this doesn't repeat the earlier-rejected "Applies To" button critique: that rejection was about a control disguised as one of many otherwise-uniform tappable tiles; this button is positioned and styled as a clearly separate kind of control from the start, the same way "Cancel" and "Create Invoice" already coexist without confusion. Preferred over Entry 12's always-visible section because most visits don't include a specialty treatment at all — the new approach keeps the default screen exactly as lean as today, costs one extra tap only when actually needed, and is more cleanly reusable across both archetypes since the modal only needs "patient" and optionally "today's selected complaints," not knowledge of which screen it's embedded in. Full ASCII wireframes of all four approaches (the original broken state, Approach A/inverted-section, Approach B/split-entry, Approach C, and this new Approach D) built and compared in `specialty-treatment-ux-mockups.md`.
- **Status:** OPEN
- **Where it lands:** Not `business-rules-log.md` (workflow domain) — belongs in the future `WORKFLOW.md` deliverable, same as Entry 6/12.
