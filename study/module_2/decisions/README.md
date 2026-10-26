# Module 2 decisions

Read by subject rather than reconstructing dated conversations. These topic pages incorporate the supplied **October 25 payment, audit and scope-continuation records**, while retaining earlier decisions on subjects those records do not settle. They describe discussion status, not deployed behavior or implementation authorization. Original records are preserved byte-for-byte with original filenames in [history/](history/).

## Topic map and reading order

| Order | Topic | Contents |
| --- | --- | --- |
| 1 | [Clinical records](clinical-records.md) | Complaints, visits, routine procedures and standalone occurrences |
| 2 | [Service catalog and branch pricing](service-catalog-and-branch-pricing.md) | O06 alternatives, availability, ownership and provisioning |
| 3 | [Checkout and invoice lines](checkout-and-invoice-lines.md) | Atomic checkout, sources, totals, immutability and duplicate billing |
| 4 | [Payments and settlement](payments-and-settlement.md) | Pending/Due/Paid, receipts, waivers, roles and reporting |
| 5 | [Packages](packages.md) | Purchase, activation, entitlement and covered visits |
| 6 | [Examination fees](examination-fees.md) | Recurrence, eligibility, fee interaction and occurrence modeling |

Each page owns its topic's current position, alternatives, open questions and source trail. The originals retain complete scenarios, rejected proposals, technical audit and migration checklists, acceptance obligations and historical wording. Consult them for detail without treating every historical status label as current.

## Where discussion resumes

Source uniqueness/retry/concurrency (O07/O13/O14), O06 catalog configuration, O08 package relationships, O11 occurrence schema and O02–O04 examinations remain open. Payments now include the supplied October 25 continuation: repeated admin receipts on old Due are allowed, saved payments cannot be edited, backdating is prohibited, split tender is supported and waiver reason is mandatory. Exact waiver representation, some admin settlement edge cases and backend enforcement still need design. No unseen package or other conversation is incorporated.

Important supersessions: ordinary therapy is flat-only in the proposed model; invoice totals derive solely from immutable lines; package-covered visits have zero-valued visit lines; all-inactive branch provisioning was reopened. October 25's intervening Pending/Due append permission and optional clinical voiding are **superseded for MVP** by the scope continuation: no post-issuance charge append/repricing or retroactive clinical correction. Ordinary admin waiver remains a settlement action, not an error-correction workflow. See the owning topic for evidence and qualifications.

## Excel tracker

The [updated workbook](clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.xlsx) retains its original filename and seven tabs. It preserves the initial 34 discussion entries alongside current interpretations and sources, adds later decisions and distinguishes active, closed and future issues. It is no longer an unchanged 16:46 snapshot; its Overview and Sources identify the reconciliation. Source paths point to the originals in `history/`.

## Upcoming decisions (staging)

The [upcoming/](upcoming/) directory contains in-flight addenda and running decision logs from the October 25�26 working sessions that are currently awaiting final consolidation into the primary topic pages:

| Record | Focus |
| --- | --- |
| [Waiver and catalog addendum](upcoming/clinic_module2_waiver_catalog_addendum_2026-10-25.md) | Dedicated `Waived` terminal status, balance zeroing, and single-clinic MVP scope |
| [Running log and follow-up](upcoming/clinic_module2_running_log_and_followup_2026-10-25.md) | Examination-only visits, 30-day recurrence rules, and calendar gap calculation |
| [Six questions / three ideas](upcoming/clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md) | Multi-installment Due repayments, partial checkout receipts, and atomic checkout |
| [Post-addendum decisions](upcoming/clinic_module2_post_addendum_decisions_2026-10-25.md) | CASH/UPI tender restriction, split tenders, derived balances, and server-time lock |
| [Running log (Oct 26, 11:30 IST)](upcoming/clinic_module2_running_decision_log_2026-10-26_1130_IST.md) | D26-01 to D26-15: `payments` table DDL invariants, BIGINT paise, and invoice deletion locks |

## Original records and precedence

| Original | Purpose |
| --- | --- |
| [October 18 checkout log](history/clinic_checkout_billing_decision_log_2026-10-18.md) | Payment lifecycle, role boundaries and scenarios, including its October 21 note |
| [October 21 addendum](history/clinic_billing_decision_addendum_2026-10-21.md) | Earlier catalog acceptance and offering context, later revisited |
| [October 22 billing log](history/clinic-billing-decision-log-2026-10-22.md) | Earlier clinical/billing decisions and reopened catalog alternatives |
| [Parent register](history/clinic_billing_decision_log_2026-10-23.md) | Broad baseline, IDs, audit/migration and acceptance obligations |
| [O06 brief, 14:37 IST](history/O06_service_catalog_branch_configuration_decision_brief_2026-10-23.md) | Catalog alternatives and reopened provisioning |
| [Successor addendum](history/clinic_module2_successor_addendum_2026-10-23_1726_IST.md) | 17:26 snapshot plus later 20:21 settlement in §§11–12 |
| [October 25 initial addendum](history/clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md) | 16:46:48 IST; repeated old-Due payments, derived financials, historical schema review and issue register |
| [October 25 successor](history/clinic_module2_successor_addendum_2026-10-25_171327_IST.md) | 17:13:27 IST; intervening append permission and conditional void safeguards |
| [October 25 audit appendix](history/clinic_module2_audit_gap_appendix_2026-10-25_184849_IST.md) | 18:48:49 IST; provenance corrections, source-model conflict and integrity risks |
| [October 25 scope continuation](history/clinic_module2_scope_decision_addendum_2026-10-25.md) | Follows 17:13; no exact time; MVP scope reversals plus continued payment/package decisions in §§6–7 |

The [chronology and overlap guide](history/2026-10-25-chronology-and-overlap.md) records unique material, supersession and duplicate findings. Timestamped order is 16:46 → 17:13 → 18:48. The scope continuation follows 17:13, but its exact temporal relation to 18:48 is unconfirmed. Reading it after the audit is a reconciliation order, not an invented creation time; apply explicit supersession only to the subjects addressed. Redundant incoming October 18/22 copies and redundant scope downloads were removed after comparison; archived baselines remain intact.

Later explicit decisions supersede earlier decisions only on the subjects they address. A later proposal is not approval. Filename dates do not override explicit later sections within records. Unresolved matters remain unresolved; owner reports are distinguished from deployed evidence.

Original maintenance instructions and internal filename references remain historical source content, unchanged. The topic pages are the reading layer. Future approved decisions should update the owning topic with status, rationale and source while preserving historical evidence. Discussion files and logs outside this folder remain untouched.
