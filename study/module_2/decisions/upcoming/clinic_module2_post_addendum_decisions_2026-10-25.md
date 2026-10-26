# Clinic Module 2 — New Decisions and Open Questions Since the Last Addendum

**Recorded:** 25 October 2026 (Asia/Kolkata)  
**Type:** Standalone incremental decision log; **new file**, not an overwrite.  
**Cutoff:** Begins **after** the creation of `clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md` and ends with the user's request for this file.  
**Read together with:** `clinic_checkout_billing_decision_log_2026-10-18.md` and `clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md`. This addendum intentionally **does not reprint** all prior decisions.

## Status key

- **LOCKED:** User explicitly approved the requirement or choice in this conversation.
- **CONFIRMED ELSEWHERE:** User said the decision had already been settled in another conversation; this record captures the user's confirmation, not an independent review of the other conversation.
- **OPEN:** No final answer; requires design/approval.
- **PARKED:** User explicitly deferred further discussion.
- **PROPOSED / NOT LOCKED:** Assistant suggestion or example, not approval.
- **CORRECTION:** Replaces an earlier inaccurate assertion or suggestion.

---

## A. Decisions explicitly locked after the last addendum

### A1. Waiver versus loss/write-off: one operational outcome, mandatory explanation — LOCKED

The user decided **not to create different invoice statuses or structured database classifications** for commercial waivers versus losses/write-offs. Both use the existing **`WAIVED`** settlement classification, with the intent explained by a reason. Example intentions may include owner-granted concession, uncollectible debt, and overcharge correction. Those are **textual explanations**, not separate enum values or accounting transaction types for this MVP.

- A waiver/write-off **must have a nonempty reason/note**; the user explicitly confirmed **mandatory**.
- The backend stores the reason as **free text**, not a fixed enum of reasons.
- The frontend may show **pre-filled suggested reasons**. Selecting one inserts/submits its text, while allowing the operator to enter other text.
- No dedicated `WRITE_OFF` invoice status or structured reason-category enum was approved.
- The user emphasized tracking how much money is actually received; waiver/write-off explanations remain traceable but are not counted as payment.
- A waiver must not create a fictitious payment record for the waived amount (previously settled; reaffirmed by this design).

**Precision:** The user confirmed that invoices with a **partial payment followed by waiver of the balance** are also classified as `WAIVED`, and said this had been settled elsewhere. That is a confirmed status outcome; do not infer other unspoken schema details.

### A2. No extra invoice waiver flag — LOCKED

The user explicitly accepted dropping `invoices.is_waived` (or an equivalent boolean). The distinct `WAIVED` invoice status is sufficient for the intended classification, together with the existing waiver metadata/reason design. Do **not** reintroduce an extra boolean without a new reason and user approval.

### A3. Payment methods: only CASH and UPI as underlying values — LOCKED

The assistant originally floated `cash`, `UPI`, `card`, and `bank transfer`. The user rejected `card`, then clarified the clinic accepts **only cash and UPI** for this flow.

- Backend payment-method enum: **cash**, **UPI** (exact SQL enum naming/casing to be chosen at implementation time).
- Frontend payment choice: **Cash / UPI / Split**.
- **Split is a UI mode, not a third backend payment method.** It represents cash and UPI payment components, with their actual methods persisted separately.
- Bank transfer and card are **not included** merely because the assistant suggested them.
- The user confirmed that hybrid/split payments had already been approved in another chat; this conversation clarified the exact cash/UPI selection.

### A4. Split payment inputs for both full and partial payment — LOCKED

For Split, the receptionist **enters both cash and UPI amounts explicitly**. The software sums the amounts and calculates the unpaid remainder. It must not assume a split payment means the invoice has been paid in full.

Example: Invoice ₹500, cash ₹100, UPI ₹150: collected ₹250; remaining ₹250 Due (not automatically Paid). This case was specifically used to reject the shortcut of auto-filling the second method amount from the full invoice total.

**Implementation note, not separately agreed as schema:** The natural payments-table mapping is separate payment entries for the actual cash and UPI components. Decisions about omitting zero-valued components, inserting both rows atomically, and exact UI validation remain implementation work.

### A5. Payment cannot exceed the outstanding amount — LOCKED

The user confirmed that an entry **must not exceed the current Due/outstanding balance**. Zero is not a real collected payment; earlier workflows may allow entering ₹0 to choose a Due/waiver outcome, but this must not be represented as a real money-received record.

**Engineering consequence for later implementation:** Enforce remaining-balance validation on the backend under concurrency, not solely by a frontend numeric input maximum. The precise SQL locking/serialization strategy is **OPEN**.

### A6. Server-assigned payment timestamp; no backdating in MVP — LOCKED

- The user selected **server time** for payment receipt timestamp creation and saving.
- They expressly rejected manually backdating/entering yesterday's receipt date and confirmed **backdating is not enabled** in this MVP.
- Distinguish `received_at` (when payment is recorded/received according to the current MVP policy) from invoice `created_at`, invoice settlement `paid_at`, and any technical record `created_at` if needed.

**Potential mismatch to revisit only if required:** An older manual staff-to-admin handoff example allows the admin to record money physically received earlier. With no backdating and server-stamped `received_at`, reporting reflects the time of entry, not necessarily the earlier physical handover. No exception or alternative rule was approved here. Do not silently backdate.

### A7. Derived outstanding balance, not stored — LOCKED

When asked whether to store the outstanding amount or calculate it, the user explicitly selected **always calculate**.

For the simple model currently discussed:

`outstanding = original billed amount − sum(real payments) − waiver/adjustment amount`

This is a **derived balance**; do not introduce a mutable `outstanding_amount` column as authoritative state without reopening the decision. Exact source-of-truth waiver amount and any minimum-zero checks remain schema work.

### A8. Treatment/invoice creation versus payment settlement: separate transactions — CONFIRMED ELSEWHERE / CORRECTION

The assistant proposed placing **treatment saving, invoice/lines, split payment records, and final status all in one transaction**. The user **explicitly rejected that** and corrected it:

1. Treatment and invoice/line items are saved in the **initial checkout/billing transaction**.
2. Payment confirmation/recording is a **second, separate transaction**.

The user stated this transaction boundary had already been established in another chat and directed us to mark it as settled, not revisit it. **Do not suggest the entire checkout plus payment is one atomic transaction again.** Atomicity within the initial billing transaction, and atomic payment-row-plus-status updates within payment settlement, are conceptually distinct.

### A9. No distinct loss/write-off mechanism for MVP — LOCKED

The earlier separate OPEN-05, "waiver versus write-off," is closed: use the same `WAIVED` classification and collect mandatory free-text reasons. There is no separate specialized loss status or reason enum. This does not mean a report should count written-off amounts as cash.

---

## B. Questions discussed and disposition

| Item | Discussed question | Disposition |
|---|---|---|
| Q1 | Do waiver and write-off need separate statuses or backend reason enums? | **CLOSED** — one `WAIVED` outcome, mandatory free-text reason; UI presets allowed. |
| Q2 | Must waiver text be mandatory? | **CLOSED** — yes. |
| Q3 | Should split payment be supported (cash + UPI)? | **CONFIRMED ELSEWHERE**, restated here. |
| Q4 | May a payment exceed remaining Due? | **CLOSED** — no. |
| Q5 | Payment method free text or enum? | **CLOSED** — enum. |
| Q6 | Payment timestamp server-side or operator-entered? | **CLOSED** — server assigned. |
| Q7 | Can staff backdate payments? | **CLOSED** — no, MVP. |
| Q8 | Are cash, UPI, card, and bank transfer all offered? | **CLOSED** — cash and UPI only. |
| Q9 | Is Split a persisted payment method? | **CLOSED** — no, UI grouping choice. |
| Q10 | Should both split amounts be entered or one auto-calculated? | **CLOSED** — enter both, including on partial payments. |
| Q11 | Should an invoice have an `is_waived` flag in addition to `WAIVED` status? | **CLOSED** — no. |
| Q12 | Does checkout commit payments alongside the initial treatment and invoice? | **CLOSED / CORRECTED** — no, separate payment transaction. |
| Q13 | Should outstanding balance be stored or calculated? | **CLOSED** — calculated. |
| Q14 | Work through idempotency keys and uncertain retries? | **PARKED** — user said later. |
| Q15 | Can a single payment record be allocated across multiple invoices? | **NOT ANSWERED** — assistant asked immediately before the request for this addendum. No approval to prohibit it was given. Cross-invoice allocation was separately parked earlier. |

---

## C. Remaining OPEN/PARKED implementation backlog

### OPEN-02: Complete database schema and constraints — PARTLY OPEN

Business decisions above are locked, but the exact migrations are not. Remaining engineering tasks include:

- Define final tables, column names/types, FK behavior, unique constraints, indexes, and per-column nullability for invoices, invoice items, payments, and waiver metadata.
- Choose the representation for invoice billed amount (line-item sum/source of truth), waiver adjustment amount, derived collected total and outstanding formula; handle zero-price cases.
- Decide exact statuses and SQL enum names/constraints that align with existing decisions (`PENDING`, `DUE`, `PAID`, `WAIVED`), including precise treatment of a partially collected but settled-as-waived invoice.
- Represent each split payment's component safely; prevent accidental zero-value payment rows.
- Apply authorization rules for initial receptionist checkout versus admin control of older Due invoices; enforce Paid/WAIVED finality as previously agreed.
- Atomicity and concurrency for each **separate** billing and payment transaction.
- Clarify impact of server timestamps on delayed admin entry for cash physically received earlier if that scenario needs accurate actual-receipt-day reporting. This is an observation, **not** a reopened decision.

### OPEN-03: Idempotency and retry mechanics — PARKED

Requirement remains locked (disable repeat submission while awaiting response; backend idempotency/duplicate protection), but concrete implementation was **explicitly deferred**. Later work: idempotency keys, transaction/request identity, conflict handling, client retry UX, concurrent requests, and duplicate source charges.

### OPEN-04: Payment implementation and UI — PARTLY OPEN

Now settled: accepted methods, split choice and two manual amounts, partial payment arithmetic, server timestamp, no backdating, no overpayment, admin-only changes on older Due invoices, separate billing/payment commits.

Still engineering work: details of form state and validation; submit/confirm choices and cancel behavior; exact Postgres functions/API and role enforcement; status transition guarantees; errors/retries; ensuring split entries and status are updated atomically within the *payment* transaction; test cases.

### OPEN: Cross-invoice payment allocation — PARKED / NOT NEWLY LOCKED

Previously, the accepted MVP operational workaround for money received toward an older invoice was human communication to admin, who records against that older invoice. **This is not approval for a general-purpose receipt-allocation system.** The assistant's final question, "one payment tied to one invoice, no split across invoices?", **was not answered**. Keep the general allocation question parked unless asked.

### OPEN: Visit/treatment/invoice cardinality — PARKED

Not discussed further here; preserve in the earlier master log, do not solve as part of this payment addendum.

### Other earlier deferred topics — NO NEW DECISION

Invoice corrections/voiding, package/examination/catalog questions, and other clinical architecture remain governed by their own logs and later decisions. This file makes **no new ruling** about them.

---

## D. Explicit corrections to the conversation record

1. **Incorrect:** All checkout records including payments should be in one initial atomic transaction.  
   **Correct:** Initial treatment+invoice billing and subsequent payment settlement are **two distinct transactions**.
2. **Incorrect:** Card or bank transfer are accepted just because they were proposed.  
   **Correct:** Cash and UPI only; Split is a frontend entry mode.
3. **Incorrect:** Any split implies full settlement or one amount can be computed from the full bill.  
   **Correct:** Operator enters both amounts; partial Split can leave a Due balance.
4. **Incorrect:** An extra `is_waived` boolean is needed.  
   **Correct:** Explicitly dropped; use `WAIVED` status and waiver metadata.
5. **Incorrect:** A separate write-off status/enum is required.  
   **Correct:** Waiver/write-off share `WAIVED`; mandatory free-text reason with frontend presets.
6. **Incorrect:** All open schema and implementation issues are settled.  
   **Correct:** Exact schema, authorization, idempotency (parked), and payment UI mechanics still need implementation decisions.
7. **Incorrect:** The proposed one-payment-one-invoice exclusivity was agreed to.  
   **Correct:** It was **only asked**, not answered, at the time this file was requested.

## E. Next-session handoff

Continue **OPEN-02 (database constraints/schema)** or **OPEN-04 (payment processing implementation)**, one small issue at a time. Do not reopen locked decisions or parked idempotency unless requested. The immediately preceding unanswered question concerned whether each payment must be associated with exactly one invoice and whether cross-invoice allocation is excluded, but the user had previously parked the broader cross-invoice allocation design. **Ask before treating this as the next required decision.**

**Document boundary:** This is an incremental addendum **only for the period since the last generated addendum**. It does not supersede or replace the full historic decision record.
