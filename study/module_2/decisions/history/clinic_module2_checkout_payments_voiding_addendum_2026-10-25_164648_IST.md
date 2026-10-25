# Clinic Module 2 — Checkout, Payments, Invoice Corrections & Clinical Voiding
## Successor discussion addendum — exhaustive checkpoint

**Created:** 2026-10-25 16:46:48 IST (`Asia/Kolkata`, UTC+05:30)  
**Chronological identifier:** `20261025-164648-IST`  
**Document type:** **NEW ADDENDUM — independent of, and not an edit to, the parent logs**  
**Discussion span:** the entire resumed conversation up to the request to create this addendum; includes the later GitHub migration review.  
**State:** decision/discussion record only. **No GitHub files, Supabase schema, SQL migrations, production data, or parent documents were changed.**

### Precedence and reading instructions

This is a **delta/addendum**, not a rewrite of the 18 October checkout log or the 23 October Module 2 canonical/addendum documents. It captures **all decisions, clarifications, proposals, challenges, small observations, and unresolved items from this conversation**. Where the newer conversation explicitly changed an earlier OPEN status, the newer explicit choice takes precedence **for that subject only**. Other decisions in the parent logs remain as they were. Statements from historical repository documents are **source evidence**, not fresh user approval or proof of the deployed Supabase state. Do not infer a new feature from a schema candidate.

**Decision tags:** `LOCKED` = expressly approved by user; `SETTLED-PRIOR` = a decision found in newer Module 2 project documents, not newly approved here; `CONDITIONAL-CHOICE` = selected design **if** that currently optional module is built; `PROPOSED-BY-USER` = suggested by user, **not** finalized; `PROPOSED-BY-ASSISTANT` = suggestion only; `ESTABLISHED` = accepted conceptual distinction; `OPEN` = a decision remains; `DEFERRED` = not planned for current MVP; `REMOVE-INTENT` = user directs omission from planned design but migration cleanup hasn't happened; `INVESTIGATE` = purpose/dependencies not understood; `SUPERSEDED` = old framing displaced. Technical choices must not silently become business decisions.

---

## 1. Chronological ledger of this conversation

1. **Resume from the 18 Oct v2.0 checkout/payment log.** It was treated as authoritative for that discussion; changes should be tracked in chat, not silently written to the uploaded original. Initially targeted its `OPEN-01` (additional partial receipts on an existing Due invoice). **[CONTEXT]**
2. **Admin receives a further partial payment on an older Due invoice.** The user explicitly accepted recording it and **keeping the remainder Due**, without automatically waiving it. Example: original ₹500, prior receipt ₹200, existing Due ₹300; later ₹100 receipt → collected ₹300, still Due ₹200. **[LOCKED; closes old OPEN-01]**
3. **Successive repayments are unrestricted by an arbitrary count.** User stated admins are the human-in-the-loop and affirmed they may accept additional partial payments repeatedly over time. Repeated payment entries do **not** imply authorization for receptionist to operate on an older Due invoice. **[LOCKED]**
4. **Central payments table ratified.** One positive payment/receipt event = one separate `payments` record tied to an invoice. Thus ₹200 + ₹100 + ₹200 against one invoice are **three** rows, not a mutable grand total, while ₹0 and waivers create **no payment** rows. **[LOCKED concept; exact DDL OPEN]**
5. **Payment event/state transition atomicity ratified.** Inserting a payment and any necessary invoice status/settlement transition must be **one atomic database transaction**; both commit or neither does. This does not by itself solve lost-response retries, duplicates or concurrent race conditions. **[LOCKED behavior; implementation OPEN]**
6. **User questioned invoice header columns.** Discussed whether `invoices` should carry `billed_amount`, `paid_amount` and `due_amount` at all, versus deriving them from related tables. Assistant distinguished mathematical totals from workflow status. **[DESIGN DISCUSSION]**
7. **Amounts are derived, not redundantly stored on invoices.** Original billed amount from invoice line amounts; collected from actual payment rows; outstanding from billed minus collected minus approved waivers. User endorsed this derived model for MVP. The Oct 23 addendum **already separately settled** no `invoices.total_amount_in_paise` in the proposed new model. Exact waiver persistence and read query/view performance remain open. **[LOCKED direction; physical schema partially OPEN]**
8. **Invoice workflow status can remain independent.** `Pending` and `Due` may have the same outstanding arithmetic, with different operational meaning. Keeping an invoice status field was an **assistant recommendation**, not an independent, newly locked column design. **[PROPOSED-BY-ASSISTANT / prior status policy established]**
9. **Historical billed lines freeze at issuance.** The prior Oct 23 Module 2 documents already say description and amount are immutable for issued invoices even when Pending/Due; paid invoices have an even stronger full financial lock. **[SETTLED-PRIOR; confirmed by targeted document inspection]**
10. **User-origin append-only proposal.** User proposed that existing issued invoice lines never be updated/deleted but **additional new lines might be appended under the same invoice if a missing/underbilled service needs adding**. This is explicitly **PROPOSED BY USER, NOT LOCKED**, optional and only if truly needed. **Do not credit the assistant as originator.** **[PROPOSED-BY-USER]**
11. **Overbilling idea also from user.** If the operator charged for a service that should not have been charged (including an incorrectly selected procedure), user suggested an **admin waiver with an explanatory note**, retaining the original invoice line rather than rewriting it. Whether this is appropriate for all pricing/clinical errors is **not settled**. **[PROPOSED-BY-USER]**
12. **Important distinction surfaced:** A waiver adjusts how much is collected, but does **not** correct an inaccurate clinical record. Example: a mistaken cupping entry may remain as if performed even after its ₹500 charge is waived. The 23 Oct canonical log explicitly warns that pricing mistakes and clinical mistakes are not automatically corrected by waivers. **[ESTABLISHED risk]**
13. **The assistant suggested “Entered in Error” clinical record marking.** Preserve source record; exclude erroneous event from true performed-treatment interpretation; optionally capture rationale and actor/time. User questioned complexity and naming. **[PROPOSED-BY-ASSISTANT]**
14. **User challenged three extra fields.** `created_by`/`created_at` already identify initial author and entry time; assistant explained these differ from who **voided** the entry and **when**. User later opted to retain correction actor/time as guardrails after all. **[CLARIFIED]**
15. **Boolean-based invalidation proposed by user.** User preferred a simple marker similar to an active flag, initially asking for a generic “invalid” rather than a specialized phrase. Assistant compared `is_invalid`, `is_voided`, `is_retracted`, and `is_valid`, recommending `is_voided`. **[PROPOSED-BY-USER, naming discussion]**
16. **User preferred `is_voided`.** User said “I think is voided is good,” then explicitly selected `is_voided`, `void_reason`, `voided_by`, and `voided_at` as a sensible **conditional** structure if a voiding feature is implemented. Field types, nullability, permissions, strict constraints and trigger implementation were not ratified. **[CONDITIONAL-CHOICE]**
17. **Whether to build voiding in MVP remains OPEN.** User explicitly requested leaving the *entire module's inclusion* undecided until thinking through more cases. Do **not** mislabel conditional field choice as MVP authorization. **[OPEN]**
18. **Who may void remains OPEN.** Assistant asked admin-only versus original receptionist/other authorized clinical staff; user did not settle this. “Admin only” must **not** be asserted as approved from unclear spoken phrasing. **[OPEN]**
19. **One-way invalidation proposed by assistant.** Suggested disallowing casual `true → false` reactivation to preserve correctness; user did **not** approve a strict one-way constraint. **[PROPOSED-BY-ASSISTANT]**
20. **GitHub explicitly authorized, with narrow scope.** First searched prior discussion and the Module 2 `/decisions/` folder for correction/immutability/waivers. User asked for targeted keyword queries rather than indiscriminate full-document reads. Inspected Oct 23 canonical log and successor addendum relevant passages. Later user expressly authorized opening the **Markdown migration**; located `docs/schema/supabase_migration.md` and audited its 16 `CREATE TABLE` definitions. **[SOURCE PROCESS]**
21. **Migration evidence: existing `clinical_notes` has no soft-delete flag.** The documented table contains `is_critical`, `created_by` and `created_at`, not `is_active`, `is_deleted`, or `is_voided`. The Markdown migration grants a broad `v1_allow_all` policy, including DELETE for authenticated roles (subject to FKs; not confirmation of deployment). **[VERIFIED IN REPOSITORY FILE]**
22. **Original `visit_services` lacks audit creation fields.** The Markdown migration's definition lacks `created_at` and `created_by`; this was identified as a future provenance gap **if voiding is built**, not an approved migration. **[VERIFIED / OPEN REVIEW]**
23. **Candidate `visit_services` voiding:** correction for performed-procedure entry error (e.g., cupping never actually performed). Strong candidate **if** feature is adopted; not locked for MVP. **[OPEN]**
24. **Candidate `visits` voiding + child consistency:** user observed that if a visit is voided, all child `visit_services` must also be treated as voided. Mentioned a **trigger**, and assistant noted alternative of deriving effective child validity from the parent. **Invariance is a proposed requirement conditional on visit voiding; physical trigger vs derived status is OPEN.** **[PROPOSED-BY-USER requirement; mechanism OPEN]**
25. **Candidate `complaint_courses` voiding:** despite deliberate creation, some could be invalid or wrongly attributed; user saw potential value. Its current `Active`/`Completed` lifecycle is **not** voiding. Scope/implementation OPEN. **[OPEN]**
26. **Candidate `clinical_notes` voiding:** user wondered whether `is_deleted`/`is_active` already exists; read-only migration inspection confirmed neither. If a note was entered incorrectly, preserve original and flag as voided rather than hard-delete was discussed, **not approved**. **[OPEN]**
27. **`patient_vitals` excluded from MVP.** User said patient vitals are not being recorded and no corresponding frontend is planned; the table was AI-added rather than requested. Whether to **drop table** or **leave dormant** is undecided. No voiding work on this table in V1. **[DEFERRED; physical removal OPEN]**
28. **`patient_alerts` slated for removal from proposed MVP.** User regards it as unnecessary duplication of clinical notes, created from a mistaken AI interpretation of demo UI. Mark as **REMOVE-INTENT BY USER**, contingent on dependency/data checks before actual migration. Do not claim already removed. **[REMOVE-INTENT]**
29. **`timeline_events` explicitly under investigation.** User has not validated its purpose and suspects unnecessary AI-added structure. The migration shows it as a clinical timeline record with optional `visit_id` and `note_id` references; **not the same as** the `daily_ledger` view. Need check real frontend usage and referential dependencies before keeping or deleting. **[INVESTIGATE]**
30. **Daily ledger / view filter concern.** The migration's `daily_ledger` view reads `visits` and hardcodes `'Complete'`, with **no `is_voided` filter**. If `visits.is_voided` is adopted, **only genuinely valid visits should appear as performed**; update/redefine view with a valid-visit filter, or otherwise expose a clear voided status. This is a **conditional future view change**, not SQL currently applied. The hardcoded paid/complete semantics also conflict with newer Pending/Due flows and are an independent pre-existing reconciliation need. **[CONDITIONAL TECHNICAL DEPENDENCY / OPEN]**
31. **Standalone performed treatments:** a separately modeled `standalone_treatments` / service-occurrence record might need the same error invalidation mechanism; existence/name/fields of that model are outside this addendum's approval. **[PROPOSED-BY-ASSISTANT / O11 OPEN]**
32. **Invoice integrity with clinical voiding:** voiding a visit or procedure **must not** silently delete or reprice an existing immutable invoice line, bypass the Paid lock, alter recorded cash, or make a previously charged source vanish. Need explicit linked financial-resolution policy. **[OPEN required audit if voiding built]**
33. **Broader candidate scan:** 16 current tables were assessed. No rationale was established to add `is_voided` to all 16. `clinics`, `profiles`, `patients`, `patient_clinic_access`, `complaint_catalog`, `services`, `clinic_service_prices` should not automatically receive the flag; existing `is_active` flags and configuration/deactivation are different semantics. `packages` and `invoices` have separate financial/lifecycle challenges; see table audit. **[ASSISTANT ASSESSMENT; NOT approval]**
34. **Request for an addendum rather than modifying canonical logs.** User asked for a timestamped, comprehensive addendum plus an Excel file, including the small proposed trigger, view filter, open items, provenance and precise statuses. **[CURRENT REQUEST]**

---

## 2. Consolidated decisions/constraints newly locked **in this discussion**

### 2.1 Older Due invoice: repeated partial receipts

**LOCKED — user expressly approved.** Admin can record another *positive* payment on an older Due invoice and keep the remaining amount Due, without finalizing and waiving it. Unlimited successive receipts within remaining balance; not an arbitrary number of repayments. Receptionist still cannot edit an older invoice. Example:

| Action | Collected to date | Outstanding | Invoice state |
|---|---:|---:|---|
| Original invoice of ₹500; patient initially pays ₹200 | ₹200 | ₹300 | Due |
| Later admin records actual ₹100 receipt | ₹300 | ₹200 | **Due** |
| Later admin records actual ₹50 receipt | ₹350 | ₹150 | **Due** |
| Ultimately receives ₹150 or formally waives outstanding (authorized admin) | ₹500 cash OR lawful settlement | ₹0 | Paid/settled |

The UI likely needs separate concepts for **record receipt, leave Due** vs **settle and waive balance**; exact button labels and validation form remain OPEN. ₹0 is not a receipt/payment row. Protect against duplicate submissions and recording more money than outstanding; mechanism not decided.

### 2.2 Payments table and transaction boundary

**LOCKED conceptual:** One common `payments` table, one row for each actual strictly positive receipt; many receipts may reference one invoice. Payment method and actual received time are necessary concepts; original decision log's suggested `invoice_id`, `amount_paise`, `payment_method`, `received_at`, `recorded_at`, `recorded_by` are **field proposals**, not signed-off DDL. The user expressly ratified **atomic** payment creation + any necessary invoice status/settlement update.

**Still OPEN:** final field types, validation, exact method enum, alternate tender splits, permission enforcement, late receipt date correction, what to do with mistakenly recorded payments before Paid, payment idempotency key, concurrent admin actions/over-collection, and enforcement of fully Paid immutability. A receipt-only transaction may keep invoice `status=Due` unchanged; it must nonetheless be validated and committed atomically with any required state change.

**Separate boundaries:** This acceptance does not itself address the treatment+invoice issuance transaction. Another, newer 23 Oct canonical register records normal checkout atomicity as settled in that workstream; do not present it as newly approved by the payment atomicity vote here. Prior 18 Oct handoff's old `OPEN-09` has newer independent project evidence.

### 2.3 Source-of-truth and derived money fields

**User-endorsed / corroborated by 23 Oct successor addendum:**

```text
Original billed  = SUM(invoice_lines.amount_in_paise)        [immutable issued charge facts]
Actual collected = SUM(payments.amount_in_paise)             [positive receipt facts]
Approved waived  = SUM(authorized adjustment/waiver amounts) [exact representation OPEN]
Outstanding      = Original billed - Actual collected - Approved waived
```

Do not persist redundant `invoices.billed_amount`, `invoices.paid_amount`, or `invoices.due_amount` *as independent authorities* in the proposed MVP design; invoices derive financial totals via aggregates/views/queries. This is separate from storing an invoice's `Pending / Due / Paid` **workflow status**. Any performance cache/materialized computation is a **future** optimization requiring consistency guarantees, not today's requirement. Invoice-line amount snapshot must remain stable despite catalog price changes.

**Repository correction:** the Markdown migration currently **does** store `invoices.amount_in_paise`, `payment_mode`, and `payment_status` with values `Paid/Pending/Overdue`, and has **no** `invoice_lines`, `payments` or `waiver_adjustments` tables; that is the **older migration blueprint**, not the newer proposed target and not live deployment proof.

---

## 3. Invoice lines, undercharges, overcharges and clinical accuracy

### 3.1 Prior settled rule versus user-origin change proposal

- **SETTLED-PRIOR:** Original billed lines/amounts/descriptions freeze at invoice issuance, even if invoice remains Pending/Due. A Paid invoice locks financial records/settlement for the MVP.
- **PROPOSED BY USER; NOT RATIFIED:** *Existing* lines remain unmodified, but **new positive lines may optionally be appended** for missed billable service/undercharge after issue. This proposal could alter the total on an issued invoice and **conflicts with the prior freeze at issuance**. It cannot be called approved or silently implemented. Need explicit new exception rule, authority/status boundary, source verification, whether old Paids remain impossible to adjust, and validation against amounts already paid/waived.
- **PRE-ISSUANCE ADDING IS DIFFERENT:** Adding a service in a frontend preview/draft **before** clicking final Create Invoice does not breach issued-line immutability. The Oct 23 successor addendum's draft-first path remained recommended, not proof that post-issuance appends were approved.
- **PROPOSED BY USER; NOT RATIFIED:** For overcharging, preserve existing line and apply an authorized **waiver with a note** rather than deleting/repricing the line. This corrects *collectibility*, not necessarily what was correctly chargeable or what clinically happened. Need decide when waiver is suitable versus genuine price correction, mistaken service/charge, credit or void, and whether a waiver reason/category must distinguish these.

### 3.2 Clinical mismatch scenario

A receptionist accidentally selects cupping even though cupping **never occurred**. It becomes a clinical occurrence, and possibly a billed ₹500 line. Admin waives ₹500, patient owes ₹0; **but the clinical record still lies** unless it can be invalidated/annotated. Proposed clinical `is_voided` would classify the occurrence as wrongly entered while retaining evidence of the original entry. Waiving an invoice is neither a substitute for validating a clinical event nor an authorization to edit past billed lines. Financial document correction after invoice Paid is explicitly deferred in the old payment log.

**Other variations needing separate answers:** (a) correct service performed but price entered too high; (b) service never performed but patient already paid; (c) truly performed but duplicated by operator; (d) genuinely performed and omitted from issued bill; (e) package-covered visit wrongly charged; (f) a visit voided while it has one or several paid/unpaid invoice lines. These are risk scenarios, **not approved processes**.

---

## 4. Conditional clinical voiding schema: fields and semantics

**MVP voiding module = OPEN.** No tables or UI modifications approved merely by defining a shape.

**CONDITIONAL-CHOICE BY USER — preferred field names if implemented:**

```sql
-- Concept sketch ONLY; do not execute as a migration.
is_voided   boolean not null default false,
void_reason text,
voided_by   uuid,
voided_at   timestamptz
```

- `is_voided`: row is preserved but should not be treated as a valid/performed occurrence. User selected this generic term over `is_entered_in_error`, `is_invalid`, `is_retracted` or `is_valid`.
- `void_reason`: explanation/notes describing why entry should no longer be treated as true. Whether this is mandatory, optional, structured, free text or role-specific remains OPEN. Do not assume `void_reason` nullable or required from example syntax.
- `voided_by`: **who performed the later void**, distinct from the original `created_by`. User accepted retaining this even after initially questioning extra audit columns. Whether FK to `profiles(id)`, server derivation via `auth.uid()` and immutability are implementation questions.
- `voided_at`: **when the later void happened**, distinct from original `created_at`; recommended DB/server-stamped and user accepted inclusion. Its exact database constraint remains OPEN.
- `created_at` and `created_by` are not substitutes for `voided_at`/`voided_by`: creation and retraction are separate events.
- Assistant suggested `false → true` only (no silent reactivation), but that was **not approved**. Likewise, allowing only admin to void was **not resolved**. UI design, undo/reactivation, note validation, authorization and historical audit behavior all OPEN.

### Suggested consistency constraints — *not yet approved*

1. If voided, later actor/timestamp should normally exist, and perhaps a reason; if unvoided those fields should normally be null; exact row CHECK and server-side actor stamping not chosen.
2. For **voided parent visit**, every child `visit_services` must be treated as not performed/valid. User explicitly identified a **trigger** as likely; assistant offered alternative **derived effective void status** (parent OR child). Choose whether to propagate child flags physically or filter/derive; a direct `UPDATE` of all children is not automatically the only correct approach.
3. Filtering reports/views/API queries is essential. A flag that only exists in the row but leaves the user-facing charts and billing/clinical interpretations untouched would be ineffective.
4. A void must not masquerade as hard deletion, undo an invoice, create a refund, modify cash, or bypass existing locking rules. Need reconcile with immutable billed source FKs and invoice lines.

---

## 5. Read-only migration review: 16 existing table definitions and voiding disposition

**Source inspected:** `docs/schema/supabase_migration.md` in GitHub, a Markdown document containing an older Supabase SQL migration. It defines **16 `CREATE TABLE` blocks**; later versions may differ from deployed Supabase. The table list below is evidence of that file, not a deployed inventory.

| # | Existing table | Relevant existing fields/behavior in source | This conversation's classification | User approval boundary / required follow-up |
|---|---|---|---|---|
| 1 | `clinics` | `is_active`, configuration, invoice counter | **NO DEFAULT VOID PROPOSAL** | Active/inactive clinic operations differ from voiding a historical entry. |
| 2 | `profiles` | role, clinic_id, created/updated metadata | **NO DEFAULT VOID PROPOSAL** | Deactivation/access lifecycle is a separate authorization concern. |
| 3 | `patients` | `is_active`, timestamps | **NO DEFAULT VOID PROPOSAL** | Existing soft-inactivation differs from treating patient data as created in error; duplicates may need a merge strategy. |
| 4 | `patient_clinic_access` | unique patient/clinic membership and FKs | **NO DEFAULT VOID PROPOSAL** | Review access revocation separately, not automatically via a generic flag. |
| 5 | `patient_alerts` | type/label, no soft-delete flag | **REMOVE-INTENT** | User says AI-created duplication of clinical notes; omit from proposed MVP, audit dependencies/data before dropping. |
| 6 | `patient_vitals` | type/value/recorded_at/trend; no void flag | **DEFERRED** | User not recording vitals; no frontend. Drop table vs leave unused is OPEN; no void work for MVP. |
| 7 | `clinical_notes` | `is_critical`, `created_by`, `created_at`; NO `is_active`/`is_deleted` | **CANDIDATE; OPEN** | Check deletion ability/current UI and consider correction via void metadata. Migration's broad RLS policy allows authenticated DELETE in blueprint, but FKs may block. |
| 8 | `complaint_catalog` | `is_active` | **NO DEFAULT VOID PROPOSAL** | Catalog availability is not retrospective clinical invalidation. |
| 9 | `complaint_courses` | `Active/Completed` status, visit relationships | **CANDIDATE; OPEN** | Mistakenly created complaint may be voided, but impact on associated visits/invoices must be defined. |
| 10 | `services` | catalog definitions, name/category/standalone price | **NO DEFAULT VOID PROPOSAL** | Inactive/unavailable catalog service (branch config) is distinct from false performed occurrence. |
| 11 | `clinic_service_prices` | per-clinic price row, updated_at | **NO DEFAULT VOID PROPOSAL** | Branch eligibility and price edits are different functionality; newer catalog design open. |
| 12 | `visits` | complaint course, date, old amount fields; no void field | **CANDIDATE; OPEN** | User proposed all child procedures must be effectively voided with parent; trigger or derived validity OPEN. |
| 13 | `visit_services` | visit FK; service snapshot/charge fields; no creator stamps | **PRIMARY CANDIDATE; OPEN** | Accidental performed-service entry; proposed void fields if module built. Audit `created_by`/`created_at`. |
| 14 | `packages` | `Active/Completed/Expired` + day_log | **SEPARATE FINANCIAL/LIFECYCLE ANALYSIS; OPEN** | Incorrect creation, cancellation, activation/redemption need bespoke policy; generic void not chosen. |
| 15 | `invoices` | older `amount_in_paise`, `payment_status Paid/Pending/Overdue`, `payment_mode`, one `visit_id` | **DO NOT AUTO-ADD VOID** | Refund/void/correction, Paid lock, invoice numbering/charge authority remain distinct. Financial voiding DEFERRED/OPEN per parent. |
| 16 | `timeline_events` | title/description/timestamp/category, optional visit/note FK | **INVESTIGATE BEFORE ANY VOID DESIGN** | User doesn't know purpose, suspects AI overbuilding; inspect frontend usage and whether its data are redundant/generated/manual. |

**Important source-specific schema facts:**

- The migration has **no `invoice_lines`, `payments` or waiver adjustment table**, notwithstanding their more recent conceptual approval/direction. Do not claim otherwise. It also lacks a standalone-treatment occurrence table under that name.
- Old `visits` comment says row existence means paid/complete, and old `daily_ledger` view hardcodes `'Complete'`: these do **not** reflect the newer Pending/Due/payment separation and require eventual reconciliation independent of voiding.
- Existing `visit_services` has `is_charged`, `charged_amount_in_paise`, whereas Oct 23 successor design calls to remove clinical occurrence charge flags/amounts and have the invoice line own the charge. This is old-vs-new blueprint difference, **not permission for changing code now**.
- `patient_alerts` and `timeline_events` may have dependent triggers, RLS policies, indexes, FK references, frontend components, and potentially existing data. Proposed cleanup requires dependency audit and safe migration path, not blind `DROP TABLE`.
- The `daily_ledger` is a **VIEW, not one of the 16 tables**. It selects `visits` joined to `patients`, and lacks any validity filter. If `visits.is_voided` is added, update the view deliberately—e.g., filter `WHERE v.is_voided = false` if invalidated visits should be entirely absent; alternatively show voided rows with explicit status if audit view is desired. User specifically requested not to lose this concern.

---

## 6. Impact chains, risks and possible tests — no implementation authorization

### 6.1 Void parent → child treatment consistency (user-identified gap)

**Goal if both rows have void semantics:** A voided `visit` must not cause its `visit_services` to appear as genuinely performed.

**Candidate A: propagate physical flags (trigger).** On a valid transition to `visits.is_voided=true`, mark all child `visit_services` voided (possibly record effective actor/time/reason); guard new child inserts under voided parents, bulk import and concurrent writes; avoid duplicate/lossy correction history. This is **not approved DDL**.

**Candidate B: derive effective void status.** Treat a child as voided when `child.is_voided OR parent.is_voided`; existing children need not be physically updated; clinical queries/views must consistently join/filter, and standalone child void still works. **Assistant alternative; not approved.**

**OPEN:** Which truth should be stored? What happens if parent voided accidentally? Should an administrator be able to unvoid? What if an invoice already references parent/child? If a complaint course is voided, should visits also inherit void? These are separate parent levels—not automatically an instruction to cascade every ancestor flag.

### 6.2 Conditional `daily_ledger` adjustment (user-requested traceability)

Current historical SQL (illustrative excerpt):

```sql
create or replace view daily_ledger as
select ...
from visits v
join patients p on p.id = v.patient_id;
-- It currently hardcodes 'Complete' and contains no visit-void filter.
```

**Conditional if visit voiding is implemented:** redefine read semantics. Likely `WHERE NOT v.is_voided` for a *performed-only* daily ledger, **OR** display an explicit voided status and exclude from performed/revenue counts. The choice of hidden vs visible voided historical entries is **OPEN**. The existing hardcoded `'Complete'` must also be revisited for Pending/Due statuses (separate earlier issue). Add regression cases: void one visit with 2 procedures; ensure daily ledger, procedure history and report counts exclude them as performed while admin can still inspect original entries.

### 6.3 Financial + authorization controls

- **Immutable billed amounts vs optional appended lines:** user append-only line proposal conflicts with issued freeze; finalize one behavior before schema. Old Paid lock remains.
- **Clinical void vs actual payment:** financial billed/collected/waived facts must not change merely because a clinical source is voided; necessary financial remedy could be waiver, void/credit/new document, or an explicit exception depending on policy—not auto-selected here.
- **Waiver with note for an overcharge:** user-origin proposal. Need distinguish discounted service legitimately performed, billing error, bad debt and fraudulent/nonperformed service; reporting classification not settled. If billing mistake already fully paid, current MVP Paid lock forbids retroactive changes.
- **Actor stamping and state transition guard:** `voided_by`/`voided_at` should not be arbitrary client-supplied values if protection matters; server derivation is recommended only. Restrict updates via role/RLS and authorize parent-child changes in one transaction if physical propagation chosen.
- **DELETE/cascade warning:** several existing FKs use `ON DELETE CASCADE` (e.g., patient→visits and visit→visit_services). That's physical deletion, **not** the proposed `is_voided` soft invalidation. Avoid unintended deletion of financial/clinical history; deletion policy requires separate review.
- **Multiple independent invoices and source integrity:** constraint/retry/idempotency policy remains OPEN in the later Module 2 successor addendum; a void flag does not solve duplicate billing or guarantee truthful invoice source references.

### 6.4 Candidate acceptance scenarios (reasoned, not executed)

1. Repeated admin receipt: old Due ₹300 → receive ₹100 → remain Due ₹200; another ₹50 later → Due ₹150. Each positive receipt is separate; no waiver on either receipt.
2. Final settlement: admin receives rest or explicitly waives it → Paid; future edits/receipts blocked by Paid lock.
3. Same server operation fails after payment insert before valid status transition → entire write rolls back; retry must not duplicate receipt.
4. Invoice billed two immutable lines ₹300+₹200, receives ₹200+₹100 → billed ₹500, collected ₹300, outstanding ₹200; no duplicated header amount is authority.
5. Undercharge discovered after issuance → user optional append-line proposal tested against original issued freeze and role/status controls; **not automatically permitted**.
6. Nonperformed cupping recorded → clinic invalidates occurrence (if built), optionally void note; original charge stays in financial history until independent settlement/correction; no false performed-service report.
7. Voided visit with ultrasound + exercise children → neither child may count as performed; test trigger vs effective-void derivation, and new inserts attempted afterward.
8. `daily_ledger` excludes (or explicitly flags) voided visits and doesn't silently count them as completed therapy; refund/payment status reconciliation unaffected.
9. Correctly performed service with incorrect price → distinguish billing correction from clinical invalidation; don't void truthful treatment record.
10. Patient vitals has no V1 frontend; patient alerts excluded from design only after confirming dependencies; timeline events purpose investigated before removal.
11. Clinic notes were recorded against wrong patient → decide correction permissions and whether note with `is_voided` remains reviewable to appropriate roles.
12. Paid invoice has associated clinical record later voided → clinical history may be flagged without changing Paid financial record; legitimate restitution workflow remains OUT OF MVP unless user reopens it.

---

## 7. Live issue register — carry forward one question at a time

| Issue | Priority | Status | Decision/question |
|---|---|---|---|
| N01 | P0 | **OPEN** | Build a clinical-voiding module in MVP at all? Which minimum entities justify it? |
| N02 | P0 if N01 yes | **OPEN** | `visits` parent void: **physical trigger propagation** to `visit_services` or derived `effective_is_voided`? How guard new child writes? |
| N03 | P0 if N01 yes | **OPEN** | Permissions to void: admin only, treating clinician, creator/receptionist, or limited combination? |
| N04 | P1 if N01 yes | **OPEN** | Are `void_reason`, `voided_by`, `voided_at` mandatory together? Server stamping/FK? One-way flag or undo allowed? |
| N05 | P0 if N01 yes | **OPEN** | View behavior: hide voided visits from normal `daily_ledger` vs display with explicit void status; adjust historic `'Complete'` assumption. |
| N06 | P0 | **OPEN** | When clinical entry already billed (Due or Paid), how does void affect invoice presentation/settlement without changing historical line? |
| N07 | P1 | **OPEN** | `complaint_courses`: can void? Should descendants become effectively void? Cross-relations/history? |
| N08 | P1 | **OPEN** | `clinical_notes`: permitted deletion/void UI; correctness/audit semantics; `is_critical` not activity flag. |
| N09 | P1 | **OPEN** | `patient_vitals`: physically drop table vs leave unused? No MVP frontend; scope DEFERRED. |
| N10 | P1 | **REMOVE-INTENT / MIGRATION OPEN** | Remove `patient_alerts` from target MVP: audit dependencies, migration/data retention, frontend uses. |
| N11 | P1 | **INVESTIGATE** | What are `timeline_events`, which screens/functions write/read them, and are they redundant with visits/notes? |
| N12 | P0 | **PROPOSED-BY-USER** | Optional append-only *new* invoice lines for undercharges: explicitly reconcile with settled freeze-at-issuance. Who/when/which statuses? |
| N13 | P0 | **PROPOSED-BY-USER** | Overcharge → admin waiver + explanation; when is this a suitable remedy vs billing correction/credit? |
| N14 | P0 | **OPEN** | Waiver storage: adjustment record or column, `has_waiver`/flag, amount/reason/actor/timestamp, partial/full, bad debt vs goodwill vs error. |
| N15 | P1 | **OPEN** | Which exact payment row fields and validators, whether payment rows editable while Due, and how Paid lock enforced across child tables? |
| N16 | P0 | **OPEN** | New-checkout receptionist partial UI including ₹0→Due edge case, method requirement and split tender. |
| N17 | P1 | **OPEN** | Admin pending-resolution path: zero receipt and full waiver; waive some and leave Due? |
| N18 | P1 | **OPEN** | Actual received date vs recorded date; late admin entry/backdate, timezone/cash EOD behavior. |
| N19 | P0 | **OPEN TECHNICAL** | Payment/checkout idempotency keys, concurrency locks, preventing duplicate invoice/payment rows and over-collection. |
| N20 | P1 | **OPEN** | Compute monetary totals via queries/views; integrity safeguards and performance; settlement status transitions despite derived amounts. |
| N21 | P1 | **OPEN** | Exact medical/financial fields and voiding in possible `standalone_treatments` table; table itself is not finalized. |
| N22 | P1 | **OPEN** | Legacy migration reconciliation: old invoice header amount/method/status and clinical charge columns vs newer immutable lines + payments design. |
| N23 | P1 | **OPEN** | Original source `visit_services` lacks `created_by`/`created_at`; if traceability matters, which fields/derivation? |
| N24 | P2 | **DEFERRED** | General Paid invoice corrections/refunds/voiding/credit-note workflows excluded from MVP unless new decision. |
| N25 | P2 | **DEFERRED** | Automated alerts for old debts and one physical receipt allocated across invoices not part of this MVP. |
| N26 | P1 | **OPEN** | `packages`: invalid creation vs cancellation/expiration/activation/refund; generic `is_voided` not automatically chosen. |
| N27 | P1 | **OPEN** | If clinical record is voided, should finance keep the original service-line source FK and display error annotation? Printer/reports semantics. |

**Newly closed in this conversation (do not reopen without scenario):** old-Due repeated admin partial receipts; central positive-receipt rows; payment-write/status atomicity; derived monetary aggregates/no duplicate invoice header balances. **Preserved prior:** receptionist limited to new checkout, admin older Due/Pending/waivers, Paid locked, receipts ≠ waivers, receipts dated by actual receipt, normal checkout atomic per newer 23 Oct log, issued lines frozen per newer 23 Oct log.

### Suggested continuation order (not an implementation plan)

First choose *whether* voiding is in V1 at all (N01). If yes, settle the parent-child validity rule (N02), role permission (N03), and the effect on issued invoices (N06) before defining SQL. Independently continue waiver representation (N14) and settle the user-origin invoice append proposal (N12), both of which intersect with issued-line immutability. Preserve the rest as open; do not reopen the deliberately parked treatment/visit/invoice cardinality by default simply because this addendum mentions it.

---

## 8. Source list / exact evidence boundaries

1. **User-provided parent checkout discussion:** `clinic_checkout_billing_decision_log_2026-10-18.md`, v2.0 (18 Oct 2026). It defined original locked flows and old OPEN-01…OPEN-17; parent content unchanged. The initial conversation resumed its old OPEN-01.
2. **Read-only GitHub canonical Module 2 log (23 Oct):** [clinic_billing_decision_log_2026-10-23.md](https://github.com/omsrivastava512/clinic-demo-be/blob/main/study/module_2/decisions/clinic_billing_decision_log_2026-10-23.md). Used for freeze-at-issue, payment layers, clinical-vs-financial mistake distinction, atomic normal checkout, current proposed source relationships. Some rows are older relative to newer successor addenda.
3. **Read-only GitHub successor addendum (23 Oct, 17:26 IST plus later update):** [clinic_module2_successor_addendum_2026-10-23_1726_IST.md](https://github.com/omsrivastava512/clinic-demo-be/blob/main/study/module_2/decisions/addenda/clinic_module2_successor_addendum_2026-10-23_1726_IST.md). Specifically §6 compares append-to-issued with prepare-before-issuance; §11.4 settles original total derived from frozen invoice lines; §11 records ordinary visit-line/standalone treatment design directions. These decisions are not recreated or silently altered here.
4. **Read-only GitHub historical Markdown migration:** [docs/schema/supabase_migration.md](https://github.com/omsrivastava512/clinic-demo-be/blob/main/docs/schema/supabase_migration.md). Audited all 16 `CREATE TABLE` definitions, clinical_notes fields, visit_services missing creation stamps, `daily_ledger` view, historical invoice columns/status and permissive v1 RLS policies. **Not deployed-state verification**.
5. **This current conversation:** primary evidence of new user approvals, user-attributed proposals, and deferrals in this addendum. Explicit user statements override suggestions and old OPEN statuses on only those subjects.

**Search procedure note:** Prior GitHub search-index requests for `overcharge waiver clinical mistake`, `immutable invoice lines append correction`, `unperformed service clinical records`, `waiver note overbilling` returned no results. Narrow direct file reads were used instead. This is not evidence that a concept didn't exist elsewhere; only identified passages were verified. The later migration audit examined table definitions (16), relevant trigger/RLS sections, and the `daily_ledger` query. No source-code test, SQL execution, RLS enforcement test, frontend verification or live Supabase inspection took place.

---

## 9. Handoff invariant summary

- Status changes newly established in this chat belong to this later addendum. The parent documents stay intact.
- Clearly mark **who suggested** each potential mechanism: append-only issued invoice lines and overcharge-waiver note were **user-origin proposals**; view-filter, one-way toggle, effective-child-status, and per-table scanning were **assistant-origin recommendations** in response.
- **Chosen conditional field names do not mean the voiding module is included in MVP.** Required core if built: `is_voided`, `void_reason`, `voided_by`, `voided_at`—user selection, exact constraints open.
- **Voiding a parent visit must effectively void its performed children** if parent voiding is eventually built; trigger or derived state is to be decided. The `daily_ledger` query must likewise reflect the chosen definition of effective validity, not list voided treatment as performed.
- Historical invoice lines remain frozen by prior decision unless the **explicitly open** user proposal to append *new* lines after issue is separately approved and designed. Never silently merge the positions.
- Removing `patient_alerts` is a user-intended future change **not yet performed**; `patient_vitals` deferred with drop-versus-dormant unresolved; `timeline_events` investigation open.
- **No artifacts other than this addendum and its Excel companion are modified or deployed.**
