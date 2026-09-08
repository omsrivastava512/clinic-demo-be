# MVP Scope & Architecture Diagnosis
### A rigorous boundary for the clinic management system — what ships now, what waits, and why


> **[COMPACTED PROMPT]**
> <details>
> <summary><b>Click to view the Compacted Scoping Prompt</b></summary>
> Evaluate and diagnose the physiotherapy clinic management system scope for a solo-developer real-world deployment, establishing the absolute minimum trustworthy MVP while explicitly decoupling schema, backend/business logic, and frontend UI timing. First, regarding **Domain Complexity & Root-Cause Diagnosis**—analyze why the project expanded during schema design, distinguishing genuine business domain complexity from unnecessary premature design. Second, regarding **True MVP Boundary & Business Workflows**—define the minimal operational boundary required for clinic trust, prioritizing core financial and data integrity invariants (pricing, consultation fees, therapy categories, waivers/discounts, package/payment tallies) over convenience features. Third, regarding **Requirement & Architectural Classification**—for each core requirement, evaluate: `Requirement → business purpose/invariant → MVP necessity → dependencies → minimum implementation → deferral strategy` (categorized strictly as: *implement now*, *model schema now but defer UI/backend*, *partial backend now*, or *defer concept entirely*). Fourth, regarding **Scope Reduction & Risk Audit**—identify aggressive simplification strategies (manual interventions, admin overrides, static configurations) while flagging premature future-proofing, edge-case inflation, and critical architectural risks. Fifth, regarding **Definition of Done**—establish objective, verifiable criteria for "MVP DONE" across the 8 specified diagnostic deliverables, utilizing bottom-line summaries from supporting project files (`DEFERRAL.md`, `specialty-decision-log.md`, newly attached draft UI specs) without re-deriving resolved decisions.
>
</details>

---

This document answers the scoping prompt directly: why the project got this large, where the true MVP boundary actually sits, a requirement-by-requirement classification with schema/backend/frontend timing separated out, the real financial and data invariants, what to model now versus never, aggressive scope cuts, the architectural risks worth losing sleep over, and an objective "MVP DONE" line. It builds on `mvp-stepback-review.md` (the previous deliverable in this project) rather than re-deriving everything in it — where this document needs a finding from that one, it says so and moves on, rather than re-arguing it from scratch.

---

## A Correction First

Before anything else: `mvp-stepback-review.md`'s item C1 is wrong, and I want to fix it in the open rather than quietly. It listed "joining fee vs. per-complaint exam fee" as an unresolved, must-decide-before-implementation blocker. It isn't. `FEECOMP_DISCUSSION.MD` — which was already in the project when I wrote that review — resolves it directly: *"the exam fee is scoped to the patient's lifetime, not the complaint. Charged once, ever. A returning patient with a brand-new complaint pays 0 exam fee."* That's stated as something you had already confirmed in that conversation, not a proposal waiting on you. I had this file, and I even worked through its content while reasoning through the previous review — I just failed to carry the correction into the final written output. That's a real miss, and it's exactly the kind of thing worth catching rather than letting compound, so I'm flagging it plainly rather than burying it in a footnote.

What's actually still open, precisely: the exam fee's **recurrence scope** (lifetime vs. per-complaint) is resolved — lifetime. What is genuinely still open is one narrower question, explicit in `FEECOMP_DISCUSSION.MD`'s own changelog: does the ₹150 lapse penalty's 10-day gap get measured against the **complaint** (an existing complaint reactivating after its own gap) or the **patient** (any gap, even one that ends with a brand-new complaint)? That's a real, live, must-decide item — I'm keeping it in this document's classification below, just correctly scoped this time. Also worth knowing: the three-table axis redesign in that same file (`exam_fee_recurrence_scope` column, `clinic_fee_sharing` table, `clinic_gap_rules` table) is explicitly marked "proposed — needs your sign-off before it's logged as settled." The *recurrence* answer is confirmed; the *schema shape* for representing recurrence-plus-sharing-plus-gap-rules generally is not, and Section 5 below has a specific recommendation on how much of that shape you actually need now.

**On the file situation more broadly, since you asked me to recheck:** the project directory is now stable at 12 files. `FEECOMP_DISCUSSION.MD` and `specialty-decision-log.md` both carry fresh timestamps from just now, but reading both in full, their content is identical to what I already had — nothing new to fold in from either. `codebase-context.md`, which was present earlier in this conversation, is no longer in the project; I don't currently have it, and I'm proceeding without it since nothing below depends on frontend implementation details it uniquely held. `FRONTEND_WORKFLOW.md`, `FRONTEND_PATIENT_PROFILE.md`, and `PACKAGES_DEFERRAL.md` are unchanged from what I already had. So: no new information surfaced by the file check, one real correction did.

---

## 1. Why the Project Has Become So Large

Three genuinely different sources, and they don't deserve the same response.

**Legitimate: twelve iterations of tenancy hardening that were fixing real bugs, not adding features.** `backend-schema-iterations-corrected.md`'s v1→v11 history is a record of catching and closing actual defects — a broken index that would have crashed the migration script, a trigger silently gutted to a no-op, an intake deadlock where two tables each waited on the other, an admin policy that locked every admin out by checking a column that's `NULL` for admins by design. None of that is scope creep. It's the ordinary cost of building a correct multi-tenant system, and it would look the same size if you'd built it fast and sloppy first and then had to fix it under pressure later — you're paying it now, deliberately, which is the cheaper way to pay it. This part of the size is earned. Don't cut here looking for MVP savings; there's very little to cut.

**Avoidable: the specialty/session/invoicing design track has been generating scope faster than it's been getting confirmed.** `specialty-decision-log.md` now holds 31 numbered entries across 13 chat sessions. The overwhelming majority are marked **OPEN** and, in several cases, explicitly *"not confirmed by Om."* In the course of that track, three things got invented that trace to no client conversation anywhere in `business-rules-log.md`: a full retail point-of-sale system (`product_sales`/`product_sale_items`, walk-in anonymous buyers, `invoices.patient_id` going nullable), a "shallow patient / service-seeker" archetype with its own polymorphic body-region linking, and a payment lifecycle (Pay Later, split cash/online/card columns, a derived Overdue status) that the schema's own comment already called "future" before any of this started. None of these are bad ideas in isolation. All of them were generated by the design process itself continuing to run, not by the clinic asking for them. That's the part of the size that's avoidable, and Sections 2, 3, and 6 below are mostly about pulling it back out.

**A subtler third source: the documentation has outpaced the code.** You have zero lines of backend or frontend implemented against this schema yet — that's stated plainly in the new prompt. What you have instead is twelve schema iterations, a fee-computation redesign, a pricing-override architecture, a packages-deferral analysis, and thirty-one specialty decisions, almost none of it built. That's not wrong to have done — reasoning carefully before writing code is good practice, and this project's own migration-safety framework (Module 9) exists specifically because "pasting an ALTER TABLE straight into the dashboard" is the failure mode being avoided. But there's a point past which more design-on-paper stops de-risking the build and starts *being* the risk, because every new decision generates updates that, per the decision log's own "Where it lands" convention, need to propagate across three or four separate files before anyone can trust them as current. You're a solo developer. That propagation cost is real, and it competes directly with the hours you'd otherwise spend writing code that ships.

**One more thing worth naming since you asked me to challenge your thinking, not just the schema.** This project is doing double duty — a real clinic tool and your primary interview artifact. That's a legitimate reason to care about doing things properly. It's also a plausible quiet driver of some of the size: a maximally-elaborated, unshipped design doesn't actually make a stronger interview story than a smaller, shipped system you can defend line by line, including a clear account of what you deferred and why. If anything, this document and the previous one *are* that account — the artifact value of "I knew what to cut and could say why" doesn't require the cut things to also exist.

---

## 2. The True MVP Boundary

Defined by what the clinic must actually be able to do end to end, not by a feature list:

1. **Register a patient** — any of the three branches, walk-in or referred, with one identity shared correctly across the chain.
2. **Open or select a complaint** for that patient — an existing active one, or a new one.
3. **Log today's visit** — which complaint(s), which services or machines, consultation or machine-only — and have the fee come out **correct** against the confirmed rules: exam fee once per patient lifetime, per-complaint therapy rate by tier, the lapse penalty when it applies, machine-only billed by itemized machine use.
4. **Generate an invoice** for that visit (or that day's batch of visits for one patient) and **record payment against it**.
5. **Keep branches isolated** — a Clinic A receptionist cannot see Clinic B's patients, visits, or invoices; the owner sees the whole chain.
6. **Look up a patient's history** — past complaints, past visits — before treating them again.

That's the boundary. Notice what's *not* in it, deliberately: packages and subscriptions, specialty treatments spanning multiple complaints in one structured record, retail sales, split or deferred payment, and the red-zone flag (real and cheap, discussed below, but not blocking — a clinic can operate its first weeks without it). Everything in Section 3 gets checked against whether it's actually load-bearing for one of these six workflows.

---

## 3. Requirement Classification

The quick-reference table first, then the judgment calls that need more than a cell can hold.

| Requirement | MVP necessity | Schema now? | Backend logic now? | Frontend now? |
|---|---|---|---|---|
| Multi-branch tenancy (`clinic_id`/`owner_id`/`patient_clinic_access`) | Must have | Done already | Done already | Needed (branch-scoped views) |
| Real RLS policies (replacing `v1_allow_all`) | Must have before real patient data | Policy SQL only | N/A | N/A |
| Four-bucket fee columns + three visit states | Must have — this is the product | Yes | Yes | Yes |
| Exam-fee lifetime scope + gap-measured-against | Must have (recurrence resolved; gap-basis still needs Om) | Yes, once gap-basis is answered | Yes | N/A (invisible to the user) |
| Fee-scope configurability (`clinic_fee_sharing`/`clinic_gap_rules`) | Not needed yet — one clinic, one confirmed scope | No — hardcode | No — hardcode | No |
| Backend fee-derivation trigger (never trust client amount) | Defer the enforcement; keep the frontend calculation correct | No | No, for now | N/A |
| Transactional visit override (`final_amount_in_paise` etc.) | Model now, simplest form; defer approval workflow | Yes (2 columns + `override_by`) | Direct-edit only | Not needed (DB edit suffices) |
| Standing per-patient override (`patient_rate_overrides`) | Confirmed real need (the ₹180 case) | Yes | Yes | Minimal (admin sets it) |
| `invoice_line_items` | Must have, minimal (visit-sourced only) | Yes | Yes | Yes, simple list |
| `visits.package_id` + `therapy_fee_in_paise` as pluggable hooks | Must exist, inert | Yes (column only) | No trigger | No |
| Full Packages module (attendance, hold, cancel/refund) | Deferred (your own call) | No | No | No |
| `treatment_events` / specialty multi-complaint linking | Defer the whole concept | No | No | No |
| Session / `session_id` concept | Defer the whole concept | No | No (atomic function instead) | No |
| Retail (`product_sales`/`product_sale_items`) | Defer the whole concept | No | No | No |
| Payment lifecycle (Pay Later, split payment, Overdue) | Defer — CHECK values already sit unused | No change | No | No |
| Patient flag / red-zone (`patient_flags`) | Confirmed, cheap — build the core now | Yes | Yes | Minimal badge/list, polish later |
| `patient_vitals` / Vitals Grid | Confirmed non-real for physio practice | Table already exists, leave inert | No | Skip or hide |
| MRN generation | Manual/free-text is fine | No change | No | No change |
| Cross-branch patient search (existing patient, new branch) | Handle manually for now | No | No | No — manual lookup by phone |

Now the ones that need the actual reasoning, not just the cell.

**RLS is the one item on this list I want to push back on in the opposite direction from most of this document.** Everything else here is me arguing for cutting or deferring. RLS is the opposite case: `v1_allow_all` is fine while you're the only person touching a dev database with fake data, and it becomes a real problem the moment an actual patient's name, phone number, and medical complaint sit in a table any authenticated user can read. That's not a hypothetical future requirement — it's the current state of a system that's meant to hold real people's health information starting soon. I'd treat "real RLS policies are live" as a hard gate on calling this MVP done, not a nice-to-have you get to when there's time. The policies themselves are already drafted as SQL comments in `supabase_migration.md` — this isn't new design work, it's turning on something already written.

**The fee-derivation trigger is the one place I'm recommending you defer something that looks, on paper, like a core invariant.** The principle — never trust a client-submitted fee amount — is correct and should be the eventual state. But building it requires the rate-table infrastructure to exist first, and it protects against a narrower risk (a client submitting a wrong number) than the one still wide open right next to it (RLS being off entirely, so a client could write directly to `invoices.amount_in_paise` and skip this protection altogether). For a pilot with a small number of known, trusted front-desk staff, I'd let the frontend's already-correct calculation logic carry this until either RLS is real or the trust model actually needs it. This is genuinely a case of "worrying about the more sophisticated problem before closing the cruder one right next to it."

**`clinic_fee_sharing`/`clinic_gap_rules` deserve a direct callout on the "worrying unnecessarily about future migrations" front you asked me to watch for.** The stated reason for preferring a lookup table over a simple CHECK-constrained column, in both `handover-fee-computation.md` and `FEECOMP_DISCUSSION.MD`, is avoiding a migration if a third tier or a second fee-sharing scope is ever needed. But this project's own Module 9 migration-safety framework already classifies "adding a CHECK constraint that all existing rows already satisfy" — which is exactly what widening a fake-enum to add one more allowed value is — as a *safe, low-risk* change. You're paying real schema complexity today (three new tables, an axis-based rule engine) to avoid a migration your own documented standards say is cheap. That's the pattern to watch for generally, not just here: check the actual cost of the thing you're avoiding before building around it.

**`treatment_events` and the session concept are the two items where I'd defer harder than "model now, build later" — I'd say don't even model them.** A schema stub sitting unused isn't free; it's a claim about the shape of a decision (standalone table vs. extending `visit_services`, day-level vs. submission-batch session) that hasn't actually been made yet, and per `specialty-decision-log.md`'s own Entry 27, the underlying question ("is `visits.complaint_course_id NOT NULL` even the right premise") is still open. Building a table now commits you to an answer before the question's settled. Logging specialty services as ordinary `visit_services` rows against one complaint is a complete model for MVP — not a stub waiting to be replaced, an actual working answer — with the one condition to verify being whether any specific specialty service is confirmed to price per-complaint rather than flat-once (if none do, there's no billing gap at all, only a reporting one).

**Patient flags are the clearest case of the opposite failure — nearly cutting something the clinic actually needs because it sits next to a pile of deferred, thematically-adjacent stuff.** It's a small table (`patient_id`, `owner_id`, `flagged_by`, `reason`, `created_at`), it reuses the same `derive_owner_id_from_patient()` pattern already live elsewhere, it was explicitly resolved as its own table (not bolted onto `patient_alerts`) in `business-rules-log.md` §1, and it protects staff from a real, named risk (disputes, abusive patients, "has a lawyer, be careful"). The only genuinely open question is whether flagging should be admin-only or open to any staff — a permissions question, not a scoping one. I'd build the table and the write path now; the UI can start as a simple badge and a filtered list, with the "surfaced near the top of some list, unmissable years later" polish coming after MVP.

---

## 4. Data and Financial Invariants

These are the properties that must hold without exception for the system to be trustworthy — not preferences, not conveniences, actual invariants.

1. **Money is integer paise, never a float, anywhere.** Already structural. Nothing changes this.
2. **A generated total must always equal the literal sum of its parts.** `grand_total_in_paise` today, the four-bucket sum once that ships — it has to stay a `GENERATED` column, full stop, even once overrides exist. The override lives in a separate `final_amount_in_paise` column specifically so the honest "what the rules actually computed" number is never lost, even when it's not what got billed.
3. **An invoice number is unique per clinic and assigned exactly once, atomically.** Already enforced via the `SECURITY DEFINER` counter-increment trigger. Nothing about MVP scoping touches this.
4. **A patient's chain identity is never ambiguous.** `unique(owner_id, mrn)` — already enforced.
5. **A staff member cannot read or write another branch's data.** This is currently **violated**, not just unenforced — `v1_allow_all` means the invariant doesn't hold today. This is the single largest gap between "invariant" and "actual state" in the whole system, and it's why Section 3 treats real RLS as a hard MVP gate rather than a normal backlog item.
6. **The amount charged must match the confirmed business rules, every time.** For MVP, this is enforced by the frontend's calculation logic being correct against `business-rules-log.md` §2 and `FEECOMP_DISCUSSION.MD`'s resolution — not yet backend-enforced (see Section 3's fee-derivation entry), which is a real, accepted gap for the pilot's trust level, not an invariant currently holding by construction.
7. **Any deviation from a computed amount is attributable to a specific admin, never anonymous.** Even in the simplified override design (two plain columns, no approval workflow), `override_by` must still be derived from `auth.uid()` at write time, not accepted as a client-submitted value. This one small piece of the override machinery isn't optional even though the rest of it is being simplified.
8. **If line-item invoicing ships, an invoice's line items must sum to its total.** Not yet built, but worth stating now since it's the one piece of the specialty track's invoicing proposal I'd keep even in the minimal version (the deferred ghost-invoice constraint trigger) — it protects real financial data cheaply.

Everything else discussed across both this document and the previous one — split payment breakdowns, Overdue auto-derivation, live attendance-day counts, multi-complaint specialty revenue attribution, a persisted session audit trail — is a convenience or a reporting nicety, not an invariant. Losing it doesn't corrupt anything; it just means a report is less granular than it could be.

---

## 5. Schema-Now vs. Schema-Later vs. Never-Yet

**Build now:** four-bucket money columns on `visits`, the gap-measured-against answer once you give it, `visits.package_id` (inert), `invoice_line_items` (visit-sourced only), `patient_rate_overrides`, the two-column transactional override, `patient_flags`.

**Model the concept, don't build the table:** clinic-level fee-sharing and gap-rule configurability. You don't need `clinic_fee_sharing`/`clinic_gap_rules` as tables yet, but the *idea* that these values are properties of a clinic, not universal constants, should live in one centralized function or config point in your application logic — not hardcoded in five different places that would all need finding later. That's the actual difference between "premature schema" and "not stupid about where the assumption lives." The schema itself can wait; the discipline about not scattering the assumption can't.

**Don't model yet, genuinely defer the whole concept:** `treatment_events`, `treatment_event_links`, the session/`session_id` table, `product_sales`/`product_sale_items`, the split-payment columns, the full Packages attendance/hold/cancel machinery. None of these get a "just in case" nullable column sitting in the schema today. Building a stub for a decision that hasn't been made is worse than not building it, because it looks finished and isn't.

---

## 6. Scope Cuts

Concrete, not abstract:

- Override approval becomes "an admin edits a field," not a pending/approved workflow with its own state machine.
- Fee-scope configurability doesn't exist as a setting; it's hardcoded to what this clinic actually does.
- MRN stays manual/free-text; no atomic-counter mechanism matching `invoice_number`'s pattern.
- The "Clinical Notes Builder" auto-categorization (fall-risk detection, etc.) on patient registration is UI intelligence you don't need yet — a plain notes field is a complete MVP.
- The Vitals Grid gets hidden or replaced with nothing; it's confirmed non-real for physiotherapy practice, and building a "correct" physio-specific version (VAS pain score, ROM) is a new feature, not a fix.
- Invoice creation is one atomic action end to end — no separate preview state, no draft invoices.
- Cross-branch patient search (an existing patient showing up at a branch they haven't visited before) is a manual workaround for now — a receptionist searches by phone, confirms identity by eye, proceeds — not a new elevated-privilege search function.
- The Billings tab shows invoice number, date, amount, payment mode, status. No PDF receipts, no refund flow, no line-item drill-down yet.
- Package hold's date-segment representation, missed-days derivation, and `day_log` reconciliation don't get touched at all — they're inside the already-deferred Packages module.

---

## 7. Major Architectural Risks

**RLS staying off while real patient data starts flowing.** Already the headline risk in Section 3 and Section 4; repeating it here because it's genuinely the most consequential item in this whole document, more urgent than any specialty-track question.

**Building `invoice_line_items`/`treatment_events` on top of an acknowledged-unresolved foundation.** `specialty-decision-log.md`'s own Entry 26 — the session write-timing question — was explicitly "handed off to a dedicated follow-up chat" and, as of the file you just gave me, still shows no resolution. Building the elaborate version of the invoicing model before that's settled means building on a foundation the design process itself hasn't finished pouring. The minimal version recommended in Section 3 sidesteps this because it doesn't need a session concept at all.

**Documentation-maintenance load competing with actual build time.** Every new decision in this project's own convention generates a "Where it lands" list spanning three or four files. For a solo developer with finite hours, that propagation cost is a real, ongoing tax, and it's worth deliberately not paying it on anything in the "never yet" list until the decision is forced by an actual build need.

**Stale scaffolding.** If any deferred piece gets partially built — just the schema, without the enforcement or the confirmed business rule behind it — it becomes exactly the kind of thing that has already bitten this project once: a function or table that looks live but isn't, waiting to be cited as current when it's actually been superseded (per your own memory of catching a removed function being described as live). The discipline that avoids this is simple: nothing on the "never yet" list gets even a placeholder column until its concept is actually confirmed and about to be built.

**Interview-narrative risk, stated plainly.** A smaller, shipped MVP with a clearly-reasoned deferral list is a stronger artifact than a larger, unshipped one — you already have the reasoning (this document and the last one), which means the marginal value of designing further, without shipping, is genuinely low right now.

---

## 8. A Concrete Definition of "MVP DONE"

Stop building and move to using the system once every line here is true, not before, and don't wait for anything beyond it:

- [ ] A receptionist, through the real UI (not mocks), can register a patient, open a complaint, log a visit with a fee that matches the confirmed rules exactly, generate an invoice, and record payment — at all three branches.
- [ ] Real RLS policies are live, replacing `v1_allow_all`, and the existing verification checklist in `backend-schema-iterations-corrected.md` has actually been run against a real project, not just read.
- [ ] A patient's complaint and visit history is viewable for follow-up care.
- [ ] The admin sees the whole chain; staff cannot see another branch's patients, visits, or invoices — verified by actually logging in as each role, not inferred from the policy text.
- [ ] Every confirmed fee scenario produces the right number on a real invoice: a plain follow-up, a first-ever visit (₹350 once), two complaints treated the same day (₹350 + 200 + 200 = ₹750), and a gap-return (per whatever the gap-measured-against answer turns out to be).
- [ ] `visits.package_id` and `therapy_fee_in_paise` exist and sit inert, confirmed unused by anything.
- [ ] `invoice_line_items` exists, supports a ₹0 line, and at least one real invoice has gone through it end to end.
- [ ] The clinic owner has run a handful of real patient-days through it without a data-integrity incident.

Everything else in this document's "defer" and "never yet" lists stays there until one of these real workflows demonstrably needs it — not until it looks elegant to have, not until a design conversation generates a good idea for it.

---

## Closing

The honest summary: the domain complexity you've actually built into the tenancy model and the fee engine is earned and mostly correct — that's not where the size problem is. The size problem is almost entirely downstream of a design process that kept generating new, unconfirmed scope (retail, sessions, treatment events, payment lifecycle) faster than the clinic could confirm it needed any of it, plus a documentation layer that's grown large enough to have its own maintenance cost. Pull the MVP boundary back to the six workflows in Section 2, hold the line on RLS specifically because it's the one place you're currently below invariant rather than above it, and the rest of this is genuinely a "later" conversation — worth having again once the pilot is running and something on the deferred list actually starts causing friction, not before.
