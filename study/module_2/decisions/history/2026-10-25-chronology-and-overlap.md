# Module 2 records: chronology and overlap

Comparison completed 25 October 2026. This guide compares the incoming records with the retained [scope addendum](clinic_module2_scope_decision_addendum_2026-10-25.md). The four distinct October 25 Markdown originals remain unchanged. With user approval, the redundant incoming October 18/22 copies were deleted and the Excel workbook was updated with reconciled statuses. This is documentary reconciliation, not new policy selection, implementation or deployed-state verification.

## Record map

| Label | Record | Distinct contribution |
| --- | --- | --- |
| B18 | [October 18 checkout log in history](clinic_checkout_billing_decision_log_2026-10-18.md) | Detailed original payment/waiver scenarios, authority matrix, outstanding questions and rationale. Fuller archive retained; redundant incoming copy deleted. |
| B22 | [October 22 billing log in history](clinic-billing-decision-log-2026-10-22.md) | Clinical workflow, catalog alternatives, therapy policy, scope and invoice-source questions; identical incoming copy deleted. |
| A1 | [Checkout/payments/voiding addendum](clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md) | October 25 initial ledger, repeated old-Due payments, payment atomicity, monetary authority, historical 16-table review and N01–N27 issue register. |
| X1 | [Updated Excel tracker](../clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.xlsx) | Original 34 A1 entries preserved, with current-status/source columns and 20 later entries. Reconciled issue register, overview, future-only cases and source references now include A2/A3/S. |
| A2 | [17:13 successor](clinic_module2_successor_addendum_2026-10-25_171327_IST.md) | Subsequent append permission, optional voiding and conditional admin/cascade/financial-floor rules, with worked examples. Some MVP permissions are subsequently withdrawn by S. |
| A3 | [18:48 audit appendix](clinic_module2_audit_gap_appendix_2026-10-25_184849_IST.md) | G01–G15 provenance corrections, source-model conflict, duplicate-submission distinctions and unresolved financial/clinical interactions. |
| S | [Retained scope addendum](clinic_module2_scope_decision_addendum_2026-10-25.md) | Subsequent scope decisions: no retroactive correction/voiding/line append in MVP; §§6–7 settle payment recording, timing, feedback, waiver reason and package scope. |

## Supported chronology versus recommended reading order

The date/time evidence establishes:

```text
B18 (18 Oct) → B22 (22 Oct)

A1 (25 Oct 16:46:48) → A2 (17:13:27) → A3 (18:48:49)
                          └→ S initial checkpoint → S §6 continuation → S §7 audit
```

The baseline dates do not make B22 a universal replacement for B18: it carries payment boundaries forward while leaving detail in the earlier record. The October 23 register and successor in [history/]() are another intervening source track referenced by A1–A3; omitting them from this incoming batch does not erase their decisions.

S explicitly follows A2 and explicitly supersedes the prior mandatory-append/optional-voiding scope. It has **no creation time** and does not establish its exact temporal relationship to A3. Read **A1 → A2 → A3 → S** as a reconciliation order, not a claim that S was written after 18:48:49. Apply S's explicit scope reversal to the subjects it names; preserve A3's distinct risks and unresolved questions. If a conflict is not explicitly reconciled, leave it open rather than manufacturing a timestamp winner.

Within S, §§6–7 and explicit supersession notes govern the earlier OPEN labels in §§1–5. Download suffixes, filesystem dates and the appendix's former claim to be the final handoff do not establish current blanket authority.

## Overlap and subject-specific resolution

| Subject | Earlier record / overlap | Treatment after comparison with S |
| --- | --- | --- |
| Additional admin receipts on old Due, remainder stays Due | B18 left this open; A1 §2.1 locks repeated positive receipts; A3 §1/G01 context preserves attribution | **Settled behavior retained**, S §§1, 4, 7. This differs from receptionist initial-checkout rights. No arbitrary receipt-count limit. |
| One payments row per actual positive receipt | B18 direction; A1 §2.2 ratifies; X1 C04 | **Settled concept retained**. No zero receipt or waiver payment row. Exact DDL remains open. |
| Payment write and required status transition atomic | A1 §2.2, A3 G05 | **Retained** by S. Checkout issuance is a separate atomic operation; atomicity does not prove retry safety. |
| Derived billed, collected, waived, outstanding amounts | A1 §2.3; A3 §3; October 23 line authority | **Retained**. Do not create independent header monetary authorities. Exact waiver storage/query implementation remains open; workflow status is distinct from arithmetic. |
| New checkout: zero/partial/full collection | B18 scenarios and A1 N16; S §§3, 6 | **Clarified/closed:** no receipt → Pay Later/Pending; actual partial → Due; full → Paid. Reception cannot create initial Due by pretending a zero receipt. Older admin Pending handling is a distinct case. |
| Preview and collection order | A1 distinguishes draft from issued amendments; S §§3, 6–7 | **Clarified:** clinical/line/price/total preview → final issuance (initially Pending) → payment choices after successful issuance. Backend-confirmed payment feedback follows successful save. Exact screen design stays open. |
| Tender method and split tender | A1 N16 and X1 open; S §§3, 6 | **Closed behavior:** method on every positive tender; split methods create separate positive rows on the same invoice. Validation and concurrency remain open technical work. |
| Saved payment edits/reversal/correction | A1 N15 and initial S checkpoint left open; older Paid lock narrower | **Closed for MVP by S §6:** all successfully saved receipts immutable, even Pending/Due. Accurate pre-save entry; no in-app retrospective repair. |
| Receipt date/backdating/late entry | B18 distinguishes actual receipt from recording; A1 N18; A3 receipt-time carry-forward | **Restricted by S §6:** same-day recording, automatic system time; no backdating or retrospective entry. Historical late-entry proposals are not MVP permissions. Cash reports use the system-recorded local collection day under this limitation. |
| Separate payment receipt artifact | Earlier receipt records/reporting can be confused with patient receipt issuance | **Closed by S §6:** no separately numbered/printed/sent payment receipt; backend record and clear UI confirmation still required. |
| Ordinary waiver | B18 admin settlement, A1 N14/N17, A3 financial distinctions | **Retained:** admin/owner only, mandatory reason in S §6. Ordinary forgiven collection is not a charge/clinical correction. Representation, partial waiver leaving Due and unresolved admin edge cases are not all settled here. |
| Post-issuance omitted-charge append | A1 §3.1 proposal → A2 §3 approved Pending/Due append → A3 G01/G02 correct origin and scoped immutability exception | **Superseded for MVP by S §§1–2, 7:** no append on issued Pending/Due/Paid. Earlier approval is historical; conditions are future design material. Original lines remain immutable throughout. |
| Clinical voiding release scope | A1 open → A2 optional → A3 optional | **Superseded for MVP by S §§1–2, 7:** retroactive clinical correction/voiding deferred. No unused void columns in initial migration. This is stronger than merely optional. |
| Void admin permission, single child, parent cascade, Paid-link block, collected-money floor | A2 §§1–2, 6; A3 G08/G09/G13 | **Future conditional requirements**, preserved rather than deleted. No current MVP void operation. Exact metadata constraints, cascade mechanism and waiver-aware floor remain unselected. |
| Overcharge via waiver / dynamic void-filter total | A1 §3.2, A2 §§4–6; A3 G06–G12 | **No dedicated correction workflow in MVP**, per S. Ordinary waiver does not validate a false clinical event. Financial/filter semantics and double-remedy conflicts remain future unresolved questions. |
| Source model: ordinary visit versus `visit_service` | A2's appended ₹150 line and generic JOIN conflict with October 23 visit-sourced flat lines; A3 G03/G09 | **Still a recorded conflict**, not a universal source-model change. Deferring append/voiding removes that correction scenario from MVP; it does not approve a general `visit_service` source or solve O11/O13. |
| Retry/idempotency, duplicate source billing and concurrent collections | A1 N19; A3 G04/G05; S §§6–7 | **Still mandatory technical work**, not solved by new-checkout meaning, atomicity or the no-corrections scope. No concrete mechanism is approved/implemented by these records. |
| Packages | B22 defers; October 23 independent track requires V1; S §6 confirms reported 60–80% use | **In MVP**. Offering, coverage, day/session meaning, activation and redemption remain dedicated design work. Refund/transfer/extension lean-scope assent is provisional, not detailed ratification. |
| Catalog, examination, standalone occurrences | B22 alternatives; A3 G03; S §4 | **Not settled by S**. Keep the October 23 records and topic-specific evidence; do not restore obsolete lifetime-exam or ordinary per-procedure assumptions from B22. |
| Historical schema inventory and cleanup | A1 §5 / X1 Table Audit; A3 §3–4 | **Unique carry-forward:** vitals deferred (drop/dormant open), alerts removal intent requires checks, timeline events purpose to investigate. S neither implements nor independently resolves these. No live schema audit performed here. |
| Frontend deselection/procedure cleanup | S §7 records an explicitly parked question | **Parked**, not an approved interface rule. Not part of this document cleanup. |

## Why the older records are not all redundant

- **A1** retains the detailed conversation origins, migration/table evidence, N01–N27 issue register and candidate tests. S carries the core decisions forward, not every evidence detail. Old issue priorities/statuses must be read through the table above.
- **A2** preserves the actual append example, the conditional void safeguards and how those positions arose. Its permissions are historical after S's explicit release-scope reversal, but its reasoning is not duplicated in full by S.
- **A3** contains distinct G01–G15 analysis: double reduction through waiver plus void-filtering, ordering that traps false clinical entries behind Paid lock, indirect Paid links, incomplete collected-only floor, and original-versus-eligible total meanings. These remain future risks, not current correction features. Source mapping and retry concerns still matter for MVP.
- **X1** originally presented A1 alone. Its seven existing tabs are retained. Original C01–C34 historical entries are preserved beside new current-status, interpretation and source columns; later decisions and risks extend the register to 54 entries. Its 33 issue rows distinguish active work, closed policy, future scope and parked questions; 15 are active open rows. Twenty illustrative checks distinguish MVP and future cases, with none claimed executed. Overview formulas were recalculated and its active-issue count verified by changing and restoring a status in the temporary authoring copy. The workbook now incorporates A2/A3/S but does not prove implementation.
- **B18/B22** preserve broader workflow and clinical/catalog rationale. S addresses changed scope/payment subjects rather than rewriting those domains.

The four October 25 originals have distinct content and remain intact in this history folder. The six topic pages now incorporate their current decisions and unresolved issues. The workbook is directly under decisions/, with source paths updated for this layout. Redundant incoming baseline copies were deleted with approval.

## Duplicate and variant findings against existing history

- Incoming **B22 was byte-for-byte identical** to [the October 22 record in history](clinic-billing-decision-log-2026-10-22.md). The redundant incoming copy was deleted after re-verification.
- Incoming **B18 was a shorter snapshot** of [the archived October 18 record](clinic_checkout_billing_decision_log_2026-10-18.md). The archive additionally contains **§15, “Open discussion added — 21 October 2026”**; otherwise the text matched after normalizing line endings. The incoming copy was deleted after re-verification. The fuller archive, including §15, remains unchanged.
- X1's filename retains its original 16:46:48 timestamp. The Overview and Sources tabs explicitly identify the October 25 reconciliation update; the workbook is no longer an unchanged A1-only snapshot. No claim of equivalence between different file formats is made.

## Remaining uncertainty and next boundary

S-versus-A3 exact creation order remains unproven. Explicit subject-specific supersession is sufficient to classify append/voiding as outside MVP without inventing that timestamp. Any future correction/filter model must resolve A3's conflicts before implementation; deferral is not technical resolution.

Authorized reorganization updated the six topic summaries and main README, moved the four October 25 originals here without changing their contents or names, retained this guide, and placed the updated workbook directly under decisions/ with corrected source paths. The temporary dec/ and unres/ folders and their redundant navigation README were removed. No application code, schema or Git-history changes were made; nothing was staged, committed or pushed.

## Scope download cleanup

The retained scope file is the expanded continuation with §§6–7. An earlier 9,031-byte checkpoint and two exact expanded duplicates were removed with approval after full comparison found no substantive content missing from the retained version. Download suffixes were not chronology identifiers. The retained original remains unchanged; only its location moved.
