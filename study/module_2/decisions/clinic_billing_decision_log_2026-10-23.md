# Clinic Module 2 — Canonical Billing & Clinical-Records Decision Log

**Consolidated:** 22 October 2026 (Asia/Kolkata)  
**Document status:** **AUTHORITATIVE DISCUSSION REGISTER — not approved DDL, deployed-state evidence, or an implementation order.**  
**Scope:** Clinic demo backend Module 2: clinical visits and performed procedures; therapy pricing and fees; catalogs and billable-source references; invoices and invoice lines; packages; examination recurrence; checkout/payment/waiver; safety, testing, and migration.  
**Immediate next subject:** O06 branch service availability-versus-pricing model remains OPEN (one table versus separate tables). O05 billing-mode storage location also remains OPEN despite the settled one-mode-per-checkout invariant. Other examination/specialty occurrence questions stay open; no schema was approved.  
**Precedence:** This register supersedes *older discussion summaries and recommendations* where they conflict. Only a later **explicit user/clinic decision**, verified implementation evidence, or a discovered defect can change a current entry. Never treat an assistant proposal as approval.

> **One-source-of-truth promise and its boundary.** For the questions discussed in the retrieved source documents, consult this file first: it integrates decisions, original-proposal recommendations, unresolved work, rejected assumptions, and acceptance/audit requirements. Earlier Markdown files remain **historical evidence**, not competing current decision logs. This file does **not** prove what is installed in Supabase or what the clinic owner will ultimately authorize; real schema/data and disputed clinical/business policies still require verification. New conversation decisions should be entered *here*, with date, status, rationale, affected tables/workflows, and prior decision superseded.

## How to interpret status and priority

- **SETTLED:** An explicit decision about business behavior or architecture; do not reopen merely for elegance. Still needs implementation and testing unless separately indicated.
- **SETTLED — AUDIT PENDING:** Accepted architecture; verify concrete invariants, relationships, edge cases, and implementation feasibility. Reopen only on a demonstrated defect or meaningful new requirement.
- **ESTABLISHED DIRECTION:** Agreed behavior or principle; physical representation/precise contract not fully selected.
- **PROVISIONAL:** Tentative rule or observation requiring explicit owner/user confirmation.
- **RECOMMENDED (NOT APPROVED):** Proposed technical design, including the October 22 architecture-resolution document. It is available for implementation debate, **not** a user decision.
- **OPEN:** Deliberately unresolved decision. An issue being discussed does not make its proposed answer binding.
- **DEFERRED:** Not approved for this release. Domain examples do not imply a feature is in V1.
- **SUPERSEDED / REJECTED:** Old proposal or assertion must not be used as current requirement.
- **IMPLEMENTATION STATUS:** Unless otherwise independently verified: design conversation only. **No migration, database write, or application change was performed in this consolidation.**

## 0. Executive checkpoint — what is actually decided

### Stable directions

1. **A treated complaint on an occasion produces one `visit`.** Two complaints treated in one physical sitting produce two visit records. One complaint has many visits over time; no new generic physical-session table was approved. **[SETTLED]**
2. **Performed clinical procedures are occurrence records** (e.g. `visit_services`), distinct from catalog definitions. An ultrasound performed twice is two occurrences even with one service definition. Preserve occurrences when procedures are bundled into a flat fee. **[SETTLED]**
3. **Normal checkout is atomic:** create confirmed visits, procedure occurrences, issued invoice and its charged details together or commit none. Printing and subsequent collection are separable from that checkout. **[SETTLED]**
4. **One checkout invoice can cover multiple complaint-visits.** A single checkout has exactly **one ordinary-therapy billing mode** (`FLAT` or `PER_PROCEDURE`) across all its complaint-visits; no mixed modes within that checkout. In flat mode, charge once **per complaint-specific visit**, not once for the entire checkout and not once per selected procedure. Specialty charges are a separate, unresolved policy—not proof of mixed ordinary billing. **[SETTLED 23 Oct]**
5. **Modes:** `FLAT` (routine complaint ₹200; rehab complaint ₹300) or `PER_PROCEDURE` (charge performed procedures individually instead of another routine therapy flat fee). A clinic has one enforced mode for MVP; different clinics may differ. Naming/configuration and historical snapshot location are open. **[SETTLED policy; OPEN storage]**
6. **Use typed nullable foreign keys for invoice-line billable sources.** Reject polymorphic `source_type` + `source_id` without real FKs and a generic shared occurrence parent for present scope. This decision is final **as a direction** and remains open **only for technical audit**. **[SETTLED — AUDIT PENDING, 22 Oct]**
7. **Freeze original billed line amounts and descriptions at issuance**, including unpaid/Pending/Due invoices. Payments, approved waivers and write-offs affect settlement rather than quietly editing the original bill. A finally Paid invoice is locked, even to the owner, in MVP. **[SETTLED]**
8. **Prepaid treatment packages are in V1** (supersedes prior deferral). A package is activated after full payment, and a covered attendance still has an invoice with ₹0/covered-by-package representation. Purchase, entitlement, and redeemed session are separate facts. **[SETTLED behavior; OPEN precise model]**
9. **Examination fee policy has changed to a return-after-inactivity model**, **not** one fee per patient lifetime. The owner stated a **30-day** inactivity threshold measured from the patient's **most recent completed visit across the clinic chain**, regardless of complaint or payment status. Exact boundary/exception details remain open. **[ESTABLISHED with PROVISIONAL boundaries]**
10. **New checkout payment options include** full receipt → Paid; Pay Later → Pending; and receptionist-recorded *partial receipt* → Due without waiver. Old Pending/Due invoices are admin-controlled; waivers are admin/owner-only. Paid means settled (possibly partly waived), **not necessarily fully collected in cash**. **[SETTLED]**
11. **Financial truth is three different facts:** original charges, actual positive receipts, and deliberate waiver/write-off adjustments. Distinguish invoice issue time, real receipt time, and final Paid time; cash reporting uses actual receipts. **[SETTLED concepts; OPEN schema]**

### Decided versus *not* decided

**Distinct concepts that must not be conflated:** the **catalog-architecture B recommendation** (separate procedure definitions and fee settings), the **typed FK invoice-line source strategy** (settled), and **product-sale occurrence design** (open) are **three separate decisions**. Settling typed FKs does not itself finalize every catalog table, examination entity, or product event.

## 1. Clinical workflow, pricing, ownership and source semantics

**Normal workflow [SETTLED]:** select patient → select one or more complaints → select/record procedures performed for each complaint → one submission that persists all confirmed treatment and issued invoice in a database transaction. Receptionist usually enters treatment; clinician can also record clinical entries. Checkout rights for every role, including clinicians and administrators, are not fully specified. A frontend multi-step form does not justify separate database commits.

**Therapy [SETTLED]:** procedure picker selects legitimate performed clinical procedures, **not** automatically applicable administrative fees. The server decides charges using permitted catalog/configuration and patient/visit history. Under flat mode, routine performed procedures are included without separate procedure charges, yet remain documented. Under per-procedure mode, charge the actual performed occurrences; no additional routine flat therapy line. Separate specialty behavior, examinations and package coverage require their own rules. No invented allocation of ₹200/₹300 across included procedures.

**Scope of the single mode [SETTLED 23 Oct]:** the mode governs every ordinary **complaint-visit in one checkout**: `FLAT` yields one ordinary therapy charge **per treated complaint-visit**; `PER_PROCEDURE` yields separate charges for eligible performed procedures (from the relevant `visit_services` occurrences), with no extra ordinary flat charge. One checkout does **not** mix modes across ordinary complaints, by explicit user confirmation; mixed specialty add-ons are distinct and OPEN. This does **not** finalize whether historical `billing_mode` is stored on `invoices`, `visits`, or both (O05). It is **not** a requirement to reprice package or examination lines as therapy. A per-visit billing-mode override is not approved. Existing clinic-level mode configuration remains separate from where historical mode is snapshotted.

**Therapy tiers and prices [SETTLED rates; OPEN mechanics]:** routine pain treatment ₹200; rehab/accident/post-operative treatment ₹300. Who assigns routine versus rehab, where the tier is stored, branch price variation, authorized correction, and grandfathering/patient agreements are not finalized. Do not infer tier from the old `FIRST`/`SUBSEQUENT` visit markers or service `STANDARD`/`PREMIUM` labels. Proposed owner: complaint-course current tier, historical snapshot at visit; **not approved**. Proposed current-rate storage: on `clinics` for only two tiers; alternative pricing table remains possible based on actual variability.

**Procedure price resolution at atomic checkout [SETTLED behavior; implementation audit pending, 23 Oct]:** the agreed normal V1 workflow records `visits`, performed `visit_services`, invoice and lines in the same atomic checkout. For `PER_PROCEDURE`, server queries may join visits → visit_services → services → branch configuration/prices; multiple joins are not themselves an architectural defect. The checkout resolves each eligible procedure's applicable configured price, snapshots its amount/description into immutable `invoice_lines`, and persists the full result atomically. The performed-on-one-day/invoiced-on-another-day hypothetical is *out of scope for the ordinary V1 checkout*. Server must ensure consistent pricing under concurrent admin edits; atomicity alone does not ensure the price row cannot change between reads. Branch service eligibility and rates are addressed separately below; the older runtime base-price fallback option is **not** the chosen path after branch initialization was proposed and accepted.

**Branch availability and initialization [SETTLED workflow, OPEN physical schema, 23 Oct]:** service definitions in global `services` do **not** imply every branch offers them; machines/personnel may differ. For **V1**, creating a branch initializes a branch-specific entry for **each existing service**, taking its initial price from `services.base_price`, but with the service **inactive** by default. Admin activates only supported services and may edit each branch's own independent price. Existing branch values are **not automatically changed** by later edits to `services.base_price` (confirmed independence of overrides). Later onboarding versions may ask the administrator to select supported services during branch creation instead of activating them afterward; same underlying configuration can serve both flows. The intended checkout path reads a configured **branch price** and does **not** silently fall back to a global base price at runtime; missing/unavailable configuration should be treated as a validation/configuration problem. Exact `is_active` placement is OPEN: same `clinic_service_prices` row versus distinct branch availability/enrollment table. No DDL approved.

**Branch-level catalog visibility [SETTLED product requirement; backend authorization pending, 23 Oct]:** a receptionist should see **only active/available procedures in the branch they are authorized to operate**, rather than the chain-wide catalog. API checkout validation must independently reject unavailable/inactive/out-of-scope service IDs even if a client bypasses the UI. Branch identity/authorization must be server-validated, not trusted from a client-supplied ID. A disabled service should remain legible in historical occurrences/invoices (not deleted out of history). Proposed implementation tests: branch A active vs branch B inactive; incorrect branch ID sent manually; inactive or missing pricing row; base price 0 vs absent; concurrent price/admin updates; new branch starts inactive. The question of new-service propagation to existing branches is **OPEN operational detail**, with default-inactive initialization a sensible recommendation—not a separately confirmed business decision.

**Visit-to-invoice membership [SETTLED requirement; RECOMMENDED representation]:** several complaint-visits belong to one checkout invoice, including package-covered ₹0 visits. Proposed schema: `visits.invoice_id` with many visits → one invoice. It resolves no-standalone-charge visits more reliably than deriving membership from charged procedure lines. No one-invoice-per-day or one physical sitting per unique date restriction is approved. A join table or split billing must earn an actual use case. Matching invoice/visit context is mandatory if both membership and line references are stored.

**Money authority [ESTABLISHED DIRECTION, NOT YET SIGNED-OFF SCHEMA]:** `invoice_lines` is the best current candidate for the *original historical billed amounts*; catalog pricing is mutable reference configuration; `visit_services` records performed work; an invoice header total can be derived or a strictly synchronized cache. Existing stored visit/procedure money must not become competing authoritative sources. Original descriptions, quantities where applicable, actual price, and pricing-path snapshot should survive catalog edits. Exact line granularity, zero-line representation, and legacy column retirement remain open.

**Old `visit_services.is_charged` issue:** Oct 21 addendum explicitly favored removing the proposed `is_charged` flag because billing links explain separately billed procedures; the current baseline may still contain it. Oct 22 design *recommended*, but did not ratify, a server-derived `billing_disposition` (`INCLUDED_IN_THERAPY` / `SEPARATELY_BILLED`) so an unbilled occurrence is not ambiguously treated as included, waived or package-covered. **[OPEN reconciliation]** Retain a reliable explanation of *why* each performed event generated a line or did not; do not blindly treat absence of a charge as inclusion.

## 2. Catalog architecture, specialty, product scope and package offerings

### 2.1 Catalog and price definition architecture [OPEN; B PREFERRED]

- **A — unified billable catalog:** one definition table for fees, procedures, packages and products with `kind`; requires strict clinical-picker filtering and subtype validation. Earlier accepted Oct 21 but later **reopened**, so **NOT final**.
- **B — separate definitions:** keep `services`/`clinic_service_prices` for clinical procedures; fee configuration separate; packages/products acquire their own definitions if V1 needs them; unify billed facts using `invoice_lines`. Current **leading recommendation**, supported by original Oct 22 proposal and subsequent conversation. **Not independently ratified as a final catalog decision.**
- **C — shared billable-item parent plus specialized child definitions:** more generalized administration; needs additional subtype consistency and joins. Not justified merely to avoid several invoice-line FKs. **Not selected.**

**Existing baseline, not proof of deployed state:** inspected repository migration describes `services`, `clinic_service_prices`, `visits`, `visit_services`, `invoices` and patient-specific `packages`, not implemented `invoice_lines` nor generic `billable_items`. The baseline's service price override is not automatically an explicit branch-enrollment constraint. A single owner/chain can keep shared definitions while branches configure availability/prices. Future independent business-owned catalogs need separate ownership semantics; not a current rule.

**Specialty treatment [OPEN MVP boundary]:** e.g. cupping may be billed separately when supported, even if regular therapy uses flat pricing. This is **not approval** to put specialty in the normal complaint picker or force a standalone specialty into an artificial complaint-visit. Decide V1 exclusion versus separate execution workflow, optional clinical link, occurrence source and invoice grouping. Owner was consulted on 23 Oct and confirmed that certain specialty treatments (e.g., cupping, IASTM, laser) are separately chargeable; this does not settle invoicing or persistence. Two explicitly open questions: (1) include specialty charges in the ordinary checkout invoice or issue a distinct specialty invoice, potentially with its own UI/API; (2) store performed specialty occurrences in `visit_services` versus a dedicated table, and if `visit_services` is used, whether `visit_id` can be absent and what integrity rules change. If ordinary checkout must exclude specialty IDs, enforce that on the server. Legacy `PREMIUM` labels are not reliable proof of specialty workflow kind.

### 2.2 Products [OPEN; V1 inclusion NOT confirmed]

Clinic actually sells goods (belts/gels/equipment). Earlier discussions **deferred retail implementation**; later conversations explicitly examined *how* its source would be recorded, but that alone does **not** approve V1 inventory/returns/product-selling functionality.

- **Candidate P-A (MVP-lean):** `invoice_lines` directly records a product sale with a real FK to product *definition*, immutable name, quantity, unit price/line amount. It is a direct sale record, not a reference to another sale occurrence.
- **Candidate P-B:** a distinct product-sale occurrence table captures the transaction (possibly later inventory, returns or fulfillment); line refers to it. Adds consistency with procedure/package events and another lifecycle/table.
- **Open questions:** Is retail actually a V1 feature? Is stock movement/return/correction needed? What is the quantity and historical price model? Does a sale need an independent identifier outside the invoice line? **Do not decide from aesthetic symmetry alone.**

### 2.3 Packages [IN V1; occurrence and rule design OPEN]

Known offering matrix: routine 10-day, routine 30-day, rehab 10-day, rehab 30-day. **Tentative** 30-day prices: routine ₹4,500; rehab ₹6,000; **10-day amounts unknown**. “10/30 days” is not confirmed to mean calendar duration versus attended treatment days. Do **not** calculate missing prices by prorating.

**Established behavior:** purchase is a separately billed event; patient-specific entitlement/coverage is not the same as an offering definition; activate only when fully paid; a redeemed covered visit still produces its own invoice, generally zero-price/covered-by-package, and a traceable use/redemption. The purchase invoice and later redeemed visit invoice are not the same event. Multiple complaints, concurrent redemption and exhaustion need explicit scope. Existing `packages` baseline is a **patient-specific package/complaint record**, not a universal product catalog.

**OPEN:** offering definitions and versioned price, purchase occurrence and charge-source key, entitlement activation semantics (does a waiver count as "fully paid"? must confirm), complaint/tier/branch scope, redemption cardinality, expiry/session balance, overlap, refund/void, mixed covered and uncovered checkouts, paid package purchase versus Pending purchase, and how a covered visit is linked to source package. **No one invoice per day limitation.**

## 3. Typed source FKs in `invoice_lines` — SETTLED, AUDIT PENDING

**Selection made 22 Oct:** typed nullable **real** source FKs, with line-`kind`-specific validation. Accept several nullable columns and an occasional schema migration for a genuinely new *source kind*; catalog rows added under an existing kind do **not** require migrations. Rejected here: polymorphic `source_type/source_id` (loses ordinary relational FK check) and universal `billable_occurrences` parent (extra polymorphic-subtype complexity). **Do not reopen this debate without a real failed invariant/new requirement.**

**Crucial shape nuance:** each line has **one logical billed source**, but can contain multiple non-null identifiers for **source plus context**. An ordinary `PROCEDURE` line may have `visit_service_id` (the charged occurrence) **and** `visit_id` (its contextual parent). `CHECK (num_nonnulls(all FK columns)=1)` would incorrectly reject it. Validate by `kind`, and enforce occurrence's visit agrees with context. Different invoice kinds (THERAPY, PROCEDURE, EXAM, PACKAGE_PURCHASE, PACKAGE_COVERED, PRODUCT if sold) require a reviewed shape; only the first two examples have clear current semantics.

**Source candidates [the first three are original proposal shapes, not signed-off DDL]:**

| Line purpose | Real billable source | Optional/required context | Current standing |
| --- | --- | --- | --- |
| Flat ordinary therapy | `visit_id` | Same checkout invoice; tier/mode/amount | Concept established; implementation audit |
| Separately priced procedure | `visit_service_id` | `visit_id` must match occurrence | Concept established; implementation audit |
| Initial or repeat examination | Exam event or validated `visit_services` event | Shared-patient chain visit timing | **OPEN exam storage and eligibility** |
| Package purchase | Actual package purchase/entitlement acquisition | Purchaser, clinic, offering, purchase invoice | **OPEN** |
| Covered package attendance | Actual redemption and corresponding visit/package link | ₹0 reason and entitlement | **OPEN** |
| Retail product | Product catalog FK on direct invoice line **or** product-sale event FK | quantity, price and possibly stock event | **OPEN** |

**Mandatory audit gates before implementation approval:**

1. **Validity and logical shape:** all required source FKs exist; nullable pattern and `kind` agree; no mixed/ambiguous line source.
2. **Context ownership:** invoice, source, patient, clinic/branch, complaint course, occurrence and package purchase/redemption all refer to the same authorized checkout; don't rely on globally unique IDs alone. Composite FKs or equivalent transaction-verified protections required.
3. **Source uniqueness across invoices:** bill the *same occurrence* only once (global uniqueness, not merely within one invoice). Repeated ultrasound is still allowed with **distinct** occurrence IDs. Prevent duplicate package purchase charges and multiple redemption of the same entitlement session.
4. **Completeness:** every charge the accepted business policy requires appears once, no extras; a valid FK alone cannot prove a needed charge is present. Flat-mode routine procedures must remain clinical but **not** gain extra charges; itemized charges must be complete.
5. **All-or-nothing + retry/concurrency:** failure rolls back invoice plus treatment; concurrent checkout, lost-response replay, exam assessment and package redemption cannot double-charge or inconsistently allocate benefits.
6. **Historical immutability and deletion:** issued labels/amounts/context frozen; deletion/corrections may not orphan old bills. RLS/authorization and database constraints are distinct obligations.
7. **New package and ₹0 cases:** explicit acceptance for purchase, paid activation, redemption, exhausted package and multi-complaint coverage.

**Concrete PostgreSQL mechanisms [RECOMMENDED, NOT APPROVED/EXECUTED]:**
- Per-kind row `CHECK` constraints for source shape; FKs prove target existence; composite FK/UNIQUE chains prove association when possible; some cross-row/business invariants need transactional validation or deferred constraint triggers (a row CHECK cannot query other tables).
- Partial uniqueness for flat therapy source `visit_id` where `kind='THERAPY'`; unique charged `visit_service_id`; equivalent specific indices for exam, package purchase/redemption and product events where those entities exist. Do not make `service_id` unique on lines or all `visit_id` uses unique—those would ban valid multiple procedures.
- Database-generated line amount/description snapshots, invoice/line reconciliation at commit, restricted issuance edits, and no cascade deletion of source history.

## 4. Examination fee policy — newer owner rule wins

**Current owner information [ESTABLISHED]:** recurrence is possible after **30 days without a completed visit**, across the clinic chain and all branches, using the **same canonical patient identity**. The relevant anchor is last **completed visit**, not last paid invoice or last examination. An unpaid, zero-value, or waived event does not vanish clinically. A return examination is not patient re-registration. A lifetime `UNIQUE(patient_id)` assessment constraint is **invalid** under this rule.

**Provisional boundary:** charging starts on **day 31** (more than 30 full days) instead of on exactly day 30. Need owner confirmation of **calendar-day versus exact timestamp** calculation and time zone. “Completed visit” requires a clear definition: how to treat cancellations/no-shows, zero-price package treatments, same-day repeat, non-treatment consultation, retroactively entered visit and cross-branch records. Is trigger patient-wide for **any complaint**? Current interpretation yes, **verify owner policy**. First-ever exam trigger and manual exemptions/override authority are not finalized.

**Fee amount:** ₹350 was the earlier **discussed** examination amount; not independently reconfirmed with the new 30-day rule. A distinct **₹150 gap-return fee** was also mentioned in prior discussions; **do not double-charge or silently equate it to the recurring examination fee** until the owner clarifies whether the old gap fee was replaced, survives separately, or is an alternative. A one-time lifetime initial exam proposal is SUPERSEDED, not a default fallback.

**Occurrence storage [OPEN]:** either record an exam under `visit_services` with appropriate service/type semantics, or use a dedicated repeatable examination event table. If dedicated, it must **not** have lifetime unique patient ID. Preserve exam eligibility and occurrence independent of whether its invoice is Pending/Paid or waived. Concurrent cross-branch checkouts and historical imported patients may require serialized patient-row eligibility evaluation and evidence for last visit. The original proposal's one-lifetime `initial_exam_assessments` design and unique index must **not** be copied verbatim.

## 5. Payment states, roles, waivers and cash reporting

### 5.1 Domain invariant [SETTLED concept]

```text
Original billed = SUM(immutable invoice-line charges)
Collected       = SUM(actual positive receipts against this invoice)
Waived/written off = SUM(authorized settlement adjustments)
Outstanding     = Original billed - Collected - Waived    [no refund/credit case modeled]
```

These are separate accounting facts. Do not create a payment row for ₹0 received or for a waiver. **Paid means no outstanding balance after real receipts and approved waiver**, not proof of full original price collected. Price mistakes and clinical mistakes are **not automatically corrected** by recording a waiver. Refund/void/credit mechanisms are deferred.

### 5.2 Accepted flows and authority [SETTLED behavior]

- **At issuance:** new invoice enters Pending as the workflow starting status, then reception handles the current checkout's collection: full payment + method → Paid; Pay Later → Pending; **partial actual payment** + method → Due with remainder collectible. Reception cannot waive. The older two-button-only formulation is **SUPERSEDED** by the explicit approved initial partial-payment path (Oct 18 log).
- **Existing Pending/Due:** only admin/owner handles later money, status and waiver; receptionist cannot modify an older invoice. A clinician's payment privileges are not established.
- **Admin settles existing Due:** record an additional received amount between ₹0 and the outstanding balance; **Mark Paid** may waive the remaining balance. Example: billed ₹500, received ₹200 previously, admin receives ₹0 and waives ₹300 → Paid with collected ₹200, waived ₹300. ₹0 is not a payment record.
- **Due after initial part payment:** receptionist collecting ₹300 of ₹500 during **that** checkout may leave ₹200 Due without waiver. **Open:** whether admin may later take another ₹100 on an old Due invoice and *keep remaining ₹100 Due*, instead of finalizing/waiving. Do not silently assume that path from schema capability.
- **Paid lock:** final Paid means no more edits to original charges, received records, methods or waivers even by owner; unpaid issued original charges are *also* frozen at issuance. Exact lock enforcement and handling of erroneous Paid entry remain technical/policy work.
- **Waiver timing correction:** obsolete "waiver only before any money is paid" is false. Owner may waive a residual Due **after** earlier partial receipt, before final Paid lock. Open whether waiving partially while remaining Due is allowed, or only during final settlement.
- **Old debt at next visit:** staff manually informs admin of money earmarked for an older invoice; no automated notification, wallet/unallocated-receipt workflow or automatic cross-invoice allocation in current MVP. Actual cash may temporarily exceed recorded old-invoice receipts until admin posts it; cannot describe this as automatically reconciled.

**Strong schema direction, not final approval:** common `payments` table with one strictly positive row per *actual* receipt, linked to invoice and containing amount, method, actual `received_at`, `recorded_at`, `recorded_by`; zero/many records per invoice. The precise payment lifecycle, idempotency, correction rules and DDL remain open. One invoice-level waiver flag/tag was expressly reserved for later discussion; a boolean without waiver amount/reason/time/actor is insufficient financial evidence. Waiver versus loss/bad debt category is likewise open. General paid-record audit-log/correction system was intentionally excluded from MVP.

### 5.3 Timestamps and reporting [SETTLED distinction; OPEN dashboard definitions]

- Invoice `created_at` / `issued_at`: original bill event.
- Payment `received_at`: real cash/UPI/card receipt time; cash collection reports use this.
- Payment `recorded_at`: database record time (recommended for delayed admin entry).
- Invoice `paid_at`: time bill reached final settled/Paid status, including potentially a waiver-funded settlement. Never use invoice `updated_at` as a substitute for receipts.

**Reporting clarification:** “Exclude Pending from EOD” refers to the discussed **paid/settled-invoice report**, *not* every financial report. A ₹300 receipt on an invoice still Due is real cash and must appear in receipt-date collections if that reporting view is used. Pending/Due should remain visible in outstanding collections reporting. Billed revenue, actual cash received, waivers and invoice counts are **different measures**. A paid invoice with a ₹100 waiver and ₹400 receipt contributes ₹400 actual cash, not ₹500. Exact report taxonomy, late backdating authority and daily cut-off time zone are open.

**Operational scope:** owner reviews unpaid bills manually; automatic chasing/escalation is not required in MVP. Payment-method list, split tender, notes, admin-first review UX, waiver source records and whether a new Pending invoice may be fully waived without any receipt are open.

## 6. Checkout and security implementation blueprint — from proposal, NOT ratified DDL

The original October 22 architecture-resolution proposal contained detailed mechanics; retaining them here **as audit-grade recommendations** prevents losing technical requirements while also preventing their mistaken elevation to settled product decisions.

**Proposed safe transaction:** one authorized database RPC/transaction creates invoice, complaint-visits, performed events, exam event (if eligible) and lines; derive all money/pricing on server; publish invoice only after validation. Payment/settlement may run in its **own authorized atomic transaction** after issuance. Do not assume several browser writes share one database transaction, even if they are one frontend submit action.

**Idempotency and concurrency [required behavior, exact method OPEN]:** preserve a stable `checkout_key` across timeout/retry; canonicalize/hash meaningful request input; uniquely scope key at least by clinic; replay an already committed matching request returns the *same* invoice; same key/different payload is rejected. Uniqueness + a transactional lock prevents parallel duplicate inserts and invoice-number side effects. A fresh key can still represent accidental duplicate treatment, so stable intent and useful recovery are essential. Checkout invoice numbering and price/config edits need coherent concurrent behavior.

**Original design suggestion, not mandate:** read/lock clinic for numbering and replay; lock canonical patient for cross-branch examination eligibility; resolve active service, branch enrollment and clinic pricing under an appropriate isolation/locking/version policy; create invoice/visits/occurrences/lines; validate source shape, completeness, totals and immutable snapshots before commit. Preserve consistent lock order. Do not hard-code the proposal's now-invalid lifetime-exam unique row.

**Validation requirements:** backend checks clinic/chain access, patient and complaint ownership, procedure status/workflow kind, branch availability, clinic mode, source ownership, fee eligibility and package entitlement. RLS is not equivalent to a source-validity invariant; privileged/security-definer RPCs require explicit caller authorization. `services` entries should not become billable just because frontend sent a guessed ID. Old catalog names and prices cannot replace issued historical snapshots.

**Open implementation questions:** actual RPC/API shape; deterministic request hash; exactly where issue and payment transactions divide; payment retry key; unique/index design; completeness validator and locking performance; how historical invoice-number trigger acts on conflicts; constraint deferrability and API authorization. **No live transaction tests or concurrency tests have been executed as part of this log.**

## 7. Migration and actual-schema reconciliation — required before coding, NOT DONE

The October 22 architecture-resolution doc inspected **repository documentation**, not a deployed Supabase database. Its existing-schema notes are useful **hypotheses to verify**:

- `services` and `clinic_service_prices` contain procedure definitions and branch rates; branch availability/activation semantics may not match the proposed explicit enrollment model.
- `visits` currently includes patient, clinic and complaint references, old type labels and monetary component/total fields. `visit_services` includes service snapshots, `is_charged` and amount fields. `invoices` has a nullable, non-unique `visit_id`. Current `packages` describe individual patient entitlements.
- The inspected migration did **not** define `invoice_lines`, a shared `billable_items` parent or a generic occurrence parent. Do not claim proposed tables exist until independently checked.
- The existing invoice `visit_id` is non-unique; historical rows may already link several invoices to one visit. A new `visits.invoice_id` cannot be reverse-filled by picking a random invoice.
- Visit totals generated from old money columns do not prove agreement with actual performed services or the invoice. Compare stored components and preserve historical discrepancies for explicit reconciliation rather than silently repricing old invoices using today's catalog.
- `FIRST/SUBSEQUENT` is not `ROUTINE/REHAB`, and an old consultation fee does not unambiguously prove a first examination, a repeat exam, or a therapy charge. Legacy patients without documented earlier exams/visits need a safe eligibility policy.
- Existing frontend mock/pricing paths and checkout implementations may calculate from current catalog and disagree with immutable invoice lines; reconcile actual code and integration contracts at implementation time. Historical architecture review referenced overlapping frontend PRs, but their exact current statuses must be rechecked rather than copied as live facts.

**Proposed migration order (NOT an executed migration):** (1) ratify release scope/rules; (2) inspect deployed schema/data and active frontend/backend write paths; (3) add new nullable fields/line or occurrence tables without destroying old data; (4) reconcile/backfill provable evidence and flag ambiguous history; (5) add and validate constraints, concurrency/retry tests and line/total reconciliation; (6) cut over checkout, invoice printing and daily ledger reads to new charge authority; (7) retire legacy columns only after measured safe compatibility. Package V1 and repeat-exam policy require revising the older proposal's originally narrower sequence.

**Non-authorization:** no code changes, SQL execution, schema migration, PR rebases or Git state changes are authorized by making this document authoritative.

## 8. Review / acceptance scenarios (reasoned, NOT executed tests)

Convert these into DB/API integration tests after exact business rules and schemas are settled:

1. **Two flat routine complaints:** knee + shoulder visits in one checkout; two `THERAPY` lines at ₹200 each; routine procedures still recorded; total ₹400, absent exam/package/specialty.
2. **Same treatments in per-procedure clinic:** each performed occurrence creates its own price-snapshotted line; no flat therapy lines. Three occurrences with illustrative ₹100 + ₹80 + ₹100 produce ₹280, not a binding catalog rate.
3. **Repeated ultrasound:** two distinct `visit_services` IDs may each be billed exactly once even with one `service_id`.
4. **Procedure misuse:** inactive/nonenrolled/specialty-in-ordinary-workflow service ID submitted via API is rejected; no clinical or financial partial commit.
5. **Context spoof:** billed procedure/visit from another patient, branch or invoice is rejected.
6. **Completeness:** a flat checkout that also charges included routine procedures, or a per-procedure checkout that omits a required line, fails final-state validation.
7. **Historical rate/edit:** changing name, clinic mode, procedure price or availability after issuance must not alter stored original descriptions/amounts and clinical occurrences.
8. **Timeout and concurrent retry:** same stable checkout identity returns same invoice; no duplicated treatment, lines, numbered invoices or payments.
9. **Examination concurrent qualification:** across branches, the new 30-day rule yields at most one appropriate exam fee for the qualifying clinical event, with fee eligibility determined by actual completed-visit history, not Paid state. Requires fresh concurrency test design; the original **lifetime-only** two-branch test is obsolete.
10. **Package purchase:** package offering, purchase occurrence, invoice charge, collected payment, and activation are distinct and correct; a partial payment does not accidentally activate package.
11. **Covered attendance:** permitted redemption links to right package entitlement/complaint and own checkout invoice with ₹0/covered reason; repeated redemption cannot consume same session twice. Mixed coverage needs approval.
12. **Receptionist partial:** ₹500 billed, ₹300 received, ₹200 Due, no waiver, positive receipt with actual method/date; older Due edits unauthorized.
13. **Owner writes off old Due:** earlier ₹200 receipt against ₹500; later admin receives ₹0, waives ₹300 and makes Paid; actual cash totals remain ₹200, not ₹500.
14. **Reporting boundaries:** Pending invoice is not shown as settled; a receipt on Due still contributes to cash collected on received date; an all-waiver Paid invoice produces no fictitious cash.
15. **Legacy migration ambiguity:** an invoice with multiple historical visit links or mismatched stored totals fails automatic backfill and is flagged rather than fabricated.
16. **Retail if V1 approved:** several units of same product produce one unambiguous itemized line with preserved quantity, pricing and source; ensure absence/presence of inventory workflow matches the chosen architecture.

## 9. Open decision register — do these one at a time

**P0 = business decision needed before dependent V1 implementation. P1 = engineering/audit design after corresponding business rules. P2 = deferred/not launch-blocking unless explicitly moved into V1.** Status values below are **not** implicit recommendations.

| ID | Priority | Decision or verification still required | Status and dependency |
| --- | --- | --- | --- |
| O01 | P2 | Retail sales, including product-sale occurrences vs direct invoice lines, quantity/returns/stock semantics | **DEFERRED — OUT OF V1 (confirmed 23 Oct)**; do not add product-sale tables or product source FK solely for V1 |
| O02 | P0 | Examination event under `visit_services` or a dedicated repeatable table; whether it is patient-wide instead of complaint-visit-scoped; whether an examination can occur without complaint treatment; correct invoice/source association and no duplicate exam charge across two complaint-visits in one sitting | **OPEN — MODELING GAP LOGGED 23 Oct**; do not invent a dummy complaint visit or choose a table yet |
| O03 | P0 | 30-day threshold: exact boundary, date vs timestamp; qualifying completed visits, any-complaint scope, exemptions, first visit and manual override | **OPEN / PROVISIONAL**; clinic-owner validation |
| O04 | P0 | Is ₹350 repeat-exam fee distinct from old ₹150 gap-return fee? Any fee stacking, price and branches? | **OPEN**; do not assume coexistence |
| O05 | P0 | Billing-mode historical authority: invoice only, visit only, both with verified consistency or derived history | **OPEN storage**; **SETTLED** one ordinary mode per checkout (23 Oct), flat charge one per complaint-visit; invoice-level snapshot favored but not explicitly ratified |
| O06 | P0 | Catalog architecture B finally accepted or amended; procedure eligibility, branch enrollment and fee-price ownership; `clinic_service_prices.is_active` vs separate branch service availability table; new-service rollout to existing branches | **OPEN physical modeling**; **SETTLED V1 branch onboarding:** initialize all service prices inactive, admin later activates; receptionist filters to branch-active only; no runtime price fallback desired |
| O07 | P0 | Invoice-line monetary authority/printed granularity; included and zero-price line semantics; visit-level totals; clinical `is_charged`/disposition transition | **OPEN schema**; **SETTLED** resolve branch-configured price at atomic checkout and snapshot original amount/description; consistency during concurrent edits for audit |
| O08 | P0 | Package V1 definition, purchase entity, invoice link, activation, validity, redemption and covered ₹0 invoice semantics; 10-day prices | **OPEN**, inclusion/coverage behavior settled |
| O09 | P0 | Exact new/old payment state transitions: admin additional part payment while keeping old Due; Pending 100% waiver; ₹0-to-Due; method, split tender and actor rights | **OPEN**; receptionist NEW partial→Due is settled |
| O10 | P0 | Payments-table final ratification; payment/waiver amount persistence, waiver flag, reason, loss/write-off distinction, old-Due correction handling | **OPEN**; financial distinctions settled |
| O11 | P0 | Specialty V1 scope and occurrence model: standalone specialty treatment may have **no associated complaint-visit**, so where does its occurrence live, and how does its invoice line reference it? Also decide same checkout invoice vs separate specialty invoice/workflow/API; and `visit_services` with optional/no `visit_id` vs a dedicated specialty occurrence table, including data-integrity consequences | **OPEN — MODELING GAP LOGGED 23 Oct**; do not force specialty into `visit_services`/a fabricated visit |
| O12 | P0 | Therapy tier assignment/approval; patient-specific negotiated-rate scope and priority; clinic price-setting rules | **OPEN**; nominal ₹200/₹300 rates settled |
| O13 | P1 | Audit typed FK line shapes with composite ownership, source uniqueness and package/exam/product concrete examples | **SETTLED — AUDIT PENDING**; reopen only on demonstrated flaw |
| O14 | P1 | Exact transaction and idempotency keys for checkout and payments; locking, retry, invoice counter, client/server trust and completeness | **OPEN technical**; atomic checkout behavior settled |
| O15 | P1 | Paid/issued lock DDL, negative balances, role/RLS enforcement, audit/waiver authorization and deletion/correction constraints | **OPEN technical**; behavior settled |
| O16 | P1 | Payment receipt, entry, settlement and service dates, late admin recording, EOD/day-boundary and reporting categories | **OPEN details**; cash-date distinction settled |
| O17 | P1 | Legacy schema/data migration: visit–invoice duplicates, charged values, old types, historical examination recency and package entitlements | **NOT STARTED**; inspect deployed DB first |
| O18 | P1 | Full package/fee/checkout acceptance + concurrency tests against implementation | **NOT STARTED**, scenarios in §8 |
| O19 | P2 | Formal refund/void/credit/correction workflow for already Paid entries, and general audit trail | **DEFERRED** MVP Paid lock retained |
| O20 | P2 | Doctor-level/branch-local examination recurrence and mixed-mode clinics; multi-business private catalogs | **DEFERRED**; not a reason for speculative tables now |
| O21 | P2 | Automated old-debt notifications/unallocated-money wallet/one physical receipt allocated among invoices | **DEFERRED**; manual handoff chosen |

**Recommended sequence (updated 23 Oct):** close O06 (availability/pricing co-location vs separate table, future new-service rollout), then O05 (historical billing-mode storage), then O07 (invoice-line detail), while retaining O02–O04 examinations, O08 packages, O09–O10 payments, O11 specialty, O12 price/tier policy, and O13–O18 technical audit. O01 retail is already deferred beyond V1. **Order is advice, not an approval of unchosen schemas.**

## 10. Original proposal-to-log reconciliation — every major area represented

The original `clinic_billing_architecture_resolution.md` (22 Oct) is a **detailed recommendation**, not a ratified schema. The checklist below preserves its useful material and documents where newer decisions overturned assumptions. It is a *coverage map*, not percentages of implementation completed.

| Original proposal block | Canonical location | Outcome after comparison |
| --- | --- | --- |
| B vs A/C, definitions versus occurrence versus charge | §§0, 1, 2 | B remains preferred; typed invoice-source FKs explicitly settled; catalog B not independently final |
| Example offerings; branch/catalog ownership; specialty scope | §§1, 2, 9 O06/O11 | Procedure/price concepts retained; retail/specialty V1 inclusion not inferred; packages now V1 |
| Clinic billing policy, flat rates, therapy tier and negotiated rates | §§0, 1, 9 O05/O12 | ₹200/₹300 and clinic policy retained; applied-mode location/tier/agreements still open |
| Invoices, visit membership, charge-line kinds and source FKs | §§1, 3, 9 O05/O07/O13 | Multi-visit invoices and typed FKs retained; exact other sources and line schema remain open |
| One-lifetime exam record and `UNIQUE(patient_id)` | §4 | **SUPERSEDED** by recurring 30-day completed-visit policy; do not copy original DDL/tests |
| Composite FK, uniqueness, completeness and historical snapshot | §§3, 6, 8 | Imported as technical audit gates/proposals, not claimed implemented |
| One DB checkout transaction, retry locks and stable key | §§0, 6, 8 | Atomic business behavior settled; exact RPC, locking and keys open |
| Nine original acceptance examples | §8 | Retained applicable examples; obsolete lifetime exam rewritten; package/payment cases added |
| Migration phases, baseline hazards, current schema/PR evidence | §7 | Preserved as preflight checklist; repository snapshot is not deployed proof |
| Remaining decisions (catalog, specialty, rates, exam, fees, payment) | §9 | Reclassified; packages moved to V1; original exam rule replaced; product/retail scope distinguished |

**Eight-area discussion coverage checkpoint (from previous log, revalidated as a *discussion* label):** catalog architecture **substantially discussed, not ratified**; visits/pricing **partially closed**; invoice structure **partially closed**; database integrity **not audited**; checkout/retry **atomic behavior settled, detailed retry not audited**; ordinary worked cases **substantially discussed, package cases untested**; migration/implementation **not started**; outstanding business rules **partially closed**. Do not convert “2 of 8 substantially reviewed” into “25% complete”; it is not an engineering progress measurement.

## 11. Supersession and discrepancy audit — no competing current truths

| Older entry / appearance of conflict | Binding treatment in this register |
| --- | --- |
| Oct21 unified `billable_items` catalog was called settled; subsequent exploration reopened it. | **Catalog A NOT FINAL**. B preferred for now; do not claim a chosen shared catalog. |
| Original Oct22 architecture recommends B and typed FKs together. | **Only typed invoice source FK choice explicitly SETTLED** on Oct22; do not mislabel catalog B as separately locked. |
| Earlier proposal deferred **packages** from V1. | **SUPERSEDED**: packages mandatory V1; new acceptance/migration work required. |
| Earlier proposal deferred **retail**. New discussion compares its reference models. | Retail **scope still unconfirmed**; its design discussion alone does not make it V1. |
| Original `initial_exam_assessments(patient_id UNIQUE)` lifetime exam. | **INVALID** for current 30-day repeat policy. New event and eligibility design pending. |
| Earlier initial examination defined as lifetime ₹350 and gap fee ₹150 discussed. | ₹350 is historical suggested rate; fee interactions and ₹150 category remain **OPEN**. |
| Old checkout only offered full payment or Pay Later. | **SUPERSEDED** by initial receptionist partial-payment→Due, no waiver. |
| Old Due meant admin had personally reviewed. | **SUPERSEDED**: Due may be created directly by receptionist for a NEW invoice's partial collection. |
| Earlier “waiver only before any payment.” | **SUPERSEDED**: owner can waive remaining Due after prior partial payments, before Paid finality. |
| “Paid” equals whole original bill in cash. | **FALSE**: Paid is settled via collection plus authorized waiver; cash tracked separately. |
| Earlier only Paid invoices were locked. | **STRENGTHENED**: original issued charges immutable even Pending/Due; Paid additionally locks settlement records. |
| Earlier Oct18 combined checkout left atomicity open. | **SUPERSEDED** by later explicit single transaction decision. |
| Older logs treated visit/invoice membership and source shape as entirely undecided. | Requirement now: multi-visit invoice, typed FKs settled; exact `visits.invoice_id` is still implementation proposal. |
| Old `is_charged` removal versus new `billing_disposition` idea. | No binary flag approved as final schema; ensure explanation of included/covered/charged occurrences before implementing. |
| EOD excludes Pending versus receipt reporting shows partial Due cash. | Different report measures; Pending excluded from **settled** reports, not from outstanding/actual-cash reporting. |
| “Already implemented” language in technical proposals. | **No implementation claimed**. Proposal's SQL/constraints and code reviews are discussion evidence, not executed checks. |

**Known unresolved contradictions, not hidden by this consolidation:** repeat exam versus original once-lifetime data model; distinct old gap fee versus new return-examination concept; payment/waiver-funded package activation; retail's V1 boundary; `is_charged`/disposition authority. These are explicitly queued above, not silently solved.

## 12. Provenance, maintenance and scope limits

**Compared reference material (historical, not active competing decision records):**

- **[P]** `clinic_billing_architecture_resolution.md`, 22 Oct 2026, original proposal: architecture A/B/C, proposed DDL/source constraints, one-lifetime exam assumption, retry design, acceptance, migration and branch evidence. **Proposals remain recommendations** unless later accepted.
- **[L]** `clinic_billing_decision_log_2026-10-22.md`, 22 Oct, the immediately preceding live decision log: accepted clinic/visit/package/30-day rules, typed FK decision, open questions, eight-area coverage and audit flags. **This canonical version replaces it at the same Library path.**
- **[E]** `clinic-billing-decision-log-2026-10-22.md`, earlier Module 2 discussion record: originally reopened catalog architecture, clinic rates, specialty/retail deferrals and invoice unresolved items. Later decisions may supersede it.
- **[A]** `clinic_billing_decision_addendum_2026-10-21.md`: earlier short-lived unified-catalog signoff, pricing context, `is_charged` discussion and package offering examples. Superseded where listed.
- **[C]** `clinic_checkout_billing_decision_log_2026-10-18.md` (including 21 Oct note): full payments/waivers/RBAC scenarios; materially preserves the latest **receptionist initial partial → Due** decision and admin-old-Due limitations.
- **[T]** User-supplied SRT discussion through 22 Oct, covering typed-FK pros/cons, rejection of polymorphic/shared-occurrence alternatives, explicit **SETTLED — AUDIT PENDING** status, and user instruction to consolidate against original proposal.

**Evidence ordering for this document:** latest explicit user choice or direct clinic-owner input (with owner-uncertainty flagged) > earlier explicit choice not superseded > established conceptual direction > recommended architecture > hypothetical scenario. An older recommendation cannot overrule a later factual clarification. User and owner decisions are still separate: an owner-policy boundary marked provisional needs the owner's confirmation.

**What this audit actually did:** read the full available original Oct22 Markdown proposal and current Oct22 log; re-examined the relevant prior Oct18/21/22 decision records and the supplied typed-FK conversation; mapped substantive proposal areas into §§0–11; identified revised scope and payment conflicts; preserved unanswered questions and test obligations. **What it did not do:** independently re-open the original clinic-owner interview, inspect live Supabase data, execute SQL, retest current GitHub PRs, or make unexpressed business approvals. Therefore this is one **decision-discussion** source of truth, not a source of truth for runtime/deployment state.

### Change history

- **18 Oct:** Payment lifecycle, receptionist initial partial payment, admin-only old-Due/waiver, locks and cash dates captured in older consolidated log.
- **21 Oct:** Unified catalog briefly accepted then reopened; applied-mode location and product/package context carried forward.
- **22 Oct earlier:** Detailed B architecture proposal generated with lifetime-exam assumption and packages initially deferred; newer owner information and V1 direction necessitated corrections.
- **22 Oct later:** 30-day examination policy superseded lifetime-only assumption; packages required in V1; typed nullable source-FK approach explicitly **settled for MVP and open for audit**.
- **22 Oct this pass:** Compared and merged original proposal plus prior decision documents into this canonical register; restored missing payment permissions; separated settled FK from preferred catalog; resolved supersession labels; imported concrete technical checks without claiming approval; added original-to-log coverage, risk backlog, package cases, and explicit remaining verification boundaries.

- **23 Oct further discussion:** confirmed one ordinary billing mode per checkout, per-complaint flat billing, atomic price snapshots, initial all-inactive branch pricing copies, branch-specific price independence, and receptionist branch-active-only catalog visibility; availability-vs-pricing table design expressly OPEN.

**Future maintenance:** update THIS document. For each decision: ID/date, exact requirement, status, rationale, superseded item, source/owner approval (if applicable), data/API/workflow impact, and test/audit coverage. Preserve older files solely as provenance; do not make parallel active decision logs.


## 13. Decision updates — 23 October 2026 (incremental; do not infer approval)

- **O01 retail/products — DEFERRED beyond V1 (explicit user decision).** Do not build product catalog/sale occurrence/quantity/stock/returns solely for this release. The earlier B-versus-C product-source discussion remains historical exploration, not a V1 implementation requirement.
- **O02 examination occurrence — OPEN, gap identified.** `visit_services` requires a complaint-specific `visit`. A patient-level examination might span multiple complaints, potentially with two visits in one sitting, or occur without complaint treatment. Attaching it arbitrarily to one complaint could misstate clinical provenance or make a repeated examination charge. Guardrails to test later: one exam charge per qualifying attendance; valid patient/branch/invoice association; correct 30-day gap logic; exam charge not swallowed by flat-therapy inclusion. Whether examinations are in `visit_services` or a repeatable standalone event table is **not decided**. The treatment-free examination example is hypothetical, not yet confirmed clinic behavior.
- **O11 specialty occurrence — OPEN, gap identified by user.** A specialty service such as cupping may be performed without a complaint-specific `visit`. The current `visit_services(visit_id)` path may then be inappropriate. Resolve standalone occurrence identity, optional complaint linkage, invoice-line source and whether specialty belongs in V1. **Do not manufacture a visit solely to satisfy a foreign key.**
- **O11 specialty — TWO QUESTIONS EXPLICITLY LEFT OPEN (owner consultation reported by user, 23 Oct):** (a) Should separately charged specialty therapy share a checkout invoice with ordinary treatment, or use a separate invoice and possibly separate UI/API? (b) Should actual performed specialty occurrences live in `visit_services` or in a dedicated table? If kept in `visit_services`, a standalone specialty treatment may have no `visit_id`, challenging the existing required visit linkage and requiring revised ownership, constraints, and billing-source handling. Concrete examples given: cupping, IASTM, laser; exhaustive catalog choice is not relevant yet. No conclusion or schema change authorized. This was raised while testing whether a single checkout could mix billing modes; it does **not** by itself establish mixed flat/per-procedure billing.
- **Typed source foreign keys — SETTLED, AUDIT PENDING.** These two gaps affect *what source entities exist*, not the already accepted mechanism for referencing them.
- **Discussion sequencing:** defer these two modeling decisions as requested; next independent short decision can be O05 (where the historical billing-mode snapshot lives). No code or DDL was changed.


### 23 October 2026 — billing-mode, branch pricing and catalog availability (latest discussion)

**Decision boundaries (do not merge statuses):**

1. **[SETTLED] One ordinary billing mode per checkout.** One checkout does not mix `FLAT` and `PER_PROCEDURE` across treated complaints. Flat-rate generates a therapy charge **for each complaint-specific visit**. Per-procedure mode charges eligible performed occurrences. The historical storage field location remains O05 OPEN. The user considered invoice-only storage earlier; it has not been explicitly selected as final.
2. **[SETTLED] Price snapshot at checkout.** The normal treatment and invoice are written atomically, so there is no ordinary V1 delay between clinically recording procedure occurrences and invoicing them. `PER_PROCEDURE` may traverse `visits` → `visit_services` → service/catalog → branch-configured prices via joins; invoice lines retain issued price and description. **[AUDIT]** concurrent price edits, consistent read/lock/version semantics, applicable branch, correct billable-vs-performed filtering, authorization, and missing-price rejection.
3. **[ALTERNATIVE CONSIDERED, NOT SELECTED] Runtime fallback to `services.base_price`.** Earlier suggestion: branch override if present else global base; risk: unconfigured branch service becomes silently billable. Later user suggestion is to initialize branch-specific pricing rows in advance so fallback is unnecessary. Do **not** treat both mechanisms as simultaneously settled requirements.
4. **[SETTLED V1 ONBOARDING]** Create a price/config row for each global service upon new-branch creation, copy the initial base price, mark it **inactive**, and let admins activate only supported services and adjust prices. Branch-specific values subsequently remain independent of the global base. **[FUTURE UI]** Select supported services at branch creation rather than enable afterwards; physical storage need not change solely for this UX difference.
5. **[SETTLED UI REQUIREMENT]** Receptionist sees **only active services at their authorized branch**, not all chain services. **[REQUIRED BACKEND GUARDRAIL]** the same availability/branch authorization must be enforced server-side, not only hidden in a dropdown. Branch-specific absent/disabled services must not be casually chargeable; preserve historical records when a service is disabled.
6. **[OPEN — O06] Availability versus pricing schema.** Can `clinic_service_prices` safely store both `is_active` and the branch price, or should branch-service enrollment/availability be separate from price configuration? Think through temporary disablement, independent branch status, pricing changes, adding services after branches exist, and unique/consistent records. The prior assistant preference for `is_active` in the price table was **a suggestion**, and user explicitly left it OPEN for brainstorming.
7. **[OPEN OPERATIONAL DETAIL]** For new global services created after several branches exist, how should branch rows be generated (e.g. initialize inactive for all existing branches), and what happens with disabled/deleted services, unavailable/NULL base prices, and price changes while checkout runs? The default-inactive roll-out was suggested, **not expressly confirmed**. The accepted branch-onboarding rule does not by itself prove this future service-creation behavior is implemented.
8. **[STILL OPEN — O02 / O11]** Patient-level examinations without a complaint-visit and standalone specialty procedures without a visit require occurrence identities and invoice-source modeling. For specialty therapy, same invoice versus separate invoice/UI/API and `visit_services` nullable visit versus dedicated occurrence table remain unanswered. Separately billed specialty does not invalidate the settled one-mode rule for **ordinary** complaint visits.

**No database migration, schema edit or application implementation occurred during this logging pass.** This entry replaces only outdated discussion status, not clinical decisions awaiting owner verification.
