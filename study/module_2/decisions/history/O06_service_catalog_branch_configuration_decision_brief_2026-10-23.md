# O06 — Service Catalog, Branch Configuration, Availability & Pricing

**Standalone architecture discussion brief**  
**Snapshot:** 23 October 2026, 14:37 IST (Asia/Kolkata)  
**Current status:** **OPEN — decision exploration; no physical schema approved**  
**Parent register:** `clinic_billing_decision_log_2026-10-23(2).md`, especially §§1, 2.1, 6–9, 11, 13; register item **O06**.  
**Provisional terminology:** `clinic_services` (not approved as an actual table rename).  
**Modification scope:** This is a **separate discussion/handoff document**. It does **not** update the canonical register, GitHub, or Supabase.

> **Reading rule:** The latest explicit user reconsideration overrides an older discussion's *decision status*, not its historical record. In particular, the canonical register previously marked “initialize every existing global service as inactive when a new branch is created” **SETTLED**; the user **explicitly reopened it later on 23 October**. It is **OPEN in this snapshot**. Established safety and checkout constraints remain intact unless explicitly reopened.

## 1. The problem in one minute

A physiotherapy chain currently has **three physical branches**. Different branches may offer different treatments (equipment and staff differ), charge different prices for the *same* treatment, and change their availability over time. We need to decide which facts belong to:

- **`services`** — historically a global catalog of treatment/procedure definitions;
- **`clinic_services`** — a **provisional** name for a branch-to-service configuration, potentially containing price and availability; or
- **branch-owned service records**, without a shared global definition.

The unresolved question is **not merely whether to add `is_active` to a pricing table**. It is whether treatment identity is shared, what a branch-to-service row means, how records are created, and what the receptionist and billing engine should read. Multiple valid designs remain under consideration.

The file is designed for **discussion**, not implementation. Do not treat an example, recommendation, table name, schema sketch, or older migration as a ratified design.

## 2. Terminology and facts to keep distinct

- **Chain / owner:** The clinic business operating multiple locations. Existing repository documentation groups branches using `owner_id` (an admin profile); there is no separately verified organization table in the discussed baseline.
- **Clinic / branch:** One physical location. **`clinics` represents branches**, not the umbrella organization. A record's `clinic_id` identifies its branch.
- **Procedure definition:** What “Ultrasound Therapy,” “IFT,” or “Cupping” *is* conceptually; potentially one identity across locations.
- **Branch enrollment / offering:** Whether a particular branch offers a particular defined service.
- **Branch availability:** Whether that offering can currently be selected/used; may be temporarily disabled.
- **Global base/standalone price:** A starting/reference price on `services`; in the inspected migration the column is `standalone_price_in_paise`. The canonical decision log sometimes calls it `base_price` conceptually. This is **not** an instruction to rename a column.
- **Configured branch price:** The price the branch should apply if the procedure is eligible and separately billed; older table `clinic_service_prices.price_in_paise`.
- **Performed procedure:** A concrete clinical occurrence (currently represented by `visit_services` linked to a `service_id`) — separate from either catalog or branch configuration.
- **Issued charge:** An immutable historical amount/description captured on `invoice_lines` under the established architectural direction, irrespective of later edits to catalog or configuration.
- **Global activation / central prohibition:** A restriction branches cannot override; different from a one-time operation disabling all branch offerings.

## 3. Architecture baseline — documentary evidence, not deployed proof

The inspected GitHub source was [`docs/supabase_migration.md`](https://github.com/omsrivastava512/clinic-demo-be/blob/main/docs/supabase_migration.md) in `omsrivastava512/clinic-demo-be`; README/docs may contain other, older or incompatible architecture proposals. Treat them as **repository documentation**, not confirmation of deployed Supabase DDL or current clinic requirements.

The inspected migration documents approximately:

```text
clinics(id, owner_id, ...)                   # one row per physical branch
services(id, name,
         standalone_price_in_paise,
         category, ...)                      # shared reference catalog
clinic_service_prices(id, clinic_id,
                      service_id,
                      price_in_paise, ...)    # unique(clinic_id, service_id)
visit_services(id, visit_id, service_id, ...)# performed event referencing services
```

The older price resolution was `COALESCE(clinic_service_prices.price_in_paise, services.standalone_price_in_paise)`; `clinic_service_prices` did **not** contain `is_active` in this inspected document. That **older fallback mechanism is not the selected direction** in the canonical decision log: branch eligibility and explicitly configured pricing were preferred for safe checkout. Whether adopting a different *override-only* design would justify revisiting fallback is now a **new decision dependency**, not implicit permission to reintroduce it.

**Caution:** The repository's older labels (`STANDARD` / `PREMIUM`, consultation/first/subsequent fees, and some README fee assumptions) do **not** automatically satisfy the revised therapy-mode, specialty, or 30-day examination rules. The canonical register wins over historical documentation for decisions. Do not assume that a `services` row being globally present makes it eligible for ordinary treatment selection.

## 4. Status ledger: what is decided, proposed, reopened, or deferred?

### A. Established constraints that O06 must respect

1. A receptionist should see **only procedures that are available at their authorized branch**. The backend independently validates branch authorization and eligibility; hiding a dropdown option is insufficient. **SETTLED product behavior; enforcement design audit pending.**
2. Two branches can configure **independent prices** for the same conceptual treatment. Changes to a global seed/reference price should **not silently overwrite** previously configured branch prices. **SETTLED direction.**
3. Normal V1 treatment/checkout writes visits, procedure occurrences, invoice and invoice lines **atomically**. For `PER_PROCEDURE`, the server resolves eligible branch price and snapshots charged amount/description to issued invoice lines. **SETTLED workflow; concurrency audit pending.**
4. Disabled or retired offerings must **not erase historical performed procedures or invoices**. **SETTLED historical-preservation requirement.**
5. The existing source strategy for invoice lines uses **typed nullable real foreign keys**, not an unchecked polymorphic `source_type/source_id`. **SETTLED — AUDIT PENDING (O13)**; O06 does not reopen it by itself.
6. One ordinary therapy billing mode per checkout (`FLAT` or `PER_PROCEDURE`) remains settled. In `FLAT`, record performed procedures even when they are included and not charged separately. **SETTLED.**

### B. Recently reopened or still open

1. **New-branch provisioning:** The canonical log initially settled “copy every global service's starting price into an **inactive** branch configuration.” After discussing the real cost and rarity of opening a new location, the user **explicitly reopened this**. **CURRENT STATUS: OPEN**, with historical settled rule retained for traceability.
2. **Physical table boundary:** price and `is_active` together in `clinic_services` versus separate availability and price tables. The combined table is **PROVISIONALLY FAVORED**, not finalized.
3. **Global `services` table:** its necessity, identity model, operational purpose, and cost are **OPEN**.
4. **Propagation of a new service to existing branches:** full inactive rows, selected-branch rows only, or no rows without overrides. **OPEN**.
5. **No-row semantics, inheritance/defaults, branch enrollment, and invoicing price path:** **OPEN / dependent on selected model**.
6. **`clinic_services` name:** user explicitly asked to keep as a **PROVISIONAL NAME**, contingent on whether the table really configures a branch's offered services.

### C. Scope restraint from the current conversation

- **Manual SQL for provisioning a fourth branch is acceptable for V1.** A new branch is a rare, substantial real-world operation; automation must earn its complexity. This is a **V1 operational allowance**, not a finalized data model or security shortcut.
- **Bulk “disable at all branches” function:** **OPTIONAL**, not a mandatory requirement; do not build speculatively.
- **Central hard lockout / safety prohibition:** **FUTURE POSSIBILITY ONLY**; no evidenced V1 requirement. This is conceptually different from setting branch flags to false.
- **Dedicated service price-change audit/history, promotional scheduling:** not established O06/V1 requirements merely because prices can change. Immutable issued prices already protect invoice history; historical administrative price changes would be a different requirement.

## 5. The three competing meanings of `clinic_services`

These are **model alternatives**, not additive features. The meaning of a missing row differs fundamentally.

### Model A — Sparse **overrides** over a global default

`services` contains baseline name, category, price and perhaps a default active flag. `clinic_services` has a row **only when a branch differs from the default**, e.g., different price or explicit inactive status.

Example (illustrative only): globally Ultrasound = ₹200, active. Branch A has **no override** and inherits the global value. Branch B has a row price=₹250. Branch C has a row active=false.

- **Advantage:** few config rows; global defaults centrally managed.
- **Hazard:** **absence means inherit**, *not* “branch doesn't offer it.” An unconfigured branch might accidentally offer/charge a treatment it lacks equipment for. Requires careful default/override precedence and nullable override semantics.
- **New tension:** A global default status, previously considered unnecessary for V1, becomes part of the model if branches inherit availability. Therefore selecting this approach would **explicitly reopen** the earlier no-global-availability direction.
- **Follow-up:** Is the baseline meant to be active for every branch, or must every new branch opt in anyway? If opt-in is mandatory, this is no longer pure sparse overrides.

### Model B — **All services configured** for every branch

`services` is a shared definition. For every `(clinic_id, service_id)` there is one `clinic_services` row, with explicit branch `price` and `is_active` (commonly initially false). Branch configurations are independent after setup.

- **Advantage:** clear explicit price/status for every pair; administratively simple “toggle an existing row”; no ambiguity from a missing pair in routine operation.
- **Cost:** more records and provisioning work whenever a branch or service is added. At **three branches**, storage volume is not itself a credible performance problem; the issue is lifecycle semantics and workflow burden.
- **Open:** Why create rows for equipment that the branch has never owned or intended to offer? Does the administration UI actually require pre-existing inactive records?
- **Historical standing:** Was previously marked settled specifically for **new-branch setup**; explicitly reopened in latest discussion.

### Model C — **Explicit branch enrollment / selected offerings**

`services` holds shared treatment identities. `clinic_services` gets a row **only for services a branch chooses to offer**, containing its own price and perhaps `is_active` for temporary suspension.

Example: Branch A offers ultrasound and IFT. Branch B offers IFT only. The two branches share the same IFT definition, but have different `clinic_services` records and possibly prices. There is **no ultrasound row for Branch B**.

- **Advantage:** **absence means not offered**; no silent global-price fallback; straightforward branch-specific visibility and branch onboarding using a manually selected list.
- **Trade-off:** adding a treatment requires inserting a configuration rather than flipping an already-created inactive record. Must distinguish “never enrolled” (no row) from “temporarily suspended” (`is_active=false`) when operationally meaningful.
- **Open:** Should a branch be allowed to register a service with price=0? How are unavailable/invalid prices handled? Does every procedure concept still need a global identity?

### Model D — **No global catalog: fully branch-owned procedures**

This was raised separately and challenges the premise of A–C: Branch A's “Ultrasound” has one ID, Branch B's “Ultrasound” another, with no shared `service_id`.

- **Advantage:** fewer parent-level concepts/joins; each branch fully owns its catalog.
- **Costs to assess, not assume fatal:** duplicate names/clinical classification, drift in definition and category, difficulty comparing same treatments across branches, migration when a patient changes branches, and inability to refer to “Ultrasound” as one shared concept without a mapping or convention.
- **Open:** Are shared treatment identity, chain-wide reporting, centralized definitions or reusable complaint-to-procedure suggestions actual needs? If they are, removing `services` may just recreate it informally elsewhere.

**Fundamental contrast:** Models A–C share **one conceptual service ID** plus branch associations. Model D gives each branch **independent service IDs**. Decide shared identity before arguing about joins.

## 6. Concrete scenarios to use during review

### Scenario S1 — Existing chain, different equipment

Branches A, B and C all exist. “Ultrasound” is a known procedure. A and B can offer it; C has no ultrasound equipment.

**Ask:** Must C have an inactive configuration (B)? No row (C)? Or inherit a global setting unless overridden (A)? What prevents a receptionist from selecting it at C?

### Scenario S2 — Fourth branch opens

The real operation of opening the facility is complicated; manual service configuration is acceptable for MVP.

**Ask:** On creation, should SQL copy **all** defined procedures inactive, insert **selected** offerings only, or insert **no exceptions** and inherit global defaults? Are prices seeded once or inherited dynamically? Which branch administrator/owner approves the offered list?

**Status caveat:** the “all inactive” answer in the canonical log is **historical, reopened**, not final as of 14:37 IST.

### Scenario S3 — Brand-new treatment offered only by Branch A

Branch A acquires a shockwave therapy machine; B/C do not.

**Ask:** Create a global “Shockwave Therapy” definition first? Then create *only A's* row? Or also empty/inactive B/C rows? Can the new treatment instead exist solely in A's own table? Who decides its category/classification and initial price?

### Scenario S4 — Temporary disable at one branch

Ultrasound machine breaks at A. Price stays ₹200. A stops offering it; B keeps offering it.

**Ask:** Is changing `clinic_services.is_active=false` sufficient? Should a suspended offering retain its branch price and record rather than be deleted? What does the UI show to staff/admin? A mere price-change event is not involved.

### Scenario S5 — Owner says “stop everywhere”

Commercial choice: stop a service at all branches, but perhaps A will resume later.

**Ask:** Do we require a special bulk operation? **No** for V1 unless actually requested. A centralized **forbid-reactivation** rule is a different future policy and not yet justified.

### Scenario S6 — Price change / discount

A charges ₹200, B ₹250. A temporarily adjusts its configured price. Previous invoices must remain unchanged.

**Ask:** Is A's price an independent current configuration, or override of a global default? Do we need explicit price history or scheduled reversal? **Not currently established.** Never recompute historical invoices from current prices.

### Scenario S7 — Receptionist registers a visit

The procedure picker must show only eligible *currently available* procedures for that receptionist's branch.

**Ask:** Does query start from `clinic_services` and join `services` for the name/category (enrollment model), or start from `services` and left-join branch overrides to evaluate inheritance (override model), or read branch-owned services directly? Who verifies eligibility if the API is called with a guessed ID?

### Scenario S8 — Checkout in `PER_PROCEDURE` mode

A performed ultrasound occurrence is entered in `visit_services`; an invoice line needs the correct **branch** price and the service label.

**Ask:** Does price come only from the explicitly enrolled/configured branch record or from an override/fallback? What if the row is missing or disabled? How do concurrent admin price edits affect the price chosen? Which FK points to the *performed occurrence* versus a definition/configuration? The invoice must snapshot the resolved price and description.

### Scenario S9 — Specialty service and category

Cupping/IASTM/laser may be separately billed. Some specialty work may not belong to an ordinary complaint-specific visit; this is separately open as O11.

**Ask:** Is specialty a subtype/category of `services`? Does the branch row need its own copy of `category`, or should it reference the single global classification? Might category ever legitimately be branch-specific? Which picker must exclude/include it? Do not use legacy `STANDARD`/`PREMIUM` labels as proof of the right business behavior.

## 7. Workflow and data-integrity questions that follow from the model

### Definition → branch offering → performed occurrence → invoice

```text
Possible shared-definition model (illustration, NOT approved DDL):

services (one treatment concept)
   | 1-to-many
   v
clinic_services (branch-specific offering, price, status)
   | consulted for branch eligibility / price
   v
visit_services (actual performed treatment; visit/branch context)
   |
   v
invoice_lines (historical issued amount and source association)
```

This is **one candidate flow**, not a finding that every arrow must be stored as a direct FK. In particular, the already established invoice-source model references **actual occurrences** for procedure lines; it does **not** automatically require invoice lines to store the branch configuration row ID as their billed source.

Questions to test for **each** proposed model:

1. **Cardinality and uniqueness:** Is there at most one `(clinic_id, service_id)` row? Can duplicates or conflicting overrides exist? Can two branch-owned IDs refer to the same treatment concept without an explicit mapping?
2. **No-row meaning:** inherited default, unavailable, never enrolled, or configuration error? Make exactly one unambiguous rule for each operation.
3. **Selection and authorization:** Can a patient/branch/clinic mismatch or guessed `service_id` be accepted through a direct API call? Which layer rejects it?
4. **Price authority:** global baseline, current explicit branch price, branch override, or negotiated patient-specific price (the latter is O12)? How is fallback precedence defined, if fallback is ever chosen?
5. **Temporal consistency:** what if the admin edits price/status during atomic checkout? The selected price must be transactionally coherent and copied onto the issued line. Mere transaction atomicity does not by itself guarantee a consistent concurrent price read.
6. **Historical integrity:** don't cascade-delete past procedures or reprice issued lines when definitions are renamed, offerings are disabled, or current rates change.
7. **Classification:** is service `category` an invariant of the treatment definition or a branch policy? If copied to multiple rows, what keeps copies synchronized and which source is authoritative?
8. **Reporting and suggestions:** do management dashboards and suggested procedures need to compare “the same treatment” across branches? Does their behavior rely on a shared `service_id`?
9. **Lifecycle:** who can create, rename, deactivate, archive, or retire a definition versus branch offering? What happens when a new branch or service appears?
10. **Availability vs billing mode:** available clinical procedure ≠ automatically billed. Flat mode may document routine treatments without individual charges. Specialty inclusion/charge rules are a distinct policy question.

**Performance note (engineering inference):** One indexed join between a shared catalog and a branch-configuration table is ordinarily inexpensive at a three-branch scale. Join count alone is not a sound reason to discard normalization. Verify actual query and index behavior when implementing, rather than assuming zero joins is inherently superior.

## 8. Consolidated OPEN decision register, organized by dependency

This section deliberately stores **questions**, not recommended answers.

### O06-A — Identity and ownership

- A1. Does the business need a **shared conceptual service identity** across branches?
- A2. Should a treatment offered by only one branch still be created in global `services`?
- A3. Can Branch A and B each have an independent ultrasound ID without unacceptable reporting, classification or data duplication?
- A4. What tangible work must `services` perform: common names, common category, base price, suggested procedures, global reporting, central governance, or something else?
- A5. If none of those shared capabilities are required, could a branch-owned catalog safely replace it?
- A6. If a global definition is retained, is it truly *chain-wide* or globally shared across unrelated businesses? A multi-business private catalog is deferred in canonical O20; do not silently introduce it.

### O06-B — Meaning and shape of `clinic_services`

- B1. Is it a sparse **override**, an explicit **enrollment**, or a **complete all-services configuration** table?
- B2. Should it hold **both price and availability**, or separate those concerns into tables? The single-table choice is only provisional.
- B3. What exactly does a missing row mean under the selected model?
- B4. If overrides only, can absence safely imply a globally active, billable service?
- B5. Is `is_active` an offering status, a temporary suspension, a service-definition retirement flag, or multiple incompatible meanings?
- B6. Does a row store only the branch's price/active flag or also a duplicated name/category? Why would duplication be needed?
- B7. Should branch prices always be explicit, including free/zero-priced procedures? What validations distinguish zero from absent?

### O06-C — Creation and propagation

- C1. On opening a new branch, copy all global services inactive, add only selected offerings, or use inherited defaults?
- C2. Is manual SQL provisioning enough for current V1? **Yes, accepted as operationally feasible**, but what validation should that manual operation observe?
- C3. When A introduces a brand-new procedure, must it become a global service definition?
- C4. Does that new procedure generate inactive rows for B/C, or only an active/selected row for A?
- C5. Do late edits to a global seed price affect already configured branches? **Established answer: no automatic overwrite.** Which explicit maintenance workflow, if any, is needed?
- C6. Who may approve new service definitions versus local branch offerings?

### O06-D — Availability, inheritance and lifecycle

- D1. Is a global `services.is_active` useful, harmful, or unnecessary?
- D2. Could `clinic_services` override a global active/default state in either direction? What are the precedence rules?
- D3. Should there be a permanent centralized safety prohibition? **Not a demonstrated V1 requirement**; record only as possible future scope.
- D4. Is mass disabling a mandatory feature? **No; optional future operation only.**
- D5. How do temporary suspension, permanent retirement and deletion differ? What stays readable in history?

### O06-E — Receptionist and backend read/write path

- E1. Which table drives procedure selection at visit registration, and which joins/filters apply?
- E2. Are absent, inactive, out-of-branch and specialty-ineligible procedures rejected at the server boundary?
- E3. If the procedure list is sourced from branch configurations, when are newly selected services inserted and made visible?
- E4. Do ordinary and specialty pickers require distinct filtered views/workflows?

### O06-F — Pricing and invoice source semantics

- F1. Should current per-procedure pricing use explicit branch configuration only, or global-default-plus-override resolution?
- F2. If a branch row is absent, should checkout reject, treat service as unavailable, or fall back to global price? **Earlier direction rejects silent fallback; new override model would require an explicit reconsideration.**
- F3. How does a pricing query identify the right branch, service, and actual performed occurrence without cross-branch misuse?
- F4. Where is current monetary authority? Where is immutable historical charged amount/description stored? **Issued `invoice_lines` snapshots remain an established requirement.**
- F5. What happens if an admin changes price/status concurrently with checkout?
- F6. How will negotiated patient rates or special business exceptions (O12) interact with the chosen price source?
- F7. Do invoice lines need to reference `clinic_services` for catalog provenance, or is a validated performed occurrence plus a snapshot sufficient? **Do not confuse provenance with the one logical billable source.**

### O06-G — Specialty and classification dependencies

- G1. Do specialties share the global `services` catalog, or require distinct definitions?
- G2. Is `category` defined once globally, copied to branch configurations, or branch-selectable?
- G3. What would justify a treatment having different classifications in different branches?
- G4. How does the category control normal-picker eligibility versus specialty workflow and pricing?
- G5. How do standalone specialty occurrences without a complaint-specific visit fit into the chosen definitions? **Occurrence and invoice grouping remain independently OPEN under O11.**

## 9. Cross-links to other canonical decisions (avoid accidental scope creep)

- **O05 (OPEN):** where historical `billing_mode` is stored. O06 must not decide it incidentally; the one-mode-per-checkout rule is already settled.
- **O07 (OPEN):** invoice-line monetary authority/granularity and explanation of included vs billed procedure occurrences; historical immutable charges established.
- **O11 (OPEN):** standalone specialty treatment occurrence and same/different invoice, plus its relationship to complaint-specific visits. O06 should not force these into `visit_services` without addressing the gap.
- **O12 (OPEN):** therapy tier, branch-rate and patient-specific negotiated-rate rules; may affect pricing precedence.
- **O13 (SETTLED — AUDIT PENDING):** typed source FK approach; O06 should preserve it.
- **O14–O15 (OPEN technical):** checkout concurrency/idempotency and server authorization/immutable invoices; service selection/price resolution must satisfy these checks.
- **O17–O18 (NOT STARTED):** migration/reconciliation and integration testing before asserting implementation.
- **O20 (DEFERRED):** independent multi-business catalogs; don't optimize O06 around a hypothetical new ownership hierarchy.

See the canonical register for full wording of **all** non-O06 issues. This file does not supersede that register.

## 10. Discussion order suggested for a focused review (NOT an architectural decision)

**Round 1 — Identity:** Is the same ultrasound therapy one service definition across the chain, or two branch-owned records? Name one concrete operational requirement that benefits from (or is harmed by) shared identity.

**Round 2 — Table semantics:** With that identity choice, decide whether `clinic_services` means overrides, enrollment, or an exhaustive matrix. Define the meaning of a *missing* row explicitly.

**Round 3 — Provisioning:** Test new branch and new service scenarios with manual V1 setup. No reason to prioritize automation unless it solves an actual operational risk.

**Round 4 — Read/checkout:** Trace one receptionist selection and one per-procedure invoice all the way from a branch-eligible procedure to a price-snapshotted `invoice_line`, including rejection cases.

**Round 5 — Classification/specialty:** Test standard ultrasound vs separately billable cupping, and mark anything depending on O11 as deferred *discussion*, not silently answered.

### Exit criteria before calling O06 settled

A final decision must say: (1) whether there is one shared `service_id`; (2) what `clinic_services` stands for; (3) missing-row meaning; (4) onboarding/new-service propagation; (5) allowed price and availability precedence; (6) receptionist/API eligibility rule; (7) relationship between definitions, performed occurrences and invoice line sources; (8) service category/specialty ownership; and (9) migration/testing consequences. Explicitly document deviations from the canonical log's earlier new-branch initialization rule.

## 11. Chronology and provenance

- **Canonical source (uploaded):** 23 Oct register marked branch initialization for all existing services as inactive **SETTLED V1 onboarding**, and catalog configuration model **OPEN**. It also recorded no runtime global-price fallback as the current direction.
- **Later 23 Oct conversation, before this file:** examined same-row price + status; price history as separate; global flag vs trigger vs bulk operation; limited real-world justifications for central lockouts; optional bulk disable; new-service propagation; whether `services` should exist; provisional `clinic_services` naming after GitHub confirmed `clinics` = physical branch.
- **Later still:** user explicitly **reopened** all-service initialization for a new branch and allowed manual SQL-based V1 provisioning, raising selected-service-only enrollment as an alternative.
- **Most recent discussion:** user added identity questions (two ultrasound IDs across branches), override semantics for price and availability, missing-row implications, receptionist query/source, itemized invoice joins/fallback, specialty catalog membership, and category duplication.
- **This document snapshot:** **23 October 2026, 14:37 IST**. Earlier conversational recap timestamps were presented approximately; they must **not** be construed as exact source timestamps or as when older business decisions were actually made.

**Nothing in this standalone file authorizes SQL writes, migrations, repository edits, or canonical-log changes.**
