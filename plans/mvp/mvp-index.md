# MVP Reports Index — Read This One First
### Covers: `mvp-stepback-review.md`, `mvp-scope-diagnosis.md`, `mvp-revalidation-four-studies.md`, `mvp-pricing-waiver-analysis.md`

This file exists so you never have to open all four again just to know where things currently stand. It gives you the *current, reconciled* answer on everything — including the two places where a later document overturned an earlier one — and points into a specific file and section only when you actually want the reasoning behind an answer, not the answer itself.

---

## Ask the clinic owner these six things first

Nothing below is final until these come back, and none of them require more design thinking on your end — each is a direct question with a direct answer.

1. Are packages being sold and used at this clinic *today*, on paper, right now — or is that still aspirational?
2. Does "settles weekly" describe real current patient-payment behavior, or was that hypothetical?
3. Does the ₹150 lapse penalty's 10-day gap get measured against the complaint (it reactivating after its own gap) or the patient (any gap, even ending in a brand-new complaint)?
4. Do the three actual clinic locations charge different tier rates from each other today?
5. Does this clinic ever treat someone for a specialty service with no complaint at all — a genuine walk-in, nothing diagnosed?
6. Is retail product sale (braces, tape, topical products at the counter) a real near-term need?

*(Where these live: all six first appear in `mvp-revalidation-four-studies.md`, Final Output §3.)*

---

## The current MVP boundary

Seven workflows, not six — the seventh is new as of the second document:

1. Register a patient, any branch, one shared chain-level identity.
2. Open or select a complaint, existing or new.
3. Log today's visit with the fee computed correctly against the confirmed rules.
4. Generate an invoice covering that visit (or the day's batch) and record payment.
5. Keep branches isolated — staff see only their branch, the admin sees the chain.
6. Look up a patient's history before treating them again.
7. **If a patient has an active pre-paid package, log the covered visit correctly (₹0 therapy charge, history intact) without the full attendance system** — conditional on confirm-question 1 above.

---

## Build now

Four-bucket money columns on `visits` (`exam_fee`, `lapse_penalty`, `therapy_fee`, `services_total`). Real RLS policies, replacing `v1_allow_all` — this is the single most urgent item in the whole series. `created_by` derived from `auth.uid()` via trigger everywhere it appears, bundled with the RLS rollout. `invoice_line_items` (minimal, visit-sourced only) together with the ghost-invoice sum-matching trigger — the two ship together, not separately, since the trigger is what makes an invoice total mean anything once line items exist. A per-patient effective rate, snapshotted automatically at registration and editable afterward, living as fields on `patients` — **not** a separate override table. A one-off, per-visit transactional override (separate field from the protected computed total, plus a reason and trigger-derived attribution) on `visits`. `patient_flags` (the red-zone table) — small, cheap, and confirmed needed.

## Build conditionally, pending the confirm-questions above

`visits.package_id` and the minimal package-visit linking (question 1). Allowing `payment_status = 'Pending'` at invoice creation (question 2). Clinic-scoping the tier-rate table (question 4).

## Defer entirely — don't design further, don't build a stub

The full Packages module (attendance tracking, hold, cancel/refund). `treatment_events` and any specialty multi-complaint linking — don't even model the table shape. The session/`session_id` concept. Retail (`product_sales`/`product_sale_items`). Split-payment columns and Overdue auto-derivation. Fee-scope configurability (`clinic_fee_sharing`/`clinic_gap_rules`) — hardcode the one clinic's actual behavior instead. A discount-type taxonomy, an approval-workflow state machine, a price-history/versioning table, and any expiring-promotional-discount mechanism.

---

## The invariants that actually matter

Money stays integer paise; a generated total always equals the sum of its parts. Invoice numbers are unique per clinic, assigned exactly once. A patient's chain identity is never ambiguous. Branch isolation — currently **violated**, not just unenforced, since `v1_allow_all` means the invariant doesn't hold today. Fee-amount correctness is frontend-trusted, not backend-guaranteed, for this pilot — an accepted, documented risk given a small trusted staff pool, not a silent gap. Any override's attribution must come from a trigger reading `auth.uid()`, never from client-submitted data. An invoice's line items must sum to its total — enforced only if the ghost-invoice trigger actually ships alongside `invoice_line_items`.

---

## What changed across the series, and why

Tracking this matters more than usual here, because two calls got reversed and one design got restructured — if you only ever read the oldest document on a given topic, you'll be working from a superseded answer.

**Correction (not a reversal — an oversight caught and fixed):** the first review flagged "joining fee vs. per-complaint exam fee" as an open, must-decide blocker. It wasn't — `FEECOMP_DISCUSSION.MD` had already resolved it (exam fee scoped to patient lifetime, charged once ever) before that review was even written. Fixed in `mvp-scope-diagnosis.md`.

**Reversal #1, later itself reversed:** `mvp-revalidation-four-studies.md` cut the transactional visit-override columns entirely, reasoning that no confirmed one-off waiver case existed. That was correct given what was confirmed at the time.

**Reversal #2:** `mvp-pricing-waiver-analysis.md` un-cut them. New business context (emergency and discretionary one-off waivers are real, confirmed) supplied exactly the confirmation that was missing. The columns are back in, for good, unless something changes again.

**A structural revision, not just a reversal:** the originally-proposed `patient_rate_overrides` — a standalone table, exceptions-only, with a discount-type enum and an expiry field — got replaced with per-patient rate fields living directly on `patients`, populated for *every* patient at registration. This wasn't a style preference; the exceptions-only design would have silently started overcharging every existing patient the moment the clinic raised its standard price, unless someone remembered to manually freeze them at the old rate first. The new shape closes that risk by construction.

**Two new findings, not reversing anything, just previously unflagged:** the ghost-invoice sum-matching trigger is required the moment `invoice_line_items` exists, not optional insurance — without it, nothing connects an invoice's total to what it claims to total. And `created_by`, used everywhere in the schema, never got the same "derive from `auth.uid()`, don't trust the client" treatment that `owner_id` did — a real inconsistency, not a deliberate choice.

---

## File-by-file: what's uniquely there, and when to go get it

**`mvp-stepback-review.md`** — the broadest scout, before the boundary was pinned down. Go here for the *why* behind the project's size: the Headline Finding and Section A trace how a design track (`specialty-decision-log.md`) generated retail commerce, a session concept, and a payment lifecycle largely ahead of any confirmed need. Section B has the fullest current-vs-simpler-vs-foundation-preserving treatment of individual deferred items (session_id, retail, treatment_events, fee-scope configurability) if you want the long-form argument behind a short conclusion stated elsewhere.

**`mvp-scope-diagnosis.md`** — the first real boundary-setting pass. Go here for Section 6, the concrete UI/workflow-level cuts that don't appear anywhere else (MRN stays manual, drop the Clinical Notes auto-categorization, hide the Vitals Grid, invoice creation as one atomic action, cross-branch patient search stays a manual phone-lookup workaround). Section 7 has the named architectural risks, including the documentation-maintenance-load and interview-narrative points.

**`mvp-revalidation-four-studies.md`** — the re-audit. Go here for Study 3 specifically if you ever need to *defend* a schema decision (in an interview, or to yourself six months from now) — it's the only place that separates "this survives because deferring it is a genuinely hard migration" (only the four-bucket columns qualify) from "this survives because of a confirmed current need, not migration cost" (everything else). Study 4 is the sharpest treatment anywhere in the series of what's actually backend-guaranteed versus merely frontend-computed-and-trusted.

**`mvp-pricing-waiver-analysis.md`** — the pricing deep dive. Go here for Study 5 if you want to actually see *why* the patient-rate mechanism changed shape — the index above only gives you the conclusion, not the walked-through scenario where the old design breaks. Study 4 is worth reading if you ever need to explain, precisely and without sounding dogmatic about it, why "just keep the rare ones on paper" doesn't hold up.

---

## If you have a specific need right now

| You want to... | Go to |
|---|---|
| Start building | The "Build now" list above — that's the actual starting point, no other file needed |
| Know what to ask the owner | The six questions above — already complete |
| Understand why the project ballooned | `mvp-stepback-review.md`, Headline Finding + Section A |
| Get concrete UI-level cuts | `mvp-scope-diagnosis.md`, Section 6 |
| Defend a schema call under questioning | `mvp-revalidation-four-studies.md`, Study 3 |
| Understand the pricing model deeply | `mvp-pricing-waiver-analysis.md`, Studies 1, 2, and 5 |
| Check whether something backend-guarantees a number or just computes it | `mvp-revalidation-four-studies.md`, Study 4 |
