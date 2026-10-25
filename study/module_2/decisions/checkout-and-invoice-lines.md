# Checkout and invoice lines

Current discussion view incorporating the supplied October 25 records and scope continuation. These are design statuses, not applied migrations; see [chronology and overlap](history/2026-10-25-chronology-and-overlap.md).

## Current position

- **Settled:** normal checkout atomically persists confirmed clinical records, issued invoice and lines, or none. Several complaint-visits can belong to one checkout invoice.
- **Settled/clarified October 25:** clinical/billing preview shows entries, billable lines, prices and total before final confirmation. Successful issuance creates the invoice initially Pending; payment selection follows issuance. Payment write plus required settlement transition is a separate atomic operation.
- **Settled:** one flat therapy line per ordinary complaint-visit; routine performed procedures remain clinical records, with no separate charged or zero-valued procedure lines.
- **Settled:** a package-covered visit has its own ₹0 visit line. An entirely covered checkout still creates an invoice. A ₹0 line is not a receipt.
- **Settled:** one separately performed standalone-treatment occurrence produces one invoice line; do not aggregate actual occurrences or presell individual sessions.
- **Settled:** immutable issued `invoice_lines` are the sole authority for original billed amounts. Derive the invoice total by summing line amounts in paise; the proposed schema does not separately store `invoices.total_amount_in_paise`.
- **Settled:** original issued descriptions/amounts remain frozen even while Pending/Due. Catalog edits do not reprice old bills. Paid additionally locks settlement records; see [payments](payments-and-settlement.md).
- **Settled MVP scope:** no post-issuance append, charge reduction/repricing or retroactive clinical correction on Pending, Due or Paid. The intervening October 25 append permission was explicitly superseded for MVP; original-line immutability remains. No unused voiding fields should be pre-built.
- **Reconfirmed:** one visit belongs to at most one invoice; a new ordinary checkout means new work, not selecting an old billed visit to rebill. This does not itself prevent duplicate submission or malicious/direct API requests.
- **Settled, audit pending:** typed nullable real source FKs. Unchecked polymorphic source IDs and a universal occurrence parent were rejected. One logical source can have additional context identifiers; counting all non-null FKs as exactly one is not a valid general rule.

## Open questions and alternatives

- **O07/O13/O14 — next integrity question:** prevent billing the same visit or standalone occurrence again across lines/invoices, retries and concurrent submissions. Distinct actual occurrences remain valid.
- Audit source shape, patient/branch ownership, completeness, authorization, deletion safety, price consistency and package entitlement. Exact constraints, locks, retry keys and RPC are not chosen.
- **O11:** final standalone source shape and combined checkout UX. **O08:** package purchase/redemption source and proof behind zero-valued coverage. Examination occurrence remains open.
- `visits.invoice_id` is a recommended membership representation, not approved DDL. No unique one-invoice-per-day rule was approved.
- Pre-issuance review then final issuance is settled; exact UI, draft persistence and combined standalone/package flows remain open. The earlier issue-at-preview-and-append approach is not an MVP permission. Distinct forms still do not force distinct invoices or transactions.
- **O11/O13 source conflict recorded by the October 25 audit:** an omitted ₹150 treatment was described as referencing `visit_service`, against the earlier flat visit-line/independent standalone-source model. The example does not approve routine procedure charge lines or a universal invoice→visit_services join. Deferring amendments removes that correction scenario from MVP without resolving general source classification.
- **Future only:** void-filtered payable totals, amended invoice status, adjustment-aware floors and waiver-plus-void interactions remain unresolved. Do not silently apply these to original billed totals or treat a waiver as clinical correction.

## Reasoning and scenarios

Knee therapy ₹200 plus package-covered shoulder therapy ₹0 yields two visit lines and a ₹200 total. Ultrasound within knee therapy adds no line. Cervical and lumbar traction performed independently produce two occurrences and two lines. One review screen could combine therapy, standalone care and package purchase, but that grouping is not fully approved.

## Changes in position and sources

- [Parent register](history/clinic_billing_decision_log_2026-10-23.md), §§1, 3, 6–8: ownership, FK rationale, audit gates, migration and acceptance scenarios. The proposed charged-`visit_service` source and cached header total are historical alternatives.
- [Successor addendum](history/clinic_module2_successor_addendum_2026-10-23_1726_IST.md), §§4–7: worked checkouts and persistence alternatives; §§11–12: authoritative later settlements on the points above.
- Successor §11.1 supersedes **O05's mode-snapshot question**: ordinary complaint therapy is flat-only. §11.2 removes routine procedure charge fields in the proposed model. §11.4 settles line authority and derived header totals.
- Parent §§7–8 retain migration, concurrency, ownership and acceptance obligations. They are not executed tests or proof of deployed schema. Retail and formal refund/void/credit correction remain deferred.
- [October 25 A2](history/clinic_module2_successor_addendum_2026-10-25_171327_IST.md), §3, approved an append exception for Pending/Due; [A3](history/clinic_module2_audit_gap_appendix_2026-10-25_184849_IST.md), G01–G03, corrects its provenance and flags source ambiguity. These remain historical, not current MVP append authority.
- [Scope continuation](history/clinic_module2_scope_decision_addendum_2026-10-25.md), §§1–3, 6–7, withdraws retroactive correction/append from MVP and settles preview/issuance/payment ordering. A3 G06–G15 retains future risks rather than a complete correction design.
