# Examination fees

Current examination/return-fee decisions as of 29 October 2026. **Examination and late/lapse fees are billing terms, not records or proof of a clinical examination.** Ongoing clinical assessment does not itself trigger these fees. Automatic fees first appear on the returned issued invoice; the earlier preview is clinical only.

## Settled fee policy

| Prior qualifying completed attendance | Applicable examination/return charge |
| --- | --- |
| First attendance | ₹350 (35000 paise) |
| Return after 0–9 calendar days, including five days | No examination/lapse fee event |
| Return after 10–29 calendar days | ₹150 (15000 paise) |
| Return after 30 or more calendar days | ₹350 (35000 paise), without an additional ₹150 |

- **Locked:** use calendar dates, not exact elapsed-hour thresholds. Day 10 and day 30 are inclusive boundaries. Clinic-local date/time-zone implementation and qualifying record selection remain audited (R16–R17).
- **Locked:** anchor on the patient's most recent qualifying completed attendance, not last formal examination, last payment, or invoice settlement. Therapy-only and fee-only attendance both reset recency. Mapping other encounter types and “completed” to actual records remains audit work.
- **Locked:** first and recurrent fee-only checkout follow the same rules. An applicable examination fee may be issued without therapy, with therapy starting another day. For example, a first ₹350 fee-only checkout followed by next-day therapy does not incur another return-gap fee. No complaint-visit, visit_services row or clinical examination record is required or fabricated for the fee (R15–R16; S23).
- **Locked:** a physical attendance with multiple complaint-visits must not multiply the examination/return charge. Opening a new complaint or performing routine clinical assessment does not itself trigger another examination fee (R14–R15).
- “Late fee” means return-gap charge, not late payment or late arrival. No appointment/scheduling module, booking reset, or cancellation/no-show policy is introduced (R13,R17).

## Charge events and checkout

- **Locked direction:** combined extra_fees stores independently recordable patient-level examination/return-gap charge events and supplies a real typed source for the invoice line. It does not require a complaint-specific visit_id or visit_services row. Keep actual standalone treatment occurrences separate (R19; S23).
- On Create Invoice, backend checkout evaluates prior qualifying attendance and calendar-day gap, chooses one applicable fee rule, and atomically records performed-care entries when present, applicable fees/calculation evidence, invoice and lines. Fee-only checkout needs no dummy clinical record. The checkout operator receives the complete invoice with the automatic charge included; no examination-confirmation gate is required (R20,R22,R76–R77).
- **Locked / Q05 settled:** automatic fees first appear on the complete itemized invoice returned after successful issuance. The earlier preview focuses on clinical accuracy and shows no automatic fees or invoice prices/totals. Existing last-visit context may remain visible (R21,R79; S19).
- A fee event states what the clinic charged. It does not prove an examination occurred or how thorough it was. Do not invent a clinical occurrence or relax invoice-source integrity to justify a charge.
- Settlement remains separate after issuance. A fee-only invoice with an applicable ₹350 fee is positive and uses ordinary settlement; this clarification creates no new zero-total exception.

## Clinical assessment and billing scope — Q03 clarified

Clinical assessment is an ongoing physiotherapy process. The application fee model does not establish whether an examination was performed. Both first-time and recurrent examination fees use the same independent billing-event and fee-only-checkout policy. Backend automatic charging depends on the settled attendance/gap rules, without a separately recorded examination prerequisite (R18,R19,R69,R76; S23).

## Open audit

- Exact independent extra_fees representation and fee-only attendance mapping; do not make charging depend on a complaint-visit or a performed-examination record (A01).
- Patient-wide prior completed-attendance identification, same-day behavior, local calendar cutoff, historical imports, and package attendance/coverage interaction (A04).
- Exact immutable evidence fields, source ownership, one-charge-per-attendance enforcement, and concurrent duplicate prevention.
- Any exemptions/overrides not explicitly decided remain open; the fixed fee bands do not approve an override UI.
- Patient registration is distinct from recurrent fee events. Do not impose a lifetime UNIQUE(patient_id) examination-fee constraint.

## Evidence

Current rules and source references: [reconciliation register](reconciliation-2026-10-27.md#decision-register). Original versions are in [history/](history/); their wording does not override the current rules above.

Related: [clinical records](clinical-records.md), [checkout and invoice sources](checkout-and-invoice-lines.md), [packages](packages.md).

Current fee clarification: [owner's billing-only and fee-only-checkout clarification](history/clinic_module2_examination_fee_semantics_2026-10-27.md). The [seven-answer checkpoint](history/clinic_module2_seven_review_answers_2026-10-27.md) remains evidence for Q01–Q07; implementation and parked audits remain unfinished.
