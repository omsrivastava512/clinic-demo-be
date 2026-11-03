# Module 2 decisions

The six topic pages state the current Module 2 decisions as of 2 November 2026. Each settled subject has one operative rule. Unfinished questions and audits are explicitly marked; they do not approve competing alternatives. Historical versions remain evidence in history/. These documents do not establish deployed behavior.

**Package source scope:** all four package records are reconciled through the 2 November 2026 voice session (S25–S28). The long offering's 26-visit ceiling and 1-per-day limit are confirmed; the 7 closure reason presets, backend date validation on "Patient Stopped Attending", post-closure factual summary, explicit rejection of queuing, and N+1 replacement start are settled; payment gating remains open for owner resolution. Other topic decisions remain governed by their current sources.

The earlier policy-clarity audit report and its coverage CSV record the S24 snapshot; their file counts and hashes predate this package incorporation.

## Topic map

| Order | Topic | Contents |
| --- | --- | --- |
| 1 | [Clinical records](clinical-records.md) | Complaints, permanent therapy classification, catalogs, anatomy, standalone care |
| 2 | [Service catalog and pricing](service-catalog-and-branch-pricing.md) | Single-clinic current settings, two flat rates, future branch migration |
| 3 | [Checkout and invoice lines](checkout-and-invoice-lines.md) | Atomic issuance, combined invoices, original snapshots and sources |
| 4 | [Payments and settlement](payments-and-settlement.md) | Positive receipts, split tender, final waiver, timestamps, reporting |
| 5 | [Packages](packages.md) | Required MVP scope, 10-visit & 26-visit offerings, 1/day limit, soft-review UX, role closure, covered-only zero-total exception |
| 6 | [Examination fees](examination-fees.md) | Billing-only meaning, calendar bands, independent extra_fees, first/recurrent fee-only checkout |

## Reading authority

**Delivery phase (S24):** the current operating phase is admin-only. A separate reception role and its authorization/RLS are planned for a later/final MVP phase. Reception references describe future permissions; clinical workflow actors do not imply extra application roles. Future reception may take initial and later payments, including after Pay Later, but only admin may waive. This is policy/phasing intent, not verified deployment state.

- Use the six topic pages and the reconciliation decision register for current rules.
- All seven review questions Q01–Q07 are settled. Parked technical audits and dedicated package design remain unfinished; no implementation completion is implied.
- The tracker's Decision Register columns G–I contain current status, interpretation and sources. Columns A–F preserve historical attribution. Table Audit is a historical schema inventory, not current permission or schema design.
- Source inventories, coverage maps and override comparisons explain provenance; old wording there cannot override a current decision. Research is background material, not clinic policy.

## Current checkpoint

The [2 November package voice session log](history/package_decision_log_2026-11-02_voice_session.md) (S28) finalizes the 7 closure reason presets, enforces backend/UI date gating on "Patient Stopped Attending" (selectable strictly on Review Overdue), mandates a post-closure factual summary with invoice/refund navigation, establishes that closure never automatically alters finances or waives money, and formally rejects queued advance package purchases (replacement purchases are blocked until prior package is Completed or Closed, starting Day N+1).

The [2 November package patch](history/clinic_packages_discussion_patch_2026-11-02_181036_IST.md) (S27) incorporates the third cumulative package record. The long offering's 26 covered-visit hard ceiling and 1-per-day coverage limit are confirmed [OWNER—REPORTED—CONFIRMED]. The 10-visit Sunday purchase day-1 counting rule, two-stage review UX, registration review-clock start, role-based closure (admin anytime, receptionist strictly on overdue status), same-complaint active purchase blocking, and independent routine/rehab pricing are settled [USER—DECIDED]. Payment-gating policy remains explicitly open for owner consultation.

The [29 October collection-authority decision](history/clinic_module2_reception_payment_authority_2026-10-29.md) (S24) settles PC08 and expands future reception payment authority beyond initial checkout. Earlier admin-only later-collection restrictions are superseded. The original seven review answers remain settled.

The [short consolidated history log](history/clinic_module2_seven_review_answers_2026-10-27.md) records all seven final answers and their explanations together. The topic pages already contain these decisions; this pass verifies coverage and links the summary.

The [later fee clarification](history/clinic_module2_examination_fee_semantics_2026-10-27.md) (S23) settles examination/lapse fees as independent billing events, not clinical examination records. First and recurrent fee-only checkout follow the same rules, without a required complaint-visit or visit_services row. A five-day return incurs no such fee.

The active non-audit business discussion in the export reaches closure: complaint classification is manually assigned and permanent; charges are flat per complaint-visit; services are catalog-backed; admin additions are required. This does not close exact schema, source/ownership, API/RLS, concurrency, idempotency, migration or validation work. Packages and complaint lifecycle remain in separate discussions.

Use the [reconciliation and question register](reconciliation-2026-10-27.md) for all source mappings, overrides and pending questions. Q01–Q04 are resolved/corrected by the [user follow-up](history/clinic_module2_reconciliation_resolutions_2026-10-27.md). Q05 is settled: clinical preview first, automatic charges shown on the returned issued invoice. Q06 is settled: standalone clinical records use service_id with a joined catalog name,without a copied clinical name. Q07 is settled: only admin may edit existing master prices and enable/disable services. Exact authorization enforcement remains audited. A01–A13 carry the remaining audit and parked work, including older cleanup and UI questions.

## Established changes

- WAIVED is a distinct final state; only actual positive payments count as collections. Final settlement uses settled_at/settled_by and waiver-specific reason, without an extra waiver flag or event table.
- Cash/UPI only; both split amounts explicit; one invoice per immutable positive payment row; created_at and received_at are both server-controlled and non-null.
- Delayed recording uses the recording day; no client backdating. An invoice is issued and returned before payment/waiver settlement.
- One-clinic services pricing/availability replaces launch-time branch configuration. Only admin may edit master prices or enable/disable services; reception cannot. Ordinary therapy remains flat, complaint-based, with no negotiated-rate engine.
- Calendar return bands are 0–9 days no gap fee, 10–29 ₹150, and 30+ ₹350, never stacked; first encounter ₹350. extra_fees is a charge source, not proof of an actual examination.
- Issued lines and saved payments are immutable. Retroactive correction/append/voiding remains outside MVP.
- Four invoice states are UNPAID, PARTIALLY_PAID, PAID and WAIVED. The user explicitly approved this naming set in S21; business behavior is unchanged.
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
| S18 | [Current decision authority](history/clinic_module2_current_decision_authority_2026-10-27.md) | Current-file authority; earlier label choice superseded by S21 |
| S19 | [Clinical preview and fee visibility](history/clinic_module2_clinical_preview_fee_visibility_2026-10-27.md) | Q05 clinical preview and fee visibility |
| S20 | [Admin catalog authority](history/clinic_module2_admin_catalog_authority_2026-10-27.md) | Q07 admin-only prices and availability |
| S21 | [Status names and standalone identity](history/clinic_module2_status_names_and_standalone_identity_2026-10-27.md) | Approved UNPAID/PARTIALLY_PAID/PAID/WAIVED; Q06 service ID with joined name; all seven review questions settled |
| S22 | [Consolidated seven-answer history log](history/clinic_module2_seven_review_answers_2026-10-27.md) | Compact final answers and owner explanations for Q01–Q07; no new policy |
| S23 | [Examination/lapse fee meaning](history/clinic_module2_examination_fee_semantics_2026-10-27.md) | Billing-only fees; independent first/recurrent fee-only checkout; attendance recency and five-day no-fee clarification |
| S24 | [Collection authority and delivery phases](history/clinic_module2_reception_payment_authority_2026-10-29.md) | PC08 resolved; future reception initial/later receipts, admin-only waiver, admin-only current phase and later reception/RLS |
| S25 | [Package handover, 28 October](history/clinic_packages_handover_2026-10-28.md) | User decisions, reported owner requirements, proposals and open questions through 28 October |
| S26 | [Package addendum, 28–29 October](history/clinic_packages_addendum_2026-10-28_to_2026-10-29.md) | Subsequent directions and changes, owner-confirmation gaps, proposals and unresolved conflicts through 29 October |
| S27 | [clinic_packages_discussion_patch_2026-11-02_181036_IST.md](history/clinic_packages_discussion_patch_2026-11-02_181036_IST.md) | Third cumulative package record: 26 hard cap, 1/day, Sunday review arithmetic, closure authority, registration clock, payment gating reopened |
| S28 | [package_decision_log_2026-11-02_voice_session.md](history/package_decision_log_2026-11-02_voice_session.md) | Fourth package record: 7 closure reasons, "Patient Stopped Attending" date rule, post-closure summary, queuing rejected, N+1 renewal start |

The upcoming staging directory has no remaining consolidated source files. Future incoming records must be compared with the current topics rather than treated as automatic replacements.

## Earlier history and precedence

All pre-existing history files are retained unchanged. The [October 25 chronology guide](history/2026-10-25-chronology-and-overlap.md) remains the earlier comparison snapshot and preserves its chronology uncertainty, duplicate findings and scope reversals. The [new reconciliation](reconciliation-2026-10-27.md) extends that work without rewriting the originals.

Later explicit decisions override earlier statements only on the subjects addressed. Dates or assistant proposals alone do not establish approval. Preserve source-qualified IDs; S14:D26-01 and S15:D-01 belong to different logs. Source reconstruction is distinguished from raw transcript evidence and from actual implementation.

The research directory remains background material and is not authority for clinic requirements. Documents outside this decisions directory, application code, database schema and Git history are outside this consolidation.

Current fee clarification: [owner's billing-only and fee-only-checkout clarification](history/clinic_module2_examination_fee_semantics_2026-10-27.md). The [seven-answer checkpoint](history/clinic_module2_seven_review_answers_2026-10-27.md) remains evidence for Q01–Q07; implementation and parked audits remain unfinished.
