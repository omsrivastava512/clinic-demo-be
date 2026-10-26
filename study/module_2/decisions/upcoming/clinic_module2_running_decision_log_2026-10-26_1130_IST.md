# Clinic Module 2 — Running Decision Log (Incremental, 26 October 2026)

**Created:** 26 October 2026, 11:30 IST (Asia/Kolkata)  
**Coverage:** From **25 October 2026, approximately 23:39 IST**, after the previously created `clinic_module2_post_addendum_decisions_2026-10-25.md`, through the latest agreement on invoice settlement fields on **26 October 2026, approximately 11:28 IST**.  
**Document type:** New, standalone **running addendum**. **Do not replace** earlier decision logs. Append future changes under a new dated entry rather than retroactively changing what was agreed.  
**Mode:** Design/discussion only. No migration, schema change, GitHub change, or Supabase database write was authorized or made as part of these decisions.

## 0. Read this first: scope and interpretation

The source of truth for this addendum is the visible conversation after the cutoff. Decisions from other chats are referenced only when the user explicitly confirmed them here. **Do not promote an assistant recommendation or proposed schema field into a locked requirement.**

**Status legend**
- **LOCKED:** User expressly accepted the business or design rule.
- **CARRIED FORWARD:** Decision was made before this addendum; reaffirmed here, not newly decided.
- **OPEN:** A decision is still required.
- **PARKED:** User expressly asked not to resolve it yet, or agreed to defer it.
- **TECHNICAL FOLLOW-UP:** Engineering specification remains to be designed despite a settled business rule.
- **SUPERSEDED / CORRECTION:** Replaces a mistaken earlier statement or proposal.

### Current checkpoint

**Next discussion:** Roadmap **04 — `payments` table schema**, beginning with its actual columns, nullability, relationships, and constraints, while preserving previously decided payment behavior. Roadmap item **03 — general settlement metadata** has just been locked. Roadmap item **02 — detailed invoice-line fields** was deliberately parked and must not be reopened without user approval.

---

## 1. Newly locked decisions since the prior post-addendum

### D26-01 — Waived amount is derived, not persisted separately — LOCKED

The user rejected a separate authoritative `waived_amount` field for the agreed one-time final waiver model.

- For an invoice ultimately marked `WAIVED`, the forgiven portion is the residual **original billed total minus actual money collected**.
- Example: ₹500 billed, ₹200 actually collected, invoice finalized `WAIVED` → ₹300 forgiven, **₹0 collectible outstanding**.
- This depends on an issued billing snapshot (immutable line-item amounts) and accurate, separately stored real payment records.
- Do not confuse **amount forgiven** with **amount received**. Waiver generates **no payment row**.
- Do not treat this rule as approval of multiple independent partial-waiver events while keeping the same invoice `DUE`; that is a different, deferred workflow.

**Proposed derivation for the currently agreed terminal-waiver model (not finalized SQL):**

```text
billed = SUM(issued invoice line amounts)
received = SUM(positive payment rows associated with this invoice)
if status = WAIVED:
    waived = billed - received
    collectible_outstanding = 0
else:
    waived = 0
    collectible_outstanding = billed - received
```

Validations and any zero/negative guards belong to later database design. The concept of deriving balances was already established in the earlier addendum and is reaffirmed here.

### D26-02 — Do not store a redundant invoice billed total for MVP — LOCKED

The user chose to **calculate the invoice's billed amount as the sum of its immutable invoice line items**, rather than persist an additional authoritative `invoices.total_amount`/`billed_amount` header column.

- Reason: a sum over a small set of immutable line items is simple; duplicating the total introduces synchronization/integrity obligations.
- The assistant raised cases where a stored/indexed total may help: large-scale aggregation, sorting/filtering by amounts, expensive calculations, or proven reporting performance problems.
- The user explicitly decided to add a stored total **only if measured performance requires it**, not speculatively for this MVP.
- **Important:** historical correctness requires invoice lines to preserve the charged values as issued. Recalculating from *current catalog prices* would be wrong; those are not the historical billing source of truth.

**SUPERSEDED:** Assistant initially recommended storing the total. The user rejected that as the MVP default and the assistant agreed.

### D26-03 — Every invoice must have at least one line item — LOCKED

The user explicitly approved the requirement that **a newly issued invoice cannot have zero line items**.

- At least **one** invoice line item is mandatory.
- This does not independently finalize the actual fields of a line item, or their relationship to visits/treatments.
- Enforcement mechanics (transaction-end integrity check, database function, etc.) remain a technical design task. A simple invoice-header constraint by itself cannot count child records.

### D26-04 — A single payment record belongs to exactly one invoice in MVP — LOCKED

The user answered **"For MVP is one"** when asked whether a payment record could cover multiple invoices.

- For MVP, each payment row is attributed to **one** invoice (`invoice_id`).
- One invoice can have **zero, one, or many** actual payment rows, a previously agreed cardinality.
- When someone physically hands over money toward a current and an older invoice, the **manual receptionist-to-admin handoff** remains the MVP workaround: the admin records the portion attributable to the older invoice.
- This rule does **not** authorize an automatic allocation engine or a unified cross-invoice receipt object.

**Note on the earlier open question:** The preceding addendum explicitly recorded one-payment-one-invoice as *unanswered*. This is **newly resolved here**.

### D26-05 — Zero amount in the UI does not create a zero-value payment row — REAFFIRMED / LOCKED

The user questioned why zero should be forbidden in the `payments` table; after the assistant explained the distinction, the user confirmed there was no new requirement.

- Some forms may legitimately accept `0` when deciding to defer collection or close by full waiver.
- **Do not persist a fictitious `payments` record with amount 0.** A payment row means positive money actually received.
- Zero collected is a financial outcome; it is **not** a payment event.

This was already settled before this addendum, then explicitly confirmed again.

### D26-06 — Payment timestamps use the server clock; no backdating — REAFFIRMED / LOCKED

When the assistant raised the difference between the actual handover time and delayed admin recording, the user expressly chose **"Just use the server stamp"** to avoid additional complexity.

- The application assigns the payment timestamp on the server when the payment is recorded.
- No operator-entered historical receipt dates or backdating in MVP.
- **Conscious limitation:** Cash physically received on one day but recorded by an admin on another appears on the **recording** day in software reports. This is not literally the physical handover date; do not claim otherwise.
- The old cash/UPI-to-admin manual handoff is still valid operationally; the discrepancy until admin entry is an accepted MVP simplification, not a new unallocated-cash feature.

The previous audit called this an inconsistency. The user re-confirmed the intentional timestamp policy. The reporting limitation should remain documented, rather than silently “fixed” by inventing a backdating feature.

### D26-07 — Reasons mandatory for waivers, not ordinary fully paid settlements — LOCKED

Earlier wording about a mandatory note on an invoice an admin "settles" was ambiguous. The assistant explicitly distinguished a full payment from a waiver; the user answered **"No. Just waivers."**

- `waiver_reason` is required when closing an invoice as `WAIVED`, including write-offs handled under that same status.
- A normal full-payment settlement ending in `PAID` does **not** require a reason note.
- Frontend quick-pick reason suggestions are allowed; the actual stored value remains free text (carried forward).

### D26-08 — Zero-total invoices are forbidden in MVP — LOCKED

Roadmap Question 01 asked whether the calculated sum of an invoice's line items could be ₹0. The user said **"Yeah, forbid."**

- Every invoice must have **a strictly positive calculated billed total** when issued.
- Combined with D26-03: at least one line item **and** total greater than ₹0.
- The user **did not** separately decide whether a zero-priced individual line is permissible as part of a positive-total invoice.
- Package-covered visits that might otherwise need a zero-rupee invoice remain part of the parked package/visit/invoice discussion. Do not invent their future treatment workflow from this one MVP constraint.

### D26-09 — Generalize invoice settlement fields for PAID and WAIVED — LOCKED

The assistant proposed a unified final-settlement timestamp, then the user suggested generalizing related waiver-only metadata. The exact three-field proposal was accepted with an explicit **"Yes."**

**Agreed invoice fields:**

1. **`settled_at`**: Time of final settlement when an invoice enters **either `PAID` or `WAIVED`**. Conceptually replaces separate `paid_at` and `waived_at` fields.
2. **`settled_by`**: User who finalized the invoice, for **either** final status. Conceptually replaces waiver-specific `waived_by`.
3. **`waiver_reason`**: Free text, **mandatory if status is `WAIVED`**, **null if status is `PAID`**.

**Important distinctions:**
- `settled_by` identifies the user who finalized the invoice, not necessarily the person who physically collected money. The payment record carries collection/recording attribution.
- `settled_at` represents invoice closure. Individual payment rows separately have their recording/receipt timestamps; an invoice can receive more than one payment before finalization.
- These were **schema design choices, not verification of current deployed columns**. The conversation mentioned `waived_at` in planning; no confirmation was established that such a column already physically existed.
- The prior no-extra-`is_waived`-flag and derived-waived-amount rules remain in force.

**SUPERSEDED as preferred new design:** Separate invoice-header `paid_at`, `waived_at`, and `waived_by` fields in the proposed schema. Do not delete or migrate existing production columns without first inspecting the actual schema and obtaining authorization.

---

## 2. User-raised concerns, auditing, and corrections discussed in this interval

These discussions do not all yield new requirements, but must be preserved to explain the resulting plan.

### A26-01 — How many of the original six questions and user ideas were resolved?

The assistant reconfirmed that the **original six questions had been covered at the business-rule level**, with **idempotency implementation still parked**. The user challenged whether everything had been counted and specifically recalled the scenario of paying extra toward yesterday's invoice.

**Counting correction:** The receptionist-to-admin old-Due payment handoff was already **Idea C** in the prior "three ideas" record. A subsequent assistant message incorrectly called it a completely separate *fourth* omitted idea. The later audit corrected that label. The actual scenario and workflow were not lost.

Original six, as referenced in this conversation:

| Earlier topic | State carried into this addendum |
|---|---|
| Multiple repayments against DUE invoices | Business requirement previously settled; payments table records positive receipts |
| Waiving a remaining DUE balance | Previously settled; `WAIVED` status for final forgiveness |
| Atomic treatment+invoice creation | Previously settled as one billing transaction |
| Duplicate prevention | Requirement locked; concrete idempotency implementation parked |
| Fully waived invoice with ₹0 received | Closed; terminal `WAIVED`, no fake payment row |
| Paid invoice immutability | Locked; no edit/correction workflow in MVP |

The previous user ideas remain: invoice waiver flag (**rejected as redundant**), receptionist accepting initial partial payment (**approved**), and manual notification to admin about money for an older invoice (**approved for MVP**). No new reopening occurred here.

### A26-02 — Older-Due plus today's checkout example revisited

The user recalled the following concrete scenario:

```text
Yesterday's invoice: ₹500 billed, ₹200 collected, ₹300 DUE.
Today's invoice: ₹500 billed.
Patient hands over ₹600 today: ₹500 for today's bill + ₹100 toward yesterday.
```

**Existing MVP handling:** Receptionist records today's ₹500 as today's payment and manually informs the admin of the extra ₹100. Admin handles the older `DUE` invoice and records its separate ₹100 receipt, reducing yesterday's outstanding balance to ₹200. No in-app notification/unallocated-wallet/automatic distribution feature is required for MVP.

**New relationship detail resolved later in this interval:** Each persisted payment row belongs to one invoice (D26-04).

### A26-03 — Deep-dive audit: gaps and omissions surfaced

The assistant audited conversation and existing decision logs, identifying the following. Preserve final outcomes, not earlier uncertainty as though still open:

| Audit finding | Outcome as of this log |
|---|---|
| Delayed admin entry vs actual physical cash receipt date | **INTENTIONAL MVP LIMITATION** after user re-confirmed server timestamp only (D26-06) |
| Is a note required for ordinary admin full-payment closure? | **RESOLVED:** mandatory for waiver/write-off only (D26-07) |
| “Fourth idea” allegedly missing | **CORRECTED:** old-invoice handoff was already recorded as Idea C |
| Early reports counted only invoices with PAID status | **SUPERSEDED:** collection reports aggregate actual positive payment rows, even while invoice remains DUE or later becomes WAIVED |
| An invoice with balance forgiven could be recorded as PAID | **SUPERSEDED:** terminal `WAIVED` covers the agreed final waiver cases |
| Finalization timestamp for WAIVED vs PAID | **RESOLVED:** unified `settled_at` and `settled_by` (D26-09) |
| Zero-value **payment** versus zero-total **invoice** | **RESOLVED distinctly:** never insert ₹0 payment rows (D26-05); forbid zero-total invoices (D26-08) |
| One billing+payment transaction proposed earlier | **CORRECTED:** initial treatment/invoice transaction, then separate payment transaction |

### A26-04 — Brief schema roadmap, then resumed session

The assistant first proposed an eight-part schema checklist (invoice header, payments, waiver metadata, financial calculations, status transitions, authorization, integrity, edge cases). The user resumed and elected to proceed with invoice header decisions first. After repeated questions and a mistaken use of an external lookup for a question about the **discussion**, the user insisted on a **preplanned, finite question sequence**.

The assistant then published the 12-item roadmap in §3 below. **Do not invent a new question outside this sequence** without adding it to the plan and justifying the dependency; do not repeatedly ask for already locked choices.

### A26-05 — Explicitly postponed invoice line-item fields

The assistant suggested storing per-line `description`, `quantity`, and `unit_price` and calculating line total. The user said **"keep it open"** and **"I'll decide that when I get there."** Thus those field choices are **PARKED**, not accepted. Visit/treatment source relationships are also still parked.

### A26-06 — Direct schema lookup explicitly rejected for this discussion

During the settlement-timestamp conversation, the user said **not to go into Supabase**, and to look only at prior discussion. The response subsequently correctly treated `waived_at` as a **mentioned proposed field**, not proof of an existing physical column. Do not assert current deployed-schema fields without checking when the user later authorizes implementation work.

---

## 3. Fixed roadmap (published before resuming Q01)

This is the **ordered discussion contract** requested by the user after previous unplanned/redundant questioning. Updates below reflect what happened after that roadmap was published.

| Roadmap | Topic | Planned work | Current status |
|---|---|---|---|
| **01** | Zero-total invoices | Allow ₹0 billed invoice? | **LOCKED:** forbidden for MVP (D26-08) |
| **02** | Invoice line-item fields | Exact source/description/quantity/price fields | **PARKED** at user's request |
| **03** | Invoice settlement timestamps and actor | Common fields for PAID and WAIVED | **LOCKED:** `settled_at`, `settled_by`, waiver-only `waiver_reason` (D26-09) |
| **04** | `payments` table | Exact columns, FKs, types, nullability, indexes | **NEXT / OPEN**; business rules partly settled |
| **05** | Waiver metadata | Field constraints, data requirements, finalization | **PARTLY LOCKED** at business level; physical SQL OPEN |
| **06** | Derived financial amounts | Query/view/function for billed, received, forgiven, outstanding | **Formula/don't-store locked; implementation OPEN** |
| **07** | Invoice status transitions | Allowed moves among PENDING, DUE, PAID, WAIVED | **Business meanings locked; precise SQL transitions OPEN** |
| **08** | Authorization | Receptionist vs admin permissions, backend enforcement | **Roles agreed; RLS/functions implementation OPEN** |
| **09** | Transaction boundaries | Separate billing commit and payment commit, each consistent | **Boundaries locked; implementation OPEN** |
| **10** | Database integrity | CHECK/FK/unique/concurrency; final-state locking | **Core prohibitions locked; SQL enforcement OPEN** |
| **11** | Payment UI and API | Request payloads, confirm/cancel paths, error handling | **OPEN** |
| **12** | Edge-case tests | Full/partial/split/waived, zero values, concurrent operations | **Test plan/implementation OPEN** |

**Deliberately excluded from the active roadmap:** Idempotency implementation (parked, requirement remains), automatic cross-invoice allocation, visit–treatment–invoice cardinality, package/examination/catalog design, and retrospective invoice correction. Do not reopen without request.

---

## 4. Field-level decisions and boundaries at this checkpoint

These are **design intentions**, not a live-database schema inventory.

### `invoices` — current locked conceptual attributes/rules

| Field/concept | Status | Interpretation |
|---|---|---|
| `id`/invoice identity | Existing design concept | Actual type/column not finalized here |
| `status` | CARRIED FORWARD | `PENDING`, `DUE`, `PAID`, `WAIVED` |
| `created_at` | CARRIED FORWARD | Original issue time concept |
| `settled_at` | **LOCKED NEW** | Closure timestamp for both PAID and WAIVED |
| `settled_by` | **LOCKED NEW** | Actor finalizing either terminal status |
| `waiver_reason` | **LOCKED/CLARIFIED** | Mandatory free text on WAIVED; null on PAID |
| `total_amount`/billed header total | **REJECTED for MVP** | Compute from immutable invoice lines; revisit only on measured need |
| `waived_amount` header | **REJECTED as redundant** | For terminal WAIVED, derive original billed minus real received |
| `outstanding_amount` stored header | CARRIED FORWARD REJECTED | Always derive, no authoritative mutable stored balance |
| `is_waived` flag | CARRIED FORWARD REJECTED | Existing WAIVED status already identifies it |
| Separate `paid_at`, `waived_at`, `waived_by` | **SUPERSEDED as proposed design** | Unified `settled_at`/`settled_by` instead |

**Invoice issuance integrity:** at least one invoice item **and** sum of items **strictly positive**. Item details remain parked.

### `invoice_items` — current boundary

- At least one per issued invoice (**LOCKED**).
- Their issued amounts must remain historical snapshots for later sums (**carried forward**).
- Specific columns (`description`, `quantity`, `unit_price`, treatment references) **not approved here**; PARKED.
- Relationship to clinical visits/treatment remains PARKED.

### `payments` — current boundary

- One row = one **positive actual payment**; no ₹0 rows.
- Each row references **exactly one invoice in MVP**.
- One invoice may have multiple payment rows.
- Cash/UPI only; Split is an input mode resulting in actual cash and/or UPI components, not its own enum value.
- Server records timestamp; no backdating.
- Do not exceed currently collectible outstanding amount; no overpayment.
- Recording payment and changing status must be consistent within the **payment transaction**, separate from issuance transaction.
- Final schema fields/types/FK `ON DELETE`, indexes, constraints, role attribution, method casing, and RPC payloads are the **next design task**.

### Final states and reporting — carried forward, not reopened

- `PAID` and `WAIVED` are terminal/locked in MVP.
- Fully waived with no receipts: `WAIVED`, balance zero, no payment row.
- Partially paid then remainder forgiven: `WAIVED`, collected money remains its real payment rows.
- Reporting: actual cash collection derives from dated payment records, **not** invoice's original billed total or a `status = PAID` filter.

---

## 5. Exact chronological question-and-decision trail (since the cutoff)

This is a compact chronological audit rather than a verbatim transcript.

1. **Oct 25, after ~23:39 — What's left?** Assistant identified active schema and payment UI, with idempotency/relationships parked. No new user approval.
2. **How many of original six/ideas?** Assistant summarized resolved business questions; user asked specifically about extra money for another invoice. Existing human handoff clarified. The count of "fourth idea" was later corrected.
3. **Anything omitted?** Deep-dive audit raised receipt-date tension, note scope, settlement timestamp, zero invoice ambiguity, reports, and superseded status terminology. These became the follow-up agenda.
4. **Which schema topics next?** Eight-part concise checklist proposed. User resumed schema session.
5. **Store waived amount?** User said always derive from billed minus receipts when terminally waived → D26-01.
6. **Store invoice billed total or SUM(line items)?** Assistant initially advocated storage; user preferred `SUM` and said only add stored total for actual performance issues → D26-02.
7. **Can an invoice have no line items?** User: **at least one** → D26-03.
8. **Which exact fields in invoice line items?** Assistant suggested description, quantity, unit price; user **PARKED** the choice → A26-05.
9. **Can one payment cover multiple invoices?** User: **one for MVP** → D26-04.
10. **May a payment row contain zero?** User asked why; assistant explained UI ₹0 versus true receipt; user accepted no new exception → D26-05.
11. **How to date delayed admin payment entries?** User reapproved **server stamp only** → D26-06.
12. **Mandatory settlement note for normal payment?** User clarified **only waivers** → D26-07.
13. **Assistant repeated waived-amount question.** User correctly pointed out duplication. It remains **already locked**, not a new open question.
14. **Assistant jumped to waiver closure timestamps.** User required the complete question plan before continuing. The 12-item roadmap was published.
15. **Roadmap 01: may total billed be ₹0?** User: **forbid** → D26-08.
16. **Roadmap 03 (02 parked): settled_at?** User requested clarification, mentioned `waived_at`; agreed that `settled_at` replaces distinct final-date fields conceptually.
17. **Generalize waiver-only metadata?** User asked whether `waived_by` and reason can also be generalized. Assistant proposed `settled_at`, `settled_by`, and specific `waiver_reason`; user: **yes** → D26-09.
18. **Immediately afterward:** Assistant announced roadmap 04 (`payments` table); user **asked for this new running log first**. No new payment-table field was approved after D26-09.

---

## 6. Open and parked items — actionable handoff

### Next active discussion: Roadmap 04 — `payments` table (OPEN)

Present an actual small proposed PostgreSQL table aligned with the locked rules and walk through only unresolved design fields:

1. Record identifier and invoice foreign key; one-to-many structure; `ON DELETE` policy.
2. Amount representation/type and positive constraint; no overpayment enforcement location.
3. Payment method enum with only cash/UPI.
4. Server-supplied timestamp and possibly a separate technical row-creation time only if demonstrably needed; no user-backdating.
5. `recorded_by`/actor FK and nullable vs required semantics.
6. Immutability, authorized insert paths, indexes, and payment-insert/status-update transaction.
7. Representing split tender with separate positive rows; error behavior if one component fails.

Do **not** turn this into a fresh series of arbitrary yes/no questions if there is a clear conventional implementation. Distinguish genuine product decisions from backend engineering details.

### Following active work (OPEN / TECHNICAL)

- **05 — Waiver schema constraints:** `WAIVED` requires nonempty `waiver_reason`, `settled_at`, and `settled_by`; `PAID` requires settlement actor/time and no waiver reason. Decide rules for nonterminal statuses and consistent transaction-end enforcement. The first two rules were approved; exact SQL details not yet.
- **06 — Derived amounts:** efficient query/view/function for billed, receipts, and waived residual while preventing duplicate sums from joining two one-to-many tables improperly. Performance should be measured before denormalization.
- **07 — Transition matrix:** specify backend enforcement for PENDING, DUE, PAID, WAIVED; forbid mutation after terminal states; user retains business-rule authority.
- **08 — Authorization:** receptionist initial checkout/partial payments, admin follow-up on older DUE invoices and waivers, restrictions implemented server-side.
- **09–10 — Atomic writes and constraints:** treatment+invoice+lines issuance, separate receipt/status commit, invoice-line existence/positive total, immutable finals, concurrency-safe no-overpay.
- **11–12 — API/UI and tests:** specify supported form submissions, split and zero input handling, confirmed-backend success UX, negative/edge tests.

### PARKED — must not be auto-reopened

- **02 — Invoice-line schema details** (user explicitly deferred).
- Idempotency key mechanism and lost-response retry recovery (**requirement already locked**, mechanism postponed).
- General automatic one-receipt/multi-invoice allocation and unallocated money workflow.
- Visit–treatment–invoice cardinality.
- Packages, covered sessions/zero-price visits, specialty billing, and other clinical catalog issues.
- Retroactive invoice/payment correction/voiding process outside the agreed locked MVP.

---

## 7. Behavioral guardrails for the next discussion

- Follow the **12-item roadmap** and announce which item is active. Never quietly switch between payment, invoice, package, and treatment design.
- Ask **one** meaningful product question at a time, only when the implementation actually depends on the answer.
- Show a concrete example and its consequence; do not pressure the user into yes/no without context.
- Do not reopen decided topics merely to create momentum; explicitly mark user-approved rules and continue.
- Do not infer existing columns from a design discussion. This log represents **decisions**, not the current deployed Supabase migration state.
- Do not inspect GitHub, Supabase, or change code without a request to do so.
- Maintain distinct LOCKED / OPEN / PARKED / PROPOSED / SUPERSEDED labels and preserve all historical overrides.

**End-of-log checkpoint:** The last explicit decision was **D26-09 (`settled_at`, `settled_by`, `waiver_reason`)**. The very next scheduled topic is **Roadmap 04 — payments table**. No payments-table fields were newly ratified after that settlement-field decision.

## 8. Post-log discussion — payments table (26 October 2026)

**Coverage:** Decisions made after the checkpoint in §7, based on the attached conversation. This is an append-only update; §7 remains an accurate record of the earlier checkpoint.

### D26-10 — Keep separate `created_at` and `received_at` timestamps — LOCKED

- Keep both `created_at` and `received_at` on payment records.
- Both are `TIMESTAMPTZ NOT NULL` with server-assigned defaults (`now()`); the client cannot supply or backdate either timestamp in the MVP.
- `created_at` represents insertion of the payment row. `received_at` represents the receipt timestamp used for collection reporting. They will normally be identical in the MVP because both are assigned by the server when the payment is recorded.
- The user accepted the extra column to preserve a distinct receipt-time field for future backdating support and avoid later migrations or historical backfills. Until backdating is added, a payment recorded later is reported on its server-recording date, consistent with D26-06.

### D26-11 — Store monetary amounts as `BIGINT` paise — LOCKED

- Use integer paise for monetary amounts across the billing model, not only for payment rows. For example, ₹250.50 is stored as `25050`.
- The user explicitly chose paise and asked not to revisit the choice. Apply the same unit consistently to invoice-line amounts and payments when those schemas are finalized.

### D26-12 — Require server-derived payment recorder — LOCKED

- `payments.recorded_by` is mandatory.
- The backend derives it from the authenticated user; it must not trust a recorder ID supplied by the React client.

### D26-13 — Payment rows are positive and immutable — LOCKED

- `invoice_id` must reference a valid invoice.
- A payment row must represent positive money received; enforce `amount_paise > 0` and do not create zero-value rows.
- Recorded payment rows cannot be edited or deleted in the MVP.

### D26-14 — Prevent invoice deletion — LOCKED

- Invoice deletion is forbidden in both the frontend and backend.
- Payment-to-invoice foreign keys use `ON DELETE RESTRICT`, preserving payment history even if another path attempts invoice deletion.

### D26-15 — Save split-payment rows and invoice status atomically — LOCKED

- When a split tender produces cash and UPI payment rows, insert the positive component rows and update the invoice status within one payment-settlement database transaction.
- If any part fails, roll back the entire payment settlement so neither a partial split nor a mismatched invoice status is persisted.
- This remains the separate payment transaction established by D26-09’s earlier checkout/payment boundary; it does not combine payment settlement with initial treatment and invoice issuance.

### Payment-table indexes — OPEN / PROPOSED

- A later assistant turn proposed indexes on `payments.invoice_id` and `payments.received_at`, for invoice payment history/balance calculations and date-range collection reports respectively.
- The user has not approved that proposal. Keep the index choice **OPEN**; do not record either index as locked based on the attached conversation.

**Updated checkpoint:** Roadmap 04 — `payments` table — remains active. The decisions above are locked. Indexes and any other unresolved implementation details remain open; no live Supabase schema was inspected and no migration was authorized by this discussion.
