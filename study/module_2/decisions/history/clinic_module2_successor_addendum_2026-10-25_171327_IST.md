# Clinic Demo Backend — Module 2 Successor Addendum

**Created:** 25 October 2026, **17:13:27 IST (UTC+05:30)**  
**Sequence:** New addendum following `clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md` (created 16:46:48 IST).  
**Scope:** All discussion *after* the preceding addendum, through the user's instruction to create this timestamped successor.  
**Provenance:** Recorded from the conversation transcript supplied in this chat; this is a conversation decision record, **not** a verified statement about the deployed database.  
**Changes to prior files / GitHub / Supabase:** None. This is an independent Markdown file.

## Status conventions

- **AGREED:** Explicitly selected or reaffirmed by the user during this discussion. These are product-level decisions, not proof of implementation.
- **CONDITIONAL AGREEMENT:** A rule agreed **if** the optional voiding feature is implemented.
- **PROPOSED BY USER:** User-originated design direction that has not been completely approved, including its implementation details.
- **RECOMMENDED BY ASSISTANT:** An assistant suggestion, not a user decision.
- **OPEN:** Unresolved policy, implementation, or technical dependency.
- **DEFERRED / OPTIONAL:** Not a required MVP-release deliverable.
- **PREVIOUSLY SETTLED:** Carried over for context; not a newly authorized change.

## 1. Release priority and scope

**AGREED — user:** Clinical-record voiding is **OPTIONAL FOR MVP**. It is neither mandatory nor fully abandoned. Its implementation must not block the mandatory checkout/billing release. Avoid diverting the ongoing session into detailed optional-feature choices while mandatory workflows remain unresolved.

**AGREED — user:** Do **not** preemptively add voiding columns to the initial migration if the feature is not being implemented. Add them in a later migration when the actual feature is built. Previously discussed conditional structure is retained for future work:

```sql
is_voided   BOOLEAN NOT NULL DEFAULT FALSE
void_reason TEXT
voided_by   UUID
voided_at   TIMESTAMPTZ
```

These are **conditionally selected fields**, not an instruction to modify schema now. Constraints, foreign keys, authorization and default behavior are not fully specified.

## 2. Conditional clinical voiding behavior

**CONDITIONAL AGREEMENT — user:** A single `visit_service` can be independently voided while its parent `visit` and other services remain valid (partial visit voiding).

**CONDITIONAL AGREEMENT — user:** Voiding an entire `visit` must make *all its associated `visit_services`* effectively void. User highlighted cascading behavior and suggested a trigger; **trigger versus derived effective-validity predicate is OPEN**. Any implementation must avoid stale child validity and preserve correction history.

**CONDITIONAL AGREEMENT — user:** Only **admin** can execute voiding. This applies to the voiding workflow discussed here; do not infer unrelated edit/delete permissions.

**OPEN:** Whether `complaint_courses` or `clinical_notes` should support voiding and the detailed policy for each. The earlier addendum's deferrals and removal/investigation items remain unchanged: `patient_vitals` feature deferred, `patient_alerts` identified for proposed removal, and `timeline_events` pending purpose/usage investigation. No decision here reinstates those tables as MVP features.

## 3. Mandatory invoice-line append behavior

**PREVIOUSLY SETTLED and reconfirmed:** Issued invoice lines cannot be edited or deleted; Paid invoices are locked. The current design allows admin to append *new* invoice lines on **Pending** and **Due** invoices, not Paid invoices. Each appended charge is a **separate visible invoice line** referencing its own `visit_service`. The original lines remain unchanged.

**AGREED — user, explicit Due/partial-payment example:** Original invoice ₹500; paid ₹200; outstanding ₹300. An actual treatment worth ₹150 was performed but not billed. Admin may append a separate ₹150 line referencing its `visit_service`. Updated gross total is ₹650; paid amount stays ₹200; outstanding becomes ₹450. After marked Paid, **no new lines may be appended**.

**PREVIOUSLY SETTLED and reconfirmed:** One visit is linked to at most one invoice, while an invoice may cover multiple visits; atomic checkout and duplicate-submission safeguards were previously discussed. Do not reopen these as fresh questions without a specific unresolved technical gap.

**Communication/process feedback — user:** Work through unresolved items one at a time; do not repeatedly reconfirm already-answered rules. Prioritize *mandatory* Module 2 issues above optional voiding. Distinguish a genuinely new issue from an implementation detail of an existing decision.

## 4. Overcharge and financial correction discussion

**PREVIOUSLY DISCUSSED — user-originated direction:** Preserve invoice lines and handle erroneous overcharges through a waiver with an explanatory note. Example discussed: ₹500 invoice, ₹200 already paid, later discover a ₹300 erroneous charge. If a ₹300 effective reduction is applicable, revised collectible amount is ₹200, equal to already-paid amount; there is no refund required *in that specific example*.

**RECOMMENDED BY ASSISTANT, NOT ADOPTED:** Distinguish explicit error adjustments from discretionary waivers for cleaner financial semantics. Avoid silently substituting this proposal for the user's waiver-based approach.

**OPEN:** Handling reductions that create overpayment, refunds/credits, tax or receipt effects, and corrections after Paid status. Voiding a clinical entry is not equivalent to a cash refund.

## 5. User-originated proposed join/filter for voided services

**PROPOSED BY USER:** Because invoice lines reference `visit_services`, retain the original invoice lines and, when aggregating relevant billable amounts, filter out `visit_services` that are voided. The user suggested a join/predicate rather than physically updating invoice lines or adding `invoice_lines.is_voided`.

**Required related predicate:** Effective validity must also consider whether the **parent `visit`** is voided, not just whether `visit_services.is_voided` is true. This supports either explicit cascading child flags or effective invalidity through parent state.

**Illustrative logic only — not production SQL:**

```sql
SELECT SUM(il.amount) AS eligible_total
FROM invoice_lines AS il
JOIN visit_services AS vs ON vs.id = il.visit_service_id
JOIN visits AS v ON v.id = vs.visit_id
WHERE il.invoice_id = :invoice_id
  AND COALESCE(vs.is_voided, FALSE) = FALSE
  AND COALESCE(v.is_voided, FALSE) = FALSE;
```

**OPEN / design caution:** This illustrates only visit-service-linked lines. Other billables, packages, consultation/flat therapy fees, specialty charges, and lines without `visit_service_id` may require separate handling. Do not drop them accidentally via an inner join. Exact source links, billing mode and query design must follow the final schema. `COALESCE` is illustrative and does not replace proper `NOT NULL` defaults.

**Disagreement documented:** The assistant initially recommended retaining an immutable historical invoice total plus explicit financial adjustments instead of dynamically re-rating issued invoices. The user countered that the system should *block clinical voiding on Paid invoices* and proposed filtering only while eligible invoices remain mutable. The parties then identified the further **partially paid Due** case. Neither the broad financial model nor production implementation has been finalized by this exchange.

## 6. Financial safeguards for clinical voiding

**CONDITIONAL AGREEMENT — user:** If any affected clinical item is already referenced by a **Paid** invoice, block the attempted void of that visit or `visit_service`. Paid invoice remains immutable; do not automatically decrease its total. When voiding a parent visit, check **all affected services and associated invoices**, not just one child.

**CONDITIONAL AGREEMENT — user (explicitly accepted):** For Pending or Due invoices, do not permit a void if the **resulting recalculated invoice total would be less than the payment already collected**. This prevents automatic overpayment from the proposed filtering approach.

Example: original invoice ₹500, already received ₹200; proposed void removes ₹400 of eligible charges. Remaining ₹100 is **less than ₹200 paid**; **block** that void under the agreed safeguard. A separate refund/credit procedure has not been designed.

**OPEN:** What to do when a Pending/Due invoice has zero or partial payment and removing a charge changes its remaining balance, whether financial waiver and clinical void can coexist, how to record the change and who can authorize all related financial operations. There is an important distinction between clinical history, the currently collectible balance, and an issued invoice's original monetary snapshot.

**OPEN — technical:** Enforce checks atomically with the voiding transaction, guard against concurrent payment/void transactions, decide whether to use triggers or service-layer transactions (possibly with database constraints/locks), and determine authorization, record traceability, failure handling, and reporting semantics. Checking only `status = 'Paid'` is **not** enough because partially paid Due invoices can also be overpaid after a void.

**Previously raised, still OPEN:** `daily_ledger` and any downstream reporting views must avoid treating voided visits as valid performed treatments where clinically applicable. But financial sales/receipts reporting must not silently rewrite Paid history based on clinical void flags. Decide which reports display historical billed amounts versus currently valid clinical care.

## 7. Chronological conversation ledger (after the 16:46:48 IST predecessor)

1. User asked what remained; assistant started voiding list; user requested **one question at a time**.
2. User selected **OPTIONAL FOR MVP** for voiding, neither mandatory nor entirely deferred.
3. Asked whether to add voiding fields ahead of the feature; user rejected unused columns; decision: schema additions only if voiding built.
4. User clarified conditional discussion was about the future voiding feature, not immediate MVP scope.
5. User approved voiding a single service without voiding the whole visit.
6. User required entire-visit voiding to affect all child services.
7. User restricted voiding permission to admin.
8. User challenged continued optional-feature discussion; assistant agreed to prioritize mandatory billing decisions.
9. User approved appending omitted performed treatments to Pending invoices; new invoice line, not increasing a total directly.
10. User rejected appending to Paid invoices.
11. User approved appending ₹150 to an already partially paid **Due** invoice of ₹500 with ₹200 paid; new gross ₹650, remaining ₹450; separate line references corresponding `visit_service`.
12. Assistant repeatedly revisited previously answered duplicate billing/retry and one-visit-one-invoice questions; user objected; established rules reaffirmed, no new redesign authorized.
13. Assistant raised overcharge correction and explicit adjustments; user recalled earlier waiver-with-note discussion; example of ₹500 gross, ₹200 paid, ₹300 error examined. Assistant's separate adjustment proposal not accepted as a new locked design.
14. User raised a new dependency: if visit/service voided, what happens to referencing invoice lines? Considered cascading flag or aggregation predicate.
15. Assistant advocated preserving issued financial history; user proposed **join/filter only non-voided services and visits** instead of changing invoice lines.
16. User clarified **Paid-linked clinical records cannot be voided**, resolving the assistant's initial Paid-total-change counterexample by prohibiting that operation.
17. Assistant found a partial-payment edge case: ₹500 gross, ₹200 paid; voiding ₹400 would produce eligible amount ₹100 and an overpayment of ₹100.
18. User agreed with guardrail: **block voiding when recomputed total would fall below already received payment**.
19. User instructed that **everything discussed since the last addendum** be captured in a new timestamped addendum, without altering the predecessor. User briefly paused the creation request, then explicitly resumed and requested the new file.

## 8. Implementation acceptance cases (future conditional feature; not MVP authorization)

- **Service-only void:** Admin voids service A; service B from same visit stays valid; clinical views show only valid performed care.
- **Visit void cascade:** Admin voids visit; all its child services are effectively invalid, whether via trigger or parent validity predicate.
- **Non-admin void:** Reject.
- **Paid linkage:** Reject void of any referenced service or a visit that would affect a Paid invoice.
- **Due, safe reduction:** Original gross ₹500, paid ₹200, proposed removal ₹100 -> eligible total ₹400, outstanding ₹200; may proceed *subject to final financial model*.
- **Due, unsafe reduction:** Original gross ₹500, paid ₹200, proposed removal ₹400 -> eligible total ₹100; reject because ₹100 < ₹200 already paid.
- **Due appended undercharge:** Original ₹500, collected ₹200; append ₹150 as separate sourced line; gross ₹650, due ₹450.
- **Paid append:** Reject any attempt to append invoice lines after settlement.
- **Historical integrity:** No silent mutation/deletion of issued invoice line amount or source reference.
- **Concurrency:** Simultaneous payment and void must not bypass paid-link or paid-amount guardrails.
- **Mixed-source invoice:** Ensure the proposed `visit_services` join does not inadvertently omit other legitimate charge types.
- **Reporting:** Valid-care summaries exclude voided clinical data; financial ledgers and receipts retain correct historical monetary provenance.

## 9. Carry-forward open items (do not mistake for settled decisions)

1. Whether optional voiding will ship at all in MVP, and in which later release if not.
2. Exact void metadata constraints and correction trace; trigger versus derived cascade.
3. Table-by-table applicability beyond `visits` and `visit_services`.
4. Whether eligible-total filtering becomes the authoritative payable amount for Pending/Due invoices, and how this interacts with original invoice snapshots, waivers, and additional billable line types.
5. Payment/refund/credit workflow if correction is needed after money is received, especially Paid invoices where voiding is blocked.
6. Atomic backend guardrails for cascading void checks across all associated invoices and payments.
7. `daily_ledger` and other clinical versus financial reporting query definitions.
8. Mandatory MVP work beyond this discussion: finish any genuinely open checkout/payment workflow details, with no unnecessary repetition of locked decisions.

## 10. Final checkpoint

This document **records** the user's decisions and proposals; it **does not implement** anything. The underlying repository, earlier logs, migrations and Supabase database are unchanged. The optional-voiding design and the financial aggregation contract must not be represented as implemented or fully finalized.
