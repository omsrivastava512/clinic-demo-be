# Module 2 reconciliation answers and corrections — 27 October 2026

Source: the user's follow-up messages in the consolidation chat after review of Q01–Q07. This is a new discussion record, not part of the archived 176-message JSON export. Existing source files are preserved unchanged. U01 is the earlier zero-invoice answer; U02–U04 are the latest clarification. Statements below distinguish the user's direction from the assistant's evidence review.

## U01 — Covered-only zero-total invoice (Q01 resolved)

The user explicitly allows an invoice to total zero when its only line, or all its lines, are covered by a package. The frontend should offer one action to mark it paid; Pay Later and other collection choices are blocked for this case.

Current interpretation: retain at least one real invoice line and validate actual package coverage on the backend. After issuance, Mark as paid closes the covered-only invoice as PAID using the existing settlement actor/time; create no payment row and count zero collections. This is not a waiver. If an uncovered therapy or applicable automatic fee makes the total positive, the ordinary collection/settlement rules apply. This is a scoped exception to S14's blanket prohibition on zero-total invoices, not approval of arbitrary free invoices. Coverage/redemption implementation remains in the package audit.

## U02 — Status terminology and provenance (Q02 corrected)

The user challenged who introduced ISSUED/PARTIALLY_PAID and identified their original terminology as pending/due.

Evidence review: in S16 M105 the assistant introduced ISSUED, PARTIALLY_PAID, PAID and WAIVED inside a question about storing status explicitly versus deriving it. M106 is the user's Yes to that compound question; M107 is the assistant's declaration that it was locked. This is evidence of agreement to explicit stored status, but is insufficient to attribute a deliberate terminology change to the user. The prior reconciliation overstated it as a separately accepted rename.

Current reading: retain PENDING (issued and no receipt), DUE (partial positive receipt and balance remains), PAID and WAIVED. These are four states, not six. PAID includes the explicitly permitted package-covered zero-total settlement from U01. Historical mentions of ISSUED/PARTIALLY_PAID remain in their original records; no enum migration or code change has occurred.

## U03 — Clinical protocol versus automatic charging (Q03 clarified)

The user explained that a doctor assesses a returning patient as clinical protocol. The app must automatically apply billing rules: reception completes the clinical entry and clicks Create Invoice; the backend checks the 10/30 calendar-day bands and includes the applicable fee in the generated invoice returned to reception.

This agrees with S15 D-35 and S16 M089–M099. It does not require a new receptionist confirmation gate or a separate recorded examination as a prerequisite to computing the fee. Clinical examination documentation remains an unfinished audit; a fee event does not prove an examination occurred. First encounter ₹350; return gaps 0–9 days no fee, 10–29 ₹150, 30+ ₹350, without stacking. Q05 below is only about whether the automatic amount also appears before Create Invoice.

## U04 — Mandatory atomic clinical/billing issuance (Q04 corrected)

The user explicitly rejects clinical-save-first/bill-later for MVP and confirms that clinical records, invoice and invoice lines belong in one atomic transaction. Applicable extra-fee records belong in that same issuance transaction. All commit or all roll back. Payment/waiver settlement remains a separate transaction after the generated invoice is returned.

Mismatch found: S12 Q3, line 52 of clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md, explicitly permits earlier clinical saving and later billing. The user says that was never agreed. Treat that sentence as a corrected source-summary error, not as proof the user knowingly approved and later reversed a clinical-first workflow. Preserve the archived source unchanged and supersede its exception in the current reading pages. Possible later invoice-line work does not authorize partial persistence of MVP checkout.

## Questions explained, still awaiting the user's answers

### Q05 — When does the automatic fee first appear?

Example: therapy ₹200 plus an applicable gap fee ₹150 produces ₹350. Must reception see ₹350 before clicking Create Invoice, or can the first display be the returned issued invoice? Earlier pre-issuance review wording and later returned-invoice wording leave that display detail unclear. U03 already confirms automatic backend charging; this question does not ask whether the fee should be automatic. The user's described flow points toward the returned invoice, but they explicitly requested an explanation before answering Q05; keep it open.

### Q06 — Must the standalone clinical record separately snapshot the service name?

Catalog identity and immutable invoice-line descriptions are settled. Common authenticated recorder/server timestamp conventions should not have been reopened: S16 M006/M008 support them. The remaining specific issue is an additional service-name snapshot on the standalone clinical occurrence itself, separate from service_id and the invoice-line snapshot. If Cervical traction is renamed later, should that clinical occurrence retain the old wording independently? This is a schema-audit detail, not an established contradictory business decision. Exact physical DDL remains audited. No answer is inferred from this clarification.

### Q07 — Who may change an existing service's price or availability?

Admin-only additions are already explicit. Editable master prices and enabled/disabled availability are required, but who may perform those edits was expanded by an assistant summary without an independently clear approval. For example, may reception change a catalog price or disable a service, or only admin? This is an authorization gap, not evidence of two approved competing policies. Do not introduce new roles, deletion policies or permissions while explaining it.

## Current checkpoint

Q01–Q04 are resolved/corrected in the current reading layer. Q05–Q07 remain open; Q06–Q07 belong to their designated technical/authorization audits. A01–A13 remain unfinished as previously scoped. Only documentation and the existing tracker are updated; no application code, database schema or Git history is changed.
