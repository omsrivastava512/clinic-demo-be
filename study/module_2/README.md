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

# [6 Aug 2026 - Module 2 conceptual pass - I - Follow up]

Good call on the file — that's exactly the right instinct, and I should've defaulted to it without you asking. I'll set it up now and keep appending to it as we go, same structure as your Module 1 handover's running log. Let me work through everything else first, then create it at the end with what we've found today.

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


