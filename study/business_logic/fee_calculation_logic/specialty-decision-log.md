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
