# Study Plan: Understanding the Clinic Schema (v2)

Scope rule: every module below is anchored to something that actually exists in your migration file — nothing generic or theoretical that you won't use.

**What's in scope now vs. later:** the mechanics of *how Supabase auth reaches your database* (the `auth.uid()` module) are in scope, because they explain why your RLS policies are written the way they are. The mechanics of *how your React app calls Supabase* (the JS client library, catching errors in components, auth flows in React) stay out of scope — those get learned when you get there. Module 10 gives you just enough advance awareness of that handoff to not be blindsided by it later, without teaching the code for it yet.

Format: one module per sitting. Plain explanation, anchored to your real tables, a hands-on exercise you actually run in the Supabase SQL Editor, ending with one light comprehension check — not a drill, just "does this actually make sense to you." No module starts until the previous one does.

---

- [ ] **Module 1 — Tables, Columns & Constraints**
  Data types, `NOT NULL`, `UNIQUE` (including composite uniques like `unique(owner_id, mrn)`), `CHECK` constraints, `DEFAULT` values, and the "store money as an integer in paise" pattern.
  Anchor: the `patients` table, read top to bottom.

  **Do this:** In the SQL Editor, try inserting a patient with `referral_mode = 'DOCTOR'` but no `referral_doctor_info`. Read the exact error Postgres gives you — that's `chk_referral_doctor_info` doing its job. Then fix it and insert successfully. Then try inserting a second patient with the same `owner_id` and `mrn` as an existing one, and watch the unique-constraint violation.

- [ ] **Module 2 — Relationships Between Tables**
  Foreign keys / `references`, `on delete cascade`, and the difference between a "reference/seed" table (`services`, `complaint_catalog` — text IDs, rarely changes) and a "transactional" table (`visits`, `invoices` — uuid IDs, grows constantly).
  Anchor: how `visits` connects to `patients`, `complaint_courses`, and `clinics`.

  **Do this:** Try inserting a `visits` row for a patient/clinic pair that has no matching `patient_clinic_access` row yet. Watch `validate_visit_clinic_id` reject it. Then insert a `complaint_courses` row for that same patient/clinic first, check that a `patient_clinic_access` row appeared automatically, and confirm the same visit insert now succeeds.

- [ ] **Module 3 — Computed Data: Generated Columns & Views**
  `generated always as ... stored` columns, and what a `view` is versus a real table.
  Anchor: `visits.grand_total_in_paise`, the `daily_ledger` view.

  **Do this:** Insert a visit with `consultation_fee_in_paise` and `services_total_in_paise` set, and confirm `grand_total_in_paise` computed itself. Then try to `UPDATE` `grand_total_in_paise` directly and see Postgres refuse — generated columns can't be written to, only derived. Then run `select * from daily_ledger` and compare it to manually joining `visits` and `patients` yourself — same data, proving the view is just a saved query, not a second copy of the data.

- [ ] **Module 4 — Functions & Triggers**
  What a `plpgsql` function is, what `before insert or update` triggers do, what `NEW` and `OLD` mean inside one.
  Anchor: `trigger_set_timestamp()` (the simplest one), then `derive_owner_id_from_patient()`.

  **Do this:** Update any `patients` row and confirm `updated_at` changed on its own. Then insert a `patient_alerts` row and deliberately pass a wrong `owner_id` in your insert statement — check afterward that the row's actual `owner_id` matches the patient's real owner, not what you typed. That's the trigger silently overwriting your input, which is the whole point.

- [ ] **Module 5 — Row Level Security**
  Why Supabase needs RLS at all (every table is auto-exposed via API unless restricted), `using` vs `with check`, `auth.uid()`.
  Anchor: the staff and admin policies on `patients` (including the commented-out "real" policies at the bottom of the SQL file).

  **What `auth.uid()` actually is:** when someone logs in through Supabase Auth, they get issued a JWT — a signed token containing their user id. Every request your frontend sends to the database carries that token along with it. Postgres reads the token *for that specific request* and makes the id available inside the query; `auth.uid()` is just a small function that reads it back out. Nothing is cached or stored as "the current session" — it's re-derived fresh, request by request.

  This is also why, if you run `select auth.uid();` yourself in the Supabase SQL Editor, you'll get `null` — the Editor connects as a superuser, not as a logged-in app user, so there's no request JWT for it to read.

  The bigger reason this matters: Supabase doesn't put a custom backend server between your frontend and the database. Postgres is reached directly. That means RLS isn't one layer of defense among several — it's the *only* one. Right now every table in your schema runs on a placeholder `v1_allow_all` policy (any authenticated user can read or write anything). That's a fine, deliberate state for development, but it's a real, currently-open gap, not a hypothetical one — the "real" policies commented out in the SQL are what closes it.

  **Do this:** Read the commented-out staff and admin policies at the bottom of the SQL file line by line. For a specific hypothetical staff profile (pick a `clinic_id`), hand-trace which rows of `patients` that policy would let them see, using the real `patient_clinic_access` logic.

- [ ] **Module 6 — SECURITY DEFINER & Self-Enforcing Triggers**
  Why some functions need to bypass RLS, and why that means they have to check security themselves instead of relying on it.
  Anchor: `process_new_complaint_course()`, `validate_owner_is_admin()`.

  **Do this:** Manually insert a `complaint_courses` row using a `patient_id` and `clinic_id` that belong to two *different* owners. Read the exact "Cross-chain violation" exception text that comes back. That message is the trigger being its own security boundary, because `SECURITY DEFINER` means RLS isn't watching this write at all.

- [ ] **Module 7 — The Whole Path, End to End**
  Synthesis: a receptionist logs in → `auth.uid()` → which RLS policy fires → which rows come back. Ties Modules 1–6 together into one mental model.
  Anchor: the "Current State at a Glance" diagram in the corrected iterations doc.

---

- [ ] **Module 8a — Business-Logic Stress Test: Does the Code Actually Work**
  Not a concept module — a mechanical verification module. Run the actual Verification Checklist from the bottom of the iterations doc against a real (dev/test) Supabase project: happy path, cross-branch isolation, cross-chain rejection, `owner_id` role guard, admin visibility. Check each box only after watching it happen, not after re-reading the trigger code.

- [ ] **Module 8b — Business-Logic Stress Test: Is the Code Solving the Right Problem**
  A separate kind of thinking from 8a — now that 8a has confirmed the schema does what it claims, ask whether what it claims is what the clinic actually needs. Walk a real day at the clinic against the schema and resolve the open questions below.

  ## Open Business Questions to Resolve
  - [ ] Are referring doctors worth their own table, or does free text stay correct forever?
  - [ ] Does "a visit row existing = session complete and paid" really match how your front desk works, or are there real in-between states?
  - [ ] Is the complaint/procedure catalog wired the way you actually want it used day to day?
  - [ ] Re-confirm the cross-branch patient identity model (one shared identity per chain) now that you can actually evaluate what that decision means, not just take it on trust.

---

- [ ] **Module 9 — Safe Schema Change Practice (Migrations)**
  The habit this module exists to break: pasting an `ALTER TABLE` straight into the Supabase dashboard whenever something needs to change. That's how you end up back where you started — a growing pile of hand-run SQL with no record of what changed, why, or whether it's safe to run again on a fresh project.

  **The actual workflow, conceptually:**
  1. Every schema change is its own small, timestamped migration file (the Supabase CLI's `supabase migration new <name>` creates one for you, e.g. `20260722_add_appointments.sql`) — not a one-off paste into the dashboard.
  2. You run it against a local or staging Supabase project first, never production first.
  3. Once verified, it gets applied to production — and the file stays in your repo as the permanent record of that change, so the *file* is the source of truth, not whatever's currently sitting in the dashboard.
  4. This is, not coincidentally, exactly what your iterations document already does by hand — it's a disciplined migration log. You already have the instinct; this just gives it a versioned file per change instead of one long document.

  **Safe vs. unsafe changes, roughly in order of how much care they need:**
  - **Safe, low-risk:** adding a nullable column, adding a brand-new table, adding an index, adding a `CHECK` constraint that all existing rows already satisfy.
  - **Needs a plan:** adding `NOT NULL` to a column on a table that already has rows (existing rows need a backfill value first), renaming a column or table (breaks anything still referencing the old name), changing a column's data type.
  - **Destructive, treat as one-way:** dropping a column, dropping a table, `truncate`. Anything here should happen against a project you have a backup or export of, not live.

  **Your appointments example:** yes — that's exactly the kind of change that becomes "v12" in your history, same discipline as v1–v11 here. New migration file, one paragraph on why, `CHECK` constraints and foreign keys designed with the same shape-first instinct as the rest of the schema, tested in a scratch project before it touches real bookings.

  One clarification on "shape vs. security," since it wasn't about features skipping this process: it's the severity-tag system already used throughout your iterations doc. 🔴 Structural changes (table shape, relationships) are expensive to reverse once real data exists — get these right the first time. 🟡 Safety-net changes (a validation rule, a trigger) are cheap to add or fix later. A new feature like appointments will have *both* kinds of changes inside it — the tag tells you which parts deserve the most upfront care, not whether the whole thing needs a migration. It always does.

- [ ] **Module 10 — How This Eventually Meets Your Frontend (Concept Only)**
  Not implementation — just enough awareness so it doesn't surprise you later.

  When a trigger's `raise exception` fires (a cross-chain violation, a missing access record, a failed `CHECK`), that's Postgres deliberately refusing a write, and it comes back through Supabase's API as a structured error — a code, a message, sometimes a hint — not a network failure or a crash. Eventually your frontend will need to catch that and turn it into something a receptionist can actually read ("this patient isn't registered at this clinic yet") instead of a raw database error or a silent failure.

  The actual code for catching this (`try/catch` around a Supabase client call, reading `error.code` / `error.message`) is exactly the kind of frontend-integration detail this plan intentionally defers — you'll pick it up naturally once you're wiring up real forms against real tables. What matters now is just knowing the handoff exists: the database is allowed to say no, and something on the other end has to listen for that.
