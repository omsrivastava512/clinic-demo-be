# Om's Execution Plan v8 — Schema + DSA + Clinic Wiring Phase
### Starting Sept 15, 2026 — target ~8 weeks, expect drift

**Supersedes `day_by_day_plan_v7.md` (May 2026).** That plan assumed the app would be
fully shipped — Supabase, realtime, auth, tests, CI — by ~July 5, with outreach
starting Sept 11. Neither happened. Not because time was wasted — real business-rules
discovery with the clinic owner (`business-rules-log.md`, `specialty-decision-log.md`,
the MVP-scoping work in `mvp-index.md`) took priority, and that was the right call.
But as of today the frontend is still 100% mock data with zero live Supabase wiring —
confirmed directly against `FRONTEND_WORKFLOW.md`. This plan starts from that real
state instead of the stale one.

**Carried forward unchanged from v7:** the four Protocols below, the DSA tracker, and
the principle of one clear thing to do each day.

**What's different:** no job search, no outreach, no Next.js, no general backend
learning — none of it, at all — until the clinic app is deployed live and verified.
That's deliberate. It all resumes in a new phase, planned in a separate future
session, once Phase E below closes.

---

## Right now

**Current phase: A. Week 1 of ~8 (week of Sept 15).**

*(Update this line by hand at the start of each week — it's the one line you actually
need to read to know where you are.)*

---

## A note on certainty — the 🔸 marker

Several tasks further down depend on a design decision that's still just a proposal —
something drafted across various chats, never actually read and approved by you.
Tasks marked 🔸 are exactly that: safe to understand, not safe to build against yet.
`decision-queue.md` is the real list of what needs your sign-off before those specific
tasks become final — go there to resolve them, not here. Everything else in this file
(no marker) doesn't depend on anything unresolved and is fine to just execute.

---

## What is explicitly OUT OF SCOPE right now

Not because these don't matter — because adding them now is parallel workload on an
already full plate. Each one resumes later, in a phase this file doesn't cover yet:

- Job applications, cold outreach, portal applications, LinkedIn, resume work
- Next.js
- General backend/Node/Express learning
- GFE machine-coding drills
- Anything already on `mvp-index.md`'s "Defer entirely" list (full packages module,
  specialty treatments, the session/encounter architecture, retail, split-payment,
  Pay Later, fee-scope configurability) — these were deferred on their own merits
  already, independent of this time-allocation question. Don't reopen them.

---

## The Four Protocols (unchanged from v7)

### Protocol 1: Bad Days
| What happened | Do this |
|---|---|
| Full day lost | Skip it. Do tomorrow's work tomorrow. No doubling up. |
| Low energy / partial day | Do ONLY the one thing most behind (DSA or clinic/module work). Even 1 hour counts. |
| Multiple days lost in a week | Saturday becomes catch-up. Do whatever's most behind. |
| Full week lost | Accept the phase slips a week. Don't compress. |

### Protocol 2: Overslept
| Woke up at... | Do this |
|---|---|
| Before 11 AM | Normal day. |
| 11 AM – 12 PM | Skip the morning focus block. Go straight to the main block. Do 1 DSA problem after dinner. |
| After 12 PM | Write off the morning. Main block only, afternoon. |

### Protocol 3: Study vs Practice (DSA)
| Video length | What to do |
|---|---|
| < 30 min | Watch + solve 1–2 problems, same session. |
| 30–60 min | Watch only. Practice next DSA day. |
| > 60 min | Split video across 2 sessions. Practice on the 3rd. |

### Protocol 4: Feature Expansion
When a task spawns unexpected sub-tasks mid-build:
1. Don't panic — normal when learning while building.
2. Add discovered sub-tasks as indented checkboxes under the parent task.
3. Each completed sub-task is visible progress, even if the parent isn't done.
4. Do NOT promote sub-tasks to new top-level items — they stay nested.

---

## Daily rhythm (applies in every phase)

| Block | What | Notes |
|---|---|---|
| First thing, ~1 hr | DSA | Protected, non-negotiable. Pull from the tracker below, wherever the spaced-repetition log says you left off. |
| Main block, ~4–4.5 hrs | Module study OR clinic dev | Which one, and which day, is set per-phase below. |

**Sunday:** DSA optional/light review only. No module study, no clinic-dev pressure —
buffer day, or genuine rest if nothing's behind.

---

## Priority Stack (current phase only)

1. **DSA** — protected daily block, no exceptions, this is what almost got dropped for two months last time.
2. **Module study** — Module 2 first (already half-done, don't let it go stale again), then Modules 4 and 5/6.
3. **Clinic dev** — bug fixes → inter-screen wiring → structural fixes → real Supabase wiring → deploy.

Nothing else competes for time until Phase E closes.

---

## Phase A — Module 2 completion + Frontend P0 bug sweep

**Target: ~2 weeks (Sept 15 – Sept 28).**
**Exit criteria:** Module 2 fully done — you can explain the `patient_clinic_access`
junction, the intake-deadlock fix (Iterations 6–7), and the cross-chain tenancy check
without looking anything up. All 7 P0 bugs below fixed on the existing mock-data
frontend.

**Day assignment:** Mon / Wed / Fri main block → Module 2 (`schema-study-plan-v2.md`,
resume where you paused mid-August, roughly half-done). Tue / Thu / Sat main block →
P0 bug fixes, in this order (source: `workflow_screens_audit.md`'s Prioritized Action
Plan, Phase 1):

- [ ] Ledger: stop `.sort()` mutating the array in place during render (`TableBody.tsx:78`)
- [ ] Ledger: add `data-selected={isSelected}` so keyboard-nav CSS actually fires (`primitives.tsx:46-52`)
- [ ] Intake: fix empty-age validation silently passing as age 0 (`index.tsx:18`)
- [ ] Invoice: fix invalid `<div>` nested directly inside `<table>` (`ItemTable.tsx:10-13`)
- [ ] Invoice: remove the conflicting `hover:disabled` CSS (disabled buttons currently turn green on hover) (`FooterActions.tsx:21-22`)
- [ ] Complaint Selector: add `e.stopPropagation()` on delete so it stops re-adding the deleted item (`ComplaintItem.tsx:43-51`)
- [ ] Complaint Selector: fix free-text entry — `Enter` currently does nothing when there's no catalog match (`index.tsx:139-168`)

---

## Phase B — Inter-screen wiring (still mock data, no backend)

**Target: ~1 week.**
**Exit criteria:** the 5 screens (Ledger, Intake, Complaint Selector, Procedure
Logger, Invoice) work as one connected flow instead of 5 isolated podium components.
Still zero Supabase calls.

**Day assignment:** this phase is short enough to run as the main block every
weekday. Module 4 (functions & triggers — what `plpgsql` and `BEFORE INSERT`/`NEW`
actually mean) runs in parallel on Mon/Wed/Fri, same slot Module 2 used.

Tasks (source: `workflow_screens_audit.md`, "Inter-Screen Integration Readiness"):
- [ ] Build a parent stepper/orchestrator — `vistitworkflow.tsx` currently renders all
      5 screens simultaneously as a vertical catalog; needs to become an actual flow
      tracking which step is active and what's been collected so far
- [ ] Ledger → Intake/Complaints: add `onSelectPatient(patient)` and
      `onRegisterNewPatient(query)` callbacks (currently zero props accepted)
- [ ] Intake → Complaints: return the created `Patient` object with a real `id`
- [ ] Complaints → Procedures: emit full `MedicalComplaint[]` objects, not bare
      `string[]` ids — Procedure Logger needs title/id/metadata, not just an id
- [ ] Procedures → Invoice: pipe the items array directly into `InvoiceBuilder` as
      props instead of two components rendering independent mock data
- [ ] Invoice → Ledger: persist payment, then return/refresh to the ledger instead of
      a raw `alert("Visit Closed")`

---

## Phase C — Structural frontend fixes (still no backend)

**Target: ~1.5–2 weeks.**
**Exit criteria:** the frontend's data shape actually matches what the schema
expects — no more age-instead-of-DOB, no more per-complaint "Consultation" checkbox,
no more legacy procedure catalog IDs. Module 5 (RLS) study runs here since it directly
sets up Phase D.

**Day assignment:** Mon/Wed/Fri main block → Module 5, then Module 6 folded in right
after (SECURITY DEFINER is really an extension of the same RLS territory, not a
separate topic). Tue/Thu/Sat main block → structural fixes, in this order:

- [ ] Replace the `age` field with a DOB picker + derived age display (`patients.date_of_birth` is the real column; age-as-string can't produce a valid insert)
- [ ] Decide + build MRN generation — sequential/trigger-based (mirroring `clinics.invoice_counter`) vs. staying free text; this needs a quick decision first, not just a build
- [ ] Extract "Consultation" out of the per-complaint procedure grid into a session-level toggle — currently two complaints checked produces two separate consultation charges, which is backwards from the confirmed once-per-session exam fee rule (Decision 6, `FRONTEND_WORKFLOW.md`)
- [ ] Swap the legacy `PHYSIO_PROCEDURES` catalog (`PROC_01`–`PROC_08`) for the real `services` table ids (`SVC-01`–`SVC-13`) — the legacy ids will fail FK inserts
- [ ] Implement `is_charged` / `charged_amount_in_paise` logic on `visit_services` instead of billing every checked procedure unconditionally
- [ ] Remove the pre-generated invoice number; show "Auto-generated on confirmation" until the real mutation returns it (the trigger mints it, the client must not)
- [ ] Thread `paymentMode` through `FooterActions` to the actual submit handler (currently selected but silently dropped)
- [ ] Preserve `complaint_catalog_id` on catalog selection instead of dropping it
- [ ] Sequence the clinical-notes write to fire only after the `patients` INSERT returns a real `id`
- [ ] Polish pass while these files are already open: replace `alert()` calls with inline errors, drop the `text-[10px]`/ALL-CAPS/font-mono overuse — this matters more once Phase E's demo has to survive a real engineer looking at it

---

## Phase D — Real Supabase wiring

**Target: ~2 weeks.**
**Exit criteria:** patient → complaint → visit → correct fee → invoice actually works
against a live Supabase project, seeded with realistic data, with real RLS in place
instead of `v1_allow_all`.

**🔸 Proposed reconciliation on the RPC design — not your decision yet, see
`decision-queue.md` item 1.** Two competing full designs exist in raw, unintegrated
chat transcripts, neither ever reviewed by you: one drafted a `sessions` table and one
atomic RPC covering session through invoice; the other drafted two separate RPCs
(a session-commit, a later invoice-commit), justified by an earlier log entry
decoupling the session boundary from the invoice boundary. Both assume a persisted
`sessions` table exists. `mvp-index.md` defers the entire session concept and
`treatment_events` entirely, which is what makes both of those premises questionable
at current MVP scope — but that's my read, not something you've confirmed. The
proposed alternative: one RPC, no `sessions` table, no `session_id`, no
`treatment_events` — committing `visits` + `visit_services` + `invoices` +
`invoice_line_items` together at "Create Invoice," with `invoice_line_items`
simplified to `invoice_id` + a single non-nullable `visit_id` + `amount_in_paise`.
Full reasoning, and the drafted `specialty-decision-log.md` entry to paste in *if you
approve this*, are in `decision-queue.md` item 1.

**🔸 Genuinely unresolved question, needs your direct word — see `decision-queue.md`
item 2.** Whether the exam fee is scoped per-complaint (a new complaint always gets a
fresh 350) or per-patient-lifetime (only the patient's very first complaint ever pays
it). Multiple documents disagree with each other on this and none of it has been
reviewed by you — full contradiction detail in the queue. Everything else in this
phase can proceed in parallel, but the exam-fee trigger specifically can't be
finalized until you answer this.

Tasks:
- [ ] Replace `v1_allow_all` with the real staff/admin RLS policies already drafted
      (commented out) in `supabase_migration.md`
- [ ] Add `created_by` derivation from `auth.uid()` via trigger, bundled with the RLS
      rollout (per `mvp-index.md`'s "Build now" list)
- [ ] Build the four-bucket money columns on `visits` (`exam_fee_in_paise`,
      `lapse_penalty_in_paise`, `therapy_fee_in_paise`, `services_total_in_paise`)
      with the same-day dedup trigger scoped to `patient_id` (not `complaint_course_id`)
- [ ] 🔸 Build the RPC per whatever `decision-queue.md` item 1 lands on — proposed
      shape (unreviewed): one RPC covering `visits` + `visit_services` + `invoices` +
      `invoice_line_items` (minimal, visit-sourced only), with the ghost-invoice
      sum-matching trigger, fired at "Create Invoice"
- [ ] Inside that RPC, validate `complaint_courses.status = 'Active'` before allowing
      a new visit against it — cheap, closes a real adjacent gap found while reading
      the Module 2 backlog. This does NOT by itself close the fuller "was this
      complaint actually part of today's legitimate selection" gap — that one stays
      deferred along with the session concept, on purpose
- [ ] Apply the role-routing model while wiring auth (from the same Module 2 backlog
      read, unreviewed but low-stakes): one `profiles` read right after login
      (`role` + `clinic_id`) drives every UI decision, since `profiles.id` is
      `auth.users.id` — no separate lookup needed. Receptionist: auto-scope to their
      own `clinic_id`, no picker shown. Admin: chain-wide by default, an optional
      clinic-switcher is pure display convenience, never a security boundary.
      Clinician: treat as receptionist-equivalent for now — whether clinicians get a
      distinct login experience at all is genuinely uncertain, not confirmed — with
      the one difference being `clinician_id`/`created_by` auto-filling from their
      own `auth.uid()` when they're the one submitting a form
- [ ] Add `visits.package_id` + minimal package-visit linking (confirmed in-scope per
      `mvp-index.md` — packages are actually being sold today, tied to rehab-tier
      patients)
- [ ] Add `clinic_tier_rates`, clinic-scoped, populated for the main clinic only —
      the two machine-only branches never touch this table
- [ ] Add the per-patient effective-rate fields directly on `patients` (not the
      exceptions-only `patient_rate_overrides` design — that one was explicitly
      replaced per `mvp-index.md`'s reversal history)
- [ ] Add the one-off transactional override field + reason + trigger-derived
      attribution on `visits`
- [ ] 🔸 Add `patient_flags` (separate table, per `business-rules-log.md` §1) — the
      table shape itself is solid, but whether flagging is admin-only or open to any
      staff member is still genuinely open. See `decision-queue.md` item 3
- [ ] Wire all 5 screens' actual reads/writes to the live project

---

## Phase E — Deploy + verify

**Target: ~3–5 days.**
**Exit criteria:** a real, public URL. This is the "v1 is live" milestone — the one
that actually discharges the responsibility to the client and gives you something
real to eventually show an engineer. Concretely, per `mvp-scope-diagnosis.md`'s own
Definition of MVP DONE, every one of these needs to produce the exact right number on
a real invoice: a plain follow-up visit, a true first-ever visit (whichever exam-fee
scope `decision-queue.md` item 2 lands on), two complaints treated the same day
(₹350 + 200 + 200 = ₹750 under per-complaint sharing, as currently confirmed in
`business-rules-log.md` §2), and a gap-return visit.

- [ ] Deploy publicly (Vercel/Netlify + the live Supabase project)
- [ ] Run the real Verification Checklist from `backend-schema-iterations.md` against
      the live deployed app, not the trigger code on paper: happy path, cross-branch
      isolation, cross-chain rejection, the `owner_id` role guard, admin visibility
- [ ] Fix whatever actually breaks
- [ ] Tell the client it's live

---

## After Phase E

Next.js, backend learning, and job search/outreach all resume here — planned in a
new session once Phase E is actually closed, not before. Not detailed in this file.

---

## DSA — pointer, not a duplicate tracker

The 36-item list that used to sit here was stale — it never had Trees, BST, Heap,
Backtracking, Greedy, or DP as categories, and undercounted real volume by roughly
3x. DSA content now lives in three dedicated files, same pattern module study already
uses (`schema-study-plan-v2.md` gets pointed to, never copied in):

- `dsa-scope-update-handoff.md` — why the old tracker was replaced, the real
  two-curriculum scope (Striver A2Z + a second course), and the pre-ranked plan for
  everything held until job-search phase.
- `dsa_study_plan.md` — Phase 1 (the old tracker's real remaining items) and Phase 2
  (Trees, BST, Heap), day-by-day.
- `dsa_phase3_plan.md` — Phase 3, detailed. **One open dispute here** — its "Part 1,
  NDSA Extra Folders" reopens at least one thing the handoff called permanently out
  of scope. See `decision-queue.md` before treating Part 1 as approved.

**Current DSA position:** *(update this line by hand — pull the actual next problem
from `dsa_study_plan.md`'s own sequence table)*

The protected block itself doesn't change: still first thing, ~1 hour,
non-negotiable, per the Daily rhythm table above. Just pull *what* to do from these
files instead of the list that used to be here.

---

## End