# Examination fees

Current examination/return-fee decisions as of 27 October 2026. The calendar-day fee bands and automatic charging at Create Invoice are settled. Clinical examination representation remains an implementation audit. Automatic fees first appear on the returned issued invoice; the earlier preview is clinical only.

## Settled fee policy

| Prior completed clinical history | Applicable examination/return charge |
| --- | --- |
| First encounter | ₹350 (35000 paise) |
| Return after 0–9 calendar days | No gap-based fee |
| Return after 10–29 calendar days | ₹150 (15000 paise) |
| Return after 30 or more calendar days | ₹350 (35000 paise), without an additional ₹150 |

- **Locked:** use calendar dates, not exact elapsed-hour thresholds. Day 10 and day 30 are inclusive boundaries. Clinic-local date/time-zone implementation and qualifying record selection remain audited (R16–R17).
- **Locked:** anchor on the patient's most recent qualifying completed clinical encounter, not last formal examination, last payment, or invoice settlement. Therapy-only and examination-only completed attendance both reset recency. Mapping other encounter types and “completed” to actual tables remains audit work.
- **Locked:** examination-only attendance is a real workflow without dummy therapy. Examination and therapy can occur together or on different days. A physical attendance with multiple complaint-visits must not multiply the examination/return charge (R15).
- **Locked:** opening a new complaint during ongoing care does not automatically trigger another charged formal examination. Informal clinical check-ins are distinct; a future mandatory new-complaint examination trigger is outside MVP (R14).
- “Late fee” means return-gap charge, not late payment or late arrival. No appointment/scheduling module, booking reset, or cancellation/no-show policy is introduced (R13, R17).

## Charge events and checkout

- **Locked direction:** combined extra_fees stores examination/return-gap charge events and supplies a real typed source for the invoice line. Keep standalone treatment occurrences separate (R19).
- On Create Invoice, backend checkout evaluates the prior qualifying history and calendar-day gap, chooses one applicable fee rule, and atomically records clinical entries, applicable fees/calculation evidence, invoice and lines. Reception receives the complete generated invoice with the automatic charge included. No additional receptionist examination-confirmation gate is required to compute the fee (R20,R76; S17 U03).
- **Locked / Q05 settled:** automatic fees first appear on the complete itemized invoice returned after successful issuance. The earlier preview focuses on clinical accuracy and shows no automatic fees or invoice prices/totals. Existing last-visit context may remain visible (R21,R79; S19).
- A fee event does not prove a clinical examination was performed. Do not invent an examination occurrence or relax invoice-source integrity merely to justify a charge.

## Clinical protocol and billing scope — Q03 clarified

The doctor assesses a returning patient as clinical protocol. The user confirms that this app must automatically apply the fee at Create Invoice, without requiring a new receptionist confirmation gate or a separate examination-record prerequisite. This is the current app requirement (R18,R20,R76).

Clinical examination representation remains audited; an extra_fees row does not prove an examination occurred. Q03 and Q05 are settled: automatic billing is required, and the fee is displayed only after successful issuance.

## Open audit

- Actual clinical examination representation: visit_services, a dedicated repeatable event, or another suitable occurrence model; no fabricated complaint-specific visit FK (A01).
- Patient-wide prior completed-encounter identification, same-day behavior, local calendar cutoff, historical imports, and package attendance/coverage interaction (A04).
- Exact extra_fees schema, immutable evidence fields, source ownership, one-charge-per-attendance enforcement, and concurrent duplicate prevention.
- Any exemptions/overrides not explicitly decided remain open; the fixed fee bands do not approve an override UI.
- Patient registration is distinct from return examination. Examinations may recur; do not impose a lifetime UNIQUE(patient_id) examination constraint.

## Evidence

Current rules and source references: [reconciliation register](reconciliation-2026-10-27.md#decision-register). Original versions are in [history/](history/); their wording does not override the current rules above.

Related: [clinical records](clinical-records.md), [checkout and invoice sources](checkout-and-invoice-lines.md), [packages](packages.md).

Current checkpoint: [consolidated seven-answer history log](history/clinic_module2_seven_review_answers_2026-10-27.md). All seven review questions are settled; implementation and parked audits remain unfinished.
