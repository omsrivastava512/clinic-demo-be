# Clinic billing and clinical records — decision log

Updated: 22 October 2026, Asia/Kolkata.

Scope: the Module 2 discussion about complaint visits, performed procedures, pricing, catalog design, invoice sources, checkout, and related payment boundaries. Consolidates the conversation available in this thread, including the supplied live-conversation transcript. This is a record of discussion, not a migration or implementation instruction.

## Start here

- **Established:** complaint-based clinical recording; one atomic checkout; clinic-controlled therapy pricing mode; routine procedures included under flat therapy; ₹200 routine and ₹300 rehab therapy rates.
- **Reopened:** unified catalog versus separate definitions versus a shared parent with specialized tables. **None is currently selected.**
- **Still open:** historical billing-mode placement; authoritative location of billed amounts; fee configuration; invoice membership; line sources and duplicate-billing constraints; specialty modeling and MVP scope.
- **Deferred:** the packages module and retail products/equipment. Their existence matters as context; their implementation is not authorized by discussing them.
- **Where we stopped:** questioning the hybrid design—why procedures reference billable items, whether other offering kinds need specialized records, and where specialty treatments belong. No subsequent architecture choice was made.

## Status legend and precedence

- **Confirmed:** an explicit user requirement or agreement.
- **Open:** not decided; an assistant recommendation does not close it.
- **Reopened:** previously accepted, then questioned or withdrawn from final status.
- **Proposed:** a candidate implementation or explanation, not a commitment.
- **Deferred:** outside the current implementation scope.
- **Existing baseline:** present in the repository migration inspected on 21 October; not proof of deployed database state or endorsement of that model.

Latest explicit user clarification overrides older recommendations. In particular, do not implement the unified catalog on the basis of its earlier acceptance: that decision was reopened.

## 1. Workflow and clinical meaning

### D01 — One visit record per complaint treated on an occasion

**Confirmed.**

The working flow is:

1. Select a patient.
2. Select the complaints being treated.
3. Select/record the procedures performed for each selected complaint.
4. Submit the treatment entry at checkout.

Treating complaint A and complaint B in one sitting produces visit A and visit B. A complaint can have many visits over its course; “one visit per complaint” means one record for that occasion, not one record for the complaint's entire lifetime.

The discussion does not establish a persisted session/sitting table, a universal one-invoice-per-day rule, or a prohibition on repeat attendance the same day.

### D02 — Procedure selection records treatment

**Confirmed.**

The procedure picker’s job is to identify what was performed. It does not decide the charge. Billing evaluates those selections separately at checkout.

A catalog definition and a performed occurrence are different facts:

- Definition: ultrasound is an available procedure.
- Occurrence: ultrasound was performed for this complaint-visit today.

All performed procedures remain in the clinical history even when included in a flat fee and therefore not charged separately.

### D03 — Atomic checkout

**Confirmed.**

One checkout submission should save the confirmed visits, performed procedures, and invoice-related records in one database transaction. Failure rolls the operation back; a failed invoice write must not leave partially saved treatment from that checkout.

Nested versus flat request JSON does not determine atomicity. The transaction boundary does.

The exact backend function/API and retry/idempotency mechanism are **open implementation details**. No claim is made that the current backend already implements this checkout.

### D04 — Combined billing for several visits

**Confirmed direction.**

The checkout must support one invoice covering several selected complaint-visits. A separate invoice per complaint is not required by the workflow.

The physical relationship expressing that membership remains open. This does not settle separate specialty, package-purchase, or retail invoice workflows.

## 2. Therapy billing policy

### D05 — The clinic owns the therapy pricing mode

**Confirmed.**

For MVP, each clinic has one enforced therapy billing policy:

- Flat fee per complaint-visit; or
- Itemized procedure pricing.

Other clinics under the same chain may use a different mode. The receptionist cannot choose different modes for individual visits. The mode is configured at the clinic level; it is not a staff-selectable override for each checkout.

Calling this a **billing mode** is clearer than using the old Consultation/Machine Only terminology. `therapy_billing_mode`, `FLAT_PER_VISIT`, and `PER_PROCEDURE` are **proposed identifiers**, not finalized or implemented column/value names.

### D06 — Flat therapy rates

**Confirmed from the user's clarification.**

- Routine pain treatment: **₹200**.
- Rehabilitation, accident recovery, and post-operative treatment: **₹300**.

These are two tiers of flat therapy pricing. Earlier examples applying ₹300 universally were incomplete and must not replace these rates.

The assignment/storage of a visit's therapy tier, configurability of those rates across branches, and patient-specific agreed-rate precedence remain open.

### D07 — Routine procedures are included in flat therapy

**Confirmed.**

Under flat therapy pricing, routine/standard procedures for that complaint-visit do not add separate charges. They are still recorded as performed.

For itemized pricing, the discussed behavior is to charge the selected procedures individually instead of adding the routine flat therapy fee. Exact applicability of other fees is a separate rule.

No allocation of the flat fee across individual included procedures has been agreed. We should not invent an ultrasound share of a ₹200 flat fee if no such component pricing actually exists.

### D08 — Specialty treatment has a separate charge when supported

**Confirmed billing rule; workflow and scope remain open.**

The user stated that specialty treatments are separately charged irrespective of whether flat therapy pricing applies.

Later, the user questioned including specialty treatments in complaint-based procedure entry at all. Therefore the billing rule is not a decision to implement specialty treatment in MVP or keep it in the ordinary procedure catalog.

### D09 — Initial examination and gap-return fees

**Established in the earlier fee discussion; not implemented by this log.**

- Initial examination was corrected from per-complaint recurrence to **once per patient lifetime**.
- ₹350 is the discussed examination amount, not one of the two flat therapy rates.
- A ₹150 gap-return fee was discussed. Its applicability, thresholds, and interaction with a new versus returning complaint remain **open**.

Any proposed “full examination reset after a long gap” is not silently incorporated into the lifetime rule. It requires an explicit decision. Also unresolved: what historical event counts as satisfying the initial-exam rule—examination performed, exam billed, or payment collected—particularly for waivers and imported patients.

Do not stack “consultation fee,” “visit fee,” and “therapy fee” merely because different documents use those names. Determine whether they describe the same fee or distinct services with different recurrence rules.

## 3. Catalog architecture — reopened

### R01 — Earlier unified-catalog acceptance is no longer final

**Reopened.**

Discussion sequence:

1. Separate clinical services plus explicit fee settings were proposed for MVP.
2. After listing therapy tiers, packages, and other billables, a shared billable catalog was recommended.
3. The user initially accepted the shared-catalog direction.
4. The user then questioned mixing clinical procedures with fees/packages/products and raised specialty's separate workflow.
5. Separate definitions were reconsidered.
6. A hybrid—shared billable identity with specialized tables—was proposed.
7. The user requested a context handoff before choosing.

**Current status: all three candidates remain open.** The last assistant recommendation is not the final decision.

### C01 — One shared catalog

**Proposed.**

Possible structure: `billable_items` contains procedures, flat therapy fees, examination fees, and later package/product offerings. A `kind` distinguishes the categories.

- The procedure picker requests only available procedure-kind rows.
- The backend validates submitted IDs again.
- Fees are genuine billable offerings, not fake clinical procedures.
- References from performed-treatment records must be restricted to legitimate procedures; a general catalog FK does not itself enforce that restriction.

Renaming/extending the existing `services` table was suggested, but no migration choice or table name was finalized.

### C02 — Separate definitions

**Proposed.**

Keep a focused services/procedures catalog. Put therapy/exam configuration in clinic fields or a fee-rate structure. Future packages/products have their own definitions when built.

Different definitions can still produce uniform invoice lines. Source references and financial authority must be designed explicitly.

Separate clinic procedure-price and fee-rate tables were illustrative proposals, not mandatory requirements. The user's concern about unnecessary tables was not resolved by accepting that sketch.

### C03 — Shared identity plus specialized definitions

**Proposed.**

Possible structure:

- `billable_items`: common offering identity and shared attributes.
- `procedures`: specialized clinical definition linked to the corresponding offering.
- `visit_services`: the performed procedure occurrence.
- `invoice_lines`: billed transaction detail, with an offering reference and separately designed source linkage.

The procedure → billable-item FK direction was intentional: a procedure can extend an offering; a fee need not have a procedure child.

The child can share the parent's primary key. Two unrelated IDs are not inherently required. Uniqueness, kind agreement, and any required subtype completeness need explicit constraints.

**Latest unresolved concern:** procedures were the only concrete specialized table in the example. The user questioned whether therapy, specialty, gels, belts, and packages would otherwise merely sit in the parent table.

The explanation offered was that a specialized table is needed when that kind has additional attributes or relationships. A fee may need no child and can still be used by invoice lines. **That explanation was not followed by user acceptance of the architecture.**

### D10 — Catalog identity does not settle invoice origin

**Technical distinction recorded during discussion; source design remains open.**

“Ultrasound” identifies an offering. “Ultrasound performed during visit A today” identifies an occurrence. A common catalog ID does not by itself show which occurrence was billed or prevent billing it twice.

Likewise, a package definition is different from one patient's package purchase. A product definition is different from a particular sale.

## 4. Specialty, packages, and retail scope

### Q01 — Specialty treatment boundary

**Open.**

The user proposed that specialty treatment such as cupping should not have to belong to a complaint-visit and should have its own workflow rather than appear in the ordinary complaint → procedure picker.

Candidates still under consideration:

- Exclude specialty treatment entirely from MVP.
- Support it through a separate form/workflow.
- Keep its definition only in a billable catalog or give it a dedicated definition.
- Record a standalone treatment event, with any complaint/visit linkage decided separately.

A separate invoice for specialty was floated, not finalized. Neither its current appearance as PREMIUM in `services` nor a separate screen determines the final table structure.

### D11 — Packages deferred; offering facts retained

**Deferred module.**

Four offerings were described:

- Routine, 10 days — price unknown.
- Routine, 30 days — tentatively ₹4,500.
- Rehab, 10 days — price unknown.
- Rehab, 30 days — tentatively ₹6,000.

Do not prorate unknown prices. “Days” has not been resolved as calendar validity versus attended treatment days.

Still untouched/open: whether covered visits create zero-price invoice lines, no new invoice, or another representation; coverage rules and mixed covered/uncovered checkout behavior.

### D12 — Retail products/equipment deferred

**Confirmed deferral.**

The clinic sells equipment such as belts. Gels/ointments were also used as future-model examples. These should be kept in mind without implementing retail, inventory, or product-sale workflows for MVP now.

Adding another belt or gel is not necessarily a new source category. Several products can share a product-definition/sale structure. No exact retail schema has been chosen.

## 5. Financial records and invoice relationships

### Q02 — Historical billing-mode location

**Open, explicitly retained by the user.**

The clinic chooses the mode. Where should the mode applied historically be remembered?

- On each visit, as proposed by the assistant?
- On the invoice, as questioned by the user?
- On a checkout/grouping/pricing record or through another model?

Do not treat the visit snapshot as accepted. Consider clinic policy changes, invoices containing non-therapy offerings, and records with no new charge. No automatic requirement to support mixed therapy modes within one checkout was agreed.

### Q03 — Authoritative home for billed amounts

**Open.**

Current schema stores money on visits, performed services, and invoices. The user challenged independent copies and asked whether billing amounts should belong exclusively to invoices/lines.

The assistant proposed invoice lines as the authoritative billed detail, with visit subtotals derived from those lines. This remains a proposal.

Distinguish:

- Current standard rate: pricing configuration.
- Historical billed charge: what was charged on an occasion.
- Amount collected: payment fact.
- Subtotal/total: arithmetic over the relevant charges.

Keeping a mutable current rate and an immutable historical charge is not accidental redundancy. Several independently editable copies of the same transaction amount require a clear authority and maintenance rule.

### Q04 — Invoice-line granularity and printed detail

**Open.**

Candidates:

- One line per visit subtotal, with preserved underlying procedure detail.
- One line per separately charged component: flat therapy fee, individual procedure, examination, etc.

Visit-level lines do not inherently lose history if historical component descriptions and amounts are retained elsewhere. They do lose reconstructability if only a subtotal survives or mutable current prices are used later.

For included routine procedures, the discussed display options are descriptive inclusion text or zero-price lines labeled “included.” Neither was selected. Zero alone is ambiguous: included, complimentary, waived, or missing a price.

### Q05 — Visit/invoice membership

**Open physical design; combined checkout requirement established.**

Options discussed:

- `visits.invoice_id`: several visits point to one invoice.
- Derive membership through invoice lines and their clinical sources.
- An explicit grouping relationship if justified.

If both visit membership and line/source membership are stored, they must agree. Deriving through lines does not automatically prevent one visit's charges being split across invoices.

A blanket unique constraint on a visit reference in invoice lines would wrongly prohibit several procedure charges from one visit. The current nullable `invoices.visit_id` is not a one-to-one relationship because it is not unique.

### Q06 — Line source design and duplicate billing

**Open.**

Patterns discussed:

- Typed nullable source FKs plus an exactly-one-populated check.
- Generic source type + source ID, with the corresponding integrity enforcement burden.
- A common charge/source parent with specialized domain links.
- A different domain-specific design, if it fits better.

Technical clarifications retained:

- Nullable FKs can enforce existence when populated. NOT NULL is a separate rule.
- PostgreSQL can enforce an exactly-one-non-null check without a trigger.
- A single ordinary FK cannot dynamically target several tables based on a type field.
- A shared parent simplifies one reference but still needs valid links to actual treatment/purchase/sale records.
- Offering links and occurrence-source links serve different purposes.

No source columns, unique indexes, triggers, or final parent/subtype design have been selected.

### Q07 — Price ownership and patient-specific rates

**Open.**

Existing schema supports shared service definitions with per-clinic price overrides. Alternatives discussed include clinic-owned offering rows with their own current prices.

The need for another price table depends on ownership and price variability, not on whether the catalog is unified or split.

Patient-specific agreed rates have been discussed, but the user explicitly postponed the override question while catalog structure was being clarified. Scope, precedence, storage, and grandfathering behavior must not be assumed settled here.

## 6. Checkout payment and waiver boundaries carried forward

These are prior discussion outcomes relevant to avoiding conflicting billing designs. They are not a completed payment-schema specification.

### D13 — Payment workflow direction

- Receptionist confirms completed treatment, creates the invoice, and either confirms payment with a method or chooses Pay later.
- Pay later creates **Pending**; the receptionist cannot subsequently edit it under the discussed workflow.
- Admin resolves Pending/Due by entering an explicit payment amount and classifying any remaining amount according to the agreed options.
- **Paid invoices are locked** for MVP.
- Pending invoices are excluded from the discussed paid/EOD reporting direction.
- A distinct payment date is needed conceptually; creation time does not establish when payment occurred. Exact fields remain to be implemented/designed.

The migration's Paid/Pending/Overdue statuses are older than this discussion. Do not describe Due or waiver handling as already implemented by that file.

### D14 — Broad waiver meaning

**Confirmed user intent.**

A waiver can represent a favor, acknowledged loss, or an overcharge/billing-selection mistake, with a note explaining the reason. The user does not want a large correction workflow solely for these MVP cases.

A waiver reduces collection. It does not fix an undercharge or correct a clinical record claiming an unperformed procedure.

### Q08 — Remaining payment ambiguities

**Open / requires reconciliation before implementation.**

- Earlier “waiver only before payment” wording and later examples waiving a balance after partial payment need one explicit interpretation. The log does not silently choose one.
- Exact distinction among waived balance, loss/write-off, and payment status is not fully specified here.
- Treatment of partial receipts in cash/EOD reporting must be defined; “Pending excluded” does not settle every report.
- A separate payments table was discussed; deferring it was the user's preference, not a completed integrity design.
- No separate audit-log table is the earlier MVP direction; final reason/attribution fields and allowed corrections still need alignment with the final schema.

## 7. Existing repository baseline

Evidence date: **21 October 2026**, from the migration file inspected earlier in this thread. Not refreshed against a live database for this log.

- `clinics`: branch and owner; old first/subsequent consultation rates; other branch settings. No explicit therapy-billing-mode field in the inspected definition.
- `patients` and `patient_clinic_access`: chain-level patient identity and branch access.
- `complaint_courses`: patient/clinic complaint course; optional complaint-catalog reference; name snapshot and clinical lifecycle fields.
- `services`: text ID, name, `standalone_price_in_paise`, category STANDARD/PREMIUM.
- `clinic_service_prices`: clinic/service price override; unique clinic/service pair.
- `visits`: patient, clinic, complaint course, date, old visit/consultation types; consultation fee and services total; generated grand total.
- `visit_services`: visit and service references; service-name/category snapshot; `is_charged`; `charged_amount_in_paise`.
- `invoices`: patient, clinic, invoice number, amount, date, payment status/mode, and nullable non-unique `visit_id`.
- `packages`: patient-specific package/complaint record with duration, expiry, attendance, amount-paid fields. Not a general offering catalog.

No `invoice_lines`, `invoice_line_items`, `billable_items`, `procedures`, or `treatment_events` table was defined in that inspected migration. Those are discussion/proposal names.

`visits.grand_total_in_paise` adds its two visit fields. It does not independently aggregate performed-service rows or reconcile the invoice. The occurrence's existing `is_charged` was explained as separately charged, not paid; its future retention/replacement is open.

## 8. Requirements any chosen design must be checked against

These are review criteria identified in discussion, not already implemented safeguards:

- Preserve clinical treatment even when no additional procedure charge applies.
- Restrict performed-procedure references to legitimate available procedures.
- Preserve issued descriptions and amounts after catalog edits and price changes.
- Allow repeat legitimate occurrences of one procedure; prevent duplicate billing of an occurrence.
- Prevent separately billing routine procedures already included in flat therapy.
- Keep patient, clinic, visit, source, and invoice relationships consistent.
- Keep transaction failure atomic and checkout retries safe.
- Enforce the intended initial-exam recurrence without duplicating it across complaint-visits.
- Keep an offering definition distinct from a patient's performed/purchased occurrence.
- Avoid implementing deferred modules merely to make a generic schema symmetrical.

## 9. Next decision queue

1. Establish the specialty boundary and its MVP scope: exclude, separate workflow, optional clinical linkage, and any invoice boundary.
2. Choose catalog structure after that scope is explicit: unified, separate, hybrid, or a better alternative.
3. Confirm definition/price ownership across clinics and the minimum rate structure.
4. Settle the authority for billed amounts and invoice-line granularity.
5. Choose invoice membership, occurrence sources, cardinalities, and constraints together.
6. Resolve historical billing-mode and classification snapshots.
7. Resume agreed-rate scope and unfinished exam/gap-fee rules.
8. Reconcile payment/waiver ambiguities before writing their implementation.
9. Audit the chosen model against one complete flat-fee checkout, one itemized checkout, and retry/price-change cases.

This is an organization of the remaining work, not a requirement to answer every item in one conversation.

## 10. Change history and documentation handling

- The user asked that running live-discussion notes remain in chat; a prior assistant had saved an addendum prematurely. That incident is not authorization to edit the repository.
- The current request explicitly authorizes this downloadable decision-log file.
- The context handoff and its visual walkthrough are explanatory views, not schema approvals.
- No project code, database, or repository migration is changed by creating this log.
- Future updates should preserve these status distinctions and explicitly mark superseded decisions rather than silently replacing them.

## References

Primary current intent: the conversation and supplied live transcript in this thread. Repository documents are supporting history; conflicting older recommendations do not overrule later user clarifications.

- [Migration baseline](https://github.com/omsrivastava512/clinic-demo-be/blob/main/docs/supabase_migration.md) — inspected file blob `421f73f8eaf5a3cee53d9d424fb969b3f5b5cdd2`.
- [Specialty and invoice decision discussion](https://github.com/omsrivastava512/clinic-demo-be/blob/main/study/business_logic/speciality_and_invoice/specialty-decision-log.md).
- [Fee-computation discussion and examination-recurrence correction](https://github.com/omsrivastava512/clinic-demo-be/blob/main/study/business_logic/fee_computation/DISCUSSION.MD).
- [Pricing and waiver analysis](https://github.com/omsrivastava512/clinic-demo-be/blob/main/plans/mvp/mvp-pricing-waiver-analysis.md).
- [Older MVP step-back review](https://github.com/omsrivastava512/clinic-demo-be/blob/main/plans/mvp/mvp-stepback-review.md) — contains positions subsequently revised/reopened.

Completeness boundary: this log covers the available billing discussion. It does not claim to reproduce every unrelated module decision or every message from conversations not supplied here.
