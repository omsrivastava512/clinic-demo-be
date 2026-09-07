> [!NOTE]
> **Notice for AI Agents:** This file contains historical discussion iterations and raw context. AI agents do **not** need to iterate through or re-evaluate this discussion. These discussion iterations serve as background context that led to the finalized specification in `study/business_logic/general/migration/handover-fee-computation.md`.

> **[COMPACTED PROMPT]**
> Evaluate three core challenges to the fee computation design in `handover-fee-computation.md` along with an open clinical rule: First (**Challenge 1 — Trigger Simplification**), why does the same-day flat fee check require verifying both visit existence and whether that prior visit actually carried the flat fee, rather than simply suppressing the fee for any subsequent visit on that calendar day, and what concrete edge cases break under this simpler rule? Second (**Challenge 2 — Scope of the First-Time Exam Fee**), correct the conflation between consultation and examination fees: why is the ₹350 initial exam fee treated per-complaint instead of strictly **once-per-patient lifetime** (where subsequent new complaints are examined free and only incur therapy charges)? Third (**Challenge 3 — Clinic Configuration Space**), how do we cleanly model multi-clinic configurability beyond the binary `PER_VISIT_DAY` vs. `PER_COMPLAINT` toggle to support: (1) flat-fee scope (lifetime, per-complaint, per-day), (2) gap recharge triggers (₹150 lapse penalty vs. full ₹350 consultation fee based on gap length/complaint identity), (3) configurable primary gap thresholds (e.g., 10 days vs. 30 days), (4) gap behavior differentiated by complaint type (same vs. new complaint), and (5) tiered secondary thresholds (e.g., >10 days = lapse penalty, >6 months = full reset as a brand-new complaint)? Finally, clarify whether this clinic's 10-day lapse penalty triggers on returning with an entirely *new* complaint or exclusively for the *same* complaint.

---

## FEECOMP-01 — Exam-fee scope correction and gap rule redesign

Working from `handover-fee-computation.md`. All three land on something real — going through each in order, then the open question at the end.

### Challenge 1 — the simplification is not correct

The two-part check isn't redundant caution — it's answering two different questions, and your version only asks one of them:

- (a) *does a visit already exist today for this patient?*
- (b) *did any of today's visits already charge this specific flat fee?*

Your rule only checks (a). But most visits don't carry the flat fee at all — a `MACHINE_ONLY` visit never does (itemized-only, no exam/lapse concept), and a plain `SUBSEQUENT` follow-up doesn't either (therapy only, by definition of that state). So "a visit exists today" tells you almost nothing about whether the fee has actually fired.

**Concrete break:** patient has an ordinary follow-up for Complaint A at 9am — 200, therapy only, no exam fee. At 11am the same day they present Complaint B for the first time ever — this visit should charge the 350, since nothing's charged it yet today. Under your rule: a visit exists today → conclude "not the first" → charge nothing. Free exam. Under the design as written: it looks specifically for a prior visit today with `exam_fee_in_paise` already nonzero, finds none (the 9am visit's is 0), and correctly charges the 11am visit.

Worth noting why this is easy to miss: the AM/PM example already confirmed in the log (neck with 150, back without) works under *either* version, because in that example the earlier visit did charge the fee. The two approaches only diverge when the day's earlier visit(s) didn't — which, given how common plain follow-ups are, will happen constantly.

One more thing: this isn't actually "two checks" in the sense of extra work. It's one query with a compound condition — `EXISTS (visit today for this patient AND exam_fee_in_paise > 0)` — a single `WHERE` clause, not two round trips. Simplifying it doesn't buy you less complexity, only less correctness.

### Challenge 2 — you're right, and the docs don't agree with themselves on this

Worth being precise about what happened, because this wasn't a case of me inventing an answer where none existed.

business-rules-log.md §2 states the per-complaint version as settled — "first-time (350 exam once + 200/300 per complaint if therapy taken)" — and the Three Visit States table in `handover-fee-computation.md` operationalizes "first-time" as *"First-ever touchpoint for this complaint."* But §9 of the same log, and that same handover file's "Open Tension" section, both flag this exact scoping question as unresolved and explicitly waiting on you — because you'd separately floated a once-ever joining fee that contradicts it. §9 even calls the per-complaint version "an already-confirmed rule" in the same breath as saying it needs your direct confirmation. That's a real inconsistency sitting in the log already, not something I introduced — but you're right that the concrete design pieces (the states table, the trigger description) proceeded on the assumption that turned out to be wrong.

**Resolution, as you've now stated it:** the exam fee is scoped to the patient's lifetime, not the complaint. Charged once, ever. A returning patient with a brand-new complaint pays 0 exam fee — the doctor examines it for free — and only therapy for that complaint gets billed.

The useful way to hold this: *"this complaint's own first touchpoint"* and *"this visit charges the exam fee"* used to be the same question under the per-complaint assumption. They're now genuinely different questions that coincide exactly once — the patient's very first complaint ever. Every complaint after that has its own clinically-true first touchpoint without that triggering the exam fee.

Nothing else moves. Therapy is untouched — still per complaint, every encounter, regardless of exam-fee history, exactly as you describe.

### Challenge 3 — redesign

Everything below is **proposed** — none of it exists in `supabase_migration.md` yet (neither does the four-bucket money split it builds on). This is design work for the eventual migration, not a description of current schema.

**Reframing item 1 first.** "Once-per-patient-lifetime / once-per-complaint / once-per-visit-day" bundles two different questions into one list. The first two are about *recurrence* — at what unit does "already paid, don't recharge" get evaluated. The third is about *same-day sharing* — when multiple complaints get examined the same day, does one 350 cover all of them or does each pay its own. Those are orthogonal; a clinic could plausibly want any combination. Splitting them avoids exactly the kind of naming collision Challenge 2 was about — I'm calling them `recurrence` and `sharing` on purpose so they can't get conflated the way "examination fee" and "consultation fee" did.

That also resolves an ambivalence already sitting in the docs — `handover-fee-computation.md` floats both a single `consultation_fee_scope` column *and* a "cleaner" two-dimensional `clinic_fee_scopes` table without picking one. Given where this is heading, the table wins.

```sql
-- Axis A — exam-fee recurrence (Challenge 2's resolution)
-- Single column: doesn't vary by fee_type or need multiple rows per clinic
-- today, so a table isn't warranted yet by the "more than one dimension"
-- test business-rules-log.md §6 already applies to calls like this.
alter table clinics
  add column exam_fee_recurrence_scope text
    not null default 'PATIENT_LIFETIME'
    check (exam_fee_recurrence_scope in ('PATIENT_LIFETIME', 'PER_COMPLAINT'));

-- Axis B — same-day sharing. Renamed from the earlier consultation_fee_scope /
-- clinic_fee_scopes naming specifically so it can't collide with Axis A.
create table clinic_fee_sharing (
  id            uuid primary key default gen_random_uuid(),
  clinic_id     uuid not null references clinics(id),
  fee_type      text not null check (fee_type in ('EXAM', 'LAPSE_PENALTY')),
  sharing_scope text not null check (sharing_scope in ('PER_VISIT_DAY', 'PER_COMPLAINT')),
  unique (clinic_id, fee_type)
);
-- This clinic: (EXAM, PER_VISIT_DAY), (LAPSE_PENALTY, PER_VISIT_DAY) —
-- both already confirmed shared in business-rules-log.md §2.

-- Axis C — gap-triggered re-charging
create table clinic_gap_rules (
  id                   uuid primary key default gen_random_uuid(),
  clinic_id            uuid not null references clinics(id),
  gap_measured_against text not null check (gap_measured_against in ('COMPLAINT', 'PATIENT')),
  threshold_days       integer not null,
  outcome              text not null check (outcome in ('LAPSE_PENALTY', 'FULL_EXAM_RESET')),
  created_at           timestamptz not null default now(),
  unique (clinic_id, gap_measured_against, threshold_days)
);
-- This clinic's one confirmed row: (COMPLAINT, 10, LAPSE_PENALTY).
-- A FULL_EXAM_RESET row, or any PATIENT row, is hypothetical until the
-- open question below is answered.
```

**Evaluation, in one line:** for the relevant `gap_measured_against` basis, find the *largest* `threshold_days` the actual gap meets or exceeds, apply that row's outcome. Your own example maps directly: `(COMPLAINT, 10, LAPSE_PENALTY)` + `(COMPLAINT, 180, FULL_EXAM_RESET)` — a 200-day gap matches both, 180 wins; a 20-day gap only matches 10. Secondary thresholds (item 5) fall out of this for free — it's another row, not a special case.

**How this composes with Challenges 1 and 2:** `FULL_EXAM_RESET` isn't a new mechanism. When it fires, the visit gets treated exactly like a true first-time visit — writes into `exam_fee_in_paise`, runs through the same same-day dedup and sharing logic from Challenge 1. It's a one-event override of the `PATIENT_LIFETIME` default, not a competing system. No new money column, no change to the dedup trigger.

| Your axis | Captured by |
|---|---|
| 1. Scope of the flat fee | Split: `exam_fee_recurrence_scope` (lifetime vs. per-complaint) + `clinic_fee_sharing.sharing_scope` (same-day sharing) |
| 2. Gap outcome: penalty vs. full reset | `clinic_gap_rules.outcome` |
| 3. Gap threshold, configurable | `clinic_gap_rules.threshold_days` |
| 4. Gap behavior by complaint type | `clinic_gap_rules.gap_measured_against` (`COMPLAINT` vs `PATIENT`) |
| 5. Secondary threshold | Another row in `clinic_gap_rules` |

**Where I stopped, deliberately.** This doesn't cross gap-length, complaint-newness, and recurrence-scope in every combination. Your first item-4 example — a clinic that recharges only when a *new* complaint appears after a patient-wide gap — needs a rule conditioned on complaint-newness *and* gap length together, which is a different shape from a gap rule alone. I haven't built that. It's a plausible future config, nothing today needs it, and a rule engine general enough to cross every axis with every other axis is exactly the over-engineering you're asking me to avoid. If it's ever needed, the honest fix is a one-column addition (a nullable `requires_new_complaint boolean` on `clinic_gap_rules`), not a redesign.

**Your open question** — new complaint vs. same complaint after 10 days — is exactly what decides whether this clinic's confirmed row stays `gap_measured_against = 'COMPLAINT'` only, or needs a second `'PATIENT'`-scoped row alongside it. I'm reading the existing log wording ("a lapse inside an ongoing complaint") as leaning `COMPLAINT`-only, but that was written before this exact question was posed this precisely — so I'd call it a lean, not a confirmed answer. Genuinely open until the doctor says which.

Two small forward-flags, not decisions: if the 350 ever becomes tier-dependent (currently flat for both), a patient whose first complaint was `REGULAR` and whose later new complaint is `REHAB` would still pay 0 under `PATIENT_LIFETIME` scope — worth a conscious call if that divergence happens. And `consultation_type` only plans to add `GAP_RETURN` — a `FULL_EXAM_RESET` is billing-identical to `FIRST`, so recording it as `FIRST` is defensible without further widening, unless you later want to distinguish the two for reporting.

---

### Session Changelog

**1. Exam-fee recurrence scope**
- Old position (business-rules-log.md §2, §9): worked from a per-complaint assumption — every new complaint's first visit charged a fresh 350. §9 flagged this as an open tension; §2's "three states total, confirmed" language stated it as settled anyway.
- New position: exam fee is scoped to the patient's lifetime — charged once, ever. A new complaint on an already-charged patient pays 0 exam fee, therapy only.
- Needs updating: business-rules-log.md §2 (three-states description) and §9 (mark the joining-fee tension resolved); `handover-fee-computation.md`'s Three Visit States table (trigger column) and Open Tension section.

**2. Same-day dedup check**
- Old position: n/a — no rule change. The existing two-part check was already correct.
- New position: unchanged. Confirmed against a proposed simplification that breaks on any day mixing a follow-up or `MACHINE_ONLY` visit with a same-day first-time or gap-return visit.
- Needs updating: optional — `handover-fee-computation.md`'s dedup paragraph could spell out why the compound check can't be simplified, so this doesn't get re-raised later.

**3. Configurability redesign**
- Old position (business-rules-log.md §9): a single two-value `consultation_fee_scope`, or the "cleaner" two-dimensional `clinic_fee_scopes` table — neither floated as final. Neither captured lifetime recurrence, a configurable gap threshold, or multiple thresholds.
- New position (proposed — needs your sign-off before it's logged as settled): `exam_fee_recurrence_scope` column + renamed `clinic_fee_sharing` table + new `clinic_gap_rules` table.
- Needs updating: business-rules-log.md §9; `handover-fee-computation.md`'s "Consultation-Fee Scope Configurability" section.

**4. New open question — lapse penalty scope**
- Old position: not previously tracked as its own item. Closest text (§2) only disambiguated "mid-course lapse" from "gap since ever-first-visiting," not same-complaint vs. new-complaint.
- New position: explicitly open — does the 10-day lapse penalty apply when a new complaint appears after a patient-wide gap, or only when an existing complaint reactivates after its own gap? Determines whether `gap_measured_against` needs a `PATIENT` row.
- Needs updating: business-rules-log.md §9 — add as its own tracked open question.