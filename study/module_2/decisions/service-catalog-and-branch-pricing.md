# Service catalog and branch pricing

Current catalog decisions as of 27 October 2026. MVP uses one operating clinic and current prices/availability in services. Future branch architecture is deferred. Catalog price/availability authority is admin-only; exact enforcement and physical schema remain audited.

## Current MVP position

- **Locked scope:** one operating clinic. Do not create a branch-by-service configuration matrix merely to launch MVP. This does not authorize deleting existing clinic identity or ownership FKs (R01).
- **Locked direction:** keep current editable catalog prices and enabled/disabled availability in services for this single clinic. Defer clinic_service_prices and per-clinic overlay workflows. Disabled services must preserve performed history and issued invoices (R02).
- **Locked:** both ordinary procedure entry and standalone treatment entry select an existing services identity. No free-text service identities during recording (R10).
- **Locked:** For new service entry, the picker exposes enabled services and the backend independently validates service availability; direct submission of a disabled service ID must not bypass that restriction. Disabling a service preserves existing clinical references and issued invoices. Concurrent disablement/effective-time handling remains audited (R02,R10; S02 §1).
- **Locked / Q07 settled:** only admin may add services, edit existing master catalog prices, or enable/disable a service. Reception cannot change catalog prices or availability. Enforce this on the backend alongside UI restrictions (R10,R80; S20).
- **Canonical Service Name Immutability (`D01` — `[USER—DECIDED / LOCKED]`, S37 §1):** Once the clinic begins registering clinical visits, canonical service names cannot be renamed or repurposed through the application. A service ID remains permanently associated with its underlying procedure (for example, an ultrasound identity can never be repurposed into cupping). Any necessary early setup corrections must be handled directly in the database before visits begin. Alternate wording can be handled through aliases; when a procedure is genuinely different, staff must create a distinct catalog identity. This rule applies to `services` and preserves admin authority over prices and availability without extending automatically to every catalog.
- **Locked:** ordinary complaint therapy is flat-fee, not separately priced per performed routine procedure. One standard ROUTINE rate and one standard REHAB rate apply per complaint-visit, independent of which bundled procedures were performed (R04–R05, R09).
- **Locked:** the clinician manually chooses the patient's complaint classification once at creation. It remains permanent; different classification requires a new complaint. It is not a service-catalog property selected anew each attendance (R07–R08).
- **Excluded from MVP:** no persistent negotiated/grandfathered patient-rate engine. Invoice standard amounts; an authorized admin discount uses real receipt(s) and waiver of the remainder, ending WAIVED (R06, R56).
- **Locked:** invoice lines retain original issued descriptions and prices. Later catalog edits do not change old charges (R31).

## Deferred branch architecture

Service definition, branch availability, configured price, performed occurrence, and issued charge remain different facts. Earlier alternatives included sparse global overrides, a complete service/branch matrix, explicit enrollment, and branch-owned catalogs.

Sparse branch overrides are a **future preference**, not a finalized fallback, availability, migration or table contract. Branch provisioning is not an MVP launch requirement.

For future branches, explicitly configured branch prices must remain independent: global reference-price edits must not silently overwrite them. Missing-row inheritance and runtime fallback are still future design choices. No branch-creation workflow is required at launch.

When a second clinic is introduced, deliberately migrate the original clinic's current prices and availability. Preserve the explicitly requested integration/rewiring dependency on clinic_services according to its eventual role. Do not assume now that both clinic_services and clinic_service_prices are necessary or that either owns a particular responsibility (R03).

## Audit and design work still open

- Exact single-clinic catalog/rate columns, where the two flat therapy rates are configured, price validation, and price-selection consistency during concurrent edits. The user mentions an aliases field for alternative wording; exact representation remains schema work and does not require a copied clinical service name (S21).
- Clinical eligibility/classification of bundled versus independently purchased services; a shared catalog identity alone does not determine its billing context.
- Exact database authorization and ownership enforcement for the settled admin-only price/availability rule. Business authority is closed; RLS/API enforcement remains audited.
- Future clinic ownership, missing-row meaning, inheritance/default precedence, new-service rollout, new-branch provisioning, and temporary/global disablement.

Do not reopen the answered single-clinic, catalog-only, or permanent complaint-classification business choices as though O06/O12 were wholly undecided.

## Evidence

Current rules and source references: [reconciliation register](reconciliation-2026-10-27.md#decision-register). Original versions are in [history/](history/); their wording does not override the current rules above.

Related: [clinical records](clinical-records.md), [checkout](checkout-and-invoice-lines.md), [payments](payments-and-settlement.md).

Current checkpoint: [consolidated seven-answer history log](history/clinic_module2_seven_review_answers_2026-10-27.md). All seven review questions are settled; implementation and parked audits remain unfinished.
