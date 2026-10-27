# Module 2 — seven review answers — 27 October 2026

Consolidated record of the owner's final answers and explanations in this chat. This adds no new policy. Earlier exchanges remain preserved in S17–S21.

## Q01 — Package-covered zero-total invoices

A package-covered-only invoice may total zero and must still exist. After issuance, offer only Mark as paid, with no Pay Later or collection choice. Close PAID without a payment row or collections. Coverage must be validated; an uncovered charge that makes the total positive uses ordinary settlement.

## Q02 — Four invoice status names

Use UNPAID, PARTIALLY_PAID, PAID and WAIVED. The owner explicitly chose this final set after initially retaining PENDING/DUE; those earlier labels are superseded, not extra states. UNPAID means no receipt; PARTIALLY_PAID means positive receipts with a remaining balance. PAID/WAIVED behavior and the covered-only zero-invoice exception remain unchanged.

## Q03 — Automatic examination/return billing

Doctor assessment is clinical protocol. Reception submits the clinical entries and clicks Create Invoice; the backend checks the patient-wide gap from the last qualifying completed clinical encounter and includes the applicable fee in the returned invoice. Use calendar days: first encounter ₹350; 0–9-day return gap no fee; 10–29 days ₹150; 30+ days ₹350, without stacking. No extra receptionist examination-confirmation gate is required.

## Q04 — Clinical records and invoice creation

Clinical records, applicable fees, invoice and lines commit in one atomic issuance transaction or all roll back. There is no clinical-save-first/bill-later MVP path; the earlier logged exception was incorrect. Discussion of possible future pending invoice lines did not authorize separate clinical persistence. Payment/waiver follows after the complete invoice is returned.

## Q05 — Clinical preview, then invoice amounts

Before Create Invoice, focus on clinical accuracy and what was done today. Do not show automatic fees or invoice prices/totals in that preview. Existing last-visit context can remain without a financial preview or approval step. Show the itemized charges and total only on the returned issued invoice; backend calculation still happens inside atomic issuance.

## Q06 — Service ID and joined clinical name

Store mandatory service_id on the standalone clinical occurrence and display the catalog name through a join; no copied clinical treatment-name snapshot. Invoice lines retain their frozen descriptions and amounts. Hypothetical catalog renaming/repurposing is not grounds for adding that clinical snapshot. The owner mentioned an aliases field for alternate wording; exact representation remains schema work. Existing recorder/server-timestamp conventions remain settled.

## Q07 — Catalog prices and availability

Only admin may edit existing master catalog prices or enable/disable services. Reception cannot. Admin-only additions were already settled. Enforce this authority on the backend alongside UI restrictions; exact authorization implementation remains engineering work.

All seven review questions are settled. Previously parked audits and dedicated package design remain unfinished. This record documents decisions; it does not claim application code, schema, migration or acceptance-test implementation.
