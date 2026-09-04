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
21. **DECIDED** — Specialty revenue is never apportioned to complaints, ever, regardless of link count — always its own separate report. Complaint-level reports stay visit-sourced only.
22. **OPEN** — Frontend UX revised again: a button opening a modal (Approach D) now recommended over Entry 12's always-visible inline section (Approach C). Supersedes, not a correction — same pattern as Entry 15.
23. **OPEN** — Double-billing prevention: four `UNIQUE` constraints proposed on `invoice_line_items`' source columns. Found this isn't sufficient alone — needs a `'Void'` invoice status and a trigger, not a partial index (which can't subquery another table).
24. **CORRECTION** — `treatment_events.charged_amount_in_paise` conflated computed vs. actual price, the exact mistake `visits`' override design was built to prevent. Renamed to `computed_amount_in_paise`; the same 4-column override pattern as `visits` added.
25. **OPEN** — Nullable-column sprawl on `invoice_line_items` evaluated against a polymorphic `source_type`/`source_id` alternative and rejected — that pattern can't carry a real foreign key. Two of three example "new billable types" (medicines, rehab belts) don't need new columns at all — they're already covered as `product_sales` catalog rows.
26. **OPEN** — Session write-timing gap confirmed real, not resolved here: Entry 15's session_id mechanism was built on top of `FRONTEND_WORKFLOW.md`'s already-unresolved Decision 2 without ever pinning it down. Handed off to a dedicated follow-up chat.
27. **OPEN** — Visit vs. `treatment_events` boundary rule sharpened, not resolved: traction can legitimately go either way (bundled visit service or standalone), breaking any simple per-service rule. Handed off alongside Entry 26.
28. **OPEN** — Session/invoice timing deepened with an actual mockup walkthrough and a real operational case for defaulting to Pending. Same handoff as Entry 26.

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

### 21. Jointly-linked specialty revenue — DECIDED (resolves Entry 14)

- **Source:** SPECIALTY-10 (opened), decided in SPECIALTY-11
- **Old position:** Entry 14 flagged the reporting ambiguity. SPECIALTY-10 narrowed it to two candidate conventions — exclude jointly-linked treatments from per-complaint totals, or split them evenly — recommending exclusion but leaving the final call to Om.
- **New position:** Decided, explicitly: specialty-treatment revenue is never apportioned to complaints, under any circumstances — not excluded-with-a-remainder, not split, structurally separate, regardless of whether a given `treatment_events` row links to zero complaints (shallow patient), one, or several. Complaint-level revenue reports query only visit-sourced `invoice_line_items`, joined through `visits.complaint_course_id`; treatment-event-sourced line items are never part of that query. Specialty revenue gets its own independent report, grouped by `treatment_events.service_id`, summing whatever line items point at a `treatment_event_id` — fully decoupled from `treatment_event_links`' complaint/region attribution, which remains a clinical record only, never a revenue-apportionment input. Verified against Om's own worked numbers (Knee Rehab ₹300 + Back Pain ₹0 + Cupping ₹500 = ₹800): reconciles exactly to real revenue by construction, since every rupee lives in exactly one bucket, never two. Zero schema change required — purely a reporting-query convention. Entry 19's `is_primary` flag loses its revenue-apportionment justification under this rule (nothing gets apportioned anymore) but keeps standalone value as a clinical-documentation field.
- **Status:** DECIDED
- **Where it lands:** `business-rules-log.md` §9; `handover-fee-computation.md`'s specialty-services section — ready to fold into the permanent record as a settled rule, not a brainstorm item.

### 22. Specialty-treatment frontend UX revised again — OPEN (supersedes Entry 12)

- **Source:** SPECIALTY-10
- **Old position:** Entry 12 recommended one shared component with inline chip-expansion, mounted as its own always-visible section inside the Procedure Logger (Approach C) plus a standalone Daily Ledger entry point for service seekers.
- **New position:** Om proposed a genuinely better option, not a correction to Entry 12 — a single `( + Add Specialty Treatment )` button, positioned outside the tappable procedure list entirely, opening a modal reused identically from both the Procedure Logger (active-patient case) and directly from the Daily Ledger (service-seeker case). Confirmed this doesn't repeat the earlier-rejected "Applies To" button critique: that rejection was about a control disguised as one of many otherwise-uniform tappable tiles; this button is positioned and styled as a clearly separate kind of control from the start, the same way "Cancel" and "Create Invoice" already coexist without confusion. Preferred over Entry 12's always-visible section because most visits don't include a specialty treatment at all — the new approach keeps the default screen exactly as lean as today, costs one extra tap only when actually needed, and is more cleanly reusable across both archetypes since the modal only needs "patient" and optionally "today's selected complaints," not knowledge of which screen it's embedded in. Full ASCII wireframes of all four approaches (the original broken state, Approach A/inverted-section, Approach B/split-entry, Approach C, and this new Approach D) built and compared in `specialty-treatment-ux-mockups.md`.
- **Status:** OPEN
- **Where it lands:** Not `business-rules-log.md` (workflow domain) — belongs in the future `WORKFLOW.md` deliverable, same as Entry 6/12.

### 23. Double-billing prevention on `invoice_line_items` — OPEN

- **Source:** SPECIALTY-11
- **Old position:** None — the unique-per-source-column question was never addressed when the four nullable columns were proposed (Entry 16).
- **New position:** Four `UNIQUE` constraints proposed, one per source column (`visit_id`, `treatment_event_id`, `package_id`, `product_sale_id`) — safe with nullable columns, since standard SQL treats `NULL` as never colliding with another `NULL`, so unlimited null rows coexist while any given non-null value is guaranteed to appear at most once, enforced by Postgres at write time. Found this isn't sufficient alone: a permanent unique constraint makes a genuinely mistaken invoice's source permanently unbillable, since there's no way to void/reissue — `invoices.payment_status` has no `'Void'`/`'Cancelled'` value today. A partial unique index was considered and rejected as syntactically invalid here (a partial index predicate can't subquery another table to check the parent invoice's status); the correct mechanism is a trigger doing the cross-table exists-check directly, scoped to exclude line items whose parent invoice is void. Requires adding `'Void'` to `invoices.payment_status`'s CHECK constraint first.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` — the invoice_line_items subsection; `supabase_migration.md`'s `invoices.payment_status` CHECK constraint.

### 24. `treatment_events` needs the same override pattern as `visits` — CORRECTION

- **Source:** SPECIALTY-11
- **Old position:** SPECIALTY-08's worked example gave `treatment_events` a single `charged_amount_in_paise` column, described ambiguously as "what it cost at the time it happened."
- **New position:** Corrected — that single column conflates the honest catalog-derived baseline with the actually-charged amount, exactly the conflation the `visits` override design (computed vs. actual, four columns) was built specifically to prevent. `charged_amount_in_paise` renamed to `computed_amount_in_paise` (the protected, catalog-derived baseline); four override columns added, mirroring `visits` exactly: `final_amount_in_paise` (nullable), `override_reason`, `override_by`, `override_status` (`'none'/'pending'/'approved'`). Effective charge reads as `COALESCE(final_amount_in_paise, computed_amount_in_paise)`, via a new `treatment_events_with_effective_charge` view (`security_invoker = true`), feeding `invoice_line_items` the same way `visits_with_effective_charge` already does. Authorization reuses the existing `validate_owner_is_admin()`-style shared function rather than a new one, consistent with the real schema's own precedent of reusing one validation function across `clinics` and `patients`.
- **Status:** CORRECTION
- **Where it lands:** `handover-fee-computation.md`'s specialty-services section; `handover-pricing-integrity-overrides.md` (extending the override pattern to a third table); `supabase_migration.md`'s `treatment_events` proposal.

### 25. Nullable-column sprawl on `invoice_line_items` vs. a polymorphic source — OPEN

- **Source:** SPECIALTY-11
- **Old position:** None — the four-column design (Entry 16) was proposed without evaluating a generic `source_type` + `source_id` alternative.
- **New position:** Polymorphic `source_type`/`source_id` considered and rejected — `source_id` can't be a real foreign key under that pattern (Postgres FKs point at exactly one table), which would silently drop referential-integrity enforcement on invoice_line_items' most important relationship, directly undercutting this schema's central, repeatedly-applied principle of database-enforced rather than application-trusted correctness. Reframed the extensibility worry: two of the three example "new billable types" (medicines, rehab belts) don't need new columns at all — they're physical products, already covered by `product_sales`/`product_sale_items` (Entry 16) as new catalog rows, not new source columns. Vouchers are a genuinely different financial shape (store-of-value, purchase separate from redemption) and would fairly earn a new column if built. Practical extensibility limit assessed at roughly 5-6 columns total, reached slowly, because most new billable items extend an existing shape rather than requiring a new one — and adding a nullable column is already categorized as a safe, low-risk migration in this project's own established migration-safety framework.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` — new subsection alongside Entry 16.

### 26. Session write-timing — flagged, not resolved here

- **Source:** SPECIALTY-12
- **Old position:** Entry 15's session_id mechanism assumes some visit/treatment_event insert happens early enough in the checkout flow to need a session assigned, without ever specifying when that insert actually occurs relative to the rest of the screen sequence.
- **New position:** Confirmed as a real gap, not a misunderstanding — it traces directly to `FRONTEND_WORKFLOW.md`'s own Decision 2 (progressive vs. coordinated writes), already flagged open before this chat existed, which Entry 15 quietly built on top of without resolving. If writes are progressive (a visit row created the moment a procedure is checked off), an interrupted checkout leaves real orphaned rows behind — worse than an empty session, since a `visits` row existing is supposed to mean "this happened, complete." If writes are coordinated (nothing hits the database until one final atomic submission, matching everything else already decided — the deferred constraint trigger, write-once invoices, no-draft Pay Later), there's no window for an orphan to exist at all. Leaning coordinated, and noting the live "Current Session Bill" panel almost certainly doesn't need backend writes to update — but not resolved here. Deferred to a dedicated follow-up chat; two handoff prompts given to Om for this and for the broader frontend↔backend communication question.
- **Status:** OPEN
- **Where it lands:** Not yet — pending the follow-up chat's resolution. Will need `business-rules-log.md` §5/§9 and `handover-index.md`'s standing note once settled.

### 27. Visit vs. `treatment_events` boundary rule — sharpened, not resolved

- **Source:** SPECIALTY-13
- **Old position:** The visit/`treatment_events` split (Entry 2 onward) was proposed without ever specifying a rule for which table a given service instance belongs in.
- **New position:** Sharpened with a concrete counter-example that breaks any simple per-service rule. Traction can legitimately be delivered as an ordinary bundled `visit_services` entry (clinician-prescribed, part of a normal visit, no extra charge) OR as a standalone `treatment_events` entry (externally prescribed, no complaint course, its own individual fee) — same catalog service, two different homes depending on instance context, not a fixed property of the service itself. Also directly questioned: whether `visits.complaint_course_id` being `NOT NULL` is even the right premise to begin with. Not resolved here — handed to the same follow-up chat as Entry 26, via a dedicated prompt.
- **Status:** OPEN
- **Where it lands:** `business-rules-log.md` §9, once resolved.

### 28. Session/invoice creation timing — deepened with the mockup walkthrough

- **Source:** SPECIALTY-13
- **Old position:** Entry 26 flagged this as unresolved, tracing to `FRONTEND_WORKFLOW.md`'s Decision 2, leaning toward coordinated writes without confirming.
- **New position:** Om independently walked the actual mockup sequence (Complaint Selector → Procedure Logger → Create Invoice → Invoice screen → Confirm Payment) and arrived at the same open question from a different angle, plus a genuinely useful addition: a concrete operational justification for defaulting a new invoice to Pending — a receptionist shouldn't have to wait for a patient to count cash before moving to the next patient, should be able to close the invoice screen at Pending and reconcile later. Still not resolved — folded into the same follow-up prompt as Entry 26.
- **Status:** OPEN
- **Where it lands:** Same as Entry 26.


---

# [03 Sep 2026] SPECIALTY-01

> **[COMPACTED PROMPT]**
> Migrating from a prior chat; resolve three interrelated open problems in order, with full rigor. First, pin down exactly WHEN visits/treatment_events/sessions/invoices should actually be written relative to the Daily Ledger → Complaint Selector → Procedure Logger → Create Invoice flow, and design or reject a session_id assignment mechanism that can't leave orphaned rows behind on an abandoned checkout — Entry 15 assumed progressive writes across the checkout without ever resolving that against the still-open "coordinated vs. progressive" write-timing question already logged in module_2_logs.md. Confirm whether the live "Current Session Bill" panel in Procedure Logger actually requires backend writes before a final commit, or is pure frontend state. Second, stress-test the visit/treatment_events split: is `visits.complaint_course_id NOT NULL` actually correct, or worth challenging? What mechanism decides whether a given service instance becomes a `visit_services` row (bundled under a complaint-tied visit) versus a `treatment_events` row (standalone) — is it a property of the service itself, or a case-by-case call? Test explicitly against traction, which can legitimately go either way: ordinarily bundled into a complaint-tied visit with no extra charge, or delivered fully standalone with no complaint course at all, on an external prescription. Third, work through the actual mockup sequence (Complaint Selector → Procedure Logger, ending in "Create Invoice" → a separate Invoice screen → "Confirm Payment") and pin down exactly which click fires which backend write: does "Create Invoice" really submit the session/visit data AND create the invoice in one coupled action? Is it the visit or the session that's being marked complete when that fires? Confirm or correct the reasoning that a Pending-by-default invoice — closable without ever clicking "Confirm Payment," so a receptionist isn't stuck waiting on a patient to count out cash — is correct behavior, not a bug.

**Editorial note, added 2026-09-03, same day:** an earlier draft in this session mislabeled this content as "SPECIALTY-12," continuing an invented cross-chat sequence number, and independently reworded it rather than archiving what was actually said. That draft is discarded, not reconciled. What follows is the real response, under its real label. Going forward, log entries are attributed to their own chat's native SPECIALTY-NN label plus the calendar date — never an invented cross-chat counter. See SPECIALTY-02 below for the fuller explanation.

## SPECIALTY-01 — Session timing, visit split, invoice flow

Quick housekeeping before the substance: FRONTEND_WORKFLOW.md isn't actually in what I have access to — not among the project files, not in this upload. What I do have that covers the same ground: module_2_logs.md's "OPEN QUESTIONS" section states almost exactly the same open "coordinated vs. progressive write" question, and schema_reconciliation_audit.md has real file/line citations on how the current mock frontend actually behaves (ProcedureLogger, InvoiceBuilder, FooterActions.tsx). Also confirmed: README.md in this project's files IS business-rules-log.md — same document, different filename in this project's listing.

**1. Session write-timing — resolved, Entry 15 superseded.**

Mapped the flow first: Daily Ledger reads a view, no writes. Complaint Selector is patient/complaint search and selection held in frontend state, no writes, including for a brand-new complaint. Procedure Logger accumulates an in-memory InvoiceItem[] — confirmed behavior, not inferred: useProcedureLogger.tsx (lines 39–46) already builds this array client-side, and validateInvoiceItems (lines 11–16) already filters that same array. The "Current Session Bill" panel is a live sum over that array — no backend writes needed to render it. Create Invoice is the only click that touches the database at all under the resolved design.

Four signals converge on coordinated writes over progressive: module_2_logs.md's own logged lean ("strongest available signal... points toward one coordinated write"); the current mock frontend is already shaped this way (accumulate, submit once); the real schema's own comment on visits — "MVP WORKFLOW: Row existing = session complete + paid. No status lifecycle" — which progressive writes would quietly violate (a visit could exist with no invoice for an indefinite stretch); and orphan-avoidance is free under one transaction (full commit or full rollback, a Postgres guarantee) versus requiring real cleanup machinery under progressive writes (nothing today defines what happens to a logged-but-never-invoiced visit).

Entry 15's mechanism (frontend omits session_id on first insert, a trigger creates and returns it, frontend echoes it on later inserts) solved a cross-request coordination problem that stops existing once there's only one request. Rejected, not refined — superseded by a single SECURITY DEFINER function, checkout_and_create_invoice(...), called once from "Create Invoice," doing all of the following inside one implicit transaction: compute the total (pass 1, from the payload — in production, recomputed server-side from rate tables, not trusted from the client, per the "never trust a client total" principle in business-rules-log.md §7) → insert sessions → insert invoices (total known, process_new_invoice() fires and mints the real number, untouched) → per visit: insert complaint_courses if new (in order, before the dependent visit, inside the same function — so the complaint-before-visit ordering rule is satisfied by statement order, not by trusting an earlier separate frontend call), insert visits, insert its invoice_line_items row → per treatment event: insert treatment_events, insert its treatment_event_links, insert its invoice_line_items row. session_id is a local plpgsql variable throughout — never sent by, or returned to, the client as something it needs to track.

```sql
create table sessions (
  id         uuid primary key default gen_random_uuid(),
  patient_id uuid not null references patients(id) on delete cascade,
  clinic_id  uuid not null references clinics(id),
  created_by uuid references profiles(id),
  created_at timestamptz not null default now()
);

create index idx_sessions_patient_id on sessions(patient_id);
create index idx_sessions_clinic_id  on sessions(clinic_id);

create or replace function checkout_and_create_invoice(
  p_clinic_id         uuid,
  p_patient_id        uuid,
  p_visits            jsonb,
  p_treatment_events  jsonb,
  p_payment_status    text default 'Pending'
)
returns uuid
language plpgsql
security definer
as $$
declare
  v_session_id     uuid;
  v_invoice_id     uuid;
  v_visit_id       uuid;
  v_te_id          uuid;
  v_complaint_id   uuid;
  v_total_in_paise integer := 0;
  visit_rec        jsonb;
  te_rec           jsonb;
  link_rec         jsonb;
begin
  select coalesce(sum((v->>'consultation_fee_in_paise')::integer
                     + (v->>'services_total_in_paise')::integer), 0)
    into v_total_in_paise
  from jsonb_array_elements(p_visits) v;

  v_total_in_paise := v_total_in_paise + coalesce((
    select sum((t->>'computed_amount_in_paise')::integer)
    from jsonb_array_elements(p_treatment_events) t
  ), 0);

  insert into sessions (patient_id, clinic_id, created_by)
  values (p_patient_id, p_clinic_id, auth.uid())
  returning id into v_session_id;

  insert into invoices (clinic_id, patient_id, amount_in_paise, date, payment_status, created_by)
  values (p_clinic_id, p_patient_id, v_total_in_paise, current_date, p_payment_status, auth.uid())
  returning id into v_invoice_id;

  for visit_rec in select * from jsonb_array_elements(p_visits)
  loop
    if visit_rec ? 'new_complaint' then
      insert into complaint_courses (clinic_id, patient_id, complaint_name,
                                      complaint_catalog_id, start_date, last_date)
      values (p_clinic_id, p_patient_id,
              visit_rec->'new_complaint'->>'complaint_name',
              visit_rec->'new_complaint'->>'complaint_catalog_id',
              current_date, current_date)
      returning id into v_complaint_id;
    else
      v_complaint_id := (visit_rec->>'complaint_course_id')::uuid;
    end if;

    insert into visits (clinic_id, patient_id, complaint_course_id, session_id,
                         date, complaint, visit_type, consultation_type,
                         consultation_fee_in_paise, services_total_in_paise,
                         clinician_id, created_by)
    values (p_clinic_id, p_patient_id, v_complaint_id, v_session_id,
            current_date, visit_rec->>'complaint_name',
            visit_rec->>'visit_type', visit_rec->>'consultation_type',
            (visit_rec->>'consultation_fee_in_paise')::integer,
            (visit_rec->>'services_total_in_paise')::integer,
            (visit_rec->>'clinician_id')::uuid, auth.uid())
    returning id into v_visit_id;

    -- visit_services rows insert here, same shape, per visit_rec->'services'.

    insert into invoice_line_items (invoice_id, visit_id, amount_in_paise)
    values (v_invoice_id, v_visit_id,
            (visit_rec->>'consultation_fee_in_paise')::integer
          + (visit_rec->>'services_total_in_paise')::integer);
  end loop;

  for te_rec in select * from jsonb_array_elements(p_treatment_events)
  loop
    insert into treatment_events (clinic_id, patient_id, session_id, service_id,
                                   computed_amount_in_paise, clinician_id, date, created_by)
    values (p_clinic_id, p_patient_id, v_session_id, te_rec->>'service_id',
            (te_rec->>'computed_amount_in_paise')::integer,
            (te_rec->>'clinician_id')::uuid, current_date, auth.uid())
    returning id into v_te_id;

    for link_rec in select * from jsonb_array_elements(te_rec->'links')
    loop
      insert into treatment_event_links (treatment_event_id, complaint_course_id, catalog_region)
      values (v_te_id, (link_rec->>'complaint_course_id')::uuid, link_rec->>'catalog_region');
    end loop;

    insert into invoice_line_items (invoice_id, treatment_event_id, amount_in_paise)
    values (v_invoice_id, v_te_id, (te_rec->>'computed_amount_in_paise')::integer);
  end loop;

  return v_invoice_id;
end;
$$;
```

This doesn't bypass any existing trigger — every insert here still fires whatever BEFORE INSERT guard already exists on that table (validate_visit_clinic_id(), process_new_complaint_course()'s cross-chain check, etc.), inherited for free because this is orchestration around ordinary inserts, not a bypass. session_id on visits/treatment_events proposed NOT NULL — safe to add directly given this project's actual current state (pre-integration, no real visit rows yet); would need a nullable-then-backfill path if added later, per the migration-safety framework in schema-study-plan-v2.md Module 9. A validating trigger confirming session_id belongs to the same patient+clinic is optional defense-in-depth, not load-bearing, given the single-function design already makes a mismatch structurally hard to produce. This also resolves frontend-schema-audit-handover.md's Complaint Selector finding (#4) as a side effect — its suggested fix (an explicit early write before advancing the stepper) is unnecessary once complaint-course creation is just the first loop iteration inside the same coordinated function. Left open: p_clinic_id derivation (staff-derived from auth.uid() vs. validated as given) is the already-logged open question in module_2_logs.md; not resolved here, but this function will eventually need an answer. Patient registration itself stays separate and earlier (SPECIALTY-10's patients_provision_initial_access), not folded into this commit.

**2. Visit/treatment_events routing — traction stress-tested, split holds.**

visits.complaint_course_id NOT NULL reaffirmed as correct, not just accepted. A visit's fee (exam fee, per-complaint therapy rate, tier, gap-return penalty, per README.md §2) is structurally complaint-scoped — there's no version of grand_total_in_paise that doesn't trace back to which complaint it's treating. Making it nullable wouldn't simplify anything; it would rebuild treatment_events' job — a bare-region link, no tier to reference — a second time, badly, inside visits.

Two independent axes decide routing, combined with OR: Axis 1 (instance-level) — does this logging pass have a complaint course to attach to right now? Axis 2 (catalog-level) — is this service a designated always-separate carve-out (dry needling/cupping/laser per README.md §2, exact list still pending Om)? A service instance lands in visit_services only when axis 1 is true AND axis 2 is false. Everything else — no complaint context, or a carve-out regardless of context — lands in treatment_events.

| | Has complaint context | No complaint context |
|---|---|---|
| **Ordinary service** | visit_services | treatment_events, bare catalog_region |
| **Named carve-out** | treatment_events, real complaint_course_id | treatment_events, bare catalog_region |

Tested against traction in three configurations: ordinary bundled (complaint context true, not a carve-out) → visit_services, matching the framing exactly. Standalone externally-prescribed (no complaint context) → treatment_events via bare region, axis 2 irrelevant once axis 1 fails. A patient with an active complaint that this traction isn't for (an unrelated external prescription landing on the same day as a real complaint-tied session) → still treatment_events, because axis 1 is about the logging context ("is this instance happening inside the visit for the complaint it's actually treating"), not a blanket "does this patient have any complaint on file" lookup — the case that would break a sloppier version of the rule. All three resolve correctly under one rule; the split doesn't need rethinking, it needed the rule written down.

```sql
alter table services
  add column is_specialty_carveout boolean not null default false;
```

Designed, not built. Considered and set aside: making this per-clinic (no evidence classification varies by clinic, and it would violate "no configuration UI for the MVP," README.md §6); leaving it as pure UI-section judgment with no backend flag (rejected — same "frontend responsible for correctness, backend never verifies" pattern already flagged as a real gap in module_2_logs.md's REAL BUGS/GAPS section).

**3. Invoice/session/payment click-mapping — resolved together with (1).**

"Create Invoice" is confirmed as the single action creating the session, every visit, every treatment event, the invoice, and its line items, all together — a direct consequence of (1)'s resolution, not an independent choice, since the deferred line-item-integrity trigger (SPECIALTY-08) requires every invoice to already have real line items at commit, which forces visits/treatment_events into the same atomic operation as the invoice they're billed on.

The visit-vs-session "which one gets marked complete" question is a false dichotomy under this model: a visits row can't exist without its parent sessions row (session_id NOT NULL, set inside the same function call), so they complete atomically, together, always. One sentence for what the button does: "Create Invoice" closes out the session, and every visit inside it is complete as a structural consequence of the session existing.

Pending-by-default confirmed as correct — and structurally required, not just convenient. If "Create Invoice" didn't actually commit right then, the button's own label would be dishonest, and a receptionist pulled away before "Confirm Payment" would believe they'd already created the invoice when nothing had actually been saved — recreating (1)'s orphaning problem, one screen later, while actively misleading about it. Pending is the state that makes the label honest. This adds no friction to the common case: Create Invoice and Confirm Payment happen back-to-back for an immediate-cash patient; Pay Later is the same mechanism with the second click simply deferred, not a separate path.

**Superseded 2026-09-03 (see SPECIALTY-02, Entry 31):** the confirm_invoice_payment() function shown originally here took three separate cash/online/card amount parameters. That design is walked back for MVP in favor of a single payment_mode parameter — see below.

```sql
-- ORIGINAL (superseded) — shown for the record, not the version to build
create or replace function confirm_invoice_payment(
  p_invoice_id             uuid,
  p_cash_amount_in_paise   integer default 0,
  p_online_amount_in_paise integer default 0,
  p_card_amount_in_paise   integer default 0
)
returns void
language plpgsql
security definer
as $$
begin
  update invoices
  set payment_status         = 'Paid',
      cash_amount_in_paise   = p_cash_amount_in_paise,
      online_amount_in_paise = p_online_amount_in_paise,
      card_amount_in_paise   = p_card_amount_in_paise,
      payment_mode = case
        when p_cash_amount_in_paise > 0 and p_online_amount_in_paise = 0 and p_card_amount_in_paise = 0 then 'CASH'
        when p_online_amount_in_paise > 0 and p_cash_amount_in_paise = 0 and p_card_amount_in_paise = 0 then 'UPI'
        when p_card_amount_in_paise > 0 and p_cash_amount_in_paise = 0 and p_online_amount_in_paise = 0 then 'CARD'
        else 'HYBRID'
      end,
      paid_by = auth.uid(),
      paid_at = now()
  where id = p_invoice_id;
end;
$$;
```

One UPDATE, everything set together, so chk_payment_breakdown_matches_total was satisfied atomically. payment_mode derived, never trusted from the client. This resolved two existing audit findings with concrete targets: schema_reconciliation_audit.md's InvoiceBuilder placeholder-invoice_number finding (the real mutation response is checkout_and_create_invoice() returning the id process_new_invoice() minted, arriving at Create Invoice, not Confirm Payment), and its FooterActions.tsx finding (Confirm Payment's onConfirm should call confirm_invoice_payment(...) directly, more specific than the audit's own vaguer suggested fix).

**Session Changelog**

**New: session write-timing (Entry 26).** Entry 15's client-echo trigger mechanism superseded by a single coordinated SECURITY DEFINER function, closing the orphaned-row risk structurally rather than procedurally. Needs updating: supabase_migration.md, resolves module_2_logs.md's open coordinated-vs-progressive question.

**New: visit/treatment_events routing rule (Entry 27).** Two-axis rule (instance complaint-context × catalog carve-out classification) written down for the first time and stress-tested against traction going both ways. complaint_course_id NOT NULL reaffirmed. services.is_specialty_carveout proposed. Needs updating: business-rules-log.md (README.md), supabase_migration.md.

**New: invoice/session/payment click-mapping (Entry 28).** Extends, doesn't contradict, SPECIALTY-11 Q1 — the same click resolved to also cover session/visits/treatment_events, and the visit-vs-session confusion resolved as a false dichotomy under the coordinated model. Needs updating: business-rules-log.md / future WORKFLOW.md.

---

# [03 Sep 2026] SPECIALTY-02

> **[COMPACTED PROMPT]**
> Multiple follow-ups, addressed one at a time. (1) Why was SPECIALTY-12 created as a separately-numbered, reworded duplicate of SPECIALTY-01, and going forward, which version is the source of truth, and who maintains the log across chats vs. within a chat? (2) Pushback on making invoices.patient_id nullable for anonymous retail sales with no fallback — even a non-patient buyer needs some identifying contact captured, for receipt/dispute purposes. (3) Direct challenge: the "override_status='pending' vs payment_status='Pending' naming collision" flag — what confusion, exactly, to whom, in what scope? (4) Clarify the Overdue-derivation recommendation precisely — is the CHECK value being removed, or just never automated? Where should the derivation actually live — frontend or backend — and does a strict "pending today, overdue starting tomorrow" rule hold up given hybrid/split payments are rare and no per-mode reporting UI is planned for MVP? (5) Pushback on the three-column cash/online/card payment breakdown — hybrid payments are rare, no reporting/filter UI is planned for MVP, the frontend doesn't capture this data yet either; should this be simplified to a single payment_mode value instead, deferring the structured breakdown entirely? (6) Draft a project custom instruction that tags every brainstormed option/approach with an MVP-scope classification (necessary vs. deferrable-with-a-cheap-backend-hook vs. defer-the-whole-module), referencing a separate prompt already added to the project explaining the underlying MVP-stripping motive. (7) Every log entry, not just session headers, should carry an explicit date, so multiple accumulating/conflicting log files across chats can be resolved by recency.

## SPECIALTY-02 — Fixing the log, MVP scoping decisions

**1. The SPECIALTY-01/12 mix-up.** A previous draft this session invented a cross-chat sequence number ("SPECIALTY-12") and independently reworded SPECIALTY-01's conclusions under it, rather than archiving what was actually said. Discarded outright, not reconciled. Fix, permanent: log entries are attributed to their own chat's native response label (which resets to 01 per chat, a fixed system rule) plus the real calendar date — never an invented cross-chat counter. The chat response is always the source of truth; the log's job is to preserve it under its real label, not generate a differently-worded twin. Maintenance split: within a chat, updates happen proactively as decisions land; across chats, there is no persistent reach — the user carries the canonical file forward by re-uploading it into each new chat, and the log gets appended to, correctly labeled and dated, at natural checkpoints.

**2. Anonymous retail sales need a captured contact, not a patient row.** invoices.patient_id nullable (SPECIALTY-09 Q2) was correct, but nullable-with-no-fallback was a real gap, not a style choice — a receipt identifying nobody is a genuine business risk. Fix is schema-level, not "create a patients row for a one-time buyer" (which would pollute a clinical table and require special-casing patient_clinic_access/RLS for someone with no clinical relationship at all):

```sql
alter table invoices
  add column walk_in_contact text;

alter table invoices
  add constraint chk_invoice_has_identity
  check (patient_id is not null or walk_in_contact is not null);
```

Every invoice needs either a real patient link or a captured contact string, never neither. Designed, not built.

**3. The "Pending" naming collision, scoped precisely by role.** To the person writing SQL: no confusion — different columns, different tables, no technical ambiguity. To a developer/DBA debugging a vaguely-worded bug report: a real but minor risk (a five-minute misdirection, not a data problem). To a future UI, only if a dashboard ever merges both concepts under one undifferentiated badge — not built yet, so a forward-looking flag, not a current issue. To the patient: not applicable. Classification unchanged: worth knowing, not worth architecting around.

**4. Overdue — clarified and derived as a view.** Correction to prior phrasing: the CHECK constraint's 'Overdue' value isn't being removed; what's being avoided is any automated process (cron, trigger) writing it into payment_status. The column itself only ever holds 'Paid' or 'Pending' in practice; 'Overdue' is computed fresh at read time, in one place, so every consumer (frontend, dashboard, future reports) reads the same derived label instead of reimplementing the day-math independently.

```sql
create or replace view invoices_with_effective_status
with (security_invoker = true) as
select
  i.*,
  case
    when i.payment_status = 'Pending' and i.date < current_date then 'Overdue'
    else i.payment_status
  end as effective_status
from invoices i;
```

The stated one-day rule (pending today, overdue starting tomorrow) maps directly onto `i.date < current_date` — no adjustment needed. Flagged, not resolved: this sits in tension with the already-logged "settles weekly" Pay Later pattern — a strict one-day threshold means routine weekly-settling patients show Overdue for most of the week as a matter of course. That may be intentional (a soft flag, no judgment implied) or may argue for a longer threshold matching actual expected settlement rhythm. Threshold value is an explicit open business decision, not resolved here — the mechanism is identical regardless of the number chosen.

**5. Payment-mode breakdown simplified for MVP — agreed, with independent justification.** The three-column design (SPECIALTY-09 Q4) was justified specifically by cash-drawer reconciliation reporting; that use case is now explicitly not wanted for MVP (no filter UI, hybrid payments rare, frontend doesn't capture per-mode amounts anywhere). Beyond simple deference: building the three columns now, with nothing in the frontend able to populate them correctly, would let a genuine hybrid payment silently record as `payment_mode='HYBRID'` with all three amount columns defaulted to zero — false precision, worse than not having the columns. Revised for MVP:

```sql
alter table invoices
  add constraint chk_payment_mode
  check (payment_mode is null or payment_mode in ('CASH', 'UPI', 'CARD', 'HYBRID'));

create or replace function confirm_invoice_payment(
  p_invoice_id   uuid,
  p_payment_mode text
)
returns void
language plpgsql
security definer
as $$
begin
  update invoices
  set payment_status = 'Paid',
      payment_mode   = p_payment_mode,
      paid_by        = auth.uid(),
      paid_at        = now()
  where id = p_invoice_id;
end;
$$;
```

This supersedes the three-parameter confirm_invoice_payment() shown in SPECIALTY-01 above. Trade-off named explicitly and accepted deliberately: any invoice recorded as 'HYBRID' during this simpler era has no recoverable cash/online/card split later. Given the stated rarity, a reasonable trade, not an oversight. The three-column upgrade remains exactly as cheap to add later as it would be now if reconciliation reporting ever becomes real post-MVP.

**6. MVP-scope tagging instruction, drafted.** The referenced project file explaining the MVP-stripping motive wasn't found in /mnt/project/ (only the same 8 files already on hand) — drafted from context given directly in this session instead. Extends the existing 🔴/🟡/🟢 severity convention rather than replacing it, adding the "does this need to exist at all right now" axis that severity alone doesn't answer:

> **MVP-scope tagging for brainstormed options** — whenever multiple approaches are being compared, tag each with: (1) MVP necessity — Required for MVP / Backend hook now, frontend deferred / Whole module deferred / Post-MVP nice-to-have; (2) Severity — 🔴 Structural / 🟡 Safety-net / 🟢 Performance (existing convention, reused); (3) Blast radius if deferred — one sentence on retrofit cost, or "none — purely additive." Attached to each option as presented, not summarized afterward.

**7. Dating every entry.** Confirmed and adopted — the actual fix for cross-chat/cross-file disambiguation that the invented counter in thread 1 was a flawed attempt at solving. Session headers keep their existing human-readable date format; every individual numbered entry's Source line now carries an explicit ISO date going forward. The 25 original entries could be backfilled from their session headers if ever wanted; not urgent.

**New entries this session**

**Entry 29 — Anonymous retail sale still needs a contact record**
- **Source:** 2026-09-03, SPECIALTY-02.
- **Old position:** SPECIALTY-09 Q2 recommended invoices.patient_id nullable for anonymous retail, with no fallback requiring any identifying info.
- **New position:** nullable is correct, but needs a fallback — invoices.walk_in_contact text, with CHECK (patient_id IS NOT NULL OR walk_in_contact IS NOT NULL). No patients row created for one-time non-clinical buyers.
- **Status:** designed, not built.
- **Where it lands:** supabase_migration.md, alongside the product_sales design from SPECIALTY-09 Q2.

**Entry 30 — Overdue is a derived label, computed in one view**
- **Source:** 2026-09-03, SPECIALTY-02.
- **Old position:** SPECIALTY-09 Q3 said Overdue "can be a read-time computation" without specifying where, or giving a concrete threshold.
- **New position:** a security_invoker view (invoices_with_effective_status), payment_status itself never written as 'Overdue' by any automated process. One-day threshold (`date < current_date`) matches the stated rule exactly; flagged tension with the "settles weekly" pattern — actual threshold value left as an explicit open business decision.
- **Status:** designed, not built.
- **Where it lands:** supabase_migration.md (view), business-rules-log.md (threshold value, once decided).

**Entry 31 — Payment-mode breakdown simplified for MVP**
- **Source:** 2026-09-03, SPECIALTY-02.
- **Old position:** SPECIALTY-09 Q4 recommended three dedicated amount columns (cash/online/card) with a sum-matching CHECK, for cash-drawer reconciliation reporting.
- **New position:** walked back — the reporting use case is explicitly not wanted for MVP, and the frontend can't populate a structured breakdown yet, meaning the columns would silently default to zero on real hybrid payments (false precision, worse than absence). Single payment_mode text column with a CHECK allowing 'HYBRID' as a value, no breakdown. confirm_invoice_payment() simplified to one parameter, superseding the version shown in SPECIALTY-01. Trade-off named and accepted: historical HYBRID invoices during this era have no recoverable split.
- **Status:** revises SPECIALTY-09 Q4; not built either way yet.
- **Where it lands:** supabase_migration.md, business-rules-log.md.

Thirty-one entries now.
