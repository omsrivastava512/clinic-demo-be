# Collection authority and delivery phases — owner decision

Recorded 29 October 2026 from the owner's reply to clarity question PC08 in this conversation. This is an attributed summary, not a verbatim transcript. Current policy pages and the reconciliation register govern operative rules; earlier historical sources remain unchanged.

## Decision

- Reception may reopen an issued invoice and finish initial collection after a browser interruption, subject to confirming the stored outcome and the existing duplicate-prevention safeguards.
- Pay Later ends that initial collection interaction. It does not prohibit future reception staff from recording subsequent actual receipts against the invoice.
- The owner expands reception authority: future reception staff may record later partial or full positive payments today, tomorrow or another day against UNPAID or PARTIALLY_PAID invoices with a collectible balance. There is no initial-checkout-only or calendar-day authorization cutoff for those receipts.
- Only admin/owner may waive a balance. Reception must never have waiver controls or backend waiver permission. Payment permission does not permit changing issued lines, saved payments or clinical history.
- The current operating/delivery phase has one application role: admin. A separate reception role and its corresponding authorization/RLS are planned for a later/final MVP phase. This records intended phasing, not verified deployed role inventory.

## Preserved constraints and implementation boundary

Cash/UPI, positive immutable receipts, no overpayment, terminal PAID/WAIVED finality, separate atomic settlement and backend-derived recorder/server timestamps remain in force. A partial receipt leaves PARTIALLY_PAID; collecting the full collectible balance closes PAID. Covered-only zero invoices retain their separate Mark as paid path with no payment row.

The older requirement that reception only handle initial checkout and hand later receipts to admin is superseded. No automatic wallet or cross-invoice allocation is approved.

Exact role/RLS implementation, state/ownership enforcement, concurrency, idempotency and interrupted-request recovery remain technical audits. This decision does not implement application code or migrations.
