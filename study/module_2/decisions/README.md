# Module 2 decisions

The six topic pages state the current Module 2 decisions as of 27 October 2026. Each settled subject has one operative rule. Pending questions and audits are explicitly marked; they do not approve competing alternatives. Historical versions remain evidence in history/. These documents do not establish deployed behavior.

## Topic map

| Order | Topic | Contents |
| --- | --- | --- |
| 1 | [Clinical records](clinical-records.md) | Complaints, permanent therapy classification, catalogs, anatomy, standalone care |
| 2 | [Service catalog and pricing](service-catalog-and-branch-pricing.md) | Single-clinic current settings, two flat rates, future branch migration |
| 3 | [Checkout and invoice lines](checkout-and-invoice-lines.md) | Atomic issuance, combined invoices, original snapshots and sources |
| 4 | [Payments and settlement](payments-and-settlement.md) | Positive receipts, split tender, final waiver, timestamps, reporting |
| 5 | [Packages](packages.md) | Required MVP scope, dedicated parked design, covered-only zero-total exception |
| 6 | [Examination fees](examination-fees.md) | Calendar bands, extra_fees, exam-only care, clinical-model audit |

## Reading authority

- Use the six topic pages and the reconciliation decision register for current rules.
- Q06 and the parked audits are unfinished; no answer is inferred. Q07 catalog price/availability authority is settled: admin only.
- The tracker's Decision Register columns G–I contain current status, interpretation and sources. Columns A–F preserve historical attribution. Table Audit is a historical schema inventory, not current permission or schema design.
- Source inventories, coverage maps and override comparisons explain provenance; old wording there cannot override a current decision. Research is background material, not clinic policy.

## Current checkpoint

The active non-audit business discussion in the export reaches closure: complaint classification is manually assigned and permanent; charges are flat per complaint-visit; services are catalog-backed; admin additions are required. This does not close exact schema, source/ownership, API/RLS, concurrency, idempotency, migration or validation work. Packages and complaint lifecycle remain in separate discussions.

Use the [reconciliation and question register](reconciliation-2026-10-27.md) for all source mappings, overrides and pending questions. Q01–Q04 are resolved/corrected by the [user follow-up](history/clinic_module2_reconciliation_resolutions_2026-10-27.md). Q05 is settled: clinical preview first, automatic charges shown on the returned issued invoice. Q06 remains a standalone clinical service-name snapshot question. Q07 is settled: only admin may edit existing master prices and enable/disable services. Exact authorization enforcement remains audited. A01–A13 carry the remaining audit and parked work, including older cleanup and UI questions.

## Established changes

- WAIVED is a distinct final state; only actual positive payments count as collections. Final settlement uses settled_at/settled_by and waiver-specific reason, without an extra waiver flag or event table.
- Cash/UPI only; both split amounts explicit; one invoice per immutable positive payment row; created_at and received_at are both server-controlled and non-null.
- Delayed recording uses the recording day; no client backdating. An invoice is issued and returned before payment/waiver settlement.
- One-clinic services pricing/availability replaces launch-time branch configuration. Only admin may edit master prices or enable/disable services; reception cannot. Ordinary therapy remains flat, complaint-based, with no negotiated-rate engine.
- Calendar return bands are 0–9 days no gap fee, 10–29 ₹150, and 30+ ₹350, never stacked; first encounter ₹350. extra_fees is a charge source, not proof of an actual examination.
- Issued lines and saved payments are immutable. Retroactive correction/append/voiding remains outside MVP.
- Four invoice states are PENDING, DUE, PAID and WAIVED. The user explicitly reconfirmed retaining them (S18).
- Clinical entries, applicable automatic fees, invoice and lines commit atomically; no clinical-save-first/bill-later exception. The invoice is returned before separate settlement.
- Pre-issuance preview focuses on clinical records and accuracy. Automatic fees and invoice prices/totals first appear on the returned issued invoice (Q05; S19).
- Package-covered-only zero invoices are allowed. After issuance, offer only Mark as paid; no payment row or collections are created.

## Tracker

The [seven-tab tracker](clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.xlsx) retains its filename, tab structure, original record IDs and historical attribution. Its current interpretation/source columns, issue states, acceptance scenarios and summary formulas now follow this reconciliation. Proposed checks remain unexecuted; its 16-table audit remains historical.

## Source archive

The five former upcoming logs are now archived below. The updated MVP log and JSON export are archived copies; their dump originals are retained unchanged. Source bytes and names are preserved; internal original-location metadata is historical. The full inventory and hashes are in the [archive manifest](reconciliation-2026-10-27.md#archive-manifest).

| Source | Archived record | Focus |
| --- | --- | --- |
| S10 | [clinic_module2_waiver_catalog_addendum_2026-10-25.md](history/clinic_module2_waiver_catalog_addendum_2026-10-25.md) | Waived terminal status; single-clinic services and future migration |
| S11 | [clinic_module2_running_log_and_followup_2026-10-25.md](history/clinic_module2_running_log_and_followup_2026-10-25.md) | Exam-only attendance, mandatory clinical re-examination, calendar bands |
| S12 | [clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md](history/clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md) | Repayments, source cardinality boundaries, clinical-first exception, idempotency requirement |
| S13 | [clinic_module2_post_addendum_decisions_2026-10-25.md](history/clinic_module2_post_addendum_decisions_2026-10-25.md) | Methods, split amounts, waiver reason/flag, no backdating, separate settlement |
| S14 | [clinic_module2_running_decision_log_2026-10-26_1130_IST.md](history/clinic_module2_running_decision_log_2026-10-26_1130_IST.md) | D26-01–D26-15 and the appended payment field/deletion decisions |
| S15 | [module2_mvp_chat_decision_log_2026-10-26.md](history/module2_mvp_chat_decision_log_2026-10-26.md) | Earlier reconstruction plus later D-26–D-44 reconciliation |
| S16 | [chatgpt-full-transcript-1793041621482.json](history/chatgpt-full-transcript-1793041621482.json) | 176-message primary export; displaced opening replies and incomplete Show more text |
| S17 | [User reconciliation answers and corrections](history/clinic_module2_reconciliation_resolutions_2026-10-27.md) | Subsequent Q01–Q04 resolutions/corrections; Q05–Q07 explanations |
| S18 | [Current decision authority](history/clinic_module2_current_decision_authority_2026-10-27.md) | One current rule per settled subject; original status names retained |
| S19 | [Clinical preview and fee visibility](history/clinic_module2_clinical_preview_fee_visibility_2026-10-27.md) | Q05 settled; Q06 discussion and Q07 authority remain open |
| S20 | [Admin catalog authority](history/clinic_module2_admin_catalog_authority_2026-10-27.md) | Q07 settled: admin-only prices and availability; Q06 remains open |

The upcoming staging directory has no remaining consolidated source files. Future incoming records must be compared with the current topics rather than treated as automatic replacements.

## Earlier history and precedence

All pre-existing history files are retained unchanged. The [October 25 chronology guide](history/2026-10-25-chronology-and-overlap.md) remains the earlier comparison snapshot and preserves its chronology uncertainty, duplicate findings and scope reversals. The [new reconciliation](reconciliation-2026-10-27.md) extends that work without rewriting the originals.

Later explicit decisions override earlier statements only on the subjects addressed. Dates or assistant proposals alone do not establish approval. Preserve source-qualified IDs; S14:D26-01 and S15:D-01 belong to different logs. Source reconstruction is distinguished from raw transcript evidence and from actual implementation.

The research directory remains background material and is not authority for clinic requirements. Documents outside this decisions directory, application code, database schema and Git history are outside this consolidation.

Current follow-up: [user instructions on current decision authority](history/clinic_module2_current_decision_authority_2026-10-27.md). Q01–Q05 and Q07 are settled; Q06 remains open.

Latest answer: [clinical preview and fee visibility](history/clinic_module2_clinical_preview_fee_visibility_2026-10-27.md).

Catalog authority: [admin-only prices and availability](history/clinic_module2_admin_catalog_authority_2026-10-27.md). Q07 is closed.
