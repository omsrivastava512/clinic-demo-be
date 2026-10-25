# Clinic Demo Backend — Module 2
## Addendum Audit Appendix 03 — Conversation Completeness, Status Corrections & Unresolved Dependencies

**Created at:** **25 October 2026, 18:48:49 IST (UTC+05:30; `Asia/Kolkata`)**  
**Chronological identifier:** `20261025-184849-IST`  
**Document type:** **THIRD, INDEPENDENT GAP / ERRATA APPENDIX** — not a replacement for either previous addendum  
**Scope of audit:** The complete **25 October 2026 conversation available in this thread**, cross-checked against the two consecutive Markdown addendums below and the previously supplied 18 October checkout/payment baseline where relevant.  
**Purpose:** Preserve any omitted nuance, incorrectly carried-forward decision status, contradiction, missing edge case, or context needed when the three files are read later as the conversation's authoritative handoff.  
**Implementation status:** Documentation review only; **no GitHub writes, migrations, Supabase changes, SQL execution, product work, or modifications of the two predecessors**.

### Files in chronological order and shorthand

- **A1:** [`clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md`](clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md), created **16:46:48 IST**, 289 lines.
- **A2:** [`clinic_module2_successor_addendum_2026-10-25_171327_IST.md`](clinic_module2_successor_addendum_2026-10-25_171327_IST.md), created **17:13:27 IST**, 148 lines.
- **A3:** **this** gap and context appendix, created **18:48:49 IST**.
- **Historical baseline:** `clinic_checkout_billing_decision_log_2026-10-18.md`, version 2.0. Relevant more recent project documents: GitHub `study/module_2/decisions/clinic_billing_decision_log_2026-10-23.md` and `study/module_2/decisions/addenda/clinic_module2_successor_addendum_2026-10-23_1726_IST.md`. Those are separate discussion tracks and were read, **not edited**, in the conversation.
- **Historical migration inspected (read-only):** [`docs/schema/supabase_migration.md`](https://github.com/omsrivastava512/clinic-demo-be/blob/main/docs/schema/supabase_migration.md). Its schema does not prove what is currently deployed.

### How these files form the source of truth

**The three timestamped addendums, read in order, are the final conversation-derived source of truth for the subjects actually discussed on 25 October.** Keep all three: A1 carries the detailed initial session, A2 carries the later decisions, and A3 corrects provenance/status and records residual omissions. They are **additive**, not three alternative specifications. For any overlapping rule: **later explicit user approval supersedes an earlier conflicting status only on that specific rule**; a newer assistant suggestion is *not* a user decision; a proposed option remains proposed until adopted. A3 must **not** be used to manufacture authorization for unapproved details. Older canonical project/checkout records remain authoritative for subjects **not addressed** on 25 October.

**Status language:** `LOCKED / AGREED` = the user affirmatively chose the behavioral rule; `CONDITIONAL AGREEMENT` = chosen *only if* the optional module exists; `PROPOSED BY USER` = user floated idea without final agreement; `RECOMMENDED BY ASSISTANT` = assistant-only option; `OPEN` = not resolved; `DEFERRED / OPTIONAL` = not required for this MVP; `SOURCE EVIDENCE` = fact found in old docs, not a fresh user decision or live implementation proof. The distinction between behavioral approval and concrete SQL/DDL approval is mandatory.

---

## 1. Cross-check result

**Outcome: A1 and A2 captured most of today's conversation, including minor details, but were NOT perfectly self-reconciling.** This third appendix is warranted. It records **status/provenance fixes**, **one substantive conflict against the October 23 source model**, **a missed workflow contradiction**, and **technical conditions not yet fully settled**. It does **not** revise either original.

### Traceability of coverage already present — do not reopen these just because they are audited

| Today's topic | Location already preserved | Current treatment after both addendums |
|---|---|---|
| Admin's additional partial receipt on older Due; keep remainder Due | A1 §1 entries 2–3; §2.1 | **LOCKED**; repeated payments allowed with no artificial count limit; older invoice admin-controlled |
| Central `payments` records, one strictly positive actual receipt per row | A1 §1 entry 4; §2.2 | **LOCKED concept**; no zero-rupee/waiver payment rows; DDL open |
| Payment insertion + corresponding state transition transaction | A1 §1 entry 5; §2.2 | **LOCKED atomicity**; retry/concurrency design still open |
| Derive billed, collected, waived and due without duplicate header monetary authorities | A1 §1 entries 6–8; §2.3 | **LOCKED direction**; exact waiver representation open |
| `Pending / Due / Paid` mean workflow states, not just arithmetic | A1 §1 entry 8; §2.3 and parent log | **Preserved prior business meaning**; SQL column details open |
| Paid fully locked; no payment or line edits on Paid | A1 §§2–3; A2 §§3, 6 | **LOCKED** |
| User-origin append-only *proposal*, before later approval | A1 §1 entry 10; §3.1 | **Historical PROPOSED BY USER** — later partly resolved by A2; see G01/G02 |
| Overcharge-waiver-note idea and clinical-versus-financial truth | A1 §1 entries 11–12; §3.2; A2 §4 | **Preserved direction/limited example**; all error types not ratified |
| `is_voided` name comparison (`is_invalid`, `is_retracted`, `is_valid`, `is_entered_in_error`) | A1 §1 entries 13–16; §4 | **Recorded faithfully** |
| Four conditional fields: `is_voided`, `void_reason`, `voided_by`, `voided_at` | A1 §4; A2 §1 | **Chosen only if voiding built**; nullability/checks/FK unresolved |
| Avoid pre-adding fields when voiding isn't implemented | A2 §1 | **AGREED**, no premature migration changes |
| Voiding optional for MVP, not a release blocker | A2 §1 | **AGREED** (resolves A1's former MVP-scope OPEN) |
| Void single `visit_service`, keep visit and sibling services valid | A2 §2 | **CONDITIONALLY AGREED** |
| Void `visit` ⇒ all its services effectively void | A1 §6.1; A2 §2 | **CONDITIONALLY AGREED**; physical trigger vs derived parent predicate open |
| Only admin can void | A2 §2 | **CONDITIONALLY AGREED**; supersedes A1's earlier permission OPEN |
| `complaint_courses`, `clinical_notes` as possible void targets | A1 §§4–5; A2 §2 | **OPEN**, not mandatory MVP work |
| Vitals feature out of MVP; no corresponding frontend | A1 §1 entry 27; §5 | **DEFERRED**; drop vs leave dormant open |
| `patient_alerts` redundant/unwanted AI-added table | A1 §1 entry 28; §5 | **User-directed REMOVE INTENT**; actual deletion/dependency checks not done |
| `timeline_events` purpose unknown, investigate | A1 §1 entry 29; §5 | **INVESTIGATE**; do not silently remove or retain |
| `clinical_notes` has `is_critical` but no active/deleted flag in old migration | A1 §1 entry 21; §5 | **SOURCE EVIDENCE**, not deployed-state claim |
| `visit_services` creation metadata absent in old migration | A1 §1 entry 22; §§4–5 | **SOURCE EVIDENCE / future traceability review** |
| `daily_ledger` hardcodes `Complete`, no `is_voided` predicate | A1 §1 entry 30; §6.2; A2 §6 | **Conditional future reporting adjustment** |
| Admin appends missing ₹150 to Due ₹500 invoice after ₹200 received | A2 §3, §7 | **AGREED**; resulting gross ₹650, collected ₹200, Due ₹450 |
| A new line is visible independently and original rows unchanged | A2 §3 | **AGREED**; new rows only, no mutate/delete existing line |
| New line cannot be appended to Paid invoice | A2 §3 | **LOCKED** |
| One visit at most one invoice; invoice may cover multiple visits | A2 §3, §7 | **Reaffirmed**; exact source/event uniqueness and retry implementation separate |
| Admin-only changes to old invoice, one issue at a time, no repeated decided questions | A1 §2.1; A2 §§3, 7 | **Behavior + collaboration preference preserved** |
| User's join/filter only non-voided billable clinical sources | A2 §5 | **PROPOSED BY USER**, not approved complete financial aggregation contract |
| No voiding clinical record tied to Paid invoice | A2 §6 | **CONDITIONAL AGREEMENT**; guard requires considering the whole affected clinical/invoice graph |
| Prevent void if recalculated total would be less than money already received | A2 §6 | **CONDITIONAL AGREEMENT**, not yet complete balance invariant |

---

## 2. Gap and correction register (references to earlier documents)

### G01 — **Material status/provenance correction: admin appends are newly approved, not merely “previously settled”**

**Affected:** A1 §3.1 (lines 99–104); A2 §3 (lines 44–50), specifically the `PREVIOUSLY SETTLED and reconfirmed` tag on A2 line 46.  
**Evidence from conversation:** After A1 was created, user explicitly approved **admin appending an omitted, actually performed treatment as a new invoice line to a Pending invoice**, then explicitly approved the same behavior for a **Due invoice with existing partial payment**, and explicitly rejected any append when Paid. User confirmed separate line display and source association.  
**Correct current status:** **LOCKED / AGREED ON 25 OCT AFTER A1**, with implementation details OPEN. **Origin:** user (first suggested append-only option; later ratified the pending/due behavior), not the assistant.  
**Do not misread:** A1's “PROPOSED BY USER; NOT RATIFIED” is an accurate **earlier snapshot**; A2's `PREVIOUSLY SETTLED` wording improperly hides the status transition. It is now **superseded on the new-line-append permission**, not a still-open yes/no.

### G02 — **State the exact exception to freeze-at-issuance, without erasing original-line immutability**

**Affected:** A1 §3.1 (lines 99–104); A2 §3 (lines 44–48); the October 23 canonical/addendum's prior freeze-at-issue policy.  
**Correct distinction:** **Existing issued invoice lines remain immutable (no UPDATE/DELETE). A newly created additional line may be INSERTed by admin only while invoice is Pending or Due. Once Paid, no further INSERT, UPDATE, DELETE, waiver change or payment mutation permitted under the MVP lock.** The exception is about **new rows** on unsettled invoices, not altering historic line amounts/descriptions or general unrestricted invoice editing.  
**Status:** **AGREED behavioral amendment on 25 Oct**, with database enforcement, permissions, amount validation, and audit/source integrity **OPEN**. “Frozen at issuance” now describes **already-issued lines**, not a prohibition against *all future line INSERTs* into an unpaid invoice. Record this as a **scoped override** of the prior broader reading, not as a retroactive claim that earlier 23 Oct documents already allowed inserts.

### G03 — **Substantive source-model conflict: a normal visit's invoice line was previously sourced from `visits`, not `visit_services`**

**Affected:** A2 §3 lines 46–48 and §5 lines 64–80; October 23 GitHub successor addendum §§11.2, 11.4, 11.5.  
**Earlier explicit domain settlement:** Under ordinary flat therapy, a **complaint-visit** produces one therapy invoice line sourced from that **`visit`**; individual routine `visit_services` are clinical occurrences *without independent invoice charges*; separately billed standalone procedures were to have independent treatment-occurrence sources, precise table design still OPEN. Package-covered visits retain their own ₹0 visit lines.  
**Later user statement (record as said):** The ₹150 omitted treatment should be an appended, separately visible line **referencing its own `visit_service`**. The later user also reasoned about a generic invoice-lines → visit-services JOIN for void filtering.  
**What this means:** This is a **real design contradiction / scope ambiguity**, not grounds to silently change the Oct 23 source strategy, nor to ignore the user's new statement. Was that ₹150 a separately billed independent treatment, a routine included modality erroneously treated as individually chargeable, or another billable type? The example did not settle that classification. `visit_services` can be valid clinical evidence without automatically being a financial line source.  
**Status:** **User's specific reference wording recorded; universal FK/source model OPEN for reconciliation.** No new table/FK chosen. Do NOT implement a universal “one invoice line per `visit_service`” rule from this example. Retain real foreign-key/source ownership and no-double-charge invariants.

### G04 — **The user explicitly clarified “new checkout” versus reusing an old visit**

**Affected:** A2 §3 line 50 and §7 item 12 (line 111), which collapse the exchange into a general reaffirmation.  
**Nuance worth preserving:** User strongly questioned the assistant's hypothetical of opening a new checkout and billing the **same old visit/service again**: in the established ordinary workflow, a **new checkout means a new visit and a new invoice**, not an operator selecting old billed occurrences for re-invoicing. The assistant rephrased the genuine failure mode as **double-submitting the same checkout** (double click, retry, lost response).  
**Status:** **WORKFLOW CLARIFICATION / EXISTING DIRECTION REAFFIRMED.** Do not invent a “reopen previous visit and rebill it in the next checkout” feature. However, new checkout meaning does **not** solve transaction retries, duplicate source links, malicious/direct API calls or concurrent writes automatically.

### G05 — **Duplicate-submission safeguards were discussed but NOT designed or implemented**

**Affected:** A2 §3 line 50's phrase “atomic checkout and duplicate-submission safeguards were previously discussed,” and A1 §7 `N19` (idempotency open).  
**Correction:** Normal checkout atomicity is supported by the 23 Oct project decision, and payment-write/state atomicity was ratified in this conversation. **Neither means idempotency keys, unique source constraints, or lost-response retry protection have been decided or installed.** User did not approve a specific DB mechanism for exactly-once behavior.  
**Status:** **Technical control remains OPEN, mandatory implementation risk**; don't ask a redundant abstract business yes/no when the underlying requirement is already clear. Resolve during concrete backend design using source-link uniqueness and idempotent checkout/payment operations.

### G06 — **Double financial reduction: void-filtering and an overcharge waiver could subtract the same charge twice**

**Affected:** A2 §4 lines 54–60 and §5 lines 62–82, §6 lines 84–94; A1 §2.3 derivation.  
**Scenario:** Original issued lines total ₹500, with ₹200 actually collected. One ₹300 erroneous service is subsequently clinically voided. **Proposed filtered eligible amount becomes ₹200.** If admin ALSO records an independent ₹300 waiver against the same mistaken service, then naïvely calculating `eligible_amount − collected − waiver` gives `₹200 − ₹200 − ₹300 = −₹300` (double correction). If instead maintaining historical original billed ₹500 for settlement, an approved ₹300 waiver gives `₹500 − ₹200 − ₹300 = ₹0` while the original charge remains visible. These are **alternative ledgers/definitions**, not combined operations by default.  
**Status:** **OPEN policy and data-model conflict discovered by audit**. Define whether void-based exclusion itself changes collectibility, whether waiver is still recorded and how it is categorized, and prevent duplicate financial remedy. **No outcome chosen here.**

### G07 — **Ordering conflict: waiving the error and marking Paid may permanently block correcting the false clinical record**

**Affected:** A2 §4, §6 (especially lines 56 and 86); A1 §3.2.  
**Scenario:** ₹500 invoice; ₹200 received; ₹300 charge was from a service wrongly recorded as performed. The already-accepted admin final-settlement flow can waive ₹300 and mark invoice **Paid**. But the later conditional rule says **do not void any clinical visit/service linked to a Paid invoice**. If the admin waives first and finalizes, clinical correction may now be forbidden even though the clinical record is false. If the admin voids first, the user-origin dynamic total filter could already reduce payable to ₹200, so adding an independent ₹300 waiver may produce G06.  
**Status:** **OPEN sequencing, clinical accuracy and financial-integrity conflict.** Do not claim that “use a waiver plus clinical void” is a finished workflow. Need future policy on order/atomic orchestration, and possibly whether an *auditable clinical error annotation* should remain possible after financial Paid lock. That latter exception is **a newly raised option, NOT approved**, and currently conflicts with the user-stated paid-link void prohibition.

### G08 — **The agreed “not below money collected” floor is necessary, but may be insufficient if waivers already exist**

**Affected:** A2 §6 lines 86–94; A1 §2.3 derived-balance formula.  
**What user explicitly approved:** **Reject conditional clinical void when the recomputed eligible total is below the sum of real collected payments.** For example, gross ₹500, paid ₹200, void ₹400 ⇒ eligible ₹100, reject. This is a real conditional decision and must remain so.  
**Further risk (not user-approved constraint yet):** If the invoice also has existing approved waiver/write-off adjustments, a total that is **at least collected** may still be **less than collected + existing effective waivers**, causing a negative outstanding amount under the original arithmetic. Whether existing waiver amounts can coexist with a Due invoice is itself OPEN.  
**Status:** **LOCKED conditional payment floor; OPEN additional adjustment-aware invariant.** Do not silently strengthen the user's approved rule into `eligible >= payments + waivers` without deciding the interaction with G06 and final-settlement-only waivers.

### G09 — **Paid-link protection must follow indirect references and entire parent-child membership**

**Affected:** A2 §5 lines 64–80; §6 line 86; October 23 rule that flat-therapy line refers to `visits`.  
**Nuance:** A routine `visit_service` may be clinically included in a **`visit`** whose flat invoice line points at `visits.id`; that service itself may have **no** direct `invoice_lines.visit_service_id`. Thus “block if *this service row* is directly referenced by a Paid invoice line” would miss an indirectly billed or included service. An entire visit may also coexist with other complaint-visits or standalone charges in one invoice.  
**Conditional desired guard:** For service or parent visit void, identify **all affected invoice relationships** (including via visit membership), not just direct service FK, and apply the user-accepted Paid-linked prohibition. The exact relationship schema remains to be reconciled (G03).  
**Status:** **Technical implication of conditional agreement; FK graph and enforcement OPEN.**

### G10 — **Historical billed amount versus dynamically eligible/payable amount requires two named meanings**

**Affected:** A1 §2.3 lines 80–93; A2 §5 lines 62–82 and §6.  
**Source-derived prior agreement:** `SUM(invoice_lines)` is the **original billed** total; frozen line amounts retain historical charge provenance. **Later user proposal:** a join-filter excludes voided clinical sources **when computing the amount that should count for billing** on unsettled invoices.  
**Not the same:** A filtered amount is **not automatically** the historically issued original gross. Dynamic filtering after clinical void may recalculate what is *currently collectible* without erasing what the original invoice displayed at issuance. UI, totals, invoice number/receipt, waiver/write-off representation and audit reporting need separate semantics.  
**Status:** **Historical-source preservation established; user-origin dynamic filter is a PROPOSAL with OPEN scope as authoritative payable.** Don't treat the illustrative SQL in A2 as signed-off production query. Existing invoices may include visit-level flats, package-covered ₹0 items and other billable sources (G03).

### G11 — **What invoice status follows an allowed void or append remains open**

**Affected:** A2 §§3, 5–6; A1 §2.3.  
**Examples:** (a) A Due invoice with ₹200 collected and eligible total reduced to **exactly ₹200** after a permitted clinical void now has computed outstanding ₹0, but nothing explicitly decides whether it becomes `Paid` automatically, requires admin action, or is kept unresolved for confirmation. (b) A Pending invoice may receive a new line and remain Pending. (c) A Due invoice might have an existing approved waiver; amending billable amount changes the reconciliation.  
**Status:** **OPEN status-transition contract**. Do not invent automatic `Paid` transitions or modify `paid_at` until business rule chosen. New-line append raises amount but is not itself a payment. The known rule that final Paid locks the invoice remains intact.

### G12 — **A service/visit clinical void and an invoice-line source must be verified as a genuine correction, not generic repricing**

**Affected:** A1 §3.2; A2 §3, §5.  
**Differentiation:** Wrongly recorded *unperformed* procedure, correctly performed procedure at wrong rate, omitted performed procedure, duplicate charge from retry, and package-covered visit all have different truth conditions. A clinical void should not invalidate a **genuinely performed** treatment merely because the price is wrong. Adding a line for an omitted service should require a genuine performed occurrence/source and must not rebill the same source.  
**Status:** **ESTABLISHED domain distinction + OPEN implementation checks.** User's waiver-with-note policy is discussed, not a universal correction instrument; do not assume discounts, bad debt, charge mistakes and clinical falsehood all share the same classification.

### G13 — **Scope of the conditional four void fields and corresponding metadata checks was chosen, not every constraint**

**Affected:** A1 §4 lines 114–140; A2 §1 lines 19–32, §2 lines 34–42.  
**Accurate combined reading:** User likes `is_voided`, `void_reason`, `voided_by`, `voided_at`; the voiding module is **OPTIONAL FOR MVP**, with **no unused columns** added ahead of the feature. When built, single-child void, parent-effective cascade and admin-only authority have been **conditionally accepted**. BUT nullability combinations, optional/required reason, auto timestamp/auth actor stamps, undo ability, trigger versus derived effective status, exact permitted tables, client RLS policies, and how these interact with payments remain **OPEN**.  
**Prior A1 statuses:** A1 said scope and void permission were OPEN when created; **A2 supersedes those points** (scope now optional; permission now admin-only if built). **Status reconciliation**, not a defect in A1's chronological snapshot.

### G14 — **Source-of-truth ordering and the Excel tracker are easy to misread**

**Affected:** A1 opening “delta/addendum” disclaimer lines 10–14, A2 opening lines 3–17, and the Excel companion created with A1.  
**Correct reading:** The two earlier Markdown addendums were created at 16:46:48 and 17:13:27 IST. This appendix comes after both. The Excel companion generated with A1 is a **structured snapshot of A1**, not an automatically synchronized live tracker with A2/A3 changes. Therefore an A1 Excel row that still says “append-only proposed,” “void scope open” or “void permissions open” is **historically correct for its timestamp but stale for current status**. Preserve Excel unchanged; read A2 and A3 for subsequent changes.  
**Status:** **Documentation-precedence clarification; no file change requested.**

### G15 — **Financial correction after Paid and erroneous clinical facts remain an explicit, accepted MVP limitation**

**Affected:** A1 §3.2, §7 N24; A2 §4 line 60, §6 lines 84–96.  
**Important caveat:** User locked Paid invoice immutability. Later, user **conditionally** prohibited clinical void of anything linked to Paid. This can mean a clinical entry recognized as inaccurate only after collection may not be voidable under the proposed feature, and overpayment/refund, legal correction or patient safety requirements could demand future remediation. **Not solved by “admin is the human in the loop.”**  
**Status:** **OPEN risk acknowledgment, not permission to weaken the Paid lock or a covert MVP feature request.** Document the accepted current restriction and revisit only with clear need/requirements.

---

## 3. Newly consolidated status ledger — exact current meanings (no migrations authorized)

| Area | Current status | Ownership / important caveat |
|---|---|---|
| Admin repeated receipts on existing Due | **LOCKED** | User; keep Due until settled; any count, positive payment rows |
| Central `payments` one row per actual receipt | **LOCKED CONCEPT** | User; exact DDL open, no zero receipts/waivers |
| Atomic payment+required settlement transition | **LOCKED** | User; retry/idempotency still open |
| Monetary header fields computed from children, not duplicate authorities | **LOCKED DIRECTION** | User; amount display views/performance/schema detail open |
| Paid fully immutable | **LOCKED** | User; financial correction workflows deferred |
| Existing issued invoice-line UPDATE/DELETE | **PROHIBITED** | User/earlier source; append exception does not allow editing |
| Admin INSERT new line on Pending/Due | **LOCKED NOW** | User, approved after A1; extra line visible; no Paid append |
| Appended new line is separately displayed | **LOCKED NOW** | User |
| Appended specific ₹150 example references `visit_service` | **USER WORDING RECORDED; GENERAL SOURCE MODEL OPEN** | Reconcile with prior visit-based flat lines / standalone occurrence |
| One visit belongs to at most one invoice | **RECONFIRMED** | User; not a new generic checkout-from-old-visit UI |
| Lost-response retry protection and unique source billing SQL | **OPEN TECHNICAL** | Mandatory integrity, not solved by atomicity |
| Overcharge treated via admin waiver/note | **USER DIRECTION; DETAILS OPEN** | One specific ₹500/₹200/₹300 example settled arithmetically; classification and lifecycle open |
| Original billed charge vs filtered eligible amount | **TWO DISTINCT CONCEPTS; AGGREGATION OPEN** | A2 SQL is illustration, not approved universal query |
| Clinical voiding included in MVP? | **OPTIONAL** | User; not blocking release; not necessarily deferred to a fixed later release |
| Pre-add unused void columns without feature | **NO** | User; if built, add when needed |
| Four void fields | **CONDITIONALLY CHOSEN** | User; `is_voided`, `void_reason`, `voided_by`, `voided_at` |
| Independently void one service | **CONDITIONAL AGREEMENT** | User; other services/visit remain valid |
| Void parent visit => all services effectively void | **CONDITIONAL AGREEMENT** | User; physical cascade trigger vs derive OPEN |
| Who can void? | **ADMIN ONLY (CONDITIONAL)** | User; A1 earlier permission OPEN superseded |
| Cannot void clinical item tied to Paid invoice | **CONDITIONAL AGREEMENT** | User; indirect source FK graph must be checked |
| Cannot void if recomputed eligible amount < collected receipts | **CONDITIONAL AGREEMENT** | User; waivers/zero balance/other sources OPEN |
| Direct `invoice_lines.is_voided` or auto line updates | **NOT CHOSEN** | User prefers join/filter proposal; not final SQL |
| Void-filter financial aggregate on unpaid invoice | **PROPOSED BY USER; SCOPE OPEN** | Original billed vs eligible collectibility needs definition |
| Daily performed-visit ledger excludes/annotates voided visits | **CONDITIONAL DEPENDENCY; DISPLAY MODE OPEN** | No old SQL view modification done |
| `patient_vitals` in MVP | **DEFERRED** | User; drop table vs leave dormant unresolved |
| `patient_alerts` table | **REMOVE INTENT** | User; not physically removed; dependencies/data first |
| `timeline_events` purpose | **INVESTIGATE** | User; no removal decision |
| `complaint_courses` and `clinical_notes` voiding | **OPEN OPTIONAL** | User considered, no feature approval |
| `packages` lifecycle and voiding | **OPEN / SEPARATE** | No generic flag imposed |
| Mandatory next topics | **WAIVERS, PENDING/DUE FORM, IDEMPOTENCY, SOURCE LINKS/DB GUARDS** | Existing questions, don't reconfirm closed business points or spend disproportionate time on optional voiding |

---

## 4. Explicit technical and workflow carry-forwards, NOT decisions

1. **Audit/payment ordering:** Define whether an admin can void a wrong clinical entry *before* settling an issued invoice, when waiver and void exclusion are alternative remediations, and how to avoid later Paid lock trapping a false entry (G06/G07).
2. **Correct source mapping:** Specify whether the appended ₹150 charge belongs to a visit-level flat bill, to an independent performed-treatment occurrence, or to a special exception that truly allows a `visit_service` source. Preserve user's example while auditing the October 23 source model (G03).
3. **Recomputed-total consistency:** Preserve historical original billing and cash. Define dynamic eligible total, waiver offsets, reporting, outstanding floor, and `Paid` status transition; do not implement the example join in A2 as a universal query (G06/G08/G10/G11).
4. **Void permission and concurrency:** When optional feature built, enforce admin-only ability server-side; guard same transaction against concurrent collection/waivers and read indirect invoice links (G09/G13).
5. **Parent/child void propagation:** If parent void happens, every service is invalid for clinical purposes. Trigger vs derived expression remains open; forbid newly inserted “valid” child under voided parent unless a future explicit corrective policy permits (A1 §6.1; A2 §2).
6. **Views/reports:** `daily_ledger` historically hardcodes `Complete`, not a financial Paid status; a valid clinical-visit report must filter/annotate voided visits if feature added; cash report remains based on actual positive receipts and their `received_at` time. No hidden mass changes to historical reports.
7. **Schema cleanup:** `patient_alerts` is a user-directed removal intent with migration safety checks; `patient_vitals` deferred UI and table disposition open; `timeline_events` purpose unexplained. Nothing dropped.
8. **Safety boundary:** Existing `ON DELETE CASCADE` FKs in old Markdown migration are physical deletion, **not** the conditional parent-visit soft-void semantics. Historical document and current Supabase deployment can differ; verify before changing any schema.
9. **Release priority:** The voiding feature remains optional and does NOT block the mandatory invoice/payment MVP. Any void-trigger and filter designs are conditional backlog items, even though we explored them thoroughly after the second session.
10. **Conversation method:** When resuming, bring up **one genuinely OPEN mandatory issue at a time**, not a previously approved business rule in another form; use step-by-step practical examples only where necessary. Do not silently write to GitHub, existing addendums, or migration scripts.

---

## 5. Provenance and immutable-predecessor verification

The two predecessor Markdown files were reviewed in their entirety during this audit against the user-visible conversation context. Their content was not edited. File SHA-256 hashes recorded before A3 creation:

```text
A1  eee8309d367a5958d9cdc7b7c6a9f9395a6fd4942a74190ab1c7f75e393977a7
A2  98c1b60cfa861d69428847739d1f53abb3c6505d607628dd6356d9e3e4286d34
```

The A1 Excel companion also remained untouched (the spreadsheet is a snapshot, not a continuously updated source of truth).

**Audit limitation:** Cross-check was against today's conversation visible in this session and the two provided addendums plus the prior supplied payment handoff. This was a **documentation completeness/consistency audit**, not a fresh crawl of every historical chat or independent verification of the currently deployed database. Where a contradiction depends on the older Oct 23 source design, it is identified as a **reconciliation question**, not resolved by assumption.

**Final instruction for future handoffs:** Read **A1 → A2 → A3** in timestamp order. For overlapping statements, take the most recent **explicit user-approved** business rule, preserve the earlier discussion as history, and leave unresolved technical mechanisms/open decisions flagged OPEN. Do not collapse a proposed design into a settled one simply because it was described in SQL.
