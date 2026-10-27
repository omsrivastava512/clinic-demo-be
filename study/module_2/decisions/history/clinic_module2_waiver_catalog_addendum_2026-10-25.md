# Clinic Module 2 — Waiver Status and Single-Clinic Catalog Addendum

**Date:** 25 October 2026  
**Type:** Independent successor addendum  
**Source:** User-provided conversation excerpt in the current chat  
**Scope:** Capture the decisions closed in that excerpt and preserve associated deferred migration requirements. This is a **requirements record**, not an implemented schema, migration, or deployment.

## Status convention

- **LOCKED — MVP:** Explicitly accepted product behavior for the first release.
- **SUPERSEDED FOR MVP:** Earlier design direction displaced by a later decision, without deleting its history.
- **DEFERRED — FUTURE:** Outside the first release, with a requirement or topic to revisit.
- **OPEN — ENGINEERING/DESIGN:** Not settled by the conversation; do not implement an assumed answer.

## 1. Invoice settlement: separate `Waived` final status

### 1.1 Status and authorization — LOCKED, MVP

- Introduce **`Waived`** as a separate terminal invoice status rather than marking a waived invoice **`Paid`**.
- **`Paid`** means the invoice was settled through actual money received. **`Waived`** means the admin forgave the entire remaining collectible balance. `Waived` can apply even when one or more partial payments have already been received.
- Both **`Paid`** and **`Waived`** are **fully settled**: their outstanding balance is **₹0**, and both are final/locked for MVP.
- A `Waived` invoice must not be included among outstanding **Pending/Due** invoices. Any payments actually collected for that invoice **remain included** in cash-collection reports.
- Waiving is **admin/owner-only**. The waiver requires a **reason**, **admin identity**, and **timestamp**. These belong to the invoice-level waiver action/metadata in the chosen MVP approach.
- Prior constraints remain: waiver settles **the entire remaining balance**; partial waiver **while keeping the invoice Due** is not part of MVP. A Pending invoice with ₹0 collected may be fully waived; a partially collected Due invoice may also be fully waived.

**SUPERSEDED FOR MVP:** The earlier convention “admin waiver settles the invoice by marking it `Paid`” is replaced with **`Waived`**. Do not silently continue using `Paid` for waived invoices in implementation or reporting.

### 1.2 Financial model — LOCKED, MVP concept

Maintain three distinct financial facts:

1. **Original billed total:** sum of the invoice's issued `invoice_lines` amounts; preserve the charge history.
2. **Actual money received:** sum of positive `payments` rows referring to the invoice; no waiver is represented as a payment.
3. **Forgiven balance:** on an invoice finalized as `Waived`, derive the waived amount as **original billed total − sum of actual payments** (for the MVP model without refunds/other financial adjustments).

**Example (agreed):**

| Record / figure | Value |
| --- | ---: |
| Original invoice lines total | ₹500 |
| Existing payment row(s) | ₹200 |
| Final invoice status | `Waived` |
| Derived waived amount | ₹300 |
| Remaining outstanding | ₹0 |

The invoice stores the **waiver reason, actor and time**; `payments` continue to store only actual receipts. A `Waived` status plus those facts is sufficient to calculate the waived amount for this **restricted MVP workflow**.

### 1.3 Waiver persistence — LOCKED MVP choice; implementation details OPEN

- **No separate `invoice_waivers` table is required for MVP** under the single-final-waiver-per-invoice model.
- Store waiver metadata with the invoice, **not** as an artificial ₹0/₹300 payment transaction.
- **OPEN — ENGINEERING:** Exact database column names/types, status constraints, backend transaction and concurrency implementation, enforcing admin-only authorization, and preventing receipts/edits after the invoice is finalized. No DDL was approved in the excerpt.
- **FUTURE REVISIT:** If later requirements introduce partial waivers while Due, multiple waiver events, reversals, or other adjustments, reconsider whether event-level waiver records become necessary. This future question does not overturn the MVP decision.

## 2. Services and clinic-specific pricing

### 2.1 Single-clinic MVP — LOCKED scope

- There is **only one clinic** in the current MVP deployment.
- **Defer use/creation of `clinic_service_prices`** and per-clinic price overrides/branch service-configuration workflows until a second clinic is introduced or clinic-specific pricing becomes necessary.
- The sole clinic still requires **editable prices** and service **enabled/disabled availability**. For MVP, the agreed simple direction is to keep those current values in the **`services` catalog**, avoiding a separate per-clinic price row merely to represent one clinic.
- An issued `invoice_line` must retain its own **billed price snapshot** (and identifying description/source as otherwise agreed). Editing a catalog price later must **not** change old invoice totals.

**SUPERSEDED FOR MVP:** Prior proposed or agreed *multi-clinic* setup steps, such as prepopulating inactive `clinic_service_prices` rows for each new clinic and independent clinic-by-clinic prices, are **not launch requirements** for the single-clinic MVP. They remain relevant to future migration planning, not present implementation.

### 2.2 Future migration and `clinic_services` integration — DEFERRED, explicitly requested

When another clinic is introduced, deliberately migrate from the single-clinic catalog arrangement to clinic-scoped service configuration:

- Revisit the earlier **`clinic_service_prices`** design and determine how pricing, enablement/availability, and clinic-service association should be represented.
- **Wire the future solution to the `clinic_services` table as appropriate to its eventual role.** The user explicitly requested preserving this integration/migration dependency. **Do not assume now** whether `clinic_services` owns availability, enrollment, pricing, or a combination, or that both tables will be needed.
- Migrate/seed the original clinic's existing service settings and prices without losing availability or creating ambiguous pricing authority; decide initialization for any newly introduced clinic.
- Update service lookup, billable-price resolution and backend access/availability enforcement to use the eventual clinic-scoped authority, while retaining previously issued `invoice_lines` unchanged.

**OPEN — FUTURE DESIGN:** Exact schema, naming, `clinic_services` responsibilities, coexistence or replacement of `clinic_service_prices`, ownership of base versus clinic prices, default activation of services in newly added clinics, migration order, and handling of future clinics. These **were not decided** by this excerpt.

## 3. Cross-check: what was and was not approved

| Topic | Outcome |
| --- | --- |
| Replace waiver-then-`Paid` with final `Waived` status | **LOCKED — MVP** |
| `Waived` final, locked, zero outstanding, absent from dues | **LOCKED — MVP** |
| Existing cash receipts retained in payment reports | **LOCKED — MVP** |
| Derive waived balance from billed total less real payments | **LOCKED — restricted MVP model** |
| Mandatory waiver reason, admin, and timestamp | **LOCKED — MVP** |
| Omit separate waiver events table in this MVP | **LOCKED — MVP choice** |
| One-clinic catalog prices and availability, with invoice price snapshots | **LOCKED — MVP direction** |
| Do not use `clinic_service_prices` yet | **LOCKED — MVP** |
| Later migration and wiring to `clinic_services` according to its role | **DEFERRED — FUTURE, explicitly tracked** |
| Physical schema, SQL migrations, UI implementation or production behavior | **NOT AUTHORIZED / NOT IMPLEMENTED HERE** |

## 4. Handoff notes

- This addendum takes precedence over **older contradictory MVP statements** only for the waiver status and the single-clinic catalog strategy described above. Do **not** erase the historical reasoning in prior decision logs.
- The **`Waived`** status should be included when designing final-state restrictions and settlement summaries; cash-collection reports must remain **payment-based**, rather than equating `Paid`/`Waived` status with money collected.
- Pricing migration is a **future requirement**, **not** a reason to implement inactive extra tables for one clinic now.
- Package and examination policies, standalone treatment source architecture, and other unrelated open items are **outside the scope of this excerpt**. None is inadvertently marked settled here.

---

*Document only. No existing Library file, code repository, Supabase project, or database was modified as part of drafting this addendum.*
