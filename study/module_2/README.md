# [6 Aug 2026 - Module 2 conceptual pass - I]

## Foreign Keys: what `references` actually enforces

A foreign key isn't a comment or a naming convention — it's a constraint Postgres checks on every write. Take this line from `patients`:

```sql
owner_id uuid not null references profiles(id)
```

This means: Postgres will physically refuse to insert or update a `patients` row if `owner_id` doesn't match an existing `id` in `profiles`. Not "should match" — *can't not match*. Try inserting a patient with a random UUID as `owner_id` and you'll get a foreign key violation before your row ever touches the table.

This is the same mechanism behind almost every relationship in your schema: `patients.clinician_id → profiles(id)`, `visits.complaint_course_id → complaint_courses(id)`, `invoices.visit_id → visits(id)`, and so on. Once you've seen one, you've seen the pattern for all of them.

## `on delete cascade`: what happens on the other side

A foreign key by itself only controls inserts/updates. It says nothing about what happens if the row being *pointed to* gets deleted. That's what `on delete cascade` controls.

Look at `patient_alerts`:
```sql
patient_id uuid not null references patients(id) on delete cascade
```

If you `DELETE FROM patients WHERE id = '...'`, Postgres automatically deletes every `patient_alerts` row pointing at that patient too. Same for `patient_vitals`, `clinical_notes`, `complaint_courses`, `visits`, `packages`, `invoices`, `timeline_events`, `patient_clinic_access` — all cascade off `patient_id`.

**Without** `on delete cascade`, the default is `RESTRICT`: Postgres blocks the delete entirely if any child row still references it. You'd get an error, not a silent no-op.

Now notice what does *not* cascade: `patients.owner_id → profiles(id)` has no cascade clause. That's deliberate — you'd never want deleting an admin's profile to silently wipe out every patient in the chain. If someone tries to delete a profile that patients still reference, Postgres blocks it. Same logic on `visits.clinic_id → clinics(id)` and `visits.complaint_course_id → complaint_courses(id)` — no cascade, so a clinic or complaint course with billing history attached can't just vanish along with its financial trail.

So the pattern across your schema is: **cascade on patient-ownership chains** (delete a patient, their whole record disappears cleanly), **restrict on organizational/reference chains** (deleting a clinic, profile, or complaint course while records still point to it is blocked, forcing a deliberate decision instead of an accidental wipe).

## Reference/seed tables vs. transactional tables

Two different shapes in your schema, and the difference is intentional:

**Reference/seed tables** — `services`, `complaint_catalog`. Text primary keys (`"SVC-01"`, `"CAT_S01"`). Seeded once, rarely touched again, small row count. The ID itself is human-meaningful and stable — you write `SVC-01` into a migration once and never generate a new one at runtime.

**Transactional tables** — `patients`, `visits`, `invoices`, `complaint_courses`, etc. UUID primary keys, generated fresh on every insert (`gen_random_uuid()`), growing constantly, potentially from multiple concurrent clients (two receptionists at two branches both creating visits at the same second). UUIDs exist here specifically because a human-readable sequential ID would need a shared counter — exactly the collision problem `invoice_number` solves separately with a real per-clinic trigger-driven counter, because *that* one actually needs to be human-readable AND sequential.

## Anchor: how `visits` connects to three tables at once

```sql
create table visits (
  id uuid primary key default gen_random_uuid(),
  clinic_id  uuid not null references clinics(id),
  patient_id uuid not null references patients(id) on delete cascade,
  complaint_course_id uuid not null references complaint_courses(id),
  ...
```

Three separate foreign keys, three separate relationships:
- **`patient_id → patients`** — cascades. Delete the patient, the visit disappears with them.
- **`complaint_course_id → complaint_courses`** — no cascade. A visit can't exist without its complaint course, but you can't delete a complaint course out from under a visit that already happened.
- **`clinic_id → clinics`** — no cascade, and also not *trusted* — this is the one your migration doesn't just declare as a bare FK. It's validated by `validate_visit_clinic_id()` against `patient_clinic_access` before the insert is allowed to complete. The FK constraint says "this clinic must exist somewhere." The trigger says "this patient must actually be allowed at *this* clinic." Two different jobs, both needed.

That's the exact mechanic your Module 2 exercise (already sitting in the handover) walks you through hands-on tomorrow: insert a `visits` row for a branch the patient has no access to → watch it get rejected → insert a `complaint_courses` row first → watch `patient_clinic_access` auto-populate → retry the visit → watch it succeed. Reading this tonight, running it tomorrow, should click into place fast since you'll be watching the FK layer and the trigger layer act as two separate checks in real time.

# [6 Aug 2026 - Module 2 conceptual pass - I - Follow up]

## 1. Does `owner_id` get set by a trigger, since it's not in your registration form?

No — and this is a real gap, not something I glossed over. Go check your handover notes: *"No trigger derives `patients.owner_id` automatically. Frontend fully responsible for submitting the correct value."*

Here's why this is different from everything else in the schema that uses `owner_id`. Tables like `patient_alerts`, `clinical_notes`, `timeline_events` all have a trigger (`derive_owner_id_from_patient()`) that looks at the row's `patient_id`, goes and finds that patient's `owner_id`, and silently writes it in — the client's input is thrown away entirely. That works because those tables have an obvious parent to copy from: the patient already exists, already has an `owner_id`, done.

But `patients` itself is the *root* of that chain. There's no parent row to derive it from — a new patient isn't "attached to" anything yet. So whoever's building the INSERT (your frontend code) has to supply `owner_id` directly. Right now the only safety net is `validate_owner_is_admin()`, which checks that whatever value you send actually belongs to *some* admin — not that it's the *correct* admin for whoever's doing the registering. That's the exact gap flagged in your notes.

Practically, what would need to happen: your receptionist is logged in, their `profiles.clinic_id` points at one specific clinic, that clinic has a `clinics.owner_id`. Your frontend would need to look that up and populate `owner_id` on the patient insert from it — but that's application logic you'd have to write, not something the database does for you. Worth sitting with, because it's the kind of thing that's invisible until it's wrong.

## 2. Is there a role check on `clinician_id`, not just an existence check?

First, roles: `receptionist`, `clinician`, `admin` — that's `profiles.role check (role in ('receptionist', 'clinician', 'admin'))`, defaulting to `'receptionist'`. Not `front_desk` — that was the *original* Iteration 1 proposal, which got scrapped before it was ever built (the note in the iterations doc calls this out explicitly — a different agent took the project a different direction). So don't worry, you weren't misremembering a real thing.

Now your actual question — this is the same shape of problem as question 1, and it's the exact concept the second message calls "FKs enforce existence, not correctness," just applied to a column that message didn't cover. `patients.clinician_id uuid references profiles(id)` checks *only* that the id exists in `profiles`. Nothing checks that the profile it points to actually has `role = 'clinician'`. You could set `clinician_id` to a receptionist's id or even another admin's id, and Postgres would accept it without complaint. There's no `validate_owner_is_admin()`-style trigger for this column anywhere in the schema — I checked. It's undocumented, same category of gap as the `owner_id` one, just never flagged in any iteration. Worth adding to your running gaps list if it isn't already.

## 3. Soft delete — and what actually is `patient_alerts`?

**Soft delete first:** yes, it's already there, you're just not clocking it as "the soft delete mechanism" — it's `patients.is_active boolean not null default true`. That's the real-world answer to "do we hard-delete patients." No, you flip `is_active` to false, the row stays forever, nothing cascades, nothing is lost. `ON DELETE CASCADE` almost certainly never fires in normal day-to-day operation of this app — it exists as a safety net for the rare, deliberate, administrative case (cleaning out test data, an actual GDPR-style erasure request), not as the everyday path.

**`patient_alerts`:** fair to not remember this one, it's easy to conflate with `clinical_notes` since they're both patient-safety-adjacent. They're different things:

- `clinical_notes` — free-form. `category` and `observation` are arbitrary text, plus an `is_critical` flag. This is the diagnostic notes feed — "Category: Diabetes, Observation: Type 2, HbA1c 7.4."
- `patient_alerts` — fixed categories only: `type check (type in ('ALLERGY', 'FALL_RISK', 'DNR', 'OTHER'))`, plus a short `label` string. This looks like it's meant to be a small set of hard safety flags rendered as a badge or banner right at the top of a patient's profile — "Penicillin Allergy," "Fall Risk" — the stuff a clinician needs to see in half a second before they even open the notes feed, not buried in a scrollable log.

I'm inferring the UI intent from the shape of the table and the source comment (`Source: PatientAlert type, types/index.tsx L71-74`) — I don't have that frontend file in front of me, so take the "badge" framing as an educated read, not a confirmed fact. But structurally, it's real and it's been in the schema since Iteration 1 — not something bolted on later.

## 4. Why not `ON DELETE CASCADE` everywhere — what does `RESTRICT` even buy you if you'd "never" delete an owner anyway?

This is a good instinct to push on. Two things:

**Why RESTRICT isn't a no-op.** You said "I cannot realistically delete an owner, so what benefit is that even giving me" — flip it around: the benefit isn't for the deletes you *intend*, it's for the one you don't. Someone fat-fingers a `DELETE FROM profiles WHERE id = '...'` in the Supabase dashboard, or a bad migration script runs against the wrong environment, or an admin panel has a bug. Without `RESTRICT`, if that FK had cascaded, you'd silently lose every patient, visit, and invoice tied to that owner with zero warning. With `RESTRICT` (the default, which is what you get when you write a plain `references profiles(id)` with no `on delete` clause), Postgres just refuses and throws an error. It's a brake that costs you nothing in normal operation and saves you exactly once, catastrophically, the one time you need it.

**Why cascade isn't applied everywhere.** The pattern in your schema is deliberate, not inconsistent: `ON DELETE CASCADE` only sits on the true patient-*ownership* chain — `patient_alerts`, `patient_vitals`, `clinical_notes`, `complaint_courses`, `visits`, `packages`, `invoices`, `timeline_events`, `patient_clinic_access` all cascade off `patient_id`, because none of those rows mean anything without the patient they belong to. Everything else — `profiles`, `clinics`, `complaint_courses` (as a *parent*, e.g. `visits.complaint_course_id`) — stays `RESTRICT`, because deleting those while records still point at them is either a mistake or a decision that needs to be made deliberately, not something that should ripple through silently. Adding `ON DELETE CASCADE` to *every* FK "for completeness" would actually be dangerous here — it would mean deleting one admin profile could silently wipe out an entire clinic chain's financial and medical history in one statement. The selectivity is the safety feature, not a gap.

## 5. Are `services` and `complaint_catalog` already populated, or just shaped?

Just shaped. Look at the migration SQL for both tables — they're `CREATE TABLE` statements only, no `INSERT` statements anywhere near them. Running the full migration gives you two empty tables with the right columns and constraints, nothing else. The actual rows (`SVC-01` through `SVC-13`, `CAT_S01`, etc.) would need separate seed inserts — most likely transcribed from your frontend's mock data files (`MOCK_SERVICES`, `complaints_catalog.ts`, mentioned in your codebase context doc), but that seeding step isn't in any of the three docs I have in front of me. Worth checking directly in your dev project tomorrow — `select * from services;` — to see if you've actually seeded these yet or if they're sitting empty.

## 6. What problem does `invoice_number` actually solve? Deep dive on the trigger.

Go back to Iteration 2's original complaint: the v1 schema used manual text IDs like `"INV-001"`, with no atomic counter. Concretely, here's the failure mode that creates: two receptionists at the same clinic both create an invoice within the same second. If invoice numbering is done client-side ("look at the last invoice, add 1, save"), both requests could read "last invoice was 041" *before either one writes back* — and now you have two invoices both claiming to be `INV-042`. For something that's supposed to be a sequential, legally-referenceable audit trail, that's a real integrity failure, not a cosmetic bug.

The fix has two layers:

**Layer 1 — the actual primary key stays a UUID.** `invoices.id uuid default gen_random_uuid()`. This is never human-facing, never sequential, and by definition can't collide. This is the "real" identity of the row.

**Layer 2 — a separate human-readable `invoice_number`, generated atomically inside the database, not the app.** Here's the trigger:

```sql
update clinics
set invoice_counter = invoice_counter + 1
where id = new.clinic_id
returning invoice_counter into next_num;
```

The reason this is safe under concurrency and client-side counting isn't: this single `UPDATE ... RETURNING` statement takes a row-level lock on that specific `clinics` row for the duration of the transaction. If two invoice inserts for the same clinic fire at the exact same moment, Postgres serializes them — one genuinely happens first, gets `invoice_counter = 42`, the second waits its turn and gets `43`. There's no window where both can read the same starting value, because the read and the write happen as one atomic operation the database itself controls, not two separate round-trips from your app.

Then: `new.invoice_number := 'INV-' || to_char(now(), 'YYYY') || '-' || lpad(next_num::text, 4, '0');` — formats it as `INV-2026-0043`. And as a backstop even beyond the trigger, there's `unique (clinic_id, invoice_number)` on the table itself — so even in some scenario where the trigger logic had a bug, an actual duplicate would still get rejected at insert time. Notice it's unique *per clinic*, not globally — Clinic 1 and Clinic 2 can both have an `INV-2026-0001`, and that's fine, because they're scoped separately.

**Why this function ended up split into two, later.** This is the part your docs walk through across iterations 4→9, and it's worth knowing because it shows *why* seemingly small trigger decisions matter:
- v4: originally there were two separate triggers (one deriving `clinic_id`, one generating the number), relying on Postgres running same-event triggers in alphabetical order — an invisible, fragile dependency. Merged into one `process_new_invoice()` function so the ordering is explicit in code.
- v9: this function is INSERT-only, and has to stay that way. If it also ran on UPDATE, then something as trivial as marking an invoice "Paid" would re-run the whole counter-increment-and-renumber logic — corrupting the sequence and detaching the printed invoice number from what's actually in the row. So a *second*, separate trigger (`validate_invoice_clinic_id`, UPDATE-only) was added just to re-check tenancy if `clinic_id`/`patient_id` ever change on an existing invoice, without touching the counter or the number at all.

## 7. "`clinic_id` isn't trusted" / "not just a bare FK" — clarify this

I phrased that sloppily last time, sorry — let me redo it properly. `visits.clinic_id` *is* a real FK: `clinic_id uuid not null references clinics(id)`. So Postgres genuinely does check "does this clinic exist" on every insert. What I meant by "not just" is: that FK check alone isn't enough for what your app actually needs, so there's an *additional* trigger layer doing something a plain FK is structurally incapable of expressing.

Here's the limit of a FK: it can only ever ask "does row X exist in table Y." It has no vocabulary for "and does this satisfy some rule that depends on a *third* table too." `validate_visit_clinic_id()` exists specifically because your real requirement — "this patient must actually have been granted access to this specific clinic" — is a three-way relationship (patient, clinic, and the `patient_clinic_access` junction row connecting them), and a column-level FK constraint simply cannot check across a third table like that. That's what triggers are for.

And there are actually two different trust postures in your schema, worth telling apart:

- **Derive (silently overwrite):** `patient_alerts`, `patient_vitals`, `clinical_notes`, `timeline_events`, `visit_services` — the trigger ignores whatever the client sent and recomputes the correct value itself from the parent row. You physically cannot get this wrong, because your input doesn't matter.
- **Validate (accept or reject, no substitution):** `visits`, `packages`, `complaint_courses`, `invoices` on update — the trigger checks your submitted `clinic_id` against `patient_clinic_access` and throws an exception if it's wrong, but it does *not* try to guess the right value for you.

Why the difference? For a `visit_service`, there's exactly one correct `clinic_id` — whatever its parent visit's `clinic_id` is, no ambiguity, so the trigger can just fill it in. But a *patient* could legitimately have valid access to two or three different clinics in the same chain — the trigger has no way of knowing which one you *meant* this visit to be at, so it can only check whether what you specified is legitimate, not pick for you.

## 8. What actually allows a patient at Clinic 1 but not Clinic 2, same chain?

The general shape of this (junction tables, the unique constraint) is what the second message covers — worth reading that first since it sets up the vocabulary. But your specific question — the concrete "chain A, three clinics" walkthrough — isn't spelled out there, so here it is in full.

Say Chain A's admin is `admin_X`, and Clinic 1, Clinic 2, Clinic 3 all have `clinics.owner_id = admin_X`. Patient P gets registered with `patients.owner_id = admin_X` — same chain. At this exact moment, `patient_clinic_access` has **zero rows** for P. P exists as a chain-level identity, but hasn't been "seen" anywhere yet.

Receptionist at **Clinic 1** opens a complaint course for P. This fires `process_new_complaint_course()`, which:
1. Looks up P's `owner_id` (`admin_X`) and Clinic 1's `owner_id` (`admin_X`).
2. Confirms Clinic 1 actually exists.
3. Checks they match — cross-chain check passes, same chain.
4. Inserts a row into `patient_clinic_access`: `(patient_id = P, clinic_id = Clinic 1)`.

Now P has exactly **one** access row — Clinic 1 only. Nothing has touched Clinic 2 or Clinic 3.

Receptionist at **Clinic 2** (same chain, same `admin_X`) now tries to log a visit for P at Clinic 2. `validate_visit_clinic_id()` fires and asks: does a row exist in `patient_clinic_access` for `(P, Clinic 2)`? **No.** Rejected — even though Clinic 2 belongs to the exact same chain as Clinic 1 where P is already allowed.

The only way P becomes valid at Clinic 2 is if someone *at Clinic 2* opens a complaint course for P there too — same trigger fires again, same chain check passes (`admin_X = admin_X`), and now a *second* access row gets inserted: `(P, Clinic 2)`. P now has two rows and can be seen at either branch — still not Clinic 3, until the same thing happens there.

So the direct answer: **chain membership is necessary but not sufficient.** `owner_id` matching is what stops a patient from a totally different chain being seen at all — that's the hard wall. But *within* the same chain, per-branch visibility is governed entirely by attendance history — whether a `patient_clinic_access` row exists for that exact `(patient, clinic)` pair — and that row only gets created the first time staff *at that specific branch* actually open a complaint course for that patient. It's modeling something true in the real world: three branches can share an owner, but that doesn't mean Branch 3's front desk has ever laid eyes on this person, and they shouldn't see the record until they have.

# [6 Aug 2026 - Module 2 conceptual pass - I - Follow up - II]

## Soft delete — confirmed, you've got it right

Yes — `on delete cascade` is a safety net for an admin/scripting accident, not something that fires in normal operation. `patients.is_active` is the actual day-to-day mechanism. Nothing more to add here, you understood it correctly.

## `patient_alerts` — your instinct is worth taking seriously

Good pattern-match, actually — this is the same shape of thing as the `complaint_catalog` dead-weight issue that got flagged and fixed back in Iteration 2 (a table existed, nothing real pointed at it). I can't confirm or deny it from the docs I have — the migration's source comment cites `PatientAlert type, types/index.tsx L71-74`, meaning at some point a real frontend type existed with this shape, but a type existing in the codebase doesn't mean a working input form exists today. I don't have your actual frontend code in front of me, so I can't check.

Your specific argument is reasonable on its face: `patient_alerts` has `type` (fixed vocabulary: ALLERGY/FALL_RISK/DNR/OTHER) and `label`. `clinical_notes` has `category` (free text) and `observation`, plus `is_critical`. If nothing in your UI treats alerts differently from a critical clinical note — no persistent badge, no separate query path, nothing — then yeah, it's plausibly redundant, and duplicating a table for something one flag already covers is exactly the kind of thing worth cutting.

But don't resolve it right now. This is precisely the kind of question Module 8b exists for — "is the schema solving the right problem," not "does the schema work as written." Deciding it today, mid-Module-2, would be exactly the kind of premature-detail jump you've flagged yourself before. I'm logging it as an open question, not a confirmed gap, and it needs an actual look at your frontend code to settle, not more schema-reading.

Your broader point — that a table with no frontend form isn't automatically a gap, since a lot of the frontend is still mockup — is exactly right, and worth stating as a standing principle: **absence of a form is not evidence of a problem, it's just missing information until you check.** I'll log the general version of this too, as a reminder to eventually audit table-by-table against real frontend usage rather than assume either way.

## The invoice trigger, step by step

Let me redo this properly, because "layer 1 / layer 2" was me being sloppy with terminology, not describing two steps in the code. Walking the actual function top to bottom:

```sql
declare
  next_num integer;
```
This declares a scratch variable, local to this one function call. It exists for the duration of this single trigger firing and then it's gone — just temporary holding space for a value we're about to fetch.

```sql
new.clinic_id := (select clinic_id
                  from patient_clinic_access
                  where patient_id = new.patient_id
                    and clinic_id = new.clinic_id
                  limit 1);
if new.clinic_id is null then
  raise exception ...
```
This checks: does a row exist in `patient_clinic_access` matching both the submitted `patient_id` AND the submitted `clinic_id`? If yes, `new.clinic_id` gets reassigned to itself — a confirming no-op. If no match exists, the subquery returns nothing, so `new.clinic_id` becomes `NULL`, and the very next line catches that and throws the exception. That's the validation step.

Now, **why `UPDATE clinics`, specifically** — this is your actual question, and the answer is: the sequential counter used to generate the human-readable number isn't stored on `invoices` at all. It's stored once per clinic, as `clinics.invoice_counter`. Each clinic has its own independent running count. So to mint invoice #43 for Clinic 1, you have to go to a completely different table — `clinics` — find Clinic 1's specific row, and bump its counter. That's literally what this line does:

```sql
update clinics
set invoice_counter = invoice_counter + 1
where id = new.clinic_id
returning invoice_counter into next_num;
```

`RETURNING invoice_counter INTO next_num` is doing two things in one breath: `RETURNING invoice_counter` tells Postgres "after you finish this update, hand me back the new value" (post-increment, so if it was 42, you get 43 back), and `INTO next_num` says "put that value into the variable I declared earlier." After this single line, `next_num = 43`.

Why does it matter that this is *one* statement instead of "read the current value, then separately write value+1"? This is the concurrency guarantee. If it were two separate steps, two simultaneous invoice creations for the same clinic could both read "42" before either one writes anything back — and both would independently compute 43, giving you two invoices claiming the same number. By folding read-and-increment into one atomic `UPDATE ... RETURNING`, Postgres takes a lock on that specific clinic's row for the duration — the second concurrent request genuinely has to wait for the first to finish before it can even start its own read. There's no window where both can see the same starting number.

Last line just formats it: `'INV-' || year || '-' || zero-padded counter` → `INV-2026-0043`.

**On `security definer`:** this function needs to `UPDATE clinics`, but a receptionist shouldn't have direct UPDATE rights on `clinics` in general — that table holds fee settings, config, stuff you don't want a receptionist editing directly. `SECURITY DEFINER` makes the trigger execute with the privileges of whoever *owns* the function, not the privileges of whoever's session triggered it. So the receptionist can insert an invoice, the trigger quietly does the counter-bump on their behalf under elevated privilege, and the receptionist never needed direct UPDATE access to `clinics` themselves.

**Correcting your v4 restatement:** not quite — let me fix this precisely, because it's a subtle but real distinction. It was never about "not relying on Postgres for the sequence." The counter-increment logic (the atomic `UPDATE...RETURNING`) was always explicit and controlled — that part didn't change across versions. The actual problem was **which trigger runs first** when two separate triggers are attached to the same table for the same event. Postgres's tiebreaker rule for that is alphabetical order by trigger name. The old setup had `invoices_clinic_id_guard` (starts with "c") and `invoices_generate_number` (starts with "g") — "c" happens to sort before "g", so the clinic-validation trigger happened to run first, and only then did the number-generation trigger run with an already-validated `clinic_id`. It worked, but only by an *accident of naming*. Rename either trigger to something that sorts differently, and the number-generator could fire first, against an unvalidated or wrong `clinic_id`, with zero warning. Merging them into one function makes the order explicit in the code itself — step 1 is always line X, step 2 is always line Y, no dependency on how anyone happened to name anything.

**The v9 split, explained properly:** `process_new_invoice()` is registered `BEFORE INSERT` only — it only ever fires when a brand-new invoice is first created, never on update. Why not extend it to also fire on update, like almost every other trigger in your schema does? Because this specific function *unconditionally* increments the counter and re-generates a fresh `invoice_number` every time it runs. If it also ran on UPDATE, then something totally mundane — a receptionist correcting `payment_mode` from "Cash" to "UPI" three days later — would silently re-mint the invoice number. The receipt already printed and handed to the patient would say `INV-2026-0043`, but the database would now say `INV-2026-0057`. Permanent, silent divergence between what was printed and what's stored.

So the responsibility got split into two separate, narrower functions:
- `invoices_process_new` — INSERT only. Only this one is allowed to touch `invoice_counter` / `invoice_number`. Fires exactly once per invoice, ever.
- `invoices_validate_clinic_id` — UPDATE only, a completely different, smaller function. Its only job: if someone changes `clinic_id` or `patient_id` on an *existing* invoice, re-check the new pairing is still legitimate. It never touches the counter or the number — no side effects there at all. And it short-circuits immediately if neither of those two columns actually changed, so routine edits (payment status, payment mode) do zero extra work.

One function owns number-minting and only ever runs once. A second, narrower function owns re-validation and only does work when the thing it cares about actually moved. That's the whole reason it's split.

## The clinic_id / owner_id derivation question — this is the good one

First, your direct factual question, answered straight: **no, clinic-scoped enforcement for receptionists is not implemented today.** I went and checked the actual RLS section rather than assume. Right now, every single table — including `patients`, `visits`, everything — runs on `v1_allow_all`: any authenticated user can read or write any row, full stop. The "real" staff-scoped policy exists, but it's written as a comment, not live:

```sql
-- create policy "staff_patients" on patients
--   for all to authenticated
--   using (
--     id in (select patient_id from patient_clinic_access
--            where clinic_id = (select clinic_id from profiles where id = auth.uid()))
--   );
```

And even once that gets turned on, notice it only covers `patients` — there's no equivalent draft shown for `visits`, `complaint_courses`, or `invoices`. And even for `patients`, it's a `using` clause, which under Postgres defaults to governing both reads *and* writes when there's no separate `with check` — but nothing here stops a receptionist from *submitting* `clinic_id = Clinic 2` on an insert in the first place, as long as the patient happens to already have a `patient_clinic_access` row there. The trigger checks "is this patient legitimately at this clinic," never "is this staff member legitimately at this clinic." That's the actual, current, real gap underneath your question — separate from whether RLS is even turned on yet.

Now the brainstorm, taken seriously, because it's a good one.

**What you're proposing** is the same pattern the schema already uses elsewhere — `derive_owner_id_from_patient()` ignores whatever the client sends and recomputes the truth server-side — just pointed at a different source. Instead of deriving from the *patient's* parent row, you'd derive from the *submitting staff member's* own profile:

```sql
-- not built yet, this is the shape of what you're proposing
new.clinic_id := (select clinic_id from profiles where id = auth.uid());
```

**Where this is strictly better than what exists today:** validation checks a claim after the fact ("is Clinic 2 valid for this patient?" — yes, if some other receptionist already put them there). Derivation removes the ability to make a false claim in the first place — if `clinic_id` is always overwritten with the submitter's own assigned clinic, a Clinic 1 receptionist *cannot* write a Clinic 2 row, period, regardless of what the frontend sends or whether the patient happens to have access there. That's a structurally stronger guarantee than anything currently in the schema for this particular question.

**Where it gets genuinely complicated, not just theoretically:**

1. **Admins break the simple version.** `profiles.clinic_id` is `NULL` for every admin, by design — that's how they span all locations. "Derive clinic_id from my own profile" gives an admin `NULL`, which is useless if an admin ever creates a visit or invoice directly. So this can't be one uniform rule — it needs a branch: derive for staff, fall back to validate-against-chain for admins. Not a dealbreaker, just means it's not as clean as the patient-derivation triggers.

2. **This would behave differently in your own hands-on testing than in the real app** — and this one's worth flagging clearly before you build anything like it. Module 5 in your study plan already explains this directly: `auth.uid()` reads from the request's JWT, and when *you* run raw SQL in the Supabase SQL Editor tomorrow — exactly what your Module 1 and Module 2 exercises have you doing — you're connected as a superuser with no request JWT at all, so `auth.uid()` returns `NULL`. A trigger built around `auth.uid()` would work correctly through your actual app, but break or behave strangely every time you test it by hand in the SQL Editor the way you've been doing all along. Worth knowing going in, not discovering as a confusing surprise mid-exercise tomorrow.

Your two scenarios, quickly separated, because they're actually different questions: **single-clinic chain, can we just guess clinic_id** — that's a UX/default-inference question (auto-select the only option), not a security one; you'd still want it enforced server-side regardless of whether the frontend conveniently pre-fills it. **Receptionist scoped to one clinic, derive instead of trust** — that's the real security question, and it's a legitimate improvement worth building deliberately, not something wrong with the current design so much as something the current design hasn't gotten to yet.

I'm not resolving this now — it's a real design decision worth sitting with with fresh eyes, probably around when you get to Module 5/6's RLS and SECURITY DEFINER material. Logging it as open.

# [8 Aug 2026] - Module 2 Conceptual Pass - I - Follow up - III

## M2-05 — Invoice Trigger Redone, Full Visit Workflow, Security Definer Scope, Admin Views

The naming convention — good idea, doing it now. Every response from here gets an ID like the one above: `M{module}-R{number}`. Retroactively, so you can point at anything already said:

- **M2-01** — FK basics: `references`, `on delete cascade`, reference vs. transactional tables, the `visits` anchor
- **M2-02** — Many-to-many junctions, FK existence-vs-correctness, optional/required FKs, denormalized snapshots, indexing FKs
- **M2-03** — Q&A: gaps log created, `owner_id`/`clinician_id` role gaps, `patient_alerts` skepticism, first invoice-trigger pass, derive-vs-validate, the chain walkthrough
- **M2-04** — "Ultimate detail": one-to-one via shared PK, circular FK, multi-FK-same-table, NULL semantics, composite-unique scoping, many-side-holds-FK rule, the denormalization gap, full dependency ordering
- **M2-05** — this one

So your two references from this message: the unread "bring it all, ultimate detail" one is **M2-04**. The one before it, with confirm/delete and `patient_alerts`, is **M2-03**. That's exactly the confusion this fixes.

Now let me redo the invoice trigger properly, with actual numbers this time, because the abstract version clearly didn't land.

---

## The three separate things, three separate tables — no ambiguity this time

- **`invoices.id`** — table: `invoices`. Real primary key, `uuid default gen_random_uuid()`. Generated instantly on row creation, unrelated to any trigger, never human-facing. The row's true identity.
- **`invoices.invoice_number`** — table: `invoices`, same table, different column, NOT the primary key. Plain text, e.g. `"INV-2026-0043"`. Set by a trigger, not the client.
- **`clinics.invoice_counter`** — table: `clinics`. A completely different table from `invoices`. Plain integer. Not an identifier of anything by itself — a running tally, per clinic, of how many invoices that clinic has had.

## Confirming directly: `process_new_invoice()` is the merged function

Yes — this is exactly the thing I described earlier as "the old two triggers combined into one." Not something different. I just wasn't consistently naming it, which is what caused the confusion. From here on, every time I mean this function, I'll say `process_new_invoice()` by name, every time.

## The trigger, walked through with real numbers

Say Clinic A's row in `clinics` currently has `invoice_counter = 42` — meaning 42 invoices have ever been created at Clinic A. A receptionist's action eventually results in this running:

```sql
insert into invoices (clinic_id, patient_id, amount_in_paise, date, visit_id)
values ('<clinic-a-uuid>', '<patient-uuid>', 50000, current_date, '<visit-uuid>');
```

**Before** this row is actually written into the **`invoices`** table — table name, explicit, every time from here — the trigger `invoices_process_new` fires. It is attached to `invoices`, it fires on `INSERT` only, and it runs the function `process_new_invoice()`.

Inside that function:

1. `declare next_num integer;` — a scratch variable, existing only for this one function call, empty so far.
2. The submitted `clinic_id` gets checked against `patient_clinic_access` — confirms this patient genuinely has access at Clinic A. Assume it passes.
3. `update clinics set invoice_counter = invoice_counter + 1 where id = new.clinic_id returning invoice_counter into next_num;`

   One line, doing all of this together: go to the **`clinics`** table (a different table from the one being inserted into), find the one row where `id` = Clinic A's uuid, take its `invoice_counter` (42), set it to 43, and immediately hand that new value straight back into `next_num`. After this line: `clinics.invoice_counter` is permanently 43 in the database. `next_num` — a temporary variable, stored nowhere — now holds 43.
4. `new.invoice_number := 'INV-' || to_char(now(), 'YYYY') || '-' || lpad(next_num::text, 4, '0');`

   Takes the 43 sitting in `next_num`, formats it as `"0043"`, glues on `"INV-"` and the year, produces `"INV-2026-0043"`, and assigns it to `new.invoice_number` — `new` meaning the row that's about to land in `invoices`.
5. Function returns, Postgres finishes the insert it was already doing, and the row that ends up in `invoices` has `invoice_number = 'INV-2026-0043'` — a value the receptionist's frontend never sent, never needed to send.

**"What needs the counter":** nothing consumes it as a reference — nothing points a foreign key at `clinics.invoice_counter`. It's memory, nothing more: "how far have we counted for this clinic," so the *next* invoice at Clinic A knows to become 44, not 43 again. It gets read, bumped, and its new value is borrowed once, briefly, to build a display string for a totally different column on a totally different table (`invoices.invoice_number`), and then it's done.

**"Returns to what":** into `next_num` — a variable that exists only inside this one function call, used two lines later, for its own internal purpose. Nothing outside this function ever sees "43" directly. The receptionist's frontend eventually gets back the finished row, including the already-formatted `invoice_number = "INV-2026-0043"` — never the raw counter value.

## What's split, what's combined — laid out directly

| Trigger | Attached to table | Fires on | Runs function | What it does |
|---|---|---|---|---|
| `invoices_process_new` | `invoices` | `BEFORE INSERT` only — never on UPDATE | `process_new_invoice()` | The merged function. Validates `clinic_id`/`patient_id`, increments `clinics.invoice_counter`, generates `invoice_number`. Descendant of the old v3 two-trigger setup — the alphabetical-ordering fix you already know. |
| `invoices_validate_clinic_id` | `invoices` | `BEFORE UPDATE` only — never on INSERT | `validate_invoice_clinic_id()` | Separate, newer (v9). If `clinic_id`/`patient_id` change on an *existing* invoice, re-checks the pairing. Never touches the counter or `invoice_number`. No old-trigger ancestor — this problem (update-time tenancy) didn't exist before v9 named it. |

Both sit on the same table. Split by *event*, not by table.

---

## The full visit → invoice workflow

Honesty check first: I have `codebase-context.md` (a prose description of the frontend) and the full schema (which tells me the required *backend* dependency order with total certainty), but not your actual current frontend code. That doc itself says Supabase integration is still "upcoming" and the app currently runs on mock data — so what follows is what the schema requires, informed by hints in that doc, not a report of confirmed current behavior. I'll flag every place I'm inferring versus stating a schema fact.

**Entry point.** Your context doc names a real component, `VisitWorkflow`, gated behind an `isStarted` flag and a "Start Workflow" button — but the *stated purpose* given for that gate is specifically "prevent the daily ledger's auto-scroll from hijacking page load on open," not explicitly "this is how a visit begins for a specific patient." It might be the same thing wearing two hats, or it might not — I can't confirm that leap from the doc alone. What I *can* say with more confidence: the same doc names a "Single-Screen Session Start" pipeline as a deliberate design goal, which pushes against your multi-screen-wizard mental model — it suggests something more consolidated than patient → button → separate complaint screen → separate procedure screen as distinct navigations. Worth opening `VisitWorkflow`'s actual source to settle this for real; I'm not going to pretend I know for certain.

**Complaint selection — your direct question, answered with full confidence.** Yes: populating this needs nothing but a read.

```sql
select * from complaint_courses where patient_id = '<uuid>' and status = 'Active';
```

A `SELECT` never fires an `INSERT`/`UPDATE` trigger, creates no row, costs nothing, and is completely decoupled from any notion of "a visit has started" in the database's eyes. You can check as many times as you like with zero side effects.

One real structural thing worth flagging here, though, tying straight back to something already in your notes: `visits.complaint_course_id` is singular — `uuid not null references complaint_courses(id)`, one value, not a list. The schema models **one complaint per visit**, not one session covering several complaints. If the UI ever lets someone select two active complaints for what feels like a single sitting (plausible, given the "Composite Key Billing" `complaintId::procedureId` pattern your own doc describes), the backend equivalent isn't one `visits` row with two complaints attached — it's *two* `visits` rows, same `date`, one per complaint. This is the exact mechanical shape of the "no session/encounter concept, only `patient_id` + `date` approximates grouping" gap you already have logged, and the Option A/B/C invoice decision sitting on your to-resolve list is precisely about what happens next — do those two visits get billed as two invoices, or batched into one. Good moment to notice exactly where that decision bites.

**Procedure logger.** Same story — a read, nothing more:

```sql
select s.*, coalesce(csp.price_in_paise, s.standalone_price_in_paise) as effective_price
from services s
left join clinic_service_prices csp on csp.service_id = s.id and csp.clinic_id = '<clinic-uuid>';
```

Populate the list, compute the effective per-clinic price, zero writes, zero triggers.

**Pre-selection / "last used" — worth taking seriously, and it's not a new idea.** Your own codebase context already names this: "**Repeat-Visit Autofill** workflow for high-volume returning therapy patients." Worth grepping your actual frontend for that exact term to see what's already built versus just named as a goal — I only have the description, not the component.

Either way, here's the good news: this needs **zero schema changes**. Everything required already exists as a queryable join across tables you already have:

```sql
-- most recent complaint course treated
select complaint_course_id from visits
where patient_id = '<uuid>' order by date desc, created_at desc limit 1;

-- services logged on that same visit
select service_id from visit_services
where visit_id = '<that visit's id>';
```

Both are ordinary reads against existing relationships — the many-to-many and FK material from earlier applies directly here, this is just that material put to work. If you ever want something smarter than "literally last time" — like a stored "prescribed program" per complaint course, distinct from whatever happened to get logged last visit — *that* would need new schema (something like a `prescribed_services` concept). Not needed today; worth keeping in the back of your mind given your own instinct to structure things for configurability now rather than overhaul later.

**The actual submit moment — this is what answers the counter-timing question.** Two pieces of your own documentation point the same direction. First, `visits` carries the comment "MVP WORKFLOW: Row existing = session complete + paid. No status lifecycle." Second, `invoices.payment_status` defaults to `'Paid'`, with the comment "Receptionist creates one entry *after* session done + paid." Third — and this is the strongest signal — the "Recursive Complaint-Orphan Stripping" safeguard operates on a "**final invoice payload**," assembled from everything selected across complaints and procedures, stripped of orphans, "**before submitting**." That phrase implies one coherent payload built up entirely in local/frontend state across the whole flow, submitted once.

Put together, the most likely design: nothing touches the database until one final confirm/submit action. At that moment — in the dependency order the schema forces, which we already mapped fully in M2-04 — the flow creates, in order: a new `complaint_courses` row only if this is a genuinely new complaint (not one being selected from the existing active list) → a `visits` row → one `visit_services` row per logged procedure → an `invoices` row. Whether that's literally one network call or a short sequence fired together by one button click, I can't know without the real integration code — but from the database's perspective, and the user's perceived experience, it's one action producing one finished visit and one finished invoice, not a visit sitting half-written while procedures are still being decided.

**Which directly answers your counter question:** `clinics.invoice_counter` increments exactly once, at the very last of those steps — the moment the `invoices` row insert actually fires. Not at "Start Workflow," not at complaint selection, not while procedures are being logged. Only there.

Concretely worth opening to confirm all this for real: `VisitWorkflow`, `ProcedureLogger/index.tsx`, `useProcedureLogger.tsx`, and whatever function eventually wires the submit action to Supabase once that integration exists.

---

## Security Definer — deferred, on purpose

This is Module 6 — "SECURITY DEFINER & Self-Enforcing Triggers," anchored on `process_new_complaint_course()` and `validate_owner_is_admin()`. Not teaching it now, exactly as you asked. You'll get the real mechanics — what it is under the hood, the syntax, where it's declared — there.

## "Which staff member is legitimate at this clinic"

Correct call — that's RLS, Module 5 (with a Module 6 self-enforcement angle once `SECURITY DEFINER` is on the table). Skipping it here.

---

## Admin views — the brainstorm

**Does the clinic-switcher idea reintroduce the "trust the frontend" problem?** No — and the reason why is worth understanding precisely, because it's a genuinely different situation from the receptionist gap we logged.

Today, `profiles.clinic_id` is `NULL` for every admin by design, and the RLS model (`owner_id = auth.uid() and role = 'admin'`) grants an admin the *entire chain*, unconditionally, with no clinic filter at the database layer. So "admin picks a clinic to view" isn't a backend access-control question at all — it's a **frontend display filter**, layered on top of access the admin already, always has.

That distinction is the whole answer. For a receptionist, trusting the frontend's submitted `clinic_id` is a real problem, because it's standing in for an actual security boundary — get it wrong (bug, tampering, a client that bypasses the UI entirely) and someone sees or writes data they were never supposed to touch. For an admin's clinic-picker, even in the worst case — the selected value is dropped, corrupted, or spoofed — the admin isn't gaining access they didn't already have. RLS already grants everything in the chain regardless of what that picker says. If the filter breaks, the failure mode is "admin sees all three clinics instead of just the one they meant to" — a UX hiccup, not a data breach. **Trusting frontend input is dangerous when it's substituting for a security boundary; it's harmless when it's a narrowing convenience layered on top of a boundary that's already correctly enforced underneath.** That's the generalizable version of what makes these two cases different, and it's worth being able to say exactly that in an interview if this ever comes up.

**How to actually build it.** Backend: nothing changes. Admins already see the full chain via the existing policy. Whatever query the switcher UI fires just adds `and clinic_id = '<selected-uuid>'` on top of a query the admin was already allowed to run — no new trigger, no new RLS policy, no schema change. Frontend: the selected clinic lives as ordinary client-side state — a React context wrapping the admin dashboard is the natural fit, given your stack currently leans on `useState`/`useReducer` with TanStack Query coming for server state. Every clinic-scoped query the admin UI fires reads that context value as an optional filter parameter. Bonus: this naturally supports an "All Clinics" option for free — that's just "don't add the extra `WHERE` clause" — no special-casing needed anywhere.

**What would an admin actually want at the chain level, not the clinic level?**

- **Cross-clinic patient lookup.** "Has this patient been seen anywhere in my chain, and where." Fully supported today: `select clinic_id, first_visit_date from patient_clinic_access where patient_id = X`, joined to `clinics.name`. This is something a Clinic 1 receptionist structurally *can't* see (no way to know a patient also attends Clinic 3) but an admin naturally can — arguably the single clearest justification for chain-level access existing at all.

- **Revenue rollup across clinics**, aggregate and per-clinic side by side — a straightforward `group by clinic_id` over `invoices` or `daily_ledger`. One honest gap here: the docs I have only draft the admin RLS policy for `patients`; there's no equivalent drafted yet for `invoices` or `visits`. Worth remembering this is genuinely undesigned territory, not something already quietly solved.

- **Clinician performance across branches** — "who saw the most patients this week, chain-wide" — `group by clinician_id`, joined to `profiles` for display names. This is a direct callback to the multi-FK-to-`profiles` material from M2-04 — exactly the kind of query that needs a clean, deliberately-aliased join, not an accident of "there's only one `profiles` join so I don't need to think about it."

- **Chain-wide invoice list, with a real UX wrinkle worth knowing now.** Since `unique(clinic_id, invoice_number)` is scoped per clinic, not globally, an admin's "all invoices" view will legitimately show `INV-2026-0012` at *both* Clinic A and Clinic B — different real invoices, identical number string. Any chain-level invoice list needs to display the clinic name alongside `invoice_number` to disambiguate, or fall back to the real `id` where a truly unique label is needed. Direct, concrete consequence of a decision already made and logged — not new work, just something to remember when that screen gets built.

- **Package oversight across branches** — active packages, expiring soon, chain-wide — same aggregate shape as the revenue rollup.

- **A point actually *in favor* of `patient_alerts`, to balance the skepticism from M2-03.** If it survives that audit, a structured `type` column is exactly what makes a chain-wide safety query trivial — `where type = 'DNR'` across every branch in one shot. Try to do the same thing against `clinical_notes.observation` (free text) and you're mining unstructured strings for a safety-critical flag, which is a real usability regression if this is the kind of thing an admin might genuinely want to scan chain-wide. Doesn't settle the earlier question — still needs the real frontend-usage check — but it's a legitimate use case in the "keep it" column, worth weighing fairly rather than assuming the table's dead weight.

# [6 Aug 2026 - Depth Pass - II]
<!-- TODO: Yet to read -->

Yeah, there's real depth left in Module 2 that I skipped over to keep the first pass digestible. Here's the rest, no Module 3 content mixed in.

## Many-to-many relationships: why `patient_clinic_access` exists at all

Everything I showed you last time was **one-to-many**: one patient, many visits. That's a single FK column on the "many" side (`visits.patient_id`), and you're done. No junction table needed.

But the patient-to-clinic relationship isn't one-to-many — it's **many-to-many**. One patient can attend multiple clinics in the chain. One clinic sees multiple patients. You cannot represent that with a single FK column on either table, because neither table can hold "multiple values" in one column cleanly. `patients` can't have a `clinic_id` column that's simultaneously A and B. `clinics` can't have a `patient_id` column that lists every patient who's ever walked in.

The fix is always the same shape, no matter what database you're looking at: a third table sitting between the two, holding pairs.

```sql
create table patient_clinic_access (
  id         uuid primary key default gen_random_uuid(),
  patient_id uuid not null references patients(id) on delete cascade,
  clinic_id  uuid not null references clinics(id),
  first_visit_date date not null default current_date,
  created_at timestamptz not null default now(),
  unique (patient_id, clinic_id)
);
```

Each row means "this patient has attended this clinic." A patient with three rows here has attended three branches. A clinic with 500 rows here has seen 500 distinct patients. Neither `patients` nor `clinics` had to change shape to support this — the relationship lives entirely in the junction table.

Notice the `unique (patient_id, clinic_id)` constraint. This is doing a different job than `unique(owner_id, mrn)` on `patients`. That one prevented two *different real-world patients* from colliding on the same MRN. This one prevents the *same relationship* from being recorded twice — a patient can't have two "first visit" rows for the same clinic. It's what makes `on conflict (patient_id, clinic_id) do nothing` in the trigger logic safe to call repeatedly: the second complaint course at the same branch tries to insert the same pair again, the unique constraint blocks the duplicate, `do nothing` just quietly moves on instead of erroring.

Contrast this with `patients ↔ visits`, which never needed a junction table, because a visit only ever belongs to exactly one patient — that's one-to-many, not many-to-many, so a plain `patient_id` column on `visits` is sufficient.

## FKs enforce existence — they say nothing about correctness

This is the one worth sitting with, because it's not obvious and it's the exact shape of a real bug already sitting in your schema.

```sql
owner_id uuid not null references profiles(id)
```

Read literally, this constraint checks exactly one thing: *does a row with this id exist in `profiles`?* That's it. It does not check what `role` that profile has. It does not check whether that profile is the "right" admin for this particular clinic chain. A foreign key is a existence check, not a business-rule check.

So on paper, nothing stops you from writing:

```sql
insert into patients (owner_id, mrn, full_name, ...)
values ('<some receptionist's profile id>', 'TEST01', 'Jane Doe', ...);
```

The FK is satisfied — that id really does exist in `profiles`. But your entire admin RLS model is built on `owner_id = auth.uid() and role = 'admin'`. If `owner_id` points at a receptionist, that patient silently vanishes from every admin's view forever, with zero error at write time. That's exactly why `validate_owner_is_admin()` had to be added as a *separate* trigger in v10 — the FK alone was never going to catch this, because catching it isn't a FK's job.

And this is precisely the gap flagged in your handover notes: `validate_owner_is_admin()` confirms `owner_id` belongs to *some* admin, but not the *correct* admin for the submitting staff member's chain. Same category of problem, one layer deeper — the FK checks existence, the trigger checks "is this an admin," but nothing yet checks "is this the right admin for this specific write." Three different questions, three different enforcement mechanisms, and each layer only catches what it was specifically built to catch. That's the pattern to internalize: **a relationship being structurally valid (FK satisfied) and a relationship being semantically correct (right admin, right chain) are two separate guarantees**, and your schema has to earn the second one explicitly — it never comes free.

## Optional vs. required relationships

Not every FK in your schema is `not null`. Compare:

```sql
patient_id uuid not null references patients(id) on delete cascade   -- visits: required
visit_id   uuid references visits(id)                                -- invoices: optional
```

`visits.patient_id` is required — a visit that isn't attached to a patient is meaningless, the row shouldn't be able to exist without it. But `invoices.visit_id` is nullable, and the migration comment tells you why: *"not all invoices link to a visit."* An invoice could be for a package purchase, or some future billing event that never maps cleanly to a single session.

This shows up in a few places once you look for it:
- `complaint_courses.complaint_catalog_id` — nullable, because free-text custom complaints have no catalog entry to point at.
- `timeline_events.visit_id` and `timeline_events.note_id` — both nullable, because a manual clinical log might not originate from either.
- `patients.clinician_id` — nullable, registration doesn't always have a clinician attached yet.

The practical consequence: when you eventually write queries joining across an optional FK, you need a `LEFT JOIN`, not an `INNER JOIN` — an inner join would silently drop every invoice that has no `visit_id`, which is most of them under this schema's design. A required FK is safe to inner-join on, because the relationship is guaranteed to exist for every row. An optional one isn't. This is a small thing that causes real, hard-to-spot bugs later if you don't know which of your relationships are guaranteed and which aren't.

## Denormalized snapshots: deliberately duplicating data the relationship already gives you

Standard relational advice says: don't store the same fact twice, reference it via FK and look it up when you need it. Your schema breaks that rule on purpose, more than once:

```sql
complaint_courses:
  complaint_name       text not null,           -- the actual stored name
  complaint_catalog_id text references complaint_catalog(id), -- the FK, nullable

visit_services:
  service_name     text not null,   -- stored name
  service_id       text not null references services(id),  -- FK

packages:
  linked_complaint_name text not null,  -- stored name
  linked_complaint_id   uuid not null references complaint_courses(id), -- FK
```

In every case, there's a FK you could join through to get the name live, and *also* a plain text column holding that name at the time the row was written. Why keep both?

Because the catalog can change later, and history shouldn't change with it. If `complaint_catalog` renames `"CAT_S01"` from "Lower Back Pain" to "Lumbar Strain" next year, every complaint course created before that rename should still display what it was actually called when the clinician wrote it down — not get silently relabeled the moment someone edits the catalog. The FK (`complaint_catalog_id`) exists so you *can* trace back to the current canonical entry if you need to. The text column (`complaint_name`) exists so the historical record is frozen and accurate regardless of what the catalog does afterward.

This is the same reasoning, applied differently, as why `visits.consultation_fee_in_paise` stores a *snapshot* of the fee at the time of the visit rather than always looking up `clinics.consultation_fee_first_in_paise` live — if the clinic raises its rates next month, last month's visit shouldn't retroactively show the new price. Relationships in this schema get chosen deliberately between "always reflect the current source of truth" (a live FK join) and "freeze what was true at write time" (a stored snapshot) — and it's not always obvious which one you're looking at until you check whether there's a plain column sitting next to the FK.

## Indexes on foreign keys aren't automatic

One thing that's easy to assume incorrectly: creating a foreign key does **not** automatically create an index on that column in Postgres. The FK constraint guarantees referential integrity — it checks the parent row exists — but a database has to *find* rows by that column efficiently too, and that's a separate concern the FK doesn't solve by itself.

That's why your migration has a whole block of manual index creation sitting apart from the table definitions:

```sql
create index idx_visits_patient_id      on visits(patient_id);
create index idx_visits_complaint_id    on visits(complaint_course_id);
create index idx_complaint_courses_clinic_id   on complaint_courses(clinic_id);
create index idx_complaint_courses_patient_id  on complaint_courses(patient_id);
```

Every FK column you'll actually filter or join on repeatedly — which, in this schema, is basically all of them — needed its index spelled out by hand. Skip it, and something like "get all visits for this patient" degrades from an index lookup into a full table scan once the table has real volume. This is a quiet but real cost of relationships: declaring the FK gives you correctness, but you still have to separately ask for the performance.

---

That covers what Module 2 actually has to offer, expanded fully — many-to-many via junction tables, the existence-vs-correctness distinction in FKs (which is the same shape as the real bug sitting in your notes), optional vs required relationships, deliberate denormalization, and FK indexing. Nothing here spills into generated columns, views, or trigger internals — that's Module 3 and 4's territory, saved for when you get there.