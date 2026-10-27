# Payments and settlement

Current payment decisions as of 27 October 2026. Settled rules below are authoritative for the proposed MVP; exact schema, authorization and API work remain audited. Evidence and current issue dispositions are in the [reconciliation register](reconciliation-2026-10-27.md).

## State meanings and authority

- **Locked:** PAID means the full billed balance is discharged through actual receipts, or through the explicitly permitted package-covered-only zero-total settlement. WAIVED means admin forgave the entire remaining collectible amount, including after partial receipts. Both are terminal with zero collectible outstanding and no subsequent receipts, waiver, or state mutation (R36,R74).
- **Locked:** four invoice states are UNPAID (issued with no receipt), PARTIALLY_PAID (positive receipts with a remaining collectible balance), PAID and WAIVED. This explicitly approved naming set replaces the earlier PENDING/DUE labels (R37,R75,R81; S21).
- **Locked:** for a positive-total invoice, reception may record the initial actual full or partial receipt: no receipt means Pay Later/UNPAID; partial actual collection leaves PARTIALLY_PAID; full actual collection means PAID. A package-covered-only zero invoice instead offers only Mark as paid after issuance, without any payment row (R38,R74).
- **Locked:** admin/owner may record repeated additional positive receipts on older PARTIALLY_PAID invoices, with no arbitrary repayment-count cap, leaving the residual due. Reception communicates old-debt collections to admin manually; no automatic cross-invoice allocation or wallet is approved (R39).
- **Locked:** only admin/owner may waive; hide waiver controls from unauthorized staff and enforce authority on the backend. Reception cannot waive or autonomously change older invoices (R40).

## Payment records and transaction boundary

- Invoice issuance commits and the invoice is returned before settlement. Payment rows and the required valid settlement transition save together in a separate transaction (R24).
- One positive actual tender component is one payment row; each row belongs to exactly one invoice, and an invoice may have zero, one, or many payments (R44–R45).
- Use BIGINT integer paise. Payments must be strictly positive; zero collection and waived money create no payment rows (R48). The package-covered-only zero invoice has a separate Mark as paid closure after issuance, with settled_at/settled_by and no receipt or waived amount; ordinary no-receipt Pay Later does not apply to that case (S17 U01).
- Only Cash and UPI are accepted backend enum values; exact SQL casing is not selected here. Split is a UI input mode. Operator enters both amounts explicitly; partial split may leave a balance and must not imply full payment (R42–R43).
- Split components and the state update are atomic: failure of any part rolls back the entire settlement. No positive tender component is silently dropped (R44).
- Do not collect more than current collectible outstanding. Backend enforcement under concurrency is required; exact function/locking/authorization implementation remains audit work (R41).
- Payment recorded_by is mandatory and derived from authenticated backend identity rather than a React-supplied recorder ID. The user baseline identifies profiles(id) UUID authentication; actual FK/default/security integration remains audited (R47).
- Keep both created_at and received_at as server-controlled TIMESTAMPTZ NOT NULL with server defaults. Neither may be client-backdated in MVP (R46).
- **Accepted recording limitation:** if staff/admin record physically received money later, software reports it on the server-recording day. Distinct historical receipt-time support is outside MVP; client backdating is not permitted.
- Saved payments cannot be edited, deleted, reversed, or retrospectively corrected, even on nonterminal invoices. Invoice deletion is prohibited on frontend/backend; payment invoice FKs use ON DELETE RESTRICT (R49–R50).

## Final waiver and metadata

- Waive the whole invoice's residual collectible balance, not individual invoice lines. A partial waiver that leaves the invoice PARTIALLY_PAID is outside this MVP model (R51, R55).
- WAIVED requires a nonempty free-text reason. UI reason presets may insert text. Ordinary PAID settlement needs no reason. Waiver/write-off share WAIVED without a WRITE_OFF enum or structured reason category (R52).
- Use settled_at and settled_by for final PAID or WAIVED closure; waiver_reason is mandatory on WAIVED and null on PAID. Nonterminal combinations and final enforcement SQL remain audited (R53).
- Keep metadata on the invoice; no separate invoice_waivers event table or redundant is_waived boolean is needed for this one-final-waiver model (R51).
- Discount workflow: issue standard prices, record actual money, then admin forgives the residual in settlement. The result is WAIVED; lines do not change. Reason examples mentioning an error do not create a retroactive invoice/clinical correction feature (R56).

## Derived money and reports

~~~text
billed = SUM(immutable issued line amounts)
received = SUM(positive payment amounts)
if final status is WAIVED:
    forgiven = billed - received
    collectible_outstanding = 0
else:
    forgiven = 0
    collectible_outstanding = billed - received
~~~

No independently authoritative header total, waived_amount, or outstanding_amount is selected. Views/functions and concurrency checks must preserve this arithmetic without multiplying line/payment totals through an unsafe join (R54, R66).

Daily collections sum actual dated positive payment rows, including receipts on still-PARTIALLY_PAID and later-WAIVED invoices. Do not equate cash with billed totals, PAID-only invoices, or forgiven amounts. Use the server-recording date limitation above, with exact local cutoff/time-zone implementation audited (R57).

No separately numbered/printed/sent patient payment receipt artifact is required. Confirm saved amount, methods, state, and balance only after backend-confirmed success; failed/unknown outcomes cannot claim success. Preserve the form for manual retry after confirmed rollback; lost responses require the separate idempotency audit (R58–R61).

## Open audit and future boundaries

Exact DDL, RLS, recorder/ownership integration, transition enforcement, RPC/API payloads, confirmation/cancel/error state, concurrency, payment indexes, queries, and migration remain unfinished (A06–A09, A12). Suggested invoice_id/received_at indexes are not approved merely by being suggested. Key/hash/invoice idempotency remains provisional. No refund, reversal, saved-payment correction, debt automation, or cross-invoice allocation is introduced.

## Evidence

Current rules and source references: [reconciliation register](reconciliation-2026-10-27.md#decision-register). Original versions are in [history/](history/); their wording does not override the current rules above.

Related: [checkout](checkout-and-invoice-lines.md), [packages and activation](packages.md), [examination charges](examination-fees.md).

Current checkpoint: [consolidated seven-answer history log](history/clinic_module2_seven_review_answers_2026-10-27.md). All seven review questions are settled; implementation and parked audits remain unfinished.
