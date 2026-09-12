# Pricing, Discount & Waiver Analysis — Six Sequential Studies
### Continuing from `mvp-revalidation-four-studies.md`


> **[COMPACTED PROMPT]**
> <details>
> <summary><b>Click to view the Compacted Pricing & Waiver Analysis Prompt</b></summary>
> Conduct a sequential, six-study architectural analysis evaluating the clinic's variable pricing, discount, waiver, and preferential rate requirements to establish the minimum viable financial model that guarantees trustworthy, auditable records without premature engine complexity. First, regarding **Pricing Concept Deconstruction (Study 1)**—distinguish genuine domain concepts (current standard clinic price, patient-specific standing/preferential rates, one-time transaction adjustments, percentage/fixed discounts, full waivers, legacy grandfathered prices, and historical price movements) from unnecessary software abstractions. Second, regarding the **Minimum Correct Financial Model (Study 2)**—establish the conceptual baseline separating "what the normal rule says," "what was actually charged," and "what was actually paid," defining the immutable audit data required for financial reporting before jumping to table DDL. Third, regarding **MVP Implementation Alternatives (Study 3)**—rigorously evaluate five architectural approaches (A: full discount/waiver engine, B: underlying pricing model with minimal UI, C: authorized manual override of computed amount with audit attribution, D: physical paper/logbook fallback for rare cases, E: superior hybrid models) across implementation complexity, auditability, operational staff friction, reconciliation risks, and revenue distortion. Fourth, regarding **Paper/Logbook Fallback Viability (Study 4)**—critique when offline manual workflows serve as legitimate simplifications versus dangerous split-brain financial risks that corrupt ledger reconciliation. Fifth, regarding **Legacy Patient Pricing & Price Increases (Study 5)**—analyze how the system remembers grandfathered rates when standard clinic pricing changes (comparing historical price tables, patient-specific overrides, effective-dated rates, or transaction adjustments). Sixth, regarding **Architectural Critique & Final Decisions (Study 6 & Deliverables)**—audit the existing schema for over-engineering, under-modelling, or future-proofing anxiety, delivering the final 9-point specification covering the core domain model, schema/backend requirements, frontend deferrals, manual process boundaries, and a definitive "do now / do later / don't build" roadmap.
>
> </details>

---

Same discipline as last time: each study closes fully before the next opens. One of these studies produces a direct, deliberate reversal of a conclusion from the previous document — not because that conclusion was wrong given what I knew, but because the business context in this prompt confirms something I'd previously called unconfirmed. I'm flagging that plainly where it happens rather than quietly folding it in as if it were the position all along.

---

## Study 1 — Identify the actual pricing concepts

**Question.** Of the eight listed — standard clinic price, patient-specific standing price, one-time transaction adjustment, percentage discount, fixed-amount discount, full waiver, historical/legacy patient price, price change over time — which are genuinely distinct domain concepts requiring separate representation, and which collapse into each other as unnecessary software abstraction?

**Evidence and alternatives.** Testing each candidate pair by asking whether knowing one tells you everything you'd need to represent the other.

*Percentage discount vs. fixed-amount discount vs. full waiver.* These are different arithmetic used to arrive at a final number, not different kinds of fact needing separate storage. What a trustworthy record actually needs is two numbers — what the rule would have charged, and what was actually charged — with the difference between them being "the discount," whatever shape it took. Storing a `discount_type` enum on top of that tells you nothing a future reader couldn't already infer from the two numbers, and a full waiver is just the boundary case where actual charged happens to be zero. I don't think this distinction earns its own field. The one place a percentage genuinely behaves differently from a flat amount is if it's meant to *track* a changing base price dynamically ("20% off whatever we're currently charging," moving automatically when the standard rate moves) rather than freezing to a number. Nothing in the business context describes that — every example given (the ₹180 case, legacy pricing) is a frozen flat number, not a floating percentage. I'm treating dynamic-percentage tracking as unconfirmed and out of scope.

*Historical/legacy patient price vs. patient-specific standing price.* These looked, going in, like they might need separate mechanisms — one arises from a deliberate individual relationship, the other from an automatic consequence of when a patient joined relative to a price change. But the *mechanism by which each arises* isn't the same thing as *what needs to be stored*. Both are, structurally, "this patient has their own ongoing rate, different from whatever's currently standard." The only real difference is what goes in a free-text reason — "relationship with the doctor" versus "predates the 2026 price increase" — not a different shape of data.

*Price change over time, as its own tracked concept.* This is the one I want to be most careful about, since Study 5 below digs into it directly. Provisionally: I don't see a confirmed need to reconstruct "what was standard on some past date" for any reporting purpose — every past invoice already permanently records what was actually charged at the time, so historical correctness doesn't depend on being able to look up historical rates later. I'm holding this as a working conclusion pending Study 5's deeper pass, not a settled one yet.

**Self-challenge.** The risk in collapsing categories together is throwing away a distinction the business actually cares about to make my own model tidier. The one I pressure-tested hardest was discount type, since "why did we give this discount" (percentage of something vs. a flat reduction) feels like it could matter for a report. But the report that would actually matter — "how much discretionary value did we give away this quarter" — only needs the *difference* between expected and actual, summed; it doesn't need to know the arithmetic shape of any individual instance. I don't think I'm losing anything real here.

**Conclusion.** Three genuinely distinct concepts survive: the clinic's current standard rate (a simple, current, mutable value); a standing per-patient rate (one mechanism, regardless of whether its cause is a personal relationship or legacy grandfathering); and a one-off, single-visit adjustment (genuinely different in scope from a standing rate, since conflating the two would let a temporary emergency waiver silently become someone's permanent new rate). Discount type, a legacy-vs-relationship distinction, and a full price-history table are all unnecessary software abstractions on top of those three.

**Decision.** Model exactly three concepts going forward. Don't build a `discount_type` taxonomy. Don't build a separate mechanism for "legacy" pricing distinct from "standing" pricing — one table, different reason text.

---

## Study 2 — Determine the minimum correct financial model

**Question.** How should "what the rule says," "what was actually charged," and "what was actually paid" be represented conceptually, before any table design, and what needs retaining for a later report to stay understandable?

**Evidence.** Working from Study 1's three concepts and the actual mechanics of a single visit being billed.

**Alternatives.** The first thing worth getting right is *what "the rule" actually means for a given patient*. It would be a mistake to treat "the rule" as the clinic's generic, vanilla standard rate for everyone — a patient with a standing override has a *different* normal rate than a brand-new patient, and that difference isn't a deviation requiring justification on every visit; it's just their normal rate, decided once. The alternative — treating every visit for a discounted patient as "a deviation from standard, please justify" — would be both wrong (it isn't a deviation for them, it's their rate) and operationally exhausting (nobody wants to write a reason on every single visit for a patient whose rate was settled months ago).

**Self-challenge.** Am I overcomplicating this by insisting on a two-layer model (personalized-normal, then one-off-on-top) instead of one flat "expected vs. actual" pair? I don't think so — the two layers answer genuinely different questions. "Is this patient's rate correct" is answered by layer one alone, checked once at the time the standing rate was set. "Did anything unusual happen on this specific visit" is answered by layer two, and only layer two needs a reason and an approver attached to it, precisely because it's the layer that's actually irregular.

**Conclusion.** Three layers, not two: (1) what the rule computes for *this specific patient*, already incorporating whatever standing rate they have — this is the honest, protected baseline, and it needs no fresh justification on any given visit, since it was already justified once when the standing rate was granted; (2) what was actually charged on this one visit, which equals layer one unless a one-off adjustment applied, in which case it's the adjusted number, captured separately, with a reason and an attributed approver; (3) what was actually paid, a genuinely separate fact from what was charged (relevant mainly if Pending/Pay-Later is ever real, per the prior document — not a pricing question specifically).

**Decision.** Layer one must never be overwritten, even when layer two differs from it — that's what keeps "how much did we discount this month" answerable later. Layer two only needs a reason and attribution *when it's actually used*; it should never be a mandatory field on every visit.

---

## Study 3 — Evaluate the MVP alternatives

**Question.** Compare full workflow (A), minimal schema with bare-minimum frontend (B), authorized manual override with attribution (C), physical-paper fallback (D), and any better option (E), against the nine named criteria, without defaulting to whichever is easiest to build.

**Evidence and evaluation.**

*Option A — full discount/waiver workflow.* High implementation complexity for a solo developer's first substantial build, against a use case the business context itself describes as occasional ("emergency, special circumstance, doctor's discretion" — not routine, high-volume traffic). A multi-state approval workflow adds real operational burden — staff have to learn and navigate a subsystem — for something that, by its own description, happens rarely. This is over-engineered relative to actual frequency and actual current scale (one on-site admin, not a large staff needing gatekeeping).

*Options B and C.* Reading these against each other, they converge once the actual mechanism is specified: a couple of fields recording the actual-charged amount separately from the protected computed baseline, a brief reason, and attribution derived by trigger from `auth.uid()` rather than trusted from client input. B implies a small, labeled UI affordance; C implies the same columns exercised through whatever generic edit capability already exists, with little or no dedicated UI. The meaningful risk sits in one specific implementation detail shared by both: "manual override of the computed amount" must mean *setting a separate field*, never *overwriting the one existing total column directly* — the latter destroys the honest baseline Study 2 said must survive, and is the actual difference between a safe simplification and a dangerous one. Implementation complexity: low. Financial integrity and auditability: good, provided the separate-field discipline holds. Frequency, operational burden, and reporting effects: all favorable, matching Study 2's model closely.

*Option D — keep rare cases on paper.* Evaluated in full in Study 4 below, since the prompt calls for a dedicated pass on exactly this question. Headline for this comparison: it fails financial integrity and reporting badly enough that no other criterion rescues it for anything touching an actual invoiced number.

*Option E.* One genuine refinement worth naming on its own: the *reason* field doesn't need to be a rich, mandatory, structured justification — a brief free-text note is sufficient, and making it heavier would add exactly the kind of front-desk friction that pushes people toward informal, undocumented workarounds in practice.

**Self-challenge.** Is there a real case for A that I'm underweighting? If staff and scale grow well past "one on-site owner personally involved in daily operations," a gate that stops any front-desk staff member from granting themselves an informal discount becomes a real control, not paranoia. That's a legitimate future trigger for building the approval layer — it just isn't the current state, and building the gate now would be solving a staffing-scale problem that doesn't exist yet.

**Conclusion.** B and C are the same recommendation at different points on a UI-investment dial: a minimal, structured, database-level mechanism, no approval gate, no discount-type taxonomy, captured via a separate field that never touches the protected baseline.

**Decision.** Build the minimal mechanism — this directly reverses the prior document's call to cut it entirely. That document's reasoning was sound given what was confirmed at the time (no specific one-off case, only the standing ₹180 one); this prompt's business context supplies exactly the confirmation that was missing.

---

## Study 4 — Specifically analyze the physical-paper fallback

**Question.** When is paper legitimate, when does it create a split-brain financial record, can totals still be reconciled, and is there a hybrid that avoids building a full feature without sacrificing trustworthiness?

**Evidence and the actual test.** The distinguishing question isn't frequency — it's whether a given piece of information feeds into a number the software claims to represent completely. A decision-making conversation between doctor and patient about whether to grant a waiver doesn't itself change any number the software totals; it can stay exactly as informal as it already is. The *result* of that conversation, the moment it changes what appears on an actual invoice, is a different kind of fact entirely — it's now part of the thing the software's revenue total claims to be complete.

**Alternatives.** Could reconciliation between a paper log and the digital totals be reliable enough to make paper safe for rare cases? I don't think so, structurally: reconciliation is a manual, effortful, skippable step, and a software system's entire value proposition as "the record" collapses the moment any real money moves through a channel the system doesn't know about — not because it happens often, but because the total is then *quietly* wrong rather than *obviously* wrong, which is worse. A once-a-year off-book transaction corrupts that period's revenue total exactly as permanently as a weekly one would.

**Self-challenge.** Is there a genuinely legitimate category of paper-only pricing information, or am I reflexively defaulting to "everything must be digital" the way I was explicitly warned against? I think there is a real category, and it's narrower than "rare cases": anything that never directly changes an invoiced number — a doctor's private sense of which longtime patients he's fond of, general goodwill, none of it tied to an actual different charge — can live however it currently lives, because it was never part of the financial record to begin with. The line isn't rarity; it's whether money is involved.

**Conclusion.** Paper is legitimate for the deliberation that precedes a pricing decision. It is not legitimate — regardless of how infrequently it happens — for the resulting number on any invoice, or for the rule that determines a standing patient's ongoing rate, because both of those are exactly the things the software's own totals claim to represent completely.

**Decision.** No pricing exception, however rare or discretionary, gets left off the system. The clinic's decision-making process can stay as informal as it already is; the number that decision produces always goes into the software, using the minimal mechanism from Study 3.

---

## Study 5 — Old patients and price changes

**Question.** When the standard price rises, existing patients keep the old price and new patients get the new one, with some patients getting further discretionary reductions on top — what does the application need to remember for future visits to bill correctly? Historical price tables, a patient override, an effective-dated rate, a transaction adjustment, or some simpler combination?

**Evidence — walking the actual scenario.** Patient A registered in January at ₹150. In April the clinic raises its standard rate to ₹200. Patient A keeps paying ₹150. Patient B registers in May and pays ₹200. In June, Patient A also gets a further one-off reduction to ₹100 for one specific visit.

**Alternatives.** I initially reached for the model I already had in hand — a clinic-current rate that everyone follows by default, with an override table used only for exceptional patients — and tried to make the grandfathering scenario work under it. It doesn't work safely. Under that model, the moment the clinic's standard rate changes to ₹200, *every* patient without an explicit override row would silently start being charged ₹200 at their very next visit — including Patient A, who's supposed to keep paying ₹150 — unless someone remembers, at the exact moment of the price change, to proactively create override rows freezing every currently-active patient at their old rate. That's a real, consequential failure mode resting entirely on a human remembering a manual bulk action, for a scenario the business context describes as the *default* expectation, not an exception requiring active intervention.

The alternative that actually matches the stated business behavior: every patient gets their own effective rate, snapshotted automatically at registration from whatever the clinic's current rate happens to be at that moment, and editable afterward for a standing exception. Under this model, Patient A's rate is set to ₹150 in January and simply never touched again when the clinic's rate changes in April — correct behavior with zero risk of anyone forgetting a step. Patient B, registering in May, gets ₹200 snapshotted automatically at that later moment. No price-history table is needed anywhere in this, because each patient carries their own current fact forward, and each past invoice already permanently records what was actually charged, independent of whatever the rate tables say today.

**Self-challenge.** Doesn't giving *every* patient a rate row, not just exceptional ones, add complexity relative to an exceptions-only table? In raw row count, marginally — one small value per patient instead of per exception. But in conceptual weight, it's smaller, not larger: it can live as one or two columns directly on the existing `patients` table (populated by a simple registration-time trigger reading the clinic's current rate, the same "derive, don't leave to chance" pattern already used elsewhere in this schema), rather than a separate table with its own type enum, expiry, and active-flag. I also checked whether this needs to vary by branch, given patients are chain-level identities visible across three locations — that's downstream of the still-open question (from the prior document) of whether the three branches actually charge independently different rates at all. If they do, this needs a nullable `clinic_id` dimension added later; if they don't, it doesn't. Either way that's a cheap, additive change, not a reason to pre-build branch-scoping now.

**Conclusion.** No price-history table. No exceptions-only override model, because it fails the exact scenario it's meant to handle. A per-patient rate snapshot, populated automatically at registration and editable afterward, correctly and safely produces "old patients keep old price, new patients get new price" with no administrative action required at the moment of a price change.

**Decision.** Replace the originally-proposed `patient_rate_overrides` (a standalone table, exceptions-only, with a type enum and expiry) with a lighter mechanism: rate fields directly on `patients`, populated for every patient at registration via a trigger, editable afterward for a standing exception, with a brief reason. This is a genuine structural simplification, not just a renaming — flagged fully in Study 6.

---

## Study 6 — Challenge the existing architecture

**Question.** Against everything concluded above, what in the currently proposed design is over-engineered, under-modelled, wrongly conflated, built from future-proofing anxiety, or missing in a way that would make financial history ambiguous?

**Over-engineered.** The originally-proposed `patient_rate_overrides` table — a standalone table with an `override_type` enum (`CONSULTATION_FEE`/`THERAPY_RATE`/`FLAT_TOTAL`), `expires_at`, and `is_active` — carries more structure than Studies 1 and 5 found necessary. `FLAT_TOTAL` specifically isn't a separate concept once expected-vs-actual is the actual storage model, and `expires_at`/`is_active` presume a time-limited-promotional-discount need that's never been confirmed as real, only speculated about.

**Under-modelled — the one genuine reversal.** The prior document cut the transactional visit-level override columns entirely, on the grounds that no confirmed one-off waiver case existed distinct from the standing ₹180 case. This prompt's business context directly supplies that missing confirmation — emergency and discretionary one-off waivers are stated as real. That cut needs to be undone. I want to be precise about what changed: the earlier conclusion wasn't wrong given what was confirmed at the time; the premise it was reasoning from has since changed.

**A second, related under-modelling.** The exceptions-only shape of the original `patient_rate_overrides` design doesn't actually handle the grandfathering scenario it was presumably meant to help with — Study 5 found it fails silently the moment a clinic-wide price change happens without a perfectly-remembered manual intervention. This wasn't previously flagged anywhere in the project's documents as a risk, because nobody had walked the actual price-change scenario against the design before now.

**A conflation worth naming precisely, for whoever eventually builds the fee-computation trigger.** The rule-computed baseline (`grand_total_in_paise` or its four-bucket successor) must be derived by checking the *patient's own* rate first and falling back to the clinic's generic rate only if the patient has none — not the reverse. Getting this lookup order backwards would silently charge every discounted or grandfathered patient the generic rate, which is exactly the kind of bug that wouldn't announce itself; it would just quietly overcharge real people. This isn't a new schema element, just a load-bearing detail worth stating explicitly before anyone writes that trigger.

**Future-proofing anxiety.** `expires_at`/`is_active` on the original override design, and (unrelated to pricing specifically, already flagged previously) `clinic_fee_sharing`/`clinic_gap_rules`.

**Missing information that would make history ambiguous.** Without the transactional override's reason and attribution now being restored, "how much discretionary waiver value went out this quarter, and who approved each instance" would be structurally unanswerable — exactly the ambiguity Study 2's model exists to prevent, and exactly what was at risk under the prior cut. On the reassuring side: nothing about historical correctness is actually missing once each visit's charged amount is permanently recorded at write time and never recalculated later — the "don't reconstruct the past, record it once and leave it alone" principle already used elsewhere in this schema fully covers this.

---

## Final Output

**1. The pricing domain model, in plain language.** Every patient has their own effective rate, set automatically when they register from whatever the clinic is currently charging, and changeable afterward if the doctor decides to give that specific patient a standing arrangement. That personal rate — not a generic clinic-wide number — is what "the rule" means for any given patient, and it needs no justification on any individual visit, because it was already justified once, when it was set. On top of that personal-normal rate, any single visit can additionally receive a one-off adjustment — an emergency waiver, a discretionary reduction — which does get a reason and a recorded approver, precisely because it's the irregular layer, not the normal one. What actually got paid is tracked as a third, separate fact from what was charged.

**2. Minimum pricing capabilities required.** A current, editable clinic/tier rate. A per-patient effective rate, snapshotted at registration, editable afterward. A one-off, per-visit adjustment with a reason and trigger-derived attribution. Nothing beyond that — no discount-type taxonomy, no approval workflow, no price-history table, no expiring promotional-discount mechanism.

**3. What should exist in schema now.** Effective-rate field(s) on `patients` (populated at registration, editable later). The transactional override fields on `visits` — reinstated per Study 3/6 — as a separate field from the protected computed total, plus a reason and an attribution column.

**4. What should exist in backend logic now.** A registration-time trigger snapshotting the clinic's current rate onto the new patient record. A trigger deriving the one-off override's attribution from `auth.uid()`, never from client input. Whenever the fee-computation logic is built, it must check the patient's own rate before falling back to the clinic's generic rate — the lookup order flagged in Study 6.

**5. What frontend can be deferred or simplified.** No approval-workflow UI. No discount-type selector. A plain, admin-editable field is sufficient for both a standing rate change and a one-off adjustment; a labeled, purpose-built screen for either can wait. Any "how much did we waive this month" reporting view is a later convenience, not an MVP requirement — the underlying numbers being captured correctly is what matters now.

**6. Whether paper is acceptable, and for what.** Acceptable only for the informal decision-making that precedes a pricing exception. Not acceptable, at any frequency, for the resulting number on an invoice or for a standing patient rate — both are exactly the facts the software's own totals claim to represent completely.

**7. The minimum design that preserves trustworthy reporting.** The computed baseline is never overwritten, and it must already reflect the patient's own rate, not a generic one. Any deviation from it lives in a separate field, with a reason and a real, trigger-derived approver. Nothing about the past ever gets recalculated once it's written.

**8. Changes to the previous documents' conclusions.** Reinstate the transactional visit-level override columns cut in `mvp-revalidation-four-studies.md`'s Study 3 — new, confirmed business context supersedes that call. Replace the originally-proposed `patient_rate_overrides` table (standalone, exceptions-only, with a type enum and expiry) with rate fields directly on `patients`, populated for every patient at registration, not just exceptional ones. Drop `FLAT_TOTAL` and the type enum generally, per Study 1. Drop `expires_at`/`is_active`, per the same reasoning already applied to future-proofing elsewhere in this project.

**9. Do now / later / don't build yet.**
- **Do now:** per-patient effective-rate field(s) on `patients` + registration-time snapshot trigger; the one-off transactional override fields on `visits` + attribution trigger; the discipline that fee-computation logic checks patient-specific rate before clinic-generic.
- **Do later, only if a real need appears:** any structured reporting view over discount/waiver activity; branch-scoping the per-patient rate (only if the still-open "do the three branches actually charge differently" question comes back yes).
- **Don't build:** a discount-type taxonomy, an approval workflow, a price-history/versioning table, an expiring-promotional-discount mechanism, and any paper-based handling of an actual charged amount.
