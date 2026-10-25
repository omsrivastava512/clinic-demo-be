# Clinic Module 2 — Successor Discussion Addendum 01

**Recorded:** 23 October 2026, **17:26 IST** (`Asia/Kolkata`)  
**Chronological position:** **After** the standalone O06 service-catalog decision brief (snapshot: 23 October 2026, 14:37 IST)  
**Status:** **ACTIVE, MODULAR SUCCESSOR ADDENDUM — discussion record only**  
**Authority:** Supplements, but does **not replace**, the canonical decision register. Explicit newer clinic-owner information and user decisions take precedence only on the particular subjects they address. Proposals and questions in this addendum do **not** silently supersede settled rules.

## 0. Source chain and boundaries

**Parent — existing canonical register (unchanged):**  
`clinic_billing_decision_log_2026-10-23(2).md`  
Title: *Clinic Module 2 — Canonical Billing & Clinical-Records Decision Log* (consolidated 22 October 2026; contains 23 October updates).

**Parallel focused workstream (unchanged):**  
`O06_service_catalog_branch_configuration_decision_brief_2026-10-23.md`  
Title: *O06 — Service Catalog and Branch Configuration Decision Brief*. It retains the deep questions about `services`, provisional `clinic_services`, branch onboarding, service availability, per-branch rates, missing-row meaning, global identifiers, and propagation.

**This successor's delta:** Records **only new information and exploration after the O06 brief**: the clinic owner's operational update, flat complaint-visit versus individually purchased treatment, possible standalone treatment occurrences, optional complaint associations, the receptionist's workflow, combined versus separate invoices, and *when* to persist checkout and invoice lines. It cross-references O06 rather than copying it.

**Versioning policy:** A new conversation may append a timestamped successor addendum. Keep earlier snapshots legible. When an earlier statement is revisited, record **previous status → current status and why**; do not erase prior decisions. Only fold these updates into the canonical register when the user explicitly requests it. This document is a discussion handoff, **not** migration DDL or implementation authorization.

## 1. 90-second briefing: what changed

1. **Operating locations:** Owner told the user that other existing branches have closed: **one operating branch today**, with **future expansion still intended**. **[NEW OWNER-REPORTED FACT; deployed data not verified]**
2. **Normal complaint care:** Owner's newly reported policy is **flat therapy billing per complaint-visit**, potentially for any future branch. It is **not** a denial that individual treatments can be sold. **[NEW OWNER-REPORTED DIRECTION; precise scope and conflict with earlier `PER_PROCEDURE` clinic mode to reconcile]**
3. **Real, nonhypothetical exception:** A person can arrive for a **single traction treatment**, commonly following an orthopedist's advice, without undertaking the clinic's ordinary complaint-course workflow. Cupping is another example. **[OWNER/USER-SUPPLIED REAL-WORLD CASE]**
4. **Same person can receive both:** A patient already having complaint-based flat therapy might also receive separately charged cupping, sometimes based on the clinician's recommendation. **[SCENARIO FOR DESIGN; invoicing/association OPEN]**
5. **Candidate clinical split:** Continue `visits` + `visit_services` for performed procedures *within complaint-based care*, and consider a separate `standalone_treatments`/`service_occurrences` table for individually charged services not necessarily belonging to a visit. **[PROPOSED — OPEN, O11]**
6. **Candidate UX/checkout:** At review, allow staff to **add another kind of treatment or a package purchase** and issue **one combined invoice**. The user **leans toward this**, but **explicitly has not approved it**. Product sales were also mentioned as a possible future addition; retail remains **DEFERRED OUT OF V1** in the parent register. **[PROPOSED — OPEN]**
7. **Pressing design fork:** Either accumulate selections in a *not-yet-issued checkout* and submit once, or create an invoice at preview and append lines afterward. These paths differ sharply in financial integrity. **[OPEN FOR DISCUSSION; current immutable-at-issuance rule remains SETTLED]**

**Do not infer:** One operating branch does not remove future branch requirements; selling a procedure individually does not prove an entire clinic runs per-procedure billing; a standalone service is not automatically a clinically undocumented retail sale; separate entry forms do not force separate DB transactions or invoices.

## 2. Precise vocabulary (avoids three different ideas called “session”)

- **Patient/person:** Existing `patients` identity can in principle represent both complaint-based patients and individuals obtaining one-off treatments. “Customer” was conversational shorthand, **not approval for a new customers table**. **[PROPOSED interpretation; confirm actual registration minimum]**
- **Complaint course:** Ongoing condition/treatment arc, e.g., shoulder pain. **[Existing domain]**
- **Complaint-visit (`visits`):** One treated complaint on one occasion; multiple complaints may yield multiple visit rows. **[SETTLED parent]**
- **Performed visit procedure (`visit_services`):** Occurrence of a procedure inside such a visit, *even when included in the flat fee*. **[SETTLED parent]**
- **Standalone performed service:** Proposed independent occurrence of traction, cupping, etc., possibly without a complaint-visit. The *name, table, FK shape and required documentation are OPEN*. **[NEW O11 proposal]**
- **Attendance / real-world appointment:** Person may receive several kinds of treatment in one physical sitting; **no generic physical-session table was approved** in the parent log.
- **Entry flow / form:** How the receptionist collects information; may be distinct for normal therapy and standalone treatment.
- **Checkout:** Logical confirmation of a selected set of chargeable/covered facts.
- **Invoice:** Financial document for issued charges. **One checkout may create one combined invoice**, if subsequently approved; an entry form is not necessarily an invoice.
- **Preview / draft:** Information shown before official invoice issuance; whether persisted is an **OPEN technical choice**. Do not conflate it with an issued Pending invoice.

## 3. Owner update: implications and contradictions to resolve

### 3.1 One operating branch, future branches still required

- **NEW FACT:** Three-branch assumption in old documents is **outdated operationally**: two prior branches are closed, leaving one operating now, per owner's report.
- **UNCHANGED INTENT:** Architecture must not assume expansion will never occur.
- **DESIGN CONSEQUENCE, NOT DECISION:** A single-branch V1 may allow manual setup and defer elaborate administration UI or bulk-propagation automation (as already noted in O06). Whether to retain global `services` plus provisional `clinic_services` remains **O06 OPEN**.
- **PREFLIGHT:** Exact live `clinics` records and historical branch data must be inspected before any archival/deletion. *Closed branch is not the same as deleting historical invoices or treatments.*

### 3.2 Flat visit fees vs one-off procedure purchases

**Previous parent baseline:** A clinic could have one ordinary `FLAT` *or* `PER_PROCEDURE` mode; one normal checkout never mixed modes across ordinary complaint-visits. **[SETTLED 23 Oct in parent]**

**New owner-reported information:** All ordinary complaint-based therapy is flat fee, including future clinics. Nevertheless, patients can pay for individual procedures (traction is a common real case) without a complaint-based treatment visit.

**Current reconciliation status: NEW EVIDENCE; ARCHITECTURAL IMPLICATION OPEN.** The owner statement may make a *clinic-wide `PER_PROCEDURE` mode for ordinary complaint-visits* unnecessary, replacing that use case with a **standalone-service workflow**. However, we have **not explicitly ratified a schema change or abolished the older mode**. Do not silently strip `PER_PROCEDURE` from the register or equate the standalone service price with the flat-visit fee.

**Question:** Can a given procedure (e.g., traction) be (a) performed within a flat complaint-visit and included in its fee, and (b) independently purchased as one separately priced service? The proposed architecture should handle this without treating the procedure's *identity* as synonymous with one billing method.

## 4. Worked scenarios to carry into review

### S1 — Ordinary flat complaint therapy only

Existing shoulder complaint; clinician provides routine treatment, potentially including ultrasound/traction as appropriate. Preserve `visit` + individual performed `visit_services` occurrences. Charge **one flat fee per complaint-visit**, not one charge per routine procedure. **[Existing settled behavior; owner corroboration]**

### S2 — Walk-in for individual traction only

A person presents for traction, perhaps with an external orthopedic referral, but does not seek ordinary ongoing complaint-based care. Candidate: one standalone performed-service occurrence with its own price and invoice line; *do not invent a complaint-course or dummy visit just to satisfy an FK*. **[User-provided real-world use case; representation OPEN]**

### S3 — Standalone cupping only

A person asks for cupping, without an existing complaint course. Candidate: short “Start Service” UI leading to a standalone occurrence and an invoice; body site/reason, practitioner, safety check and notes remain to define. **[PROPOSED — OPEN]**

### S4 — Normal therapy plus separately priced cupping in one attendance

An ongoing shoulder patient receives ordinary flat therapy and clinician-recommended cupping in the same visit to the premises. Candidate: `visits`/`visit_services` **and** a standalone occurrence, with an **optional clinically meaningful link to the shoulder complaint course**, and either:

- **Option 4A:** One combined invoice with two appropriately sourced line types and one settlement flow. **[USER-LEANING, NOT SETTLED]**
- **Option 4B:** Two checkouts/invoices, despite one physical attendance. **[ALTERNATIVE, NOT SELECTED]**

The question “same treatment room / same day” alone does **not** determine whether one or two clinical occurrences, entry forms, persistence transactions, or invoices are needed.

### S5 — Normal flat therapy plus package purchase at checkout

Reception logs complaint treatment; at final review the patient also wishes to buy a package. Candidate: both original therapy charges and an actual package-purchase source generate lines on one invoice. **[USER-LEANING, NOT SETTLED]**

**Important preserved package rule:** Package offering, purchase/entitlement, and later redemption are separate facts. Package coverage is activated only after the defined full-payment condition; **whether a newly purchased package covers the very same checkout and what “fully paid” means under waivers remain OPEN**. A combined invoice cannot automatically bypass those rules.

### S6 — Standalone service plus package, without ordinary visit

Potential boundary test only, **not a confirmed clinic workflow**: Can checkout contain standalone service/purchase items without any complaint-specific `visit`? Does the invoice creation path support zero ordinary visits? **[OPEN — do not infer from UI sketches]**

### S7 — Product sale mentioned at final review

The user mentioned adding products as another kind of checkout item. The parent canonical register **explicitly defers retail products beyond V1**. Record this as a **future extensibility thought**, not a V1 approval, product table, stock workflow, or required invoice source FK.

## 5. Candidate persistence and UI structure — discussion only

### 5.1 Separate clinical occurrence table

Provisional names: `standalone_treatments`, `standalone_services`, or `service_occurrences` (no selected table name).

**Possible conceptual attributes to discuss later:** performed service reference; patient; clinic/branch; clinician; performed date/time; treatment site and referral/reason; optional complaint-course linkage; checkout/invoice context as needed. **None of these is approved DDL**.

**Open invariants:** A real occurrence should exist even if later billed at ₹0/waived; duplicate service performance must be distinguishable from double-billing one occurrence; invoice source references must have real typed FK integrity; an optional complaint association must match the correct patient/chain/branch and must not pretend to be an ordinary complaint-visit.

**Critical clinical caveat:** No complaint-course does not necessarily mean no clinical record. Traction, especially, may require referral, contraindication, clinical authorization, consent and procedure details. Define the minimum record using actual clinic practice rather than treating these services as generic merchandise.

### 5.2 Candidate UI flows

**Ordinary care:** `Select patient → select complaint(s) → record performed visit procedures → Checkout review`.

**Standalone-only:** `Select/register person → Start Service → choose standalone treatment(s) → capture applicable context → Checkout review`.

**Combined:** In a common review screen, the receptionist may choose `Add standalone treatment` or `Add package purchase`, return to review, and then finalize. An add-on's *separate form* need not represent a second invoice or database transaction.

**OPEN:** Should this combined ability exist only within an ordinary visit checkout, or should the review screen be a general cart for several supported source types? Should there also be an independent single-service flow? Is optional complaint association offered in the add-on form? What constraints distinguish included routine traction from separately sold traction?

## 6. The pressing design fork: **when do we issue the invoice?**

### Approach A — Build a checkout draft first; persist once at final confirmation

1. Receptionist selects complaints and performed procedures in the UI.
2. **Checkout preview** shows the proposed items and amounts; user may add a standalone service or package purchase.
3. Until confirmation, this remains **non-issued draft state** (possibly client-side, unless a durable draft requirement is separately proven).
4. **Final confirmation** sends one coordinated request; the server authoritatively checks branch/service eligibility, prices, package rules and ownership, then atomically creates clinical occurrences, invoice and immutable issued lines.
5. Payment collection/settlement is recorded under its separately authorized flow, following the agreed invoicing/payment boundaries.

**Assessment — RECOMMENDED CANDIDATE, NOT APPROVED:** Closely fits the parent register's **settled atomic checkout + frozen-on-issuance invoice**. UI forms may be distinct while persistence is one atomic submission. A larger structured request is not, on its own, a reason to fragment atomic writes.

**Questions:** Browser failure before confirmation; how drafts persist, if at all; price changes while a draft is open; how preview amounts are labeled provisional; checkout idempotency and retry; adding a service after confirmation requires a **new transaction/document or explicit correction policy**, not silent mutation of the old issued line.

### Approach B — Issue an invoice at preview and append lines as add-ons are confirmed

1. Receptionist reaches preview; backend already creates an invoice.
2. Adding cupping/package causes further invoice-line inserts, recalculations and associated clinical records.
3. Receptionist eventually declares the invoice final and collects payment.

**Issue:** If the preview invoice is already **issued**, this **conflicts with the settled rule** that even Pending/Due original invoice charges and descriptions are frozen at issuance. Incremental writes may also create abandoned/partially formed financial documents, reconciliation gaps and complicated retry/concurrency behavior.

**Only conceivable reframing:** Define a **separate non-issued draft-invoice lifecycle**, with clear visibility, expiry/abandonment, authorization, numbering, finalization transaction, and no accidental inclusion in financial reporting. **This is new scope; not presently required or approved.** Merely calling an issued Pending invoice “draft” does not satisfy the existing immutability invariant.

**Status: ALTERNATIVE UNDER EVALUATION; NOT COMPATIBLE WITH THE EXISTING ISSUED-INVOICE LOCK WITHOUT A NEW DRAFT MODEL OR AN EXPLICITLY REOPENED POLICY.**

### Approach C — Split completed flows into independent invoices

Ordinary treatment and extra cupping are separately submitted and invoiced. Simpler isolated requests, but the receptionist handles two invoices/payment balances for one attendance, with potential reconciliation and patient-experience costs. **[ALTERNATIVE; NOT USER-PREFERRED, NOT REJECTED]**

**Do not conflate:** *two forms* ≠ *two HTTP calls* ≠ *two database transactions* ≠ *two invoices*. Decide each separately based on the established financial and clinical invariants.

## 7. Invoice/source implications to examine (O07 / O11 / O13 / O14)

- **Original charge authority:** Issued `invoice_lines` preserve description/amount; clinic/service catalogs remain mutable configuration; received cash and waivers are distinct from original charges. **[SETTLED concepts]**
- **Correct sourced lines:** Flat therapy comes from its `visit`; a separately priced standalone procedure should reference its **actual performed occurrence**, not pretend a bare catalog definition is evidence it was performed. Source FKs remain **typed, nullable, real relational foreign keys — SETTLED, AUDIT PENDING**.
- **Mixed invoice completeness:** A combined checkout must include all billable required sources exactly once; no double charge for routine procedures included in flat therapy; no fabricated encounter/visit for standalone service; validate membership, source ownership, and per-clinic pricing.
- **One payment vs two:** Combined invoice allows one original balance and later partial receipts. Multiple invoices require clear receipt allocation when collecting a single sum; neither choice erases payment-state rules or waivers.
- **Package purchase:** A line needs a genuine purchase/entitlement-acquisition source (precise O08 model still open). Full payment/activation and possible zero-price coverage must not be inferred from simply inserting a package line.
- **Branch rates:** With only one operating branch, pricing may be operationally simpler, but **O06 still decides** whether per-branch `clinic_services` exists, what missing configuration means, how rate lookup works, and whether the global `services` catalog remains. Do not silently restore runtime base-price fallback.
- **Time and idempotency:** One final submission still needs consistent price validation under concurrent changes, safe retries, unique occurrence billing and no orphaned clinical records. **[Technical audit OPEN]**

## 8. Questions explicitly OPEN for next sessions

### A. Clinical classification and data model — O11 / O05

1. Is a single table for independently performed services appropriate for both **cupping** and **ordinary modalities sold singly** (not just a hardcoded “specialty” class)?
2. For the **same traction service**, what clinical circumstance distinguishes inclusion in a flat complaint-visit from a standalone charged occurrence? Can staff explicitly select context, or must the server derive it?
3. May an independently billed treatment **optionally reference an ongoing complaint course** without being stored as a normal `visit_service`? Is an association to the exact `visit` or to the complaint course more clinically accurate?
4. Which standalone-service fields are medically/operationally mandatory (referral/prescription, body region, notes, clinician, consent, safeguards)? Is a one-off individual recorded in `patients`, and how much intake is necessary?
5. Is `PER_PROCEDURE` as a clinic-wide mode for **ordinary complaint-visits** now obsolete given the owner's flat-for-all-locations statement? Which exact rules from O05 should be amended, and does `billing_mode` storage remain necessary?
6. Do recorded individual treatments always create their own billable occurrence and charged line, or could they be included/waived/covered by package? Which no-charge dispositions are legitimate?

### B. Checkout composition and UX — O11 / O07

7. Must the receptionist be able to add standalone treatments **during the normal therapy flow**, and also create a **standalone-only checkout** with no complaint-visit?
8. Is **one combined invoice** the default for distinct treatments performed in one physical attendance, or should staff optionally split them? On what genuine business rule?
9. At what moment are clinical occurrences, package acquisitions and financial lines **committed**: only at final confirmation, or does the product need a separately persisted non-issued draft?
10. If staff opens a preview and adds an item, is the displayed amount provisional until the backend resolves prices/coverage? How should approval of a changed total work?
11. If staff forgets cupping **after an invoice has already been issued**, should that be a new invoice/checkout rather than an edit? What correction mechanism, if any, is truly V1?
12. Can a combined checkout contain **multiple complaint-visits plus one or several standalone services and a package purchase**? Are quantities, duplicate treatments, and one-service-only cases handled unambiguously?
13. Does purchase of a package on the **same invoice** ever immediately cover a service/visit on that invoice, or only later eligible attendances? How do partial receipts/waivers impact activation?
14. How should one aggregate payment against a combined invoice be reported, and what happens to split invoices where money is collected together?

### C. Boundaries and dependencies — O06 / O08 / O12 / O13–O18

15. Where do independent service prices, eligibility, and normal-vs-standalone classification live while **O06 remains open**? Does a catalog `category` dictate the workflow, or does the performed occurrence's context?
16. If the owner currently has one branch, what concrete multi-branch behavior must the V1 schema preserve, and what branch-onboarding UI/automation can remain manual?
17. How do we enforce invoice-line source uniqueness, cross-patient/branch association, rolled-back failure, and checkout retry safely when the request mixes occurrence types?
18. What is the **minimum V1 UI/backend scope** that supports the real traction-only and regular-plus-cupping situations without prematurely implementing generic shopping carts or retail inventory?

**No question above is closed by recording it.**

## 9. Decision-status delta against the parent register

- **Branch count:** Earlier docs assume 3 live branches → **NEW OWNER REPORT: 1 currently operating, expansion possible**. *This changes operational facts, not the need for branch-safe design.*
- **Ordinary therapy billing:** Parent `FLAT`/`PER_PROCEDURE` clinic-policy choice → **NEW OWNER REPORT: ordinary complaint-based visits flat; individual procedures also sold separately**. *Potential supersession of clinic-wide itemized mode is OPEN pending precise reconciliation.*
- **O11 occurrence:** Parent specifically flagged standalone specialty without `visit` as a gap → **EXPANDED OPEN QUESTION:** one independent-service occurrence model might cover cupping **and** traction sold alone; optional complaint association to examine.
- **O11 invoices:** Parent same-versus-separate specialty invoice OPEN → **USER LEANING:** combined review/check-out with add-ons when same attendance, plus independent standalone-only flow. **NOT SETTLED**.
- **O07/O14 checkout:** Parent atomically persists normal visit and invoice with immutable issued lines → **NEW FORK:** collect additions in draft and submit once, or issue early and append. *Existing issued-immutability invariant remains SETTLED; append-to-issued is conflicting unless separately redesigned.*
- **O08 packages:** Package V1 inclusion and paid activation remain as before → **NEW OPEN INTEGRATION:** optional purchase as an extra item in shared checkout; same-invoice coverage semantics not decided.
- **O01 retail:** **Still DEFERRED OUT OF V1**, despite being mentioned as a potential extensible future review item.
- **O06 shared catalog/branch rates:** **Still OPEN** in the independent O06 file. The current one-branch fact is not approval to remove `clinic_services` or global `services`.
- **O13 typed source FKs:** **Still SETTLED — AUDIT PENDING**, unchanged.

## 10. Suggested next conversation checkpoint (not a verdict)

First address **whether a single combined checkout is conceptually required when an ordinary complaint-visit and an individually charged treatment happen together**. Then determine whether the frontend can **collect both kinds of entry before issuing a single invoice**. Keep the separate-table choice open until we test standalone-only traction, regular-plus-cupping, and package purchase against the same source-integrity rules.

A short decision question for resumption:

> **If regular therapy and cupping happen during the same attendance, should staff be able to add both before pressing one final “Create Invoice” action — without creating any issued invoice during the preview?**

**No code, SQL, schema changes, GitHub edits, or modifications to the parent canonical register or standalone O06 file were made by preparing this addendum.**

---

## 11. Successor settlement update — 23 October 2026, 20:21 IST

**Chronology:** Decisions and refinements established **after** this addendum's original 17:26 IST snapshot. The original snapshot and its earlier OPEN labels are preserved above as historical context. This section is authoritative for **these specific subsequently discussed points**; it is not a blanket approval of unresolved schema, package, checkout, or payments work.

### 11.1 O05 — ordinary billing-mode fork becomes obsolete

**Previous state:** Whether to store a historical `FLAT`/`PER_PROCEDURE` clinic billing-mode snapshot on visits and/or invoices was open; the parent policy admitted both ordinary modes.

**Newer user-confirmed direction — SUPERSEDED IN THE PROPOSED MODEL:** Ordinary *complaint-based* therapy visits are **always flat-fee**; a separately priced procedure is instead represented as an **independent performed-treatment occurrence** (candidate table `standalone_treatments`, naming/schema not yet ratified). Thus the proposed model **no longer needs a `FLAT` versus `PER_PROCEDURE` switch for ordinary visits**, and the associated O05 question about *where to snapshot that discarded switch* is not an active design decision.

**Scope limit:** This does not yet approve a final standalone-occurrence table definition or require modifications to existing database migrations. O11 questions about clinical linkage and requisite documentation remain open.

### 11.2 O07 — routine procedures are clinical, not invoice-line sources

**SETTLED — user confirmed:**

- For every ordinary complaint-specific `visit`, the invoice records **one flat therapy line sourced from that visit**. Example: knee therapy ₹200; ultrasound and exercises performed within it are **not additional chargeable lines**.
- `visit_services` still records the **individual procedures actually performed** for clinical history, but **ordinary invoice lines do not source from `visit_services`**.
- Do **not** add ₹0 invoice lines merely to enumerate routine procedures included in flat therapy. These are distinct from the explicitly required **₹0 package-covered *visit* line**.
- In the **proposed new schema**, remove `visit_services.is_charged` and `visit_services.charged_amount_in_paise`; procedure-occurrence rows hold **clinical facts, not amounts or charge-status flags**. This is a *design settlement*, not confirmation those existing database columns have been dropped.

**Worked example:**

```text
visits: knee complaint-visit
    └── visit_services: ultrasound; exercises  [clinical records only]
invoice_lines:
    └── Knee therapy — ₹200                 [source: that visit]
```

**Caveat:** Separately purchased traction/cupping and other individually priced services belong to the separately modeled standalone-treatment context, not a financially charged `visit_service` inside an ordinary flat visit.

### 11.3 O07 / O08 — invoice describes the entire checkout, including package-covered visits

**SETTLED — explicit user preference after comparing alternatives:** An invoice represents the **entire completed checkout**, not only items that generate additional charges. Therefore:

- A package-covered complaint-visit appears **as its own ₹0 invoice line**, rather than being omitted.
- A checkout consisting **entirely of package-covered visits still produces an invoice**, even if the invoice total is ₹0.
- An invoice containing one chargeable visit and one covered visit contains both lines. Example: knee therapy ₹200 + package-covered shoulder therapy ₹0 = invoice total ₹200.
- ₹0 on the invoice is **not** a fictional ₹0 receipt or payment; financial settlement remains a distinct workstream.

**Trade-off considered and deliberately rejected for V1's stated checkout meaning:** Omitting covered visits from invoice lines (and creating no invoice when everything was covered) avoids zero-valued financial documents but makes the invoice an incomplete picture of checkout and requires other records to reconstruct it.

**Still OPEN — O08:** How the ₹0 line proves or references the specific package, whether through a redemption occurrence, its visit source, an explicit package context FK, or a constrained relationship. **Do not substitute an unvalidated zero price for entitlement enforcement.** Package purchase, activation, covering eligibility, and redemption cardinality are not settled by this choice. Accounting/reporting treatment of zero-total invoices must be distinguished from receipts/revenue.

### 11.4 O07 — calculate original invoice totals from immutable invoice lines

**SETTLED — user confirmed:** `invoice_lines` are the sole authority for original invoiced amounts. Do **not** redundantly store `invoices.total_amount_in_paise` in the **proposed schema**. Instead, derive the original total from the sum of the issued invoice line amounts in paise. The issued line amounts and descriptions stay frozen.

- Rationale: avoid duplicate persisted representations that might diverge; performance optimization is not yet a demonstrated V1 need.
- Payments, due/outstanding balances, waivers, receipts, and payment-table design are **explicitly outside this O07 discussion** and are being worked through in a separate payment-module conversation.
- A derived invoice total is **not the same thing** as what has been paid or is outstanding. No payment-policy decisions are introduced here.

### 11.5 O07 / O11 — one performed standalone occurrence, one invoice line

**SETTLED — user confirmed:** Record **each actually performed standalone treatment occurrence separately** and create **one corresponding invoice line per occurrence**. Do not aggregate multiple performed occurrences into one financial line merely because they share a catalog procedure name.

- Example: cervical traction and lumbar traction at the same attendance represent two distinct clinical occurrences and two invoice lines.
- **Do not enforce** an artificial database rule that a procedure cannot be performed twice on the same body region: two actual occurrences should remain representable if the real workflow requires them.
- The clinic **does not pre-sell individual standalone treatment sessions**, so quantity-based advance sale of such sessions is **not a supported V1 scenario**. This does **not** remove separately sold prepaid **packages** (O08).
- Table name, required clinical fields, optional complaint relationship, and the final specific FK layout for independent occurrences remain **O11/O13 OPEN**; this settlement applies to the conceptual occurrence-to-line cardinality.

### 11.6 O07 — next unresolved integrity problem

**OPEN — next topic:** Enforce that the **same visit** or **same standalone treatment occurrence** is not billed on multiple invoice lines or invoices through duplicate checkout submissions, retries, or concurrent processes. Audit typed source FKs, unique constraints, source/invoice patient and branch consistency, and transaction/idempotency safeguards (**O07/O13/O14**). No particular SQL constraints or implementation have yet been selected.

## 12. Current checkpoint and status reconciliation

**Settled in the current proposed design:**

1. Ordinary complaint-visit billing is flat; routine performed `visit_services` remain clinical only. **O05's former visit-vs-invoice billing-mode snapshot question: SUPERSEDED** by discarding that ordinary-mode fork.
2. One visit produces one therapy line; package-covered visits produce their own ₹0 visit lines; even all-covered checkout gets an invoice. The invoice describes the **whole checkout**. **O07: SETTLED** on these points.
3. `visit_services` has no charge-status or amount fields in the *proposed schema*. **O07: SETTLED**, no DB migrations applied.
4. Invoice total is **computed from immutable `invoice_lines`**, not separately stored on `invoices`. **O07: SETTLED**.
5. Independently performed standalone treatments generate **one clinical occurrence and one line each**; no aggregation and no presale of individual sessions. **O07: SETTLED**; occurrence schema remains **O11 OPEN**.

**Open and not silently settled:**

- **O08:** Package redemption/entitlement relationship supporting a ₹0 covered visit, package purchase/activation and same-checkout coverage.
- **O11:** Exact standalone-treatment table/model, required clinical data, optional complaint linkage; combined checkout UX and persistence timing beyond previously settled invoice immutability.
- **O13/O14:** Verified FK layout, same-source duplicate billing prevention, transaction and retry controls.
- **O06:** Service catalog and per-branch availability/pricing remains its own workstream.
- **Payments/waivers:** Separate module/conversation; no update made to its unresolved details here.
- **O01:** Retail products deferred beyond V1.

**No implementation action:** This document update is a decision-log amendment only. The canonical parent register, O06 brief, source code, migrations and database are unchanged.
