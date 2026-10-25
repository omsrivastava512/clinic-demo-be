# Payments and settlement

Current discussion view incorporating the supplied October 25 payment addenda and scope continuation. These record approved behavior, not deployed schema or implementation. Exact chronology and subject-specific precedence are documented in the [comparison guide](history/2026-10-25-chronology-and-overlap.md).

## Current position

- **Settled concept:** original billed amount, actual receipts, authorized waiver/write-off and outstanding balance are separate facts. Outstanding = billed − collected − authorized adjustments within the modeled no-refund case.
- **Settled:** a new invoice begins Pending. Reception may collect full payment → Paid, choose Pay Later → Pending, or collect an initial partial payment → Due. Reception cannot waive.
- **Settled:** later action on older Pending/Due invoices belongs to admin/owner. Admin may collect money and settle with an authorized waiver of the remainder. Paid does not mean the entire original bill was received in cash.
- **Settled on October 25:** admin can record further positive receipts on an older Due invoice and leave the rest Due, repeatedly without an arbitrary repayment-count limit. For a ₹500 bill with ₹200 collected, another ₹100 means ₹300 collected and ₹200 still Due; it does not automatically waive the remainder.
- **Settled concept:** one common `payments` table with one row per actual strictly positive receipt. Payment insertion and any required invoice-state/settlement change are atomic; exact DDL, concurrency and replay controls remain open.
- **Settled:** each positive tender specifies a method; split payment on one invoice uses separate rows, such as cash plus UPI. Reception's initial zero receipt means Pay Later/Pending, never a fake ₹0 receipt or zero-payment Due action. Partial actual collection means Due; full collection means Paid.
- **Settled MVP:** successfully saved payments are immutable at Pending, Due and Paid. No edit, reversal or in-app correction after saving. Staff verify the actual amount before submission; saved mistakes have no in-product repair in MVP.
- **Settled MVP:** same-day payment recording with automatic system timestamp; no manual backdating, historical date selection or retrospective entry. Today's receipt on an older invoice counts toward today's collection. A missed receipt cannot be entered retrospectively under this MVP policy.
- **Settled:** only admin/owner may waive outstanding collection, with a mandatory reason. This ordinary settlement waiver does not correct issued charge amounts or false clinical records.
- **Settled:** no ₹0 receipt or payment row for a waiver. Example: ₹500 billed, ₹200 received earlier, ₹300 waived at final settlement → Paid, collected ₹200.
- **Settled:** original issued charges stay frozen even unpaid; final Paid locks charges, receipts, methods and waivers, including for the owner in MVP.
- **Settled MVP:** no distinct numbered/printed/sent payment receipt artifact. Backend payment recording remains required. Only confirmed backend save permits UI success, showing amount, methods and current balance/status; failed or uncertain save must not report success.
- Old debt is handed to admin manually. Automatic chasing, unallocated-money wallet and automated cross-invoice allocation are deferred.

## Open questions

- **O09/O10:** can a partial waiver leave the invoice Due, or is waiver only part of final settlement? Detailed admin Pending-resolution and zero-receipt/full-waiver edge cases are not all closed by the repeated-receipt decision.
- Exact method enum, form layout, waiver classification/storage and actor/time evidence, clinician privileges and backend validation. Split tender, mandatory waiver reason and saved-receipt immutability are no longer open business questions.
- **O14–O16:** lock/RLS enforcement, idempotency, concurrent collection/overcollection prevention, local reporting cutoff and balance/status consistency. Automatic timestamps do not independently choose all time-zone details.
- **Future only:** late-entry exceptions, saved-payment reconciliation and refund/credit/retroactive correction workflows. No permission for these is inferred from ordinary waiver support.

## Reporting and reasoning

Distinguish bill issuance, collection and final settlement. Cash collections include positive receipts on still-Due invoices and use their system-recorded local collection day under the same-day MVP rule. Older separate received-at/recorded-at and late-entry proposals are historical, not backdating permissions. Excluding Pending from a settled-invoice report does not exclude it from outstanding reports or exclude Due receipts from cash collections. Revenue, cash, waiver totals and invoice counts are different measures. Saved entry errors can leave incorrect collections without an in-product repair; that limitation is recorded, not solved.

## Changes in position and sources

- [October 18 log](history/clinic_checkout_billing_decision_log_2026-10-18.md), §§3–5: detailed permissions and scenarios A–H; §§7–11: reversals, open lifecycle issues and risks.
- [Parent register](history/clinic_billing_decision_log_2026-10-23.md), §5 and §11: initial partial collection supersedes the older full-payment/Pay-Later-only flow; waiver may follow earlier partial receipts; Due does not imply admin review; issuance lock is stronger than Paid-only charge locking.
- [Successor](history/clinic_module2_successor_addendum_2026-10-23_1726_IST.md), §§11.3–11.4 and 12: zero-value lines are not receipts; payment policy is outside that settlement update.
- [October 25 A1](history/clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md), §§2.1–2.3: closes old-Due repeated receipts, ratifies positive receipt rows and atomic payment writes, and derives monetary totals from their sources.
- [October 25 audit](history/clinic_module2_audit_gap_appendix_2026-10-25_184849_IST.md), G04–G05 and §3: safeguards remain technical work; checkout atomicity does not prove retry safety.
- [Scope continuation](history/clinic_module2_scope_decision_addendum_2026-10-25.md), §§3, 6–7: closes split-tender/timing/correction/feedback decisions and mandates waiver reason. Its later sections supersede its own earlier OPEN labels. No exact scope-versus-audit timestamp is asserted.

Related: [original charges and invoice totals](checkout-and-invoice-lines.md), [package activation ambiguity](packages.md).
