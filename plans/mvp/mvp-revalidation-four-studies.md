# MVP Boundary Re-Validation — Four Sequential Studies
### Continuing from `mvp-scope-diagnosis.md`


> **[COMPACTED PROMPT]**
> <details>
> <summary><b>Click to view the Compacted Four-Study Re-Validation Prompt</b></summary>
> Conduct a sequential, four-study deep architectural re-validation continuing from `mvp-scope-diagnosis.md` to establish the exact scope boundaries, hidden dependencies, schema necessities, and financial invariants before finalizing database design. First, regarding **MVP Boundary Re-Validation (Study 1)**—test the proposed core workflows against actual clinical operational purpose to determine if any included feature can be safely cut, if any deferred feature is mission-critical to avoid misleading financial or patient history records, and if any supposed edge case is a load-bearing operational case. Second, regarding **Deferred Features & Hidden Dependency Auditing (Study 2)**—evaluate all major deferred features (full package lifecycles, specialty treatment events, payment complexity, discounts/waivers, advanced invoicing) against the concrete-failure test (*what specific operational/financial defect occurs if not modeled today?*), categorizing each into one of five implementation buckets. Third, regarding **Challenging "Schema-Now" Decisions (Study 3)**—critique every proposed schema-now/frontend-later element against migration difficulty, premature commitment, and table pollution to establish the absolute minimum stable database model. Fourth, regarding **Financial & Data-Integrity Invariant Audit (Study 4)**—rigorously audit all financial rules (pricing calculations, consultation/exam fees, therapy categories, patient-specific rates, waivers/overrides, invoice sums, payment tallies, branch tenancy, and audit attribution), establishing where enforcement must strictly reside in backend/database triggers versus client UI to ensure complete ledger trustworthiness. Finally, deliver the 5-part synthesis: the revised MVP boundary, a 4-way classification table (*implement now*, *model now*, *partially supported*, *defer entirely*), mandatory pre-schema decisions, items to discard, and a definitive scope certainty recommendation.
>
> </details>

---

Each study below is fully closed out — question, evidence, assumptions, alternatives, self-challenge, conclusion, decision — before the next one opens. Two of the four studies genuinely change a call made in the previous document rather than just re-confirming it; I've flagged both plainly where they happen instead of letting them slide by as if they were always the position.

---

## Study 1 — Re-validate the MVP boundary

**Question.** Is the six-workflow boundary from `mvp-scope-diagnosis.md` (register → complaint → visit+fee → invoice+payment → branch isolation → history lookup) actually sufficient, or does something inside it deserve removal, and does something outside it turn out to be load-bearing once tested against the real clinic's actual day rather than against a feature list?

**Evidence.** `FRONTEND_WORKFLOW.md`'s five real screens map onto those six workflows closely, with one addition: the Daily Ledger's live `Waiting`/`In Therapy`/`Paid` status board, which — per that same document's Decision 3 — has zero schema backing today and contradicts the schema's own stated philosophy ("row exists = complete, no status lifecycle"). Separately, `business-rules-log.md` and `PACKAGES_DEFERRAL.md` both treat packages as a real, currently-operating part of this clinic's business, not a hypothetical future one — Om's own ₹180 override case and the package hold/cancel rules were derived from an actual owner conversation, not invented.

**Assumptions and what's unresolved.** I don't actually know, from anything in the documents, whether packages are being sold and consumed *right now*, on paper, at this clinic — or whether they're a feature the owner described wanting but hasn't started yet. That distinction matters more than anything else in this study. Same for "settles weekly": it's mentioned once, informally, as something Om floated, never confirmed as the clinic's actual current payment behavior.

**Alternatives.** For the status board: either build it (a real `visits.status` lifecycle column, a genuine 🔴 structural change) or leave it purely as informal, verbal front-desk coordination, same as any clinic without this software already manages today. For packages: either treat them as fully future (my prior stance) or treat a package-covered visit as something that can walk through the door on day one of the pilot.

**Self-challenge.** My instinct going into this was that the six-workflow boundary was basically right and this study would mostly reconfirm it. Applying the actual test given — not frequency, but whether exclusion makes the system misleading, unreliable, or unusable — breaks that instinct in two places, not zero. The status board fails the test (its absence doesn't make anything the software *does* claim untrue; the clinic simply keeps coordinating the way it always has). But packages, if they're actually live today, pass the test in the other direction: a receptionist facing a real package patient with no way to represent "this session was covered" will either silently skip logging the visit (a real gap in patient history — the exact kind of thing this whole system exists to prevent) or log it as a normal paid visit (a real overcharge). Neither is a UI inconvenience; both are the system actively producing a wrong record. Likewise, if "settles weekly" describes real current behavior, forcing every invoice to read `Paid` immediately isn't an incomplete feature — it's the software asserting something false about money that hasn't actually been collected yet.

**Conclusion.** The boundary's *shape* is right; its *coverage* was drawn one notch too far toward "pure pay-per-visit, nothing else exists yet." The status board correctly stays out. Packages and payment timing are conditionally in, pending two confirmations I can't make for you.

**Decision.** Add a seventh item to the MVP boundary: *a patient with an existing pre-paid package can have a covered visit logged correctly — ₹0 therapy charge, visit history intact, package revenue already recorded at sale time — without the full attendance-tracking system.* Ask Om directly: (a) are packages being sold and used at this clinic today, on paper, right now? (b) does "settles weekly" describe how patients actually pay today, or was it a hypothetical? If (a) is yes, the minimal package-linking work from Study 3 below moves from "conditional" to "build now." If (b) is yes, allow (don't force) `payment_status = 'Pending'` at invoice creation, using the CHECK value that already exists unused. If either answer is no, both items revert exactly to how the prior document scoped them — inert hook, forced-Paid MVP.

---

## Study 2 — Deferred features and hidden dependencies

**Question.** For every feature currently marked deferred — packages, specialty workflows, payment complexity, discounts/waivers, advanced invoicing — which of five buckets does it actually belong in: whole feature deferred; domain concept needed now, implementation later; schema needed now, frontend later; backend logic needed now, UI later; or nothing needed until it's actually built? The test throughout: what concrete problem does *not* modeling it today actually create?

**Evidence.** I'm working from the same source set as the prior two documents — `business-rules-log.md`, `specialty-decision-log.md`, `handover-fee-computation.md`, `handover-pricing-integrity-overrides.md`, `PACKAGES_DEFERRAL.md`, `module-2-gaps-log.md` — but applying the concrete-problem test to each deferred item individually rather than as a group, since a wholesale "the specialty track over-designed, defer all of it" verdict is exactly the kind of grouped judgment this study is supposed to break apart.

**Assumptions and what's unresolved.** Same two as Study 1 (packages live today? settles-weekly real?), plus one more this study surfaces on its own: does this clinic, today, ever treat someone with a specialty service and *no* complaint at all — a walk-in for a massage or cupping session with nothing diagnosed? `specialty-decision-log.md` Entry 10 states plainly this "shallow patient" idea "was not previously discussed anywhere" before the SPECIALTY chat invented it — meaning it's speculative until confirmed, exactly like retail.

**Alternatives and per-item resolution.**

*Packages (full module).* Concrete problem from not modeling attendance tracking, hold, cancel/refund: none identifiable — the clinic presumably tracks this on paper today, and the software not automating it doesn't make anything the software claims false. Whole feature deferred, confirmed.

*Package-visit linking (narrower than the above).* Concrete problem from not modeling: real, per Study 1 — visit history gaps or double-billing the moment a real package patient shows up, *if* packages are live. Bucket: schema+minimal backend now, conditional on Study 1's confirmation; UI can wait.

*Specialty multi-complaint linking (`treatment_events`).* Concrete problem from not modeling: none for billing (Entry 1's finding — specialty money already lands in `services_total_in_paise`, untouched by anything package-related, regardless of how many complaints it's attributed to). The only loss is reporting granularity. Whole feature deferred, don't even model the table shape, since `specialty-decision-log.md`'s own Entry 27 still hasn't settled whether `visits.complaint_course_id NOT NULL` is even the right premise to build against.

*Shallow-patient / complaint-free specialty visits.* Concrete problem from not modeling: unconfirmed, same as retail — I don't know if this happens today. Deferred, but added to the confirm-with-Om list rather than assumed either way.

*Split payment columns, Overdue auto-derivation.* Concrete problem from not modeling: none — the CHECK values already exist, unused, at zero cost, and no reporting UI needs them for MVP. Whole feature deferred.

*Pay Later / `Pending` status itself.* Different from the above — covered in Study 1, conditional on the settles-weekly confirmation.

*Standing per-patient override (`patient_rate_overrides`).* Concrete problem from not modeling: real and current — the ₹180 patient either gets overcharged (real financial harm to an actual person) or requires a manual, unaudited workaround every single visit, forever. Schema and minimal backend now; UI can be an admin-only form later.

*`invoice_line_items` (minimal, visit-sourced).* Concrete problem from not modeling: real and current — `module-2-gaps-log.md` confirms multi-complaint same-day sessions actually happen, and `invoices.visit_id` is singular. Without some line-item mechanism, either the combined-bill UX already shown in `FRONTEND_WORKFLOW.md`'s screenshots has to be walked back, or the connection between an invoice and the several visits it covers gets silently dropped. Schema and minimal backend now; UI can be a plain list.

*Retail (`product_sales`/`product_sale_items`).* Concrete problem from not modeling: unconfirmed — no client conversation anywhere names this. Deferred, added to the confirm list rather than dismissed outright, since these logs come from summarized voice recordings and could simply be missing a real conversation.

**Self-challenge.** My going-in instinct, carried from the prior document, was "the specialty track over-designed relative to confirmed need, defer nearly all of it as one block." Going through it item by item instead of as a block, that instinct holds for most of it — but it would have wrongly swept the package-linking need into the same bucket as retail and shallow-patients, which don't deserve the same confidence of dismissal. Treating "came out of the SPECIALTY design track" as itself a reason to defer would have been sloppy; the actual test is whether the underlying business activity is real, not which chat happened to generate the schema idea for it.

**Conclusion.** Most of the deferred surface holds as a confident, wholesale "no" — no concrete problem is created by leaving it unmodeled. Two items don't hold as unconditional deferrals: package-visit linking and Pay-Later's `Pending` status both convert from "deferred" to "build now" the instant Om confirms the underlying business activity (packages, weekly settlement) is already real today rather than aspirational.

**Decision.** Build `invoice_line_items` (minimal) and `patient_rate_overrides` now, unconditionally — both have confirmed, current concrete problems behind them. Build package-visit linking and allow `Pending` status only after Study 1's two confirmations come back. Everything else on the deferred list — full packages, `treatment_events`, shallow-patient archetype, retail, split-payment/Overdue automation — stays deferred wholesale, added to the confirm list only where genuinely unconfirmed (shallow-patient, retail), and simply left alone where the "no concrete problem" verdict is already solid (split payments, Overdue).

---

## Study 3 — Challenge every schema-now decision

**Question.** For each item currently classified "schema now, implementation later," what exact future migration does building it now actually prevent, how hard would that migration really be if deferred, and is keeping it out now premature commitment or a genuine hedge against a real cost? Explicit instruction I'm holding myself to: don't keep something merely because it's cheap; don't reject something merely because it's technically addable later.

**Evidence.** The project's own migration-safety taxonomy, from `schema-study-plan-v2.md` Module 9, is the actual yardstick here: adding a nullable column, adding a new table, or widening a CHECK constraint that existing rows already satisfy are all explicitly categorized as safe, low-risk changes — cheap now *and* cheap later. Anything that requires backfilling a value for existing rows, or decomposing one existing column's data into several new ones, is categorized as needing a real plan, and is expensive specifically in proportion to how much real data already exists by the time it happens.

**Assumptions and what's unresolved.** The single fact this whole study leans on is one you stated directly in the new prompt: zero backend or frontend has been implemented yet, meaning zero real rows exist in any of these tables today. Every migration-cost comparison below is a comparison between "cheap, because nothing exists to migrate" and "expensive, because real rows will exist by then" — and that comparison only cuts one way while the first half remains true.

**Alternatives and per-item challenge.**

*Four-bucket money columns.* What migration does building now prevent? Splitting one flat, already-populated `consultation_fee_in_paise` into four named buckets, after real visits exist, isn't a safe additive change — it's decomposing historical financial data with no reliable way to reconstruct which portion of an old flat number was "exam" versus "therapy" for any specific past row. This is the one item in the whole review where deferring genuinely means a hard, lossy migration later, not a safe one. Build now — this conclusion gets *stronger* under this test, not weaker.

*`visits.package_id`.* What migration does building now prevent? Almost nothing — adding a nullable FK to a table that already has rows is the textbook safe case; every existing row just gets NULL, correctly. So the honest reason to build this now was never "the migration would be hard later" — it's cheap either way. That means building it now has to be justified by an actual current need, not by "it's free, why not," which is precisely the reasoning this study told me to reject. Its correct status is exactly what Study 1 already made it: conditional on packages being confirmed live, not an unconditional pluggability hook.

*`invoice_line_items` (minimal).* What migration does deferring this actually cost? Less than I'd have guessed before running this test. If deferred and added later, every existing invoice under the old one-invoice-per-visit model becomes, structurally, the simplest possible case of the new model — one invoice, one line item, matching its existing `visit_id`. That's not a hard migration. So the real argument for building this now isn't migration-avoidance at all; it's the same argument from Study 2 — a confirmed, already-happening operational need (multi-complaint sessions, already screenshotted in the real frontend design) that the system can't currently represent correctly. Build now, but I want to be honest that the reason is operational, not migration-cost.

*`patient_rate_overrides`.* Same shape as the above — deferring this doesn't require migrating anything later (a forward-only feature; existing rows are simply silent about overrides that didn't exist yet). The real justification is the same confirmed, current, ongoing operational cost of a manual workaround for a real named patient. Build now, operational reasoning, not migration-cost reasoning.

*Transactional visit-level override (`final_amount_in_paise`, `override_reason`, `override_by`).* This is the item this study actually changes. What migration does building it now prevent? Nothing meaningful — three nullable columns added to a table with existing rows is the safest possible category of change, and every historical row correctly reads as "no override" via NULL. And unlike the standing per-patient case, there is no confirmed current instance of a one-off waiver need distinct from the ₹180 patient (who's served by the separate table). So this was being kept for exactly the reason I'm told to reject: it's cheap, so why not. Reversing the prior document's call: **don't build these columns for MVP.** If a genuine one-off waiver need appears during the pilot, the honest workaround — a direct manual edit to the relevant row, which the project's own stated MVP philosophy already treats as acceptable elsewhere ("editable only through direct database access") — costs nothing and leaves the real signal (does this keep happening?) intact for deciding whether to actually build the columns later, at which point adding them is exactly as cheap as it would have been today.

*Fee-scope configurability (`clinic_fee_sharing`/`clinic_gap_rules`).* What migration does keeping this *out* of schema now prevent? None to avoid — widening a CHECK-constrained column to add a value later is explicitly the safe category, by the project's own taxonomy. This reconfirms the prior document's call, now on firmer footing: the axis-table design isn't a defensible hedge against a real migration cost, it's schema built to avoid a migration the project's own standards already say is cheap. Stays out.

*`clinic_tier_rates`'s clinic-scoping.* This is the one item where the migration costs are genuinely close in both directions — building unscoped now and needing to add scoping later means a small, bounded, single-pass data copy (a few rate rows, not transactional history); building scoped now and never needing it just means one unused column dimension forever. Neither direction is dangerous. This stays a real toss-up, appropriately unresolved, not something this stricter test manages to settle either way.

**Self-challenge.** Running this study, I expected it to mostly reconfirm the prior document's "build now" list with better-articulated reasons. It did that for most of the list, but it also found one item — the transactional override columns — that survived the previous review's reasoning ("cheap, keep it") without ever surviving *this* review's actual test ("cheap is not, by itself, a reason"). I'd rather flag that reversal plainly than quietly keep the old recommendation because it already made it into a finished document once.

**Conclusion.** The items that survive as genuine "build now" cases split into two different kinds of justification, and it matters which kind applies: the four-bucket columns survive because deferring them is a real, hard, lossy migration once data exists. Everything else that survives — `invoice_line_items`, `patient_rate_overrides` — survives because of a confirmed, current operational need, not because of migration cost (those migrations would actually be cheap later too). The transactional override columns had neither justification and are cut. `package_id` has neither justification on its own and stays conditional on Study 1.

**Decision.** Remove `final_amount_in_paise`/`override_reason`/`override_by` from the MVP schema entirely — not even the simplified two-column version. Keep everything else exactly as the prior document scoped it, with the reasoning now split explicitly into "hard-migration-avoidance" (four-bucket columns only) versus "confirmed-current-need" (everything else that survived), so a future re-read of this project doesn't conflate the two kinds of argument.

---

## Study 4 — Financial and data-integrity audit

**Question.** What invariants must actually hold for this MVP to be trustworthy, and for each one: is it enforced today, where should enforcement live, is frontend calculation sufficient for the pilot, and does deferring backend enforcement create unacceptable risk? The distinction I'm holding myself to throughout: the frontend *computing* the right number is not the same fact as the system *guaranteeing* it.

**Evidence.** Walking the specific list the prompt named, against the real schema in `supabase_migration.md` as it exists today, not against what any handover file proposes building.

**Assumptions and what's unresolved.** None new here — this study is mostly a matter of reading the actual SQL carefully rather than resolving an open business question, and it surfaced two things worth naming precisely because nobody had asked this exact question of the schema before.

**Alternatives and per-item audit.**

*Therapy/service pricing and consultation/exam fee rules.* Enforced today: not at all — `visit_services.charged_amount_in_paise` and every four-bucket fee column are plain, client-submitted values, with no trigger cross-checking them against the catalog or the confirmed business rules. What *is* backend-guaranteed is narrower than it looks: `grand_total_in_paise` being `GENERATED` guarantees the total always equals the sum of whatever four numbers were submitted — it guarantees internal arithmetic consistency, not that those four numbers are the *correct* ones under the business rules. Is frontend calculation sufficient for MVP? For a small, known, trusted staff pool, yes — the same way it's already accepted that RLS being off is tolerable at this trust level. But this is a decision to *accept*, explicitly, not a gap to overlook: today, and even after every "build now" item above ships, the system does not guarantee fee correctness. It guarantees that whatever gets submitted adds up internally. The guarantee of business-rule correctness lives entirely in the frontend's code being right and the humans running it not tampering with it.

*Therapy categories/tiers.* Same shape — a CHECK constraint (once tiers are modeled) guarantees a *valid* tier value exists, not that the *correct* rate was actually charged for it.

*Patient-specific pricing.* Once `patient_rate_overrides` exists, the override *rule itself* is a real, persisted database fact. But nothing cross-checks that a given visit's submitted `therapy_fee_in_paise` actually reflects an active override for that patient — a receptionist can simply forget to look it up, and nothing catches that. This is a genuine, narrow residual gap, distinct from the table not existing at all: the table records the rule; it doesn't enforce that the rule was applied. Acceptable for MVP because the failure mode is a single overcharge to a known, small-population patient, likely to be noticed quickly — but worth naming rather than assuming the table alone closes the loop.

*Invoice totals.* This is where the audit surfaces something the prior two documents didn't say plainly enough: `invoices.amount_in_paise` is, today, a plain client-submitted column with no structural connection to the visits or line items it's supposed to represent. Once `invoice_line_items` exists, that gap doesn't close by itself — only the ghost-invoice deferred constraint trigger (checking that an invoice's line items sum to its total, at commit time) actually closes it. Without that trigger, `invoice_line_items` is a nice display feature with no guarantee it reflects the number actually being billed. I'm elevating this from "worth building alongside" (the prior document's framing) to "required the moment `invoice_line_items` exists" — it's not insurance for a hypothetical bug, it's the only thing that makes the invoice total mean anything at all once there's more than one column claiming to represent it.

*Payment records.* `payment_status`/`payment_mode` record a real-world fact (did money change hands) that software fundamentally cannot verify independently of the humans reporting it — this isn't a backend-enforceable invariant in the same sense as pricing correctness is, in any system, and shouldn't be treated as a gap to close so much as a fact to record honestly.

*Reporting/tallying.* Fully derived from invoice-total correctness — no separate invariant of its own. A `SUM(amount_in_paise) GROUP BY clinic_id` is only as trustworthy as the ghost-invoice trigger makes the underlying totals.

*Branch isolation.* Already covered in Study 1 on the privacy/trust side; the financial-integrity side is that `v1_allow_all` also leaves per-branch revenue attribution unprotected — nothing stops a Clinic A action from being misrecorded under Clinic B, corrupting exactly the reporting invariant above.

*Attribution/auditability.* Here's the second thing this audit surfaces that nobody had flagged before: `created_by`, used across almost every table in the schema, has no derivation trigger anywhere — it's client-submitted everywhere it appears. Compare this to `owner_id`, which *is* derived by trigger specifically because trusting a client-submitted value for it was judged unacceptable. The exact same misattribution risk that justified deriving `override_by` from `auth.uid()` in the pricing-integrity design was never extended to `created_by`, even though `created_by` is used far more widely. This isn't a deliberate scoping decision anywhere in the documents — it reads as an inconsistency in how the "derive, don't trust" principle got applied, not a considered exception.

**Self-challenge.** It would be easy to end this study by simply reasserting "trust the frontend for now, it's fine at this scale" across the board, since that's the conclusion I've reached for fee correctness in every prior document. Holding myself to the actual distinction the prompt drew — computed-right versus guaranteed-right — is what surfaced the two items above; a looser pass would have stopped at "yes, invoice_line_items needs a trigger, sure" without noticing that the trigger isn't optional polish once there are two numbers (an invoice total and a set of line items) that could silently disagree, or that `created_by` was never given the same treatment as `owner_id` for no stated reason.

**Conclusion.** This MVP, even fully built per every "build now" item above, guarantees arithmetic and tenancy-derivation consistency but does not guarantee business-rule correctness of fee amounts or the honesty of its own audit trail — both rest on a small, currently-trusted human actor pool rather than the database. That's a legitimate, bounded risk posture for a pilot with known staff. It stops being legitimate the moment the actor pool grows past people the owner personally trusts, or the moment real financial disputes start needing a trustworthy paper trail to resolve.

**Decision.** The ghost-invoice sum-matching trigger moves from "worth building" to "required," effective the same commit that introduces `invoice_line_items` — they ship together or not at all. Explicitly record, in whichever document becomes the actual architecture reference, that fee-amount correctness and `created_by` attribution are accepted as frontend-trusted for the pilot — a documented decision, not a silent gap. Add "derive `created_by` from `auth.uid()` via trigger, on every table that has the column" to the same worklist as turning on real RLS policies — both are cheap, both close the same class of trust-the-client gap, and both should happen in the same pass rather than separately.

---

## Final Output

### 1. Revised MVP boundary

The six workflows from `mvp-scope-diagnosis.md`, plus one addition from Study 1:

1. Register a patient, any branch, one shared chain-level identity.
2. Open or select a complaint, existing or new.
3. Log today's visit with the fee computed correctly per the confirmed rules.
4. Generate an invoice covering that visit (or that day's batch) and record payment.
5. Keep branches isolated — staff see only their branch; the admin sees the whole chain.
6. Look up a patient's history before treating them again.
7. **New:** if a patient has an active pre-paid package, log the covered visit correctly — ₹0 therapy charge, history intact, revenue already recorded at sale — without the full attendance system. *(Conditional on Study 1's confirmation that packages are already live today.)*

### 2. What's implement now / model now / partial / defer

| Item | Status |
|---|---|
| Four-bucket money columns (`exam_fee`, `lapse_penalty`, `therapy_fee`, `services_total`) | **Implement now** — hard migration if deferred |
| Real RLS policies (replace `v1_allow_all`) | **Implement now** — currently a violated invariant, not a missing feature |
| `invoice_line_items` (visit-sourced, minimal) | **Implement now** — confirmed current need |
| Ghost-invoice sum-matching trigger | **Implement now**, bundled with the line above — not optional once it exists |
| `patient_rate_overrides` | **Implement now** — confirmed current need (₹180 patient) |
| `created_by` derivation via `auth.uid()` trigger | **Implement now**, bundled with the RLS rollout |
| `visits.package_id` + package-visit linking | **Partially supported** — conditional on confirming packages are live today |
| `Pending` payment status allowed at invoice creation | **Partially supported** — conditional on confirming "settles weekly" is real |
| Gap-measured-against (lapse penalty: complaint vs. patient) | **Decision required**, not yet a schema question until answered |
| Fee-scope configurability (`clinic_fee_sharing`/`clinic_gap_rules`) | **Defer entirely** — hardcode; the migration it avoids is already cheap |
| `clinic_tier_rates` clinic-scoping | **Model now conceptually, confirm before building** — genuine toss-up, ask Om |
| Transactional visit-level override columns | **Defer entirely** — reversed from the prior document; no confirmed need, cheap either way |
| Full Packages module (attendance, hold, cancel/refund) | **Defer entirely** — your own prior instruction, reconfirmed |
| `treatment_events` / specialty multi-complaint linking | **Defer entirely** — don't even model the table shape |
| Session / `session_id` concept | **Defer entirely** |
| Retail (`product_sales`/`product_sale_items`) | **Defer entirely**, pending a confirm question |
| Shallow-patient / complaint-free specialty archetype | **Defer entirely**, pending a confirm question |
| Split payment columns, Overdue auto-derivation | **Defer entirely** — zero cost either way |

### 3. Decisions genuinely required before schema implementation

- Are packages being sold and used at this clinic *today*, on paper, right now — or is that still aspirational?
- Does "settles weekly" describe real current patient-payment behavior, or was it hypothetical?
- Does the 10-day lapse penalty gap get measured against the complaint or the patient (`FEECOMP_DISCUSSION.MD`'s still-open item)?
- Do the three actual clinic locations charge different tier rates from each other today, or would one shared rate table be accurate?
- Does this clinic ever treat someone for a specialty service with no complaint at all — a genuine walk-in with nothing diagnosed?
- Is retail product sale (braces, tape, topical products at the counter) a real near-term need, or does it not exist as an idea outside the design chat that generated it?

None of these are open-ended design work — they're each a direct yes/no or a specific number, answerable in one conversation with the clinic owner.

### 4. What to stop thinking about for now

The full Packages attendance/hold/cancel machinery, `treatment_events` and any specialty multi-complaint linking, the session/`session_id` concept, retail's actual table design, split-payment columns, Overdue auto-derivation, and — newly — the transactional visit-level override columns. None of these need another design pass, another comparison table, or another schema sketch until a real, observed need forces one. If you find yourself refining any of these further before that happens, that refinement is the thing to stop, not the underlying idea.

### 5. Final recommendation

Yes, with one condition: you have enough scope certainty to move to schema/architectural finalization *once* the six confirm-questions in Section 3 come back. They're fast to get — none require new design thinking, only the clinic owner's actual answer to a specific question that's already precisely stated. I would not proceed to finalizing the schema with those six still open, because three of them (packages-live, settles-weekly, gap-measured-against) directly change which column set ships in the first migration, and getting that migration right the first time is exactly the kind of thing Study 3 just showed is cheap now and expensive later. Everything else in this document is settled enough to build against today.
