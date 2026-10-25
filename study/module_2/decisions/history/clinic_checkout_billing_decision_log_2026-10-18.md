# Clinic Checkout, Invoicing & Payment — Consolidated Decision Log

**Version:** 2.0 — comprehensive conversation checkpoint  
**As of:** 18 October 2026 (Asia/Kolkata)  
**Scope:** All decisions, examples, architectural discussions, reversals, deferred ideas, and outstanding questions from the provided checkout-design excerpt and the subsequent conversation.  
**Purpose:** A durable handoff document for resuming the design discussion. This is **not** a database migration, implementation specification, or claim that code has been updated.

> **Source boundary:** This log is reconstructed from the excerpt and conversation only. The assistant **did not inspect GitHub** during the later discussion, because the user explicitly asked not to do so at that point. Assertions about existing frontend/backend code are therefore *claims in the earlier excerpt*, not fresh repository verification. The user also explicitly chose **not to decide visit/treatment/invoice cardinality yet**.

---

## 0. How to read the decision status

| Label | Meaning |
|---|---|
| **LOCKED** | The user explicitly chose/accepted this rule or restated it decisively. Treat as the current MVP requirement unless the user changes it. |
| **ESTABLISHED CONCEPT** | The participants converged on the domain model or distinction; exact implementation is not locked. |
| **PROPOSED / NOT LOCKED** | A concrete suggestion or example was discussed, but no explicit final decision was made. |
| **OPEN** | Requires a user decision or a precise rule before implementation. |
| **DEFERRED** | Intentionally postponed or excluded from the MVP conversation. |
| **SUPERSEDED** | Earlier wording/interpretation was changed by a later decision. Never implement it as the current rule. |
| **OPERATIONAL INPUT** | A stated observation about clinic practice, not a software design rule. |

**Guardrail:** Do not convert an assistant recommendation into an approved requirement merely because it sounded sensible. Where the transcript is ambiguous, this document says so.

---

## 1. Executive snapshot: where the conversation ended

1. **LOCKED —** Receptionists normally enter treatment information; clinicians also have access to make entries. The ordinary flow records treatment, generates an invoice, then handles payment separately.
2. **LOCKED —** An invoice may be created even though collection, waiver approval, or a dispute is unresolved. The initial state is **Pending**.
3. **LOCKED —** Three invoice/payment-resolution statuses currently exist: **Pending**, **Due**, **Paid**. **Due** is now also reachable at **initial checkout by a receptionist who records a partial payment**; it no longer necessarily means the owner has personally reviewed the invoice.
4. **LOCKED —** At initial checkout the receptionist may (a) confirm full payment with a payment method, (b) choose **Pay Later** and leave the invoice Pending, or (c) **record a partial payment and leave the balance Due**. The third option was the **most recent explicit decision**.
5. **LOCKED —** Waivers are **admin/owner-only**. Receptionists can create an outstanding balance; they cannot waive it.
6. **LOCKED —** After the receptionist selects Pay Later, subsequent processing of that old Pending invoice is reserved for the admin. An **existing Due invoice** is likewise administered by the admin, not retroactively edited by the receptionist.
7. **LOCKED —** An admin can settle a Due invoice by entering an additional amount from **₹0 to the outstanding balance**, then marking it Paid; any remaining unpaid portion is treated as waived. The user specifically motivated this with an old uncollectible balance being written off.
8. **LOCKED —** Once an invoice is Paid, it is **fully locked for the MVP**, including to the admin. No correction flow/audit log for paid invoices is required now; backend must enforce immutability.
9. **LOCKED —** Actual money received and waivers are different quantities. The original billed total is not the same as collected cash or the outstanding balance.
10. **ESTABLISHED CONCEPT / STRONG DESIGN DIRECTION —** Use one common **payments** table across invoices, with **one row per actual payment** and **zero or more payment rows per invoice**. A Pending invoice can have zero rows; a partially collected Due invoice has one or more; a fully waived invoice might have zero. The user requested to see how this schema lands but did **not unambiguously approve a complete DDL**.
11. **LOCKED —** Separate invoice creation time, final-settlement time, and payment-receipt time. Daily **cash collection** uses **actual payment receipt dates**, not invoice creation date or generic `updated_at`.
12. **LOCKED —** For the existing-Due-invoice / new-visit scenario, staff will **communicate manually** to admin (call/text/in person) about money applicable to the older debt. Do not build automated notifications or an unallocated-payment workflow for the MVP.
13. **OPEN —** Exactly how repeated *partial* repayments on an existing Due invoice work, and whether the admin has a **Record Payment, Keep Due** action.
14. **OPEN —** Whether recording treatment and creating the invoice **must be one atomic database transaction**; the user had not made the final call.
15. **OPEN / INTENTIONALLY PARKED —** Whether one visit can include several treatments and how visits/treatments map to invoice lines and invoices. Do **not** reopen this by default when the user returns.
16. **OPEN —** A proposed invoice-level **waiver flag/tag** and its exact meaning, storage, and rules. The user explicitly asked to remember this for later discussion.

---

## 2. Agreed domain language and arithmetic

### 2.1 Three distinct layers — ESTABLISHED CONCEPT

- **Clinical/treatment records:** What happened clinically (complaints, treatments, procedures, possibly multiple clinical relationships). Not identical to what is charged.
- **Invoice + invoice lines:** What the clinic billed. One invoice may have multiple billable line items. Lines describe the charged components; not every procedure necessarily merits a separate charge.
- **Payments + waivers:** How the bill was settled or reduced. Payments are **money actually received**; a waiver is **an amount deliberately not collected**, not a payment.

These distinctions coexist. They are not competing alternatives.

### 2.2 Financial quantities — LOCKED concept; persistence strategy OPEN

For an invoice with no refunds or credit notes:

```text
Original billed amount = SUM(invoice line-item charges)
Collected amount       = SUM(actual payments recorded against the invoice)
Waived amount          = SUM(approved waiver/write-off adjustments)
Outstanding amount     = Original billed - Collected - Waived
```

The **original billed amount** should not silently be rewritten to match collected cash. Distinguish what was invoiced from what was collected and what was waived.

**Implementation not locked:** Whether `billed_amount`, `collected_amount`, `waived_amount`, `outstanding_amount` are stored, derived, cached, or reconciled is not yet decided. Checks such as preventing negative outstanding balances or duplicate collection should be enforced at the backend regardless of storage choice.

### 2.3 Cardinality — ESTABLISHED CONCEPT

Cardinality means the allowed number of relationships between records (one-to-one, one-to-many, many-to-many); *optionality* addresses whether zero linked rows are permitted.

- **Invoice → payments:** **One invoice to zero or many payments** (concept accepted).
- **Invoice → invoice lines:** Proposed **one invoice to one or many lines** for itemized charges; exact minimum and connection to treatments not ratified.
- **Visit → treatment(s) → invoice(s):** **OPEN**, deliberately postponed. Do not assume that a visit maps to exactly one treatment or that each treatment receives its own invoice.
- Invoice lines describe **composition** of what was charged; payments describe **settlement**. The two child collections can both reference the same invoice.

### 2.4 The evolving meaning of statuses — LOCKED as currently understood

| Status | Current business meaning | Typical entry path | Note |
|---|---|---|---|
| **Pending** | Invoice exists, but the payment decision/processing was deferred; nothing has been confirmed as collected in this workflow. | Invoice creation + Pay Later. | **Not** simply a synonym for an owed balance. Admin can later review. |
| **Due** | An outstanding balance is explicitly recognized as still collectible. | Admin reviews/records less than full amount **or** receptionist records a partial payment at initial checkout. | **SUPERSEDES** earlier definition “owner has personally reviewed.” |
| **Paid** | Fully settled; no outstanding balance, through real payments and/or approved admin waiver. | Receptionist confirms a fully paid initial checkout; admin settles a Pending/Due invoice. | Triggers permanent MVP lock. A Paid invoice can include a waiver, so Paid **does not mean full billed amount was collected**. |

**Open nuance:** If a receptionist initially records ₹0 paid but expressly confirms full balance Due instead of clicking Pay Later, is that a permissible initial-checkout path? The user explicitly approved *partial paid → Due* and admin *₹0 → Due*, but did not clearly approve receptionist *₹0 → Due*. Do not assume.

---

## 3. Locked decisions with detailed rationale

### DEC-01 — Normal clinical entry ownership — LOCKED / OPERATIONAL INPUT

- The **receptionist** is the usual person recording treatment information before checkout.
- The **clinician** also has access to create treatment entries.
- This is role access and operational context, **not** proof that clinical charting and billing occur at separate times.
- The owner/admin participates in billing decisions, especially waivers and unresolved debts.

**Implication:** Do not justify two database commits merely by pointing out that clinicians could make entries. Establish an actual delayed handoff if one exists.

### DEC-02 — Create the treatment record and invoice in the normal checkout flow — LOCKED at the workflow level

- The receptionist submits completed treatment information; the backend determines charges and creates a bill.
- The UI can present a confirmation that the treatment was saved and the invoice created, then offer a payment action.
- It is legitimate to create the invoice while a waiver request, payment, or package-coverage dispute is unresolved.
- Payment is a **separate subsequent user action/backend request**.

**Not yet locked:** Whether the treatment write and invoice generation are wrapped in **one all-or-nothing database transaction**. The original excerpt argued strongly for this; no explicit final user acceptance occurred.

### DEC-03 — Initial invoice state is Pending — LOCKED

- New invoices are created with **Pending** payment status until checkout/payment processing chooses an outcome.
- This is a normal workflow state, not necessarily an exceptional database error.
- A Pending invoice can legitimately exist even if the patient has departed.

### DEC-04 — Receptionist has an immediate full-payment path — LOCKED

- After invoice creation, receptionist can confirm that the billed amount has been paid.
- They record the **payment method** (cash, UPI, card, etc.).
- Backend records the actual receipt and finalizes invoice **Paid**.
- Once Paid, the invoice becomes immutable under DEC-10.

**Open form details:** Confirmation UX, whether receipt time is captured automatically or entered, permitted payment methods, multi-method tender at initial checkout, and retry protection are not fully specified.

### DEC-05 — Receptionist may select Pay Later — LOCKED

- The receptionist can close the payment prompt by selecting **Pay Later**.
- The invoice remains **Pending**.
- The receptionist has **no later edit authority** over that old Pending invoice.
- The admin/owner is responsible for future resolution; the application is not required to automatically escalate or notify them.

**Clarification:** Closing the modal is not the same as cancelling or deleting the invoice.

### DEC-06 — Receptionist may take a *partial* payment during the initial checkout and mark the rest Due — LOCKED (latest decision)

This was the final newly accepted requirement:

> Patient owes ₹500 now, can pay ₹300 now, promises ₹200 later. Receptionist should be able to record the ₹300 and explicitly leave ₹200 **Due**, without any authority to waive it.

Required behavior:

- Capture the actual initial payment amount and method.
- Record an actual payment (not a fake waiver).
- Set/leave the remaining amount as **Due**.
- No waiver option for receptionist.
- The user responded **“Yes, with that”** to the precise proposed rule that a receptionist can mark a **new invoice** Due after taking partial payment.

This **changes** the earlier UI assumption that receptionist can only **Confirm full payment** or **Pay Later**. Initial checkout now requires a third valid outcome.

**Permission scope:** This new power applies to the **invoice being checked out now**. It **does not** grant receptionist permission to alter an old Due or old Pending invoice.

**Open:** Whether the new partial-payment UI should have a single “Record partial payment / Mark Due” action or some other layout, and whether ₹0 is allowed on this route.

### DEC-07 — Old Pending/Due invoices are admin-controlled — LOCKED

- If reception leaves an invoice Pending, the admin manages subsequent processing.
- If there is an older Due invoice, the admin handles subsequent payments/waivers for that invoice.
- Reception can settle the **new invoice** from a later session, but cannot personally allocate money to the **old** invoice via an unauthorized old-bill edit.
- Admin is the responsible role for follow-up; no claim that admin checks every outstanding invoice daily was established.

### DEC-08 — Owner/admin can settle a Due invoice with waiver/write-off — LOCKED

For an old invoice in Due state:

- Owner opens its resolution form (same general idea as the Pending invoice processing form).
- Enter an **additional received amount**, default **₹0**.
- Allowed range: **₹0 through the remaining outstanding amount**.
- Available action explicitly accepted: **Mark Paid**; or **Cancel** (make no changes).
- If the additional collected amount is less than the outstanding balance, the rest is considered **waived** when admin presses Mark Paid.
- Mark Paid changes status to **Paid** and invokes the permanent lock.

Concrete example:

```text
Original invoice:       ₹500
Already collected:      ₹200
Currently Due:          ₹300
Owner enters now:         ₹0
Owner presses Mark Paid
=> Additional payment:    ₹0 (NO payment transaction for zero)
=> Amount waived:       ₹300
=> Outstanding:           ₹0
=> Final status:        Paid (locked)
```

The user proposed this specifically to clear a debt that is no longer worth collecting (a business loss/write-off). The system must distinguish that waiver/write-off from cash.

**Open:** Whether “waiver” and “bad-debt write-off” should be separate accounting categories for reporting; the user uses the word *loss* for this specific motivation but did not demand separate categories.

### DEC-09 — Waiver authority is admin-only — LOCKED

- Receptionists do not approve or create waivers.
- The owner/admin may waive outstanding amounts when settling an invoice.
- A waiver after an invoice is already **Paid** was rejected.
- The older broad phrase **“waiver only before any payment”** has been narrowed/superseded: the user explicitly approved waiving the remaining balance of an invoice that is still **Due** even if some money was collected earlier.
- Therefore the operative restriction is **no waiver/change after final Paid lock**, not “no waiver after any partial money was received.”

**Open:** Whether an owner can record a waiver while leaving part of the bill Due, or can waive only as part of final settlement; current accepted Due workflow is Mark Paid + waive the residual.

### DEC-10 — Paid is final/immutable for this MVP — LOCKED

- Once status becomes **Paid**, staff **including admin/owner** cannot change the bill, payment amount, payment method, or waiver.
- Enforce at the **backend/database/application authorization layer**, not only by disabling UI buttons.
- The user explicitly accepted the drawback that a mistakenly entered Cash payment might actually have been UPI and could not be corrected in this MVP.
- Do **not** implement a paid-invoice admin correction/audit trail as part of this decision.

**Trade-off:** Reduced scope and stricter process at the cost of permanently preserving some human input errors. Prevent accidental finalization with sensible confirmation UX (exact UX not decided).

**Open technical definition:** Precise database tables/events covered by the lock, and how to handle legally necessary voids, refunds or data corrections in the future. Do not quietly bypass this rule for convenience.

### DEC-11 — Record actual receipts, do not misrepresent waivers as payments — LOCKED

- Invoice has an original charge, which is **not** replaced by amount actually collected.
- Collected amount, waived amount, and due amount must be separately discernible.
- An invoice billed at ₹500, paid ₹400, waived ₹100 is **Paid** but represents **₹400 cash inflow**, not ₹500.
- Full ₹500 waiver with zero money can be represented as settled without inventing a ₹500 payment (the general concept follows from admin Due settlement at ₹0). The exact handling of a **new Pending invoice fully waived without any receipt** remains a UX/schema edge case to verify.

### DEC-12 — Every real receipt belongs in the payment model — ESTABLISHED CONCEPT / direction strongly accepted

The discussion converged on:

- One common **`payments`** table, rather than separate ad-hoc payment columns for Pending or Due invoices.
- All actual receipts go there: immediate reception payment, initial receptionist partial payment, admin payment later, and repayments of older Due balances.
- **One row per actual payment**; same invoice can have zero, one, or multiple payment rows.
- No payment row when **₹0** was collected; do not create payment rows for waivers.
- Each payment should capture invoice reference, amount, method, actual received time, system recorded time, and actor who recorded it.
- A payment insertion and corresponding invoice-state/balance change should be **atomic**—this was proposed as the clean design and explained, but the user did **not** give a fresh unambiguous “yes, lock that transaction constraint” after the explanation. Treat atomic processing as **strong proposal / unresolved ratification**, not implementation already approved.

**Decision-strength caution:** The user asked how the clean payments-table design works and requested a schema walkthrough. This indicates directional agreement, but the **complete schema, payment event lifecycle, and treatment/invoice/payment atomicity** are not all finalized. Do not claim exact column types, DB constraints, or a deployed migration are locked.

### DEC-13 — Time dimensions and cash reports — LOCKED

Distinguish:

1. **Invoice `created_at`**: date/time the bill was generated.
2. **Invoice `paid_at`**: date/time the bill reached final **Paid** state.
3. **Payment `received_at`**: date/time money actually arrived.
4. **Payment `recorded_at`**: date/time the system recorded it (**proposed** for late admin entry; recommended and included in example DDL, but exact schema still not locked).

Do **not** use `updated_at`/`modified_at` to calculate cash collections: later edits could shift historical collection dates.

**Example:** Bill ₹500 issued Monday. ₹400 genuinely received Wednesday and ₹100 waived. Monday is invoice creation; Wednesday is collection of **₹400**. A financial report for Wednesday based on payment `received_at` reports ₹400. A report by original invoice date answers a different question.

**Note:** Recording `received_at` accurately is operationally important when money is physically taken by reception and entered later by admin.

### DEC-14 — Reports need not be separate persistent tables — ESTABLISHED CONCEPT

- Views and queries can present up-to-date invoice/payment statuses.
- Cash collection views should report **actual receipts** by receipt date.
- Pending/Due invoices should remain visible in relevant outstanding/unresolved-work reports rather than being hidden from **all** reporting.
- The user considered excluding Pending from certain revenue reports, but no universal rule to ignore Pending/Due invoices in every metric was locked.

**Open:** Which accounting/revenue reports group by service date, invoice date, payment date, and waiver/write-off date; exact historical reporting definitions.

### DEC-15 — Manual staff communication, no software notification/unallocated-money workflow in MVP — LOCKED

Concrete real-world scenario raised by user:

- Old invoice: ₹500 originally, ₹200 collected, ₹300 Due.
- Patient returns for a new treatment billed ₹500.
- Patient hands over ₹600 total: ₹500 for today's invoice and **₹100 intended against old debt**.
- Receptionist settles today's new ₹500 invoice.
- **Only admin** can subsequently record the additional ₹100 against the **older Due invoice**.
- Staff should **call, text, or directly inform admin** of the extra cash and which old debt it relates to.
- The app will **not** implement automated notifications, a general unallocated-payment wallet, or cross-invoice payment allocation at this stage.

**Deliberately accepted risk:** Before the admin records that extra ₹100, the physical money held and payments recorded in the app may temporarily differ. Communication and timely entry are an operational obligation. The software should not falsely claim that difference is reconciled.

### DEC-16 — Payment reason/waiver note for traceability — ESTABLISHED DIRECTION; details OPEN

- User wanted a note/prompt explaining why a billed amount was waived or otherwise settled for less than face value.
- Example: billed ₹500, actual collected ₹400, owner waived ₹100, explanation recorded.
- An explanation attached to a settlement is **not equivalent to a comprehensive historical audit log**.
- The user declined adding a general-purpose editable-paid-record audit trail in the MVP.

**Open:** Mandatory or optional note? Is note attached to invoice, waiver event, decision, or all? Required for every waiver, only exceptional waivers, or bad-debt write-offs? Do we need a reason code?

### DEC-17 — Owner review workflow is allowed to be imperfect rather than fully automated — LOCKED operational choice

- Owner/admin has responsibility to find and resolve outstanding invoices.
- If admin does not review Pending for an extended period, the user views that as an operational problem, not grounds for building an automatic notification/escalation system right now.
- Reconsider only with observed clinic failures, not speculation.

---

## 4. Current workflow illustrated with scenarios

The scenarios below demonstrate **agreed behavior where possible**. Any unapproved extension is marked.

### Scenario A — Full initial payment (LOCKED)

```text
Treatment entered by receptionist or clinician
  -> backend saves treatment and creates invoice for ₹500
  -> invoice begins Pending
Receptionist takes ₹500 and selects Confirm Payment
  -> records ₹500 with payment method
  -> status Paid
  -> invoice is permanently locked
```

**Accounting:** billed ₹500; paid ₹500; waived ₹0; due ₹0.

### Scenario B — No initial payment; Pay Later (LOCKED)

```text
Invoice for ₹500 created (Pending)
Receptionist selects Pay Later
  -> modal closes; invoice remains Pending
  -> receptionist cannot later edit this Pending invoice
Owner/admin is responsible for future resolution
```

**Accounting:** billed ₹500; collected ₹0; waived ₹0; outstanding ₹500. Still **Pending** until explicitly processed.

### Scenario C — Initial part-payment by receptionist (LOCKED, newest)

```text
Invoice for ₹500 created (Pending)
Patient pays ₹300 immediately; promises ₹200 later
Receptionist records ₹300 plus payment method
  -> ₹300 recorded as actual payment
  -> ₹200 explicitly left Due
  -> invoice status Due
Receptionist CANNOT waive ₹200
```

**Accounting:** billed ₹500; collected ₹300; waived ₹0; due ₹200.

**Significant revision:** `Due` **does not mean** “only admin has touched the invoice.” Receptionist may create Due **at the original checkout**.

### Scenario D — Admin records a part payment but keeps remainder Due (OPEN)

```text
Existing Due invoice: billed ₹500; collected ₹200; outstanding ₹300
Patient pays another ₹100
Desired accounting if supported: collected total ₹300; outstanding ₹200; status Due
```

This is precisely the **Record Payment, Keep Due** feature posed earlier. The user discussed its rarity, gave a realistic next-visit example, and asked about payment table implications; there was **no final explicit yes/no** on allowing the old Due invoice to remain Due after another partial receipt. Payments table makes it technically straightforward but does **not** automatically settle the business/UI decision.

### Scenario E — Admin settles old Due and waives remainder (LOCKED)

```text
Existing Due: billed ₹500; already paid ₹200; outstanding ₹300
Admin enters ₹100 additional payment and presses Mark Paid
  -> new ₹100 payment (if actually received)
  -> remaining ₹200 waived
  -> Paid / final lock
```

Also valid by explicit owner choice: enter **₹0**, press Mark Paid, waive entire ₹300 outstanding; zero is **not** a payment.

### Scenario F — Patient has a new visit and old debt (LOCKED for role boundary/manual handoff; repayment lifecycle OPEN)

```text
Old invoice: Due ₹300
New invoice: ₹500
Patient brings ₹600, intending ₹500 new + ₹100 old
Receptionist can settle NEW ₹500 invoice
Receptionist contacts admin manually about extra ₹100 for OLD debt
Admin must make any recording/change to OLD invoice
No automated notification/unallocated payment ledger in MVP
```

What admin can do when receiving **only part** of the old ₹300 while preserving ₹200 Due remains **OPEN** (Scenario D).

### Scenario G — Package coverage dispute (HYPOTHETICAL, not proof of need)

Patient says session is included in package and should cost ₹0; receptionist sees exhausted coverage; owner unavailable. The earlier excerpt used this as an example of an unresolved charge. User correctly observed that such a patient may refuse to pay while the claim is disputed. The agreed system can still create an invoice Pending, but **package entitlements, prices, and adjustment rules are not yet specified**. Do not treat this imaginary example as confirmed clinic behavior.

### Scenario H — Waiver request while owner is busy (HYPOTHETICAL)

Earlier discussion posited that clinical record may need saving before an owner decides a waiver. Clinic input later clarified: waiver requests are uncommon, typically go to owner, and the owner **usually decides immediately**. Therefore a long unresolved waiver process was not demonstrated as a routine need. This weakens—but does not mathematically disprove—the case for separate treatment-save commits.

---

## 5. Authorization matrix (current behavioral intent)

| Action | Receptionist | Clinician | Admin/owner | Status |
|---|---|---|---|---|
| Enter treatment | Yes, usual actor | Yes, has access | Not discussed | LOCKED for first two |
| Create normal treatment checkout / invoice | Receptionist normal workflow | Clinician treatment entry allowed; exact checkout authority unclear | Not specified | Partly OPEN |
| Confirm full payment during **new** checkout | Yes | Not specified | Yes in admin processing | Receptionist LOCKED |
| Select Pay Later on **new** checkout | Yes | Not specified | N/A | LOCKED |
| Record part-payment, leave **new** invoice Due | **Yes** | Not specified | Admin processing concept applies | **LOCKED (latest)** |
| Reopen old Pending invoice | **No** after Pay Later | Not specified | **Yes** | LOCKED |
| Change/collect against an **existing Due** invoice | **No** | Not specified | **Yes** | LOCKED |
| Grant waiver/write-off | **No** | No permission established | **Yes** | LOCKED |
| Edit an invoice once **Paid** | **No** | **No** | **No** | LOCKED MVP |
| Record additional partial payment on old Due **without closing** | **No** | Not specified | **OPEN: feature not decided** | OPEN |

**Enforcement:** The immutable Paid rule and role restrictions should be checked by backend authorization, not solely frontend affordances. Exact RBAC implementation is open.

---

## 6. Data model: what was discussed vs. what is actually approved

### 6.1 Conceptual entities

```text
PATIENT
   |
   +--- INVOICE (what was billed; created_at; status; paid_at)
             |
             +--- INVOICE_ITEM(s) (charged components)
             |
             +--- PAYMENT(s) (actual receipts; method; received_at; recorded_at; actor)
             |
             +--- WAIVER / ADJUSTMENT RECORD?   <-- OPEN: design

VISIT / TREATMENT / PROCEDURE RELATIONSHIPS    <-- OPEN: intentionally parked
```

- A conceptual one-to-many relationship is **not** equivalent to a signed-off SQL migration.
- Payment rows should be related to a specific invoice. The discussed first-pass design has `payments.invoice_id`.
- The later next-visit example presumes staff/admin identify which invoice an extra receipt is intended for.

### 6.2 Fields previously illustrated (PROPOSED, not approved schema)

**`invoices`** possible fields:

```text
id
patient_id
billed_amount_paise          -- or derive total from invoice items; unresolved
status                       -- pending | due | paid
created_at
paid_at
... waiver representation OPEN
```

**`payments`** possible fields:

```text
id
invoice_id                   -- foreign key -> invoices
amount_paise                 -- positive for actual receipts; never zero for waiver
payment_method               -- cash / UPI / card / etc.
received_at                  -- actual money receipt date/time
recorded_at                  -- database entry date/time
recorded_by                  -- staff user reference
```

The earlier demonstration used `BIGINT` monetary values in **paise**, `TIMESTAMPTZ`, and constraints for positive amounts. **Those specific types were illustrations, not ratified schema.** The assistant also used illustrative `patients`/`users` references without confirming live repo table names. Inspect actual migrations before writing DDL.

**`invoice_items`** possible fields (PROPOSED):

```text
id
invoice_id
charge_description / charge_source reference
amount
...
```

Do not finalize clinical source foreign keys, visit ID, and number of lines until the intentionally parked cardinality question is reopened.

**Waiver model:** Not chosen. Options could include an aggregate `waived_amount`, append-only adjustment records, reason/note fields, and/or a derived boolean. This is future discussion, not current approval.

### 6.3 Why a separate payments table was argued for

If a ₹500 invoice receives ₹200 Monday, ₹100 Wednesday, and ₹200 Friday, three payment records preserve each amount, date, method, and actor. A single mutable `paid_amount` on invoices loses that event-level history and makes date-specific cash reporting unreliable.

However, the user questioned MVP overhead and **has not finished the exact design decision for repeated partial receipts on an old Due invoice**. The shared table has broad conceptual support because every actual receipt should be represented consistently, regardless of invoice status.

### 6.4 Transaction boundaries — separate unresolved questions

**A. Treatment + invoice creation:** Should one checkout request run treatment writes, invoice creation, and invoice-line creation inside **one all-or-nothing database transaction**? **OPEN.** This was the original architectural debate.

**B. Payment recording + balance/status update:** Proposed that insertion of a payment and related settlement/status transition be **atomic**, so no payment is recorded without a matching valid invoice state. **Strong recommendation discussed; explicit user ratification still pending.**

**C. Retries and concurrency:** Atomicity does **not** ensure exactly-once behavior if response is lost or simultaneous requests race. Idempotency, unique request keys, and proper locking/constraints remain **OPEN technical design**, not optional problems solved automatically by a transaction.

---

## 7. Decisions superseded, clarified, or explicitly *not* made

| Earlier claim or framing | Current correction |
|---|---|
| “Due means admin has looked at invoice.” | **SUPERSEDED:** New-invoice Due can result from receptionist recording a partial payment during checkout. Due means a recognized collectible outstanding balance. |
| Receptionist only has **Pay Later** or **Confirm full payment**. | **SUPERSEDED:** Receptionist additionally can record a partial payment and mark remaining amount Due **during initial checkout**. |
| Waiver must occur **before any payment whatsoever**. | **SUPERSEDED:** Owner may waive the balance **after previous partial payment(s)** while invoice remains Due; only Paid is final. |
| A waived invoice showing Paid is identical to fully collected cash. | **REJECTED:** Waived sum and actual cash receipts are separate; Paid indicates settlement, not original billed cash collected. |
| Every Due invoice necessarily has at least one payment. | **INCORRECT:** Admin can confirm a full ₹500 balance Due with ₹0 collected; Due does not imply a payment row. |
| Every Paid invoice necessarily has a payment row. | **INCORRECT as universal rule:** A 100%-waived invoice can be settled with ₹0 receipt. |
| One request necessarily equals one database transaction. | **INCORRECT:** Backend may make multiple commits in one request; transaction boundaries must be explicit. |
| An invoice must wait for waiver/price dispute to be resolved. | **REJECTED:** Invoice can be created Pending before such issues are resolved. |
| A printer/PDF issue should force treatment resubmission. | **REJECTED conceptually:** Printing/receipts should operate on already saved invoice; exact PDF implementation not planned. |
| Multiple invoice lines are an alternative to multiple payments. | **INCORRECT:** Lines are charged components; payments are money received. They can each form one-to-many relationships off one invoice. |
| The assistant can choose combined checkout architecture immediately. | **NOT AUTHORIZED:** User insisted on being questioned about actual clinic needs and retaining final decision authority. |
| “Five questions pending” means exactly five settled categories. | **NOT RELIABLE:** Multiple original gaps were answered, split, combined, or added. Keep a live issue tracker, not a fictitious fixed count. |

---

## 8. Outstanding decisions — structured backlog for resuming

The order below groups closely related items and avoids confusing the number of questions. **Do not answer these on behalf of the user. Ask one question at a time when they return.**

### P0 — Needed to implement current billing/payment behavior correctly

**OPEN-01 — Can admin add partial payments to an existing Due invoice and still leave it Due?**

- We already know admin can **Mark Paid** and implicitly waive residual; Cancel makes no changes.
- Missing action: patient gives ₹100 against ₹300 Due; admin records ₹100 and leaves ₹200 Due without waiving.
- Realistic case specifically raised: patient pays extra toward old debt during a later therapy visit.
- Need yes/no, UI action, whether successive payments are unlimited, and constraints.
- Do **not** conflate this with the now-LOCKED receptionist ability to create Due at **initial checkout**.

**OPEN-02 — Ratify payments table and atomic payment write semantics.**

- Shared model heavily discussed: one positive payment row per receipt across all invoices; zero/many per invoice.
- Need explicit final agreement on whether that is MVP schema and whether inserting payment and changing invoice settlement/state must be atomic.
- Decide whether all payment records are immutable (especially while invoice Due) and how errors are handled **without violating locked Paid immutability**.

**OPEN-03 — Finalize new-checkout partial-payment rules and form.**

- LOCKED: receptionist can accept ₹300 on ₹500 and leave ₹200 Due, no waiver.
- Still decide: allowed payment amount range (must be positive? can be zero?), exact prompts/buttons, mandatory method, whether payment can be split across methods, confirmation and role restrictions.

**OPEN-04 — Choose invoice-level waiver representation.**

- User expressly reserved idea: **waiver flag/tag/column** to indicate invoice was waived partly or in full.
- Questions: boolean derived from `waived_amount > 0` vs stored boolean? Distinguish partial/full? Preserve amount, reason, actor, timestamp? Flag alone does not preserve financial meaning.
- Consider distinction between goodwill waiver and **bad-debt write-off**.
- Do not treat the flag as approved merely because user wanted to discuss it later.

**OPEN-05 — Define adjustment/waiver persistence, notes, and rules.**

- Where is adjustment amount recorded?
- Must the note be mandatory? Is note supplied at initial waiver, final Due settlement, every write-off?
- Can a waiver be applied but invoice left Due? Current explicit admin flow only waives residual **when Mark Paid**.
- How to prevent a negative amount due or waiver greater than outstanding?

**OPEN-06 — Define pending-to-admin resolution in full.**

- Admin can open Pending and explicitly record amount, settle or leave Due.
- What exactly happens when admin enters ₹0 and chooses Due? User allowed it in original discussion.
- May Pending become Paid via 100% waiver without cash? Suggested by broader concept but initial Pending full-waiver path not separately ratified.
- When is `paid_at` set, particularly for zero-money final settlement?

**OPEN-07 — Specify Paid lock in schema/security.**

- Business rule LOCKED; implementation not yet specified.
- Must prevent updates/deletes of invoice, its line items, existing payments, and waiver record once Paid; define how referential updates and automated tasks interact.
- Address mistake handling without contradicting no-correction MVP choice.

**OPEN-08 — Resolve payment dates and late recording behavior.**

- Receipt date vs entry date: concept accepted.
- Can admin backdate `received_at` to when receptionist physically took extra cash? Likely needed by example, but permission/validation not decided.
- Define report time zone and what “today” means; never substitute `updated_at`.

### P1 — Architectural issues from the original debate

**OPEN-09 — Atomic treatment + invoice creation or separate commits?**

- Proposed ordinary checkout: one request and one database transaction saves treatment, invoice, and lines.
- Alternative: saved treatment survives failed invoice and is billed later, requiring a recoverable unbilled-treatment state.
- Operational input: clinicians can record treatments; receptionist normally does; waiver requests rare and owner usually decides immediately; invoice can be Pending before amount dispute resolved.
- Need user decision after clinic scenarios. Do not infer from number of UI screens.

**OPEN-10 — What if checkout/payment succeeds in DB but response is lost?**

- Resubmitted identical checkout/payment can create duplicates unless idempotent.
- Need keys/uniqueness behavior and protection against concurrent duplicate invoicing/payment.
- Separate concerns: atomic all-or-nothing is not duplicate prevention.

**OPEN-11 — Treatment/visit/invoice/line-item cardinality.**

- Was intentionally deferred at user's explicit request.
- Need repo inspection only after user reauthorizes; they had requested **do not inspect GitHub right now**.
- Do not assume invoice-per-treatment or invoice-per-visit.
- Existing excerpt suggested treatment records may have multiple clinical relationships; how invoice lines link to clinical sources is not finalized here.

**OPEN-12 — Can one physical payment cover multiple invoices?**

- In the realistic ₹600 example, user allocated ₹500 new invoice and ₹100 old invoice operationally.
- A simple `payments.invoice_id` handles this as **two invoice-specific payment entries**, but does not model the single ₹600 cash handover as one parent receipt across two invoices.
- User explicitly rejected unallocated-payment/notification complexity now; cross-invoice parent receipt/allocation structure is **DEFERRED**, not secretly implemented.

### P2 — Future policy/reporting details

**OPEN-13 — Distinguish waiver, discount, bad debt/write-off, refund and credit.**

- Owner may clear uncollectible Due as a *loss*; the user also described discretionary waiver.
- Whether these need separate codes or accounting treatment is undecided. A generic “waiver” may be enough for MVP, but reporting semantics should be explicit.

**OPEN-14 — Reporting definitions beyond cash collected.**

- Revenue billed by service/invoice date; collected cash by actual receipt date; outstanding by status/date; waived amounts; retrospective history.
- Query/view implementation is favored but not all report calculations are approved.
- Define whether Paid invoices with waivers count as paid count, waived count, or both in non-cash dashboards.

**OPEN-15 — Package coverage and package sales.**

- Buying a package and receiving package-covered treatments are different events.
- A covered visit may have a ₹0 treatment charge while specialty procedures incur separate charges.
- Alternative future design: covered attendance recorded with no fresh invoice; not decided.
- Package coverage dispute remains an illustrative scenario, not confirmed daily workflow.

**OPEN-16 — Payment methods and multiple tenders.**

- Cash/UPI/card examples exist; exact enum/validation and whether one invoice checkout may be split across payment methods are not finalized.

**OPEN-17 — Later corrections, refunds, voids.**

- Out of current MVP because Paid is locked. Might eventually require auditable reversal/credit mechanisms rather than directly mutating paid entries; do not design silently now.

---

## 9. The original six-gap review: current disposition

The assistant originally raised approximately six immediate gaps, then the conversation branched. The mapping below prevents lost threads.

| Original issue | Where it stands now |
|---|---|
| 1. Additional partial repayment on Due | **OPEN-01**; receptionist’s initial partial is **separately LOCKED**. |
| 2. Waiver after earlier part-payment | **LOCKED:** admin may waive remaining Due and finalize Paid; timing/records finer points **OPEN-05**. |
| 3. Treatment/invoice transaction | **OPEN-09**. |
| 4. Duplicate requests/responses lost | **OPEN-10**. |
| 5. Fully waived invoice (₹0 collected) | Owner can settle Due with ₹0 and waive remainder; detailed Pending UX and reporting **OPEN-06 / OPEN-14**. |
| 6. Paid lock and accidental wrong method | **LOCKED:** no edits for anyone in MVP; technical enforcement **OPEN-07**. |

Additional topics later added: shared payments table/atomic payment updates; manual admin handoff; initial receptionist partial-to-Due; waiver flag; time/date fields; full schema and cardinality.

---

## 10. Original excerpt’s reasoning — retained, but NOT all adopted

This section preserves the architectural context that launched the conversation.

### Strongest case for **combined saving** (PROPOSED)

At an ordinary reception-led checkout, treatment details and charges are already known. One submit could save treatment, create invoice and lines, and commit everything together. This avoids normalizing a half-finished “treatment saved, no invoice” state.

- Database transaction gives all-or-nothing behavior.
- Code can be modular even if commit is atomic: separate treatment-writing and invoice-building functions under one checkout transaction.
- Payment confirmation remains a later request.
- Invoice line items do not require treatment and billing to commit separately.
- The earlier excerpt says the specialty discussion and MVP review favor this approach; **not independently verified here**.

### Strongest case for **separate saving** (HYPOTHETICAL unless confirmed)

If clinician needs to finish clinical charting while billing price/coverage/waiver is undecided, a durable treatment-only save permits a real handoff to reception. But that demands an explicit unbilled-treatment work queue, retries, deduplication, and resolution rules.

- Clinic input did **not** establish that this regularly happens.
- Ability to invoice an older treatment later does **not** force every normal checkout to use two commits now.
- One HTTP request can still contain two database commits; it is transaction boundaries, not button count, that matter.

### Line items and clinical linking (ESTABLISHED CONCEPT; schema OPEN)

Original example:

```text
Knee therapy charge: ₹300
Back therapy charge:  ₹0
Cupping charge:       ₹500
Invoice total:       ₹800
```

A clinical cupping procedure might relate to both complaints but must not be charged twice solely because it has two complaint associations. Clinical records describe what occurred; invoice lines describe charges. Bundled treatments should not generate duplicate procedure-level charges.

### Package and payment distinctions (CONCEPTUAL)

- A package purchase can carry a charge separate from covered therapy attendance.
- A package-covered visit may produce no new positive therapy charge, while another chargeable specialty item remains.
- It is not yet decided that every clinical event always generates a fresh invoice.
- Receipt printing/PDF should be retriable from saved invoice; failure to print should not imply resaving treatment.

### Concurrency and frozen invoices (PROPOSED technical caution)

- “Check whether already invoiced” before insertion is vulnerable to two concurrent requests; transaction alone does not prevent every race.
- If invoice lines are frozen after issuance, adjusting only a treatment-side price can create inconsistency. Any post-invoice corrections need explicit adjustment policy.
- Deferred invoice integrity checks do not logically force treatment and invoice creation into the same transaction if foreign keys can reference earlier records.

---

## 11. Conflict and implementation risk register

These are not extra features; they are where an engineer could misread the current rules.

| Risk | Correct interpretation / unresolved action |
|---|---|
| Status overloaded with “admin reviewed” | Do not use `status = due` to prove admin reviewed: receptionist can now set Due at new checkout. A separate actor/event field would be needed if “admin reviewed” must be reported. |
| “Paid” mistaken for money received | Paid means balance settled. Actual receipts come only from payments; waiver is not cash. |
| No payment row on a zero receipt | Do not insert a ₹0 payments row merely to signal settlement. If entirely waived, record waiver/settlement separately. |
| Old invoice controlled by receptionist | The new permission is limited to **initial checkout**. Old Due/Pending remains admin-controlled. |
| Partial receipt on old Due forces waiver | The currently described admin form has Mark Paid / Cancel; OPEN-01 asks whether to add Record Payment, Keep Due. |
| Full lock makes mistakes permanent | This is consciously accepted MVP behavior, not a claim mistakes cannot occur. Confirm before finalizing. |
| Reporting by paid_at overstates cash timing | `paid_at` is *final settlement time*. Partial money could have arrived days earlier. Cash collection must use **payment.received_at**. |
| Admin records old cash later | Physical cash may temporarily exceed recorded cash. Manual handoff accepted; date semantics OPEN-08. |
| Waiver flag used as complete accounting model | Boolean alone cannot represent ₹ amount, reason, actor, or partial/full proportion. It must not replace those facts without explicit decision. |
| Invoice total equal to payments aggregate | They can differ during Pending/Due and when waivers exist. Outstanding = billed − collected − waived. |
| Premature visit assumptions | Do not hard-code one invoice per treatment or one treatment per visit before OPEN-11. |
| Race/double submit | Atomic transaction does not inherently ensure no duplicate attempts. OPEN-10 must be addressed in implementation design. |

---

## 12. Suggested *resume order* — process guidance, not a business decision

The user asked to be questioned and challenged with **one concrete scenario at a time**, and not to have recommendations presented as settled decisions. When conversation resumes:

1. **Start with OPEN-01**: Given old invoice Due ₹300 and another ₹100 received, can admin record ₹100 **and keep ₹200 Due**? This is the most immediate unresolved functional conflict.
2. Then **OPEN-02**: Explicitly ratify the central payments table and transactional insertion/settlement update after showing actual schema constraints; avoid conflating this with treatment + invoice transaction.
3. Then **OPEN-03**: Finish receptionist initial partial-payment form, including zero and method rules.
4. Then **OPEN-04/05**: Revisit the promised **waiver flag** idea, and decide how waivers/write-offs, amounts, reasons, and actors are stored.
5. Then **OPEN-09/10**: Resume combined vs separate treatment/invoice persistence and duplicate-submission handling with one concrete clinic scenario each.
6. Keep **OPEN-11**, visit/treatment cardinality, parked until user reopens it. Do **not** proactively search their GitHub.

This is a recommended order to minimize circular discussion, **not** six questions already answered or approved.

### Ready-to-use first question for next session

> “An existing invoice is **Due ₹300**. Patient brings **₹100** during a later visit. Only admin can touch the older invoice. Should admin be able to record that ₹100 and keep the remaining ₹200 **Due** (instead of having to mark the invoice Paid and waive ₹200)? Yes or no?”

After answering, do not jump straight into schema changes; confirm whether the user wants a deeper challenge or the next question.

---

## 13. Mini change log: what changed since the earlier decision log

The first long decision log had 14 numbered items, an operational facts section, and outstanding architecture questions. Subsequently:

- **More explicit waiver-on-Due rule:** Admin can waive/write off the balance of a partly paid Due invoice and finalize Paid, including entering **₹0** new money.
- **Owner Due modal clarified:** Enter a further amount between ₹0 and the current due total, default ₹0; Mark Paid or Cancel. The distinction between this finalization and partial repayment *without* finalization remains open.
- **Real-world next-visit example:** Patient pays new ₹500 invoice and hands over extra ₹100 against an old ₹300 debt. Only admin handles old debt.
- **No in-product handoff infrastructure:** Call/text/in-person owner notification, not unallocated funds or automated alerts.
- **Payments table strongly developed:** One row per actual receipt across every invoice status; zero, one, or many per invoice; include method, received time, recorded time, actor. User requested actual schema explanation.
- **Invoice-item and payment cardinality clarified:** Independent one-to-many child entities; cardinality of visit/treatment to invoice intentionally deferred.
- **Initial receptionist partial checkout added and explicitly approved:** New invoice ₹500, receive ₹300, mark ₹200 Due. Receptionist has no waiver authority.
- **Status Due definition updated:** Admin review is not required when receptionist initially records partial payment; earlier admin-only provenance definition superseded.
- **Waiver flag idea preserved:** User wants to discuss invoice-level waived flag/tag/column later; **not yet approved**.
- **Still unresolved:** Additional part-payment on old Due without closing it, final schema and transaction behavior, combined treatment/invoice transaction, idempotency, schema cardinality, zero-money Pending waiver flow, auditing details.

---

## 14. End-of-break checkpoint

**Do not lose these three distinctions:**

1. **What was billed** (invoice lines, original invoice amount), **what was collected** (actual payment records), and **what was forgiven/write-off** (waiver/adjustment) are three different financial facts.
2. **Who can act** depends on *new checkout vs old unresolved invoice*: receptionist may record part payment and mark a **new** invoice Due, but admin alone revisits an **old** Due/Pending invoice and approves waivers.
3. **Paid means settled and locked**, not “all original charges were collected in cash.”

**Most recent explicit decision:** Receptionist **can** record a partial payment on a newly generated invoice and mark the remainder **Due**; waiver remains admin-only.

**Most important immediate open question:** Can admin take an **additional partial payment** on an older Due invoice **without** marking it Paid or waiving the remainder?

**User-reserved topic:** Invoice-level **waiver flag/tag**; discuss later, without assuming a boolean alone solves accounting.

**Parked topic:** Visit–treatment–invoice cardinality; do not inspect GitHub unless user reopens it and requests inspection.

---

*Prepared as a faithful decision record from the conversation, including explicit uncertainties and superseded statements. This is intended to be edited as decisions are made, rather than treating discussions as requirements by default.*

## 15. Open discussion added — 21 October 2026

**OPEN — Applied therapy billing-mode snapshot: necessity and location.**

- Question: Should the historical applied therapy billing mode be stored on the visit, on the invoice, or is that snapshot needed at all? No choice has been finalized. Earlier assistant recommendations for either location are proposals, not accepted decisions.
- Stated clinic policy: The clinic controls its therapy billing mode; the backend resolves the applicable setting rather than accepting a per-visit choice from the frontend.
- Case to examine: A package-covered visit might have no invoice or invoice line. Whether package-covered visits should be omitted from invoices is itself unresolved. If omitted, would preserving flat-versus-per-procedure mode have any useful meaning for that visit? Do not assume it does.
- Additional consideration raised by the user: Invoices can cover visits, package purchases, specialty treatments, and other chargeable items. Examine how a therapy-specific mode fits that broader invoice scope.
- Preserve the distinction between package coverage and therapy billing mode; they are separate facts.
- Do not assume invoices combine visits from different dates or checkouts. That hypothetical was introduced by the assistant and is not an established requirement.
- User instruction: Log this as an open discussion; do not resolve it or continue arguing the design now.
