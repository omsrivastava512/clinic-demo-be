# Checkout and invoice lines

Current checkout decisions as of 27 October 2026. Settled rules below are authoritative for the proposed MVP; pending questions remain explicitly marked. No DDL or migration is applied here. Evidence and current issue dispositions are in the [reconciliation register](reconciliation-2026-10-27.md).

## Current position

- **Locked:** MVP checkout atomically saves confirmed clinical records, applicable extra-fee records, invoice and lines, or none. No clinical-save-first/bill-later path is permitted (R22–R23,R77).
- **Locked:** issue and return the complete invoice first. Payment/waiver settlement is a separate later atomic transaction. No transaction is held open across screens or user decisions (R24).
- **Locked / Q05 settled:** Before Create Invoice, review what was done today for clinical accuracy. The preview shows clinical records, without automatic fee amounts or invoice prices/totals. Existing last-visit context may remain visible. After successful atomic issuance, display the complete itemized invoice with automatic fees and total; settlement follows separately. Backend fee calculation remains inside issuance (R21,R79; S19).
- **Locked:** one flat therapy line per treated complaint-visit; two complaints in one physical attendance create two visits. Bundled routine procedures remain clinical records with no separate charged or zero-value procedure lines (R04–R05).
- **Locked:** complaint therapy, separately performed standalone care, and an applicable examination/return fee may share one checkout invoice as distinct lines. One standalone performance means one actual billable occurrence, not an aggregate or presold individual session (R25–R26).
- **Locked direction:** examination/return-gap charge events use the combined extra_fees source. A charge is not proof a clinical examination occurred. Clinical examination representation remains audited (R19–R20, R69).
- **Locked:** original billed amount is the sum of immutable issued line snapshots. Do not store an additional authoritative header total or reprice old lines from current catalog values (R31, R54).
- **Locked:** original issued descriptions/amounts remain frozen even while unpaid/partially paid. No post-issuance append/remove/reprice, replacement or retroactive clinical correction is permitted in MVP (R63).
- **Locked:** invoice deletion is forbidden; positive payment rows are separately immutable and their invoice FK uses ON DELETE RESTRICT (R49–R50).

## Covered-only zero-total invoices — Q01 resolved

At least one invoice line is mandatory. A package-covered-only checkout may have a zero total; the backend must validate actual coverage. This is the permitted zero-total case. Other zero-total invoices and extra zero-value bundled-procedure lines are not authorized (R32–R33,R74).

After the complete invoice is issued and returned, offer only Mark as paid for this zero-total case: no Pay Later or collection choice, no payment row, zero collections. Close PAID with the existing settlement metadata. If an uncovered line or automatic fee makes the total positive, ordinary settlement applies. Coverage/redemption implementation remains audited.

## Source integrity and open audit

- **Working default / audit required:** typed nullable real source FKs, with exactly one logical billable source per line. Only source FKs participate in that count; patient/clinic/context FKs do not become extra sources (R34).
- Prevent billing the same occurrence twice across lines/invoices. The uniqueness key is the actual visit, standalone occurrence, or applicable fee event, never the reusable service_id. Distinct actual performances remain valid (R35).
- One visit belongs to at most one invoice. visits.invoice_id is an earlier recommended membership representation, not approved physical DDL. No one-invoice-per-day rule is approved.
- Include extra_fees without treating a dedicated examination_occurrence_id as finalized. Final source column names, source shape, ownership, completeness, deletion safety, and exact CHECK/unique constraints remain audited.
- Flat ordinary therapy is sourced from the complaint-visit; bundled visit_services do not become separate charge lines. Exact source FKs and existing-schema mapping remain audited.
- Backend idempotency is still required. A disabled button, source uniqueness, and an atomic transaction solve different problems; the key/hash/invoice proposal remains provisional (R60–R61).
- Exact draft persistence, request payloads, locking, retry recovery, error handling, and migration remain unfinished. General form boundaries do not force transaction or invoice boundaries.

## Future-only correction material

Archived conditional void/cascade rules, waiver-plus-void double reductions, financial floors, false-record/Paid-lock traps, and original-versus-eligible totals remain future design risks. Do not apply void-filtered totals or adjustment-driven repricing to current immutable invoice history (R64).

## Evidence

Current rules and source references: [reconciliation register](reconciliation-2026-10-27.md#decision-register). Original versions are in [history/](history/); their wording does not override the current rules above.

Related: [clinical records](clinical-records.md), [payments](payments-and-settlement.md), [packages](packages.md), [examination fees](examination-fees.md).

Current authority: [user instructions and consistency corrections](history/clinic_module2_current_decision_authority_2026-10-27.md). Q01–Q05 and Q07 are settled; Q06 remains open. Earlier answers are preserved in [S17](history/clinic_module2_reconciliation_resolutions_2026-10-27.md).

Latest answer: [clinical preview and fee visibility](history/clinic_module2_clinical_preview_fee_visibility_2026-10-27.md).

Catalog authority: [admin-only prices and availability](history/clinic_module2_admin_catalog_authority_2026-10-27.md). Q07 is closed.
