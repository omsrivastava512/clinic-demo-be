# [17 Aug 2026] Question about Package Logic

> [!NOTE]
> **Notice for AI Agents:** This file contains historical discussion iterations and raw context. AI agents do **not** need to iterate through or re-evaluate this discussion. These discussion iterations serve as background context that led to the finalized specifications in `README.md` and `DEFERRAL.md`.

> **[COMPACTED PROMPT]**
> Please review our packages schema, related tables, and business logic README to evaluate how package attendance and expiration should be architected across four key areas: First, since both `visits` and `packages` link directly to `complaint_courses`, why is an explicit `visits.package_id` foreign key necessary rather than deriving package redemption through the shared complaint course? Second, how should the database model the two distinct real-world clinic packaging models—consecutive working-day subscriptions (where prepaid care elapses across consecutive active clinic days) versus session-based slot vouchers (where a fixed count of visits can be redeemed on arbitrary dates)? Third, how should operational disruptions and attendance exceptions be handled: how do we track clinic off-days, mid-day emergency closures, or holidays without complex calendar maintenance (evaluating whether to store explicit dates like `expiry_date` vs. integer day counters vs. day logs), and how do we support manual package freezes or patient-specific leave waivers without penalizing unexcused lapses? Finally, how can we design a clean, hierarchical configurability model (with precise domain naming conventions) that allows clinics to define a default packaging behavior while supporting per-patient or per-package overrides for special accommodations (such as paralysis cases requiring flexible visit vouchers in a subscription-based clinic)?

Confirmed against the live files — `packages.linked_complaint_id` has no `unique` constraint, and `visits` has no `package_id` column anywhere in the DDL. That fact is going to matter in point 1 below. Also re-read Section 7 and Section 8 of the README in full — I'll be leaning on Section 7's override pattern for point 4.

Before I get into it — two things I want to nail down first, because my answer actually changes shape depending on them:

**A.** For a pure session-based package (your friend's case) — is there *any* outer time boundary at all, even a loose one ("sessions must be redeemed within a year"), or is it genuinely open-ended, gated only by attendance, forever? This decides whether session-based packages need any date columns at all, or none whatsoever.

**B.** For the per-patient default in point 4 — do you want that setting to be *binding* (a receptionist literally cannot pick a different mode for that patient without a deliberate override step), or a *smart pre-fill* (shows up pre-selected on the form, but is freely changeable per package, same spirit as the `clinics.default_visit_type` decision already sitting in Section 2)? This decides whether it needs a validation trigger or is pure UI convenience.

I'll answer everything below using my best judgment either way, and flag exactly where the answer to A or B would change what I'm proposing. Let's go in order.

---

## 1. The `package_id` pushback

Your instinct — "we already have a link, why add another one" — is the *right* instinct to default to. Redundant identifiers are how schemas rot. So let me actually show the reasoning this time instead of just asserting a new column, which is a fair thing to have called out.

Here's what's actually true, checked directly against the file I just re-read:

```sql
-- visits
complaint_course_id uuid not null references complaint_courses(id)

-- packages
linked_complaint_id uuid not null references complaint_courses(id)
```

You're right that both point at `complaint_courses`. That link is real, and it already answers "what complaint is this visit for" and "what complaint was this package sold against." But it answers a *different question* than "which specific package should this visit's day count against" — and those two questions usually have the same answer, but not always, for one concrete structural reason: **`linked_complaint_id` has no `unique` constraint.** Nothing in the schema stops a single `complaint_course_id` from having more than one `packages` row.

And that's not a hypothetical edge case for you specifically — it's the normal shape of a long-running complaint. A `complaint_course` can span months (`total_sessions` keeps growing, `status: Active` the whole time). A patient buys a 10-day package against it, fully uses it, the complaint isn't resolved yet, they buy a second 10-day package against the *same* `complaint_course_id`. Today, nothing even stops both packages from showing `status = 'Active'` at the same time. If a visit gets logged for that complaint next week, and the only link is `complaint_course_id`, there's no way to say which of the two packages that visit should draw down — you'd have to guess, or add a rule like "the most recently purchased active one," which is exactly the kind of implicit, un-auditable logic this schema has spent ten iterations trying to get rid of.

There's a second, quieter reason too: even with only one package ever, `complaint_course_id` tells you "same complaint," not "should consume a package day." Suppose the package is fully exhausted but `status` hasn't flipped yet (day 11 comes in before anyone's checked), and the patient pays standalone for that visit instead of it silently eating into a package that's already spent. If the only signal is "same complaint_course_id, package still shows Active," that visit gets miscounted. An explicit link records a *decision made at logging time* — not an inference reconstructed later from two rows that happen to share a foreign key.

So: keep `linked_complaint_id` exactly as it is — it's still doing real work (the clinical relationship, "what is this package for"). Add `visits.package_id`, nullable, as a *separate* fact — "did this specific visit consume a day of this specific package." They're correlated, not identical, and the moment sequential packages against one complaint become possible (which they already are, today, with zero schema change needed), conflating them stops being safe.

```sql
alter table visits add column package_id uuid references packages(id);
```

Two more things worth doing alongside this, both cheap:

**A partial unique index**, independent of the `package_id` question, closing the "two Active packages on one complaint" gap directly:
```sql
create unique index uq_one_active_package_per_complaint
  on packages (linked_complaint_id)
  where status = 'Active';
```
This is a good idea regardless of anything else in this message — it's a real data-integrity hole sitting there right now.

**How `attended_days` gets kept correct**, now that a real link exists. My earlier answer reached for the `invoice_counter` pattern (a trigger-maintained counter) without checking whether that comparison actually holds up — it doesn't, and I should've caught that the first time. `invoice_counter` needs a trigger specifically because it must be *atomic and sequential* under concurrent writes — two invoices can't get the same number. `attended_days` has no such requirement; it's a plain count, and counts don't care what order they're computed in. A trigger-maintained counter also has to correctly handle insert, update (package_id reassigned), *and* delete to avoid drifting — three paths, more surface area, in a project that's explicitly deferred a full test suite. Given that, I'd compute it at read time instead:

```sql
select count(*) from visits where package_id = $1
```//

Zero drift possible, by construction, and it automatically self-corrects if a visit is ever edited or removed — no second trigger to maintain. Given you're deliberately trying to keep this MVP lean, this is the lower-maintenance choice, not just the "more correct" one.

---

## 2. Date-based vs. session-based — naming the actual distinction

Your friend's case draws a real, clean line, and it's worth naming precisely because the two models are almost opposites in how their clock runs:

- **Date-based (subscription).** The clock runs on the calendar, not on attendance. "10 working days from today" — every clinic-open day that passes counts against the total, whether the patient shows up or not. Not showing up is a loss, on the patient.
- **Session-based.** The clock only moves when the patient actually attends. No calendar pressure at all — 10 visits is 10 visits, whenever they happen.

Worth being direct about this: **your current schema is built entirely for the date-based model, and has zero support for session-based today.** `duration_days`, `expiry_date`, `exclude_sundays`, `missed_days` — every one of those columns only makes sense if the clock is calendar-driven. There's no "sessions remaining, no calendar involved" concept anywhere. That's not a criticism, it's just useful to say plainly, because it means the session-based mode isn't a tweak to the existing shape — it's a second, genuinely different mode that the schema needs to be able to hold *alongside* the first one. That's what points 3 and 4 build toward.

For the date-based model specifically — here's the mechanism, worked as a real week, because that's what actually clarifies it. Patient buys a 10-working-day package on Monday, Aug 3:

| Date | Clinic status | Patient shows up? | Result | Days used |
|---|---|---|---|---|
| Mon Aug 3 | Open | Yes | Attended | 1/10 |
| Tue Aug 4 | Open | No | Missed | 2/10 |
| Wed Aug 5 | **Closed** (festival) | — | Doesn't count | 2/10 |
| Thu Aug 6 | Open | Yes | Attended | 3/10 |
| Fri Aug 7 | Open | Yes | Attended | 4/10 |
| Sat Aug 8 | Open | No | Missed | 5/10 |
| Sun Aug 9 | Excluded by default | — | Doesn't count | 5/10 |
| Mon Aug 10 | Open, then closed at 2pm (emergency) | Yes, came at 10am | Attended | 6/10 |
| ... | | | | continues to 10/10, then auto-expires |

That Aug 10 row is doing a lot of work, and it's the exact scenario you raised in point 3 — let me carry it there.

---

## 3. Dates vs. days, the closure mechanism, and the mid-day case

You asked the right question directly: *"should I store dates, or should I store days?"* — **store days (computed), not a mutable `expiry_date`.** Here's the concrete reason, not just a preference.

If `expiry_date` is a real, stored date that gets pushed forward every time the clinic closes, then pressing your "we're closed today" button has to go and `UPDATE` every currently-active package's `expiry_date` — and only the ones where the patient hasn't shown up yet today, which means the update needs to know, at the moment it fires, who's already been seen. That's exactly the "run midway" complication you were wrestling with — it forces you to care about *timing*, whether the button was pressed before or after a given patient's visit.

If instead nothing is stored except `purchase_date` and `duration_days`, and "days used" is *computed* at the moment it's needed, the timing problem disappears entirely. The computation just asks two independent questions per day, and neither depends on when during the day anything happened:

1. Was there a visit logged against this package on that date? → **attended**
2. If not — is that date marked as a clinic closure? → **doesn't count.** Otherwise → **missed.**

That's it. A patient who came in at 10am before a 2pm emergency closure already has a visit row for that date, so question 1 already resolved it — question 2 never even needs to look at them. A patient who hadn't come in yet gets caught by question 2 and is waived. You don't need to know *when* the button was pressed relative to anyone's visit — the two facts (attended? closed?) are independently true or false for that date, and they compose correctly no matter the order they happened in. This is genuinely the cleaner design, and it's the one your own phrasing in point 3 was already leaning toward.

The closure "button" itself, then, doesn't need to touch a single `packages` row — it's one insert:

```sql
create table clinic_closures (
  id           uuid primary key default gen_random_uuid(),
  clinic_id    uuid not null references clinics(id),
  closure_date date not null,
  reason       text,
  created_by   uuid references profiles(id),
  created_at   timestamptz not null default now(),
  unique (clinic_id, closure_date)
);
```

Reactive, not a maintained calendar — a row only exists for a day that was *actually* closed, inserted whenever the decision is made, whether that's 9am or 2pm. Nothing to pre-populate, nothing to keep in sync, which matches exactly what you said you wanted ("I don't want to keep track of all that calendar — it's on the clinician"). Sundays stay handled by the existing `exclude_sundays` flag on the package, since that's a standing rule, not an exception — no need to write a row for every Sunday that ever happens.

And your last point in message 3 — the individual patient waiver, "on the owner's mercy" — is the same shape, scoped one level tighter, to a single package instead of the whole clinic:

```sql
create table package_day_waivers (
  id           uuid primary key default gen_random_uuid(),
  package_id   uuid not null references packages(id),
  waived_date  date not null,
  reason       text,
  granted_by   uuid references profiles(id),
  created_at   timestamptz not null default now(),
  unique (package_id, waived_date)
);
```

This is the same three-question logic, just extended to three cases instead of two: attended → missed → *waived* (this table) → clinic-closed (the other table). The default, with nothing in either table, is exactly what you described: a missed day on a day the clinic was open is on the patient. `granted_by` gives you the accountability trail "on the owner's mercy" implies, and I'd gate writes to this table to `role = 'admin'`, same as everywhere else in Section 7's override philosophy — a receptionist shouldn't be able to grant these unilaterally.

One more thing this resolves for free: `status` flipping from `'Active'` to `'Expired'` doesn't need a nightly cron job checking every package. Nothing in this schema runs background jobs today, and I wouldn't start here — the same "row existing = truth, checked when it matters" instinct that already governs `visits`/`invoices` applies just as well: check whether a package's day-budget is exhausted at the moment someone tries to log a visit against it (or opens its detail page), and correct `status` right then if it's stale. No new infrastructure category needed.

`expiry_date` as a column can still exist, purely as a *display estimate* ("should be done around Aug 15") — but it shouldn't be the thing that actually gates anything. `days_used >= duration_days`, computed live, is the real authority.

---

## 4. Configurability — clinic default, patient hint, package fact

Default-plus-override is the right shape, and you don't actually have to design it from scratch — you've already built this exact pattern twice in this same schema, for a different attribute:

- `clinics.default_visit_type` — a clinic-level default that's *purely a pre-fill convenience*, "cheap, no enforcement needed, just a starting value" (that's your own Section 2 language).
- The Section 7 override table — component-scoped rows (`override_type`, `granted_by`, `expires_at`, `is_active`) rather than a single flag, specifically because a grant deserves its own reason and history.

Point 4 is the same two ideas, applied to package mode instead of price. I'd propose three layers, not two, and here's why a third layer earns its place:

```sql
-- Layer 1: clinic-wide default, pure pre-fill
alter table clinics add column default_package_mode text
  check (default_package_mode in ('DATE_BASED', 'SESSION_BASED'))
  not null default 'DATE_BASED';

-- Layer 2: optional patient-level hint, ALSO pure pre-fill (pending your answer to B above)
alter table patients add column default_package_mode_hint text
  check (default_package_mode_hint in ('DATE_BASED', 'SESSION_BASED'));
  -- nullable — most patients have no hint, fall through to the clinic default

-- Layer 3: the actual binding fact, decided once, at the moment this specific package is sold
alter table packages add column package_mode text not null
  check (package_mode in ('DATE_BASED', 'SESSION_BASED'));
```

The reason for three layers instead of collapsing patient-level into "the" answer: your friend's case wasn't really a trait of *that person* — it was a trait of *that situation* (a paralysis case that's genuinely hard to schedule around). If that same patient later buys a completely unrelated package for a sprained wrist, there's no reason it should inherit session-based mode just because one earlier package needed it. So I'd treat `packages.package_mode` as the only place the decision is actually *binding* — chosen at sale time, snapshotted, and from then on immutable for that package's lifetime, the same discipline this schema already applies to `consultation_fee_in_paise` ("snapshot of fee at time of visit") and every other denormalized snapshot column here. The patient-level hint is real and worth having — it saves the receptionist from re-remembering "oh right, this one's different" every time — but it's a convenience that pre-selects the dropdown, not a rule the dropdown is locked to.

That said — this is exactly where your answer to **B** changes the design. If you actually want it *binding* at the patient level (a receptionist genuinely cannot override it without a deliberate step), that's buildable too, but it needs a trigger validating `packages.package_mode` against `patients.default_package_mode_hint` at insert time, closer in spirit to `validate_owner_is_admin()` than to `default_visit_type`. Say the word and I'll design that version instead.

**On naming** — since you flagged it as something to get right: I'd go with `'DATE_BASED'` / `'SESSION_BASED'`, matching the CHECK-constraint style already everywhere in this schema (`'CONSULTATION'`, `'MACHINE_ONLY'`, `'WALKIN'`) and reading unambiguously to anyone grepping the schema cold in six months. A terser pair like `'CALENDAR'` / `'SESSION'` works too if you want less typing. Either way, this is genuinely an Om-call kind of decision, same as "consultation therapy" sitting as a placeholder name in Section 2 — I wouldn't lock it in without him seeing it.

One structural consequence worth flagging now rather than discovering later: `duration_days`, `expiry_date`, and `exclude_sundays` only mean something for `DATE_BASED` packages. A `SESSION_BASED` package needs a "sessions promised" number instead, which isn't the same thing as "days promised" even though it's tempting to reuse one column for both. This is the exact same shape as `visit_type`/`consultation_type` already in your schema — a value that's only required (and only *makes sense*) conditional on a sibling column. I'd follow that precedent directly rather than inventing something new:

```sql
alter table packages
  alter column duration_days drop not null,
  alter column expiry_date drop not null,
  add column sessions_promised integer,
  add constraint chk_package_mode_fields check (
    (package_mode = 'DATE_BASED' and duration_days is not null and expiry_date is not null)
    or
    (package_mode = 'SESSION_BASED' and sessions_promised is not null)
  );
```

This is also where your answer to **A** lands directly: if session-based really is meant to be open-ended forever with zero calendar pressure, this constraint above is complete as written — no expiry concept needed on that side at all. If there's supposed to be some outer boundary even for session-based packages, that's one more nullable date column and one more clause in the CHECK, not a redesign.

---

**Where this leaves things**, in the resolved/open shape your own DECISIONS.md and GAPS.md already use:

*Fairly confident, pending your answers above:* `visits.package_id` + the partial unique index on `linked_complaint_id`; `attended_days` computed at read time, not trigger-maintained; days-based (not date-mutation-based) tracking via `clinic_closures` + `package_day_waivers`; the three-layer `package_mode` design with snapshotting at sale time.

*Genuinely open, need your input:* whether session-based has any outer time boundary at all (A); whether the patient-level default should be binding or advisory (B); the final name for the two modes — Om's call.

None of this is written into `supabase_migration.md` yet — everything above is proposal-stage, same as Sections 5–7 of your README already are, not a decision until you say it's one.

# [18 Aug 2026] (Unresolved)

> **[COMPACTED PROMPT]**
> Reviewing our package architecture, I need clarification on five key areas and request that we begin maintaining a persistent decisions log: First, for attendance-based packages, can `expiry_date` remain optional (null for indefinite, but configurable if a clinic wants an outer ceiling)? Second, could you clarify what "binding" versus "advisory" defaults mean—since admins configure patient-level overrides while receptionists sell packages, should exceptions (like mode overrides, holds, or cancellations) be strictly admin-only or support an approval workflow where receptionists submit pending requests? Third, how do two proposed integrity constraints—allowing at most one visit per complaint per day and at most one active package per complaint course—help eliminate ambiguity in visit day counting and package drawdown? Fourth, please explain in plain English (without raw SQL) what the partial unique index on active packages does and how read-time counting operates. Finally, where did `exclude_sundays` originate, and how should we model weekly schedules, holidays, and mid-day emergency closures so that unvisited patients receive day waivers without penalizing patients who already attended that morning (and could a standalone visit later be converted into day 1 of a package)?

## 0.a — expiry for attendance-driven packages

Got it — revising: expiry stays *optional*, not absent, for that mode too. Null = indefinite (your friend's actual case), but a clinic can set one if it wants a ceiling even on a purely attendance-driven package. So the CHECK constraint from my last message needs loosening — `expiry_date` shouldn't be required for one mode and forbidden for the other, it should just be nullable under both, with the *other* field (`duration_days` vs. a sessions counter) being the one that's actually mode-conditional. I'll fold this into the design below.

## 0.b — "binding" vs. "advisory," explained, and where I actually land

Sorry, that was jargon without an example. Concretely: **advisory** means the setting only decides what shows up *pre-selected* on a form — the receptionist can still freely pick something else with zero friction, zero error, zero permission check. **Binding** means the database itself would *reject* a choice that contradicts the setting — you'd need a trigger checking it, the same way `chk_consultation_type` rejects a `MACHINE_ONLY` visit that also has a `consultation_type` set. Advisory = a pre-filled dropdown. Binding = a locked dropdown, enforced server-side even if someone bypasses the UI.

Now — your own reasoning while working through it: one correction, then I think you've actually already answered the real question yourself.

**The correction:** `visits.visit_type` (`'CONSULTATION'` vs `'MACHINE_ONLY'`) is a *per-visit billing model* — bundled clinician-directed therapy vs. itemized machine charges. That's not the same axis as the package attendance-mode we've been designing since your paralysis example (does the clock run on the calendar or on attendance). You called both of them "session based" at different points, which is exactly why they got tangled — and honestly, that's on me too, since I'm the one who named the package mode `SESSION_BASED` in the first place, which collides directly with how you already use "session" to mean a visit. I'm renaming it below to `VISIT_BASED` to kill that collision at the source rather than leaving a landmine in the schema.

**What you actually resolved:** the pattern you described — clinic default, admin can override per-patient, receptionist doesn't get to touch it — *is* the default+override pattern, and you named it correctly without needing me to explain "binding" first. Where you landed: admin-only to create a non-default, for both the (new) visit-type patient default *and* the package-mode/expiry override, same restriction already on Hold and Cancel/Refund. That's a clean, safe call.

On your actual open question — should any of these eventually get the approval-workflow treatment like the fee waiver — here's my real take, and it's more differentiated than a flat yes/no:

The test that actually matters isn't "how important is this decision," it's: **does it have to be decided right now to let something in progress finish, with no safe state to just sit in — or can it sit "requested, not yet acted on" without anything breaking or defaulting to the wrong thing?**

- **Package mode/expiry at the moment of a *new* sale — no safe pending state, structurally, not just as a rule.** The sale is happening at the counter. Either it waits for an admin (bad for business) or it gets created under a guessed mode and starts running under the wrong clock immediately. This one should stay admin-only-final even long-term — it's not that receptionists aren't trusted, it's that there's nothing safe to hand them a "pending" version of.
- **Hold and Cancel/Refund — actually do have a safe pending state**, and this is worth knowing even though Om's already settled them as admin-only in your README: while a request sits unactioned, the package just keeps running exactly as it was. Nothing forces a wrong decision in the meantime. That's the same shape as the fee waiver (request now, admin finalizes later, nothing broken between). So structurally, if Om ever wants to loosen "receptionist can't even *ask*," these two would support it cleanly — that's a separate question from whether he wants to loosen it now, which he hasn't, and I'm not suggesting you revisit that.
- **The new visit-type patient-default hint — lowest stakes of all of these**, since `visit_type` is already freely re-picked per visit regardless of what's pre-filled (already resolved in Section 2). A wrong pre-fill costs one extra click. Admin-only is fine and safe, but there's very little actually at risk even if that one specifically opened up.

## Item 1 — the two new constraints

Both are good, and I want to be precise about what each one actually fixes, since they fix *different* things.

**One visit per complaint per day** structurally dissolves your literal question — "does it increase visits or days" — because with that constraint, there can never be more than one visit for a given complaint on a given date. So "was there a visit that day" and "how many visits that day" become the same fact, permanently, by construction. There's no longer a version of reality where those two numbers could disagree. And it's not a new assumption on top of your fee model — it's *already implied* by how Section 2 describes same-day multi-complaint billing (one shared exam fee, one per-complaint therapy charge, per day) — this constraint just makes that implicit assumption something Postgres actually enforces instead of something the frontend has to remember not to violate.

**At most one active package per complaint at a time** — this is the same idea I'd already sketched as a unique index last message, just arrived at independently from your other conversation. Genuinely useful, and it changes my answer on `visits.package_id` in a way I want to be honest about rather than just declaring victory for my earlier recommendation.

With both constraints in place, the *concurrent* ambiguity — "which of several currently-active packages does this visit belong to" — mostly goes away, because there's at most one candidate at any moment. What it doesn't solve: reconstructing that fact *after time has passed*, once a package's status has moved on to Completed/Expired and a new one may have started. And that specifically matters here because we already decided a package's effective end isn't a fixed stored date — it's computed live, from closures and waivers that can get logged after the fact. A query trying to re-derive "was this old package active on that date" later is asking a question whose answer can shift depending on what closures got added afterward. So: both constraints are worth adding regardless, on their own merits — but `visits.package_id`'s job narrows rather than disappears. It's no longer "resolve the ambiguity" (the new constraints mostly do that now), it's "durably record the decision once it's made, so it can't be reinterpreted later by data that didn't exist yet at the time."

## Item 2 — that unique index, in plain English

Fair ask, I leaned on the SQL too hard last time. What it does: it tells Postgres "never let two rows in `packages` share the same linked complaint while both of them currently say `Active`." That's it. Old, finished packages don't count against it at all — you can have five retired packages against the same complaint, no problem. It only steps in the moment someone tries to create (or reactivate) a *second* currently-active one while a first is still running, and it rejects that outright rather than letting it happen silently. It's exactly the rule you described independently in item 1 — "at one time, only one active package is allowed for a complaint course" — just already written down as an actual enforceable thing rather than a sentence. Still valid, still worth adding.

# [18 Aug 2026] (Unresolved)

> **[COMPACTED PROMPT]**
> Regarding response formatting: I do want to see SQL queries, but ensure text explanations and tables are rendered as proper markdown rather than getting trapped inside code blocks, and always provide a brief explanation of what the SQL executes and why it is structured that way; please re-state the mid-day emergency closure mechanism legibly, explain where `exclude_sundays` originated, and keep our decisions log updated.

**The mid-day mechanism, re-sent legibly.** The core idea from before: a package's "day used" status gets decided by asking two independent, small questions about each date — did the patient actually get a visit logged that day? And separately, was that date marked as a clinic closure? If there's a visit, the day counts as attended, full stop, regardless of anything else that happened that day. If there's no visit and the date is marked closed, the day doesn't count at all. If there's no visit and the date is NOT marked closed, it counts as missed. Nothing in this depends on *when during the day* the closure got declared relative to anyone's visit — a patient seen at 10am before a 2pm emergency closure already has a visit row, so the closure declared later that afternoon never even needs to look at them. That's the whole trick that dissolves the "run midway" problem — it's genuinely already resolved, that section just landed unreadable.

**Where `exclude_sundays` actually came from.** Traced it — it's not something Om asked for in a business conversation. It's inherited straight from the original frontend TypeScript type (`PackageRecord.excludeSundays`), which already existed as a plain per-package boolean before any of the backend/schema work started. It got carried into the migration simply because the frontend already had the field, not because anyone deliberately decided "Sunday specifically deserves special treatment." That's worth knowing — it makes your instinct to question it a legitimate re-examination of an inherited default, not you second-guessing a real business decision.

**My actual take, since you asked for it directly:** don't remove it, generalize it. The real problem isn't Sunday — it's that Sunday is hardcoded as the *only* day a clinic can have a standing weekly closure on. A clinic that's closed Wednesdays instead, or not closed on any fixed day at all, has nowhere to say that today. I'd replace the boolean with a small list of which weekdays (if any) a given clinic is standardly closed on:

```sql
alter table clinics add column recurring_closed_weekdays integer[] not null default array[0];
```

What this does: it's a list of weekday numbers, using Postgres's own numbering where 0 is Sunday and 6 is Saturday — so `{0}` is today's current default (Sunday-only), `{}` (empty) means no standing weekly closure at all, and a clinic that's actually closed Wednesdays gets `{3}`. It's an array rather than a single value because a clinic closed on two fixed days a week is a real possibility and shouldn't need a schema change to express. This becomes one half of the calendar picture — the "always" half. `clinic_closures` from before stays as the other half — the "just this once" exceptions (festivals, emergencies). Together they cover both without hardcoding Sunday as special.

**Your specific new scenarios, checked against the design directly, not just asserted as covered:**

- *"Open only for new patients today"* — for package-tracking purposes this is functionally a closure for anyone with an active package, even though the doors are technically unlocked. Same mechanism: insert a row into `clinic_closures` for that date, with the reason field literally saying "new patients only." No new structure needed — the `reason` column already exists to carry exactly this kind of nuance.
- *"Doctor closes early after one hour"* — this is the exact case the two-question model was built for. Anyone already seen in that first hour keeps their attended day. Anyone not yet seen gets waived the moment that day gets marked closed, whenever that happens. Already correct.
- *"Individual patient, genuine emergency, waived on the owner's mercy"* — this is `package_day_waivers` from before, unchanged: one row, one date, one package, admin-gated.

**The "convert a logged visit into day 1 of a new package" idea** — noted as a future feature, not being designed right now, exactly as you flagged it yourself. It's in the log below so it doesn't get lost, not something I'm building out today.

Now, the log file.That file's already sitting there, complete and current — covers everything from the `package_id` reasoning through today's `exclude_sundays` trace and the admin-only test, matching everything just discussed. No need to rewrite it.That's everything — the mid-day mechanism restated cleanly, the `exclude_sundays` trace and the `recurring_closed_weekdays` proposal, all three of your new scenarios checked individually rather than just asserted, and the log file is up top with all of it plus the full thread from before. Keep going whenever you're ready on the rest.

# [19 Aug 2026] (Unresolved)

> **[COMPACTED PROMPT]**
> Please provide a deep, sequential architectural evaluation across four core package mechanics while keeping our log updated: First, explain in detail how `package_day_waivers` operates, what problem it solves, and why it is admin-gated; second, break down how `clinic_closures` works, why it was proposed, and how `packages.status` dynamically flips from 'Active' to 'Expired' at read/logging time without background cron jobs; third, provide a comprehensive trade-off comparison (usability, reliability, complexity, scalability) between pure manual tracking, `clinic_closures`, login-absence auto-detection, and daily staff login prompts (`closure_type`: FULL vs. NO_PACKAGE_PATIENTS) alongside `recurring_closed_weekdays`; finally, detail the approval workflow where receptionists sell packages at clinic defaults immediately (without blocking sales) while mode/expiry overrides sit as pending requests—addressing invoice/profile status rendering, whether pending cancellations or holds should block visit logging or day counting, and how configurable permissions for `visit_type` overrides should be handled.

Taking these one at a time, in order, the way you asked.

## 1. `package_day_waivers` — the full mechanics

**The problem it solves:** an exception that's about *one specific patient's* circumstances, not about the clinic's operating status. `clinic_closures` protects everyone at a branch on a given date — `package_day_waivers` protects one patient's one package on one date, while the clinic is otherwise running completely normally and everyone else's packages tick as usual.

**How it fits into the day-by-day computation:** with this table in the picture, the check for any given clinic-open day now has three questions instead of two, and they're checked in this order:

1. Was a visit actually logged against this package on that date? → **Attended.** This wins regardless of anything else — if they came in, it counts, full stop.
2. If not — is there a `package_day_waivers` row for this exact (package, date)? → **Waived.** Doesn't touch the day-budget at all, same treatment as a clinic-wide closure, but scoped to just this one package.
3. If neither → **Missed.** Consumes a day, on the patient — the ordinary default.

**Worked through concretely:** a patient has a 10-day `CALENDAR_BASED` package. On day 6, a genuine family emergency keeps them away. Normally that's just a missed day — the budget still gets consumed whether they showed up or not, that's the whole premise of calendar-based tracking. The admin decides this one deserves an exception. One row goes in: this package's id, that date, a reason, and who granted it. From then on, whenever that package's days-used gets recomputed, that date resolves to "waived" instead of "missed" — as if the clinic itself had been closed, but only for this one patient. Everyone else at the clinic that day has an ordinary Tuesday.

Why not just use `clinic_closures` for this instead? Because that would incorrectly protect *every* active package at that branch that day, not just this one patient's. Two tables exist because they're answering genuinely different questions — "was the clinic open" vs. "should this one patient's day count" — and collapsing them would mean either over-protecting everyone to help one patient, or under-building the mechanism entirely.

**Why it needs to stay admin-gated:** this is inherently a discretionary, no-formula judgment call — there's no rule that decides whether a given circumstance "deserves" an exception, a person has to decide that. Since it's discretionary and, left ungated, could be quietly abused (a receptionist waiving days for friends or family), requiring `granted_by` to actually be an admin — enforced the same way `validate_owner_is_admin()` already enforces ownership elsewhere in this schema — keeps every waived day answerable to a specific person's decision. And to be clear: nothing here is automatic. No day ever gets waived by inference — someone always has to actively decide it, which is the same "the app should never auto-flag" principle Om already stated for the Flag/Red Zone feature, just showing up again here.

## 2. `clinic_closures` — mechanics, purpose, and the status-flip explained properly

**How it works:** one row per (clinic, date) that's been declared closed, for any reason. It's the *second* question in the model above — but at the clinic level rather than the per-package level. If a date has a row here for a given clinic, then for *every* currently-active package at that branch, that date is excluded from the day-budget, regardless of who did or didn't show up (moot anyway, since the clinic wasn't running).

**Why it exists, concretely:** it came directly out of the mid-day-closure problem — you needed some way to record "this clinic wasn't operating, fully or partially, on this date" that could be declared reactively, any time during the day, rather than predicted in advance. The design goal from the start was "no calendar to maintain, just something to press when it's actually needed."

**Is it still proposed?** Yes — nothing's been built yet, and question 3 below is a real re-evaluation of whether it's still the right mechanism, not a rubber stamp. I'll get to that.

**Where it's used:** its only consumer is the days-used computation for packages. Nothing else in the schema reads it. The write side — whatever UI actually creates these rows — hasn't been designed yet; that's still open, and question 3's discussion feeds directly into what that UI should look like.

**Now, the status flip — how "Active" actually becomes "Expired," mechanically, since I described that too loosely before.** The idea is that `packages.status` stops being something that has to be kept continuously correct, and becomes something that gets corrected exactly at the two moments it actually matters:

- **The moment someone tries to log a new visit against this package.** Right before that write is allowed to go through, the live days-used gets computed using the three-question model above. If it turns out the budget is already exhausted — even though the stored `status` column still says `'Active'` because nobody's checked since — two things happen in that same moment: the stored `status` gets corrected to `'Expired'` right then, and the attempt to link this new visit to the now-expired package gets rejected (the receptionist would need to sell a new package, or bill this visit standalone — which one is a business call I haven't made for you, just flagging it needs one).
- **The moment anyone opens that package's detail view** — say, on a patient's profile. The same live computation runs, and if it disagrees with the stored `status`, the corrected value is what actually gets shown to the person looking at it, and the stored column gets opportunistically updated to match.

So `status` as a stored column is really more of a *cache of the last time anyone checked*, not a continuously-accurate value — the live computation is always the real authority, the two touchpoints above are just where that authority gets asked. Worth being honest about the one real gap this leaves: if a package quietly runs out and genuinely nobody ever tries to use it or look at it again, its stored status can sit at `'Active'` indefinitely — harmless for anything that actually matters (nothing depends on it being instantly accurate), but it would mean a report like "how many active packages does this branch have right now" could overcount slightly. Not worth solving preemptively — if that kind of reporting accuracy becomes a real need later, a periodic correction pass would close it, but I wouldn't build that until something actually needs it.

## 3. `exclude_sundays` / `recurring_closed_weekdays` — the real comparison you asked for

Let me actually stress-test all three approaches on the table, not just the one I originally proposed.

| | **A — Pure manual** (no tracking at all) | **B — `clinic_closures`** (what I proposed) | **C(i) — Auto-detect from login absence** (your new idea) | **C(ii)+(iii) — Onboarding prompt + close button** (your new idea) |
|---|---|---|---|---|
| **Usability** | Poor — every affected package needs a separate manual edit, every time | Good — one action protects every affected package at once | Looks effortless on the surface | Good — same protection as B, friendlier trigger |
| **Reliability** | Depends entirely on someone remembering | High — explicit, deliberate, unambiguous | Low, and I'll defend that below | High — same mechanism as B |
| **New infrastructure** | None | None | A scheduled job, plus logic to reconcile late/backdated entries | None — pure UI layer on top of B |
| **Scalability** | Fails outright at 3 branches with real package volume | Scales cleanly, one row per event, cheap lookups | Gets *worse*, not better, with more branches and staff | Scales exactly as well as B |
| **Build complexity** | Lowest (nothing to build) — but it just relocates the cost onto endless manual labor forever | Low–moderate | Highest of everything on this table, despite sounding simplest | Low — mostly front-end, reuses B's schema |

**Why I'd actually reject C(i) specifically**, since it's the interesting new idea and deserves real scrutiny rather than a wave-through: "nobody logged in" is a genuinely weak proxy for "the clinic was closed." A few realistic ways it misfires — an internet or power outage where the clinic still saw patients on paper and entered everything the next morning; a receptionist who just forgot their password that day while the clinic ran completely normally; a small operation where the clinician treats patients directly and someone else does data entry a day later. In every one of these, the clinic was genuinely open, but "auto-detect closed" would have already fired overnight, incorrectly protecting every active package that day — which is actually worse than doing nothing, because now you'd need extra reconciliation logic for "wait, backdated visits just arrived for a day I already marked closed," and that reconciliation logic is exactly the kind of complexity this whole design has been trying to avoid. It also genuinely requires a scheduled job to even evaluate "was there activity yesterday" — the one piece of new infrastructure everything else in this design has managed to avoid needing. So: sounds automatic, is actually the most fragile and the most work.

**C(ii) and C(iii) are a different story — genuinely good, and I'd adopt both.** They're not a different data model at all, just a much better way of *triggering* the same `clinic_closures` mechanism. A quick daily prompt when someone logs in ("clinic running today?", "taking package patients today?") does the exact same insert a manual button would, just with less friction to remember it exists. And "are you taking package patients today" as its own yes/no is sharper than what I had — it cleanly captures the "open, but not for package patients" case that I was previously just leaving to free text. Worth a small schema addition to make that distinction structural rather than buried in a text field:

```sql
alter table clinic_closures add column closure_type text
  check (closure_type in ('FULL', 'NO_PACKAGE_PATIENTS')) not null default 'FULL';
```

This just gives the closure a queryable category — `FULL` for a genuinely shut clinic, `NO_PACKAGE_PATIENTS` for "open, but package patients specifically aren't being seen today" — while `reason` stays as free-text color on top of that, same as before. The close-button-with-consequences idea (showing which active packages would be affected before confirming) is a good confirmation layer too, and it doesn't actually need new logic underneath — it's just a query against currently-Active packages at that clinic, displayed before the same insert happens.

**And to your root question — do we even need this, could it just be manual?** Given you're at 3 branches with ongoing package volume, Approach A genuinely doesn't hold up; it's not that it's "simpler," it just pushes all the work onto a person doing it by hand, forever, with nothing catching mistakes. But I don't think that's actually what B feels like in practice — one button press, one row, covers every affected package automatically. That's about as close to "manual" as a structured mechanism gets. I'd still keep `recurring_closed_weekdays` alongside it, specifically because the onboarding prompt only fires on days someone actually logs in — a clinic that's predictably closed every Sunday shouldn't need someone to log in just to say "no" on a day everyone already knows the answer to. `clinic_closures` handles the unpredictable one-offs; the recurring setting handles the known-in-advance pattern. Different jobs, same as before.

## 4. The approval workflow, refined

**Direct answer to your direct question: no, that's not what I was recommending, and I should've said it more precisely the first time.** Admin-only applies to *deviating from the default* — not to creating a package at all. A package sold at the clinic's default mode should be a completely normal, fast, receptionist-only action, exactly like any other sale today. Nothing about that should ever need an admin in the room.

**Your proposed workflow is genuinely the right fix, and it resolves something I'd flagged as unsolvable.** I'd said package-mode overrides have "no safe pending state" because the sale has to happen right now — but I was wrong to treat that as unsolvable, because the *default* mode itself is exactly that safe state. The package gets created immediately, at the default, a fully valid transaction on its own. A request to override sits as a separate, pending fact layered on top. If approved, it changes one or two columns on the existing row. If not, nothing changes and the package simply stays at whatever it already validly was. That's the same shape as the visit-fee waiver, and I should have seen that the first time rather than treating packages as categorically different.

**One real wrinkle worth naming, though — specific to a *mode* change, not an expiry-only change.** An expiry adjustment is trivial to reconcile retroactively; it's just a boundary, and moving it doesn't require re-litigating anything that already happened. A *mode* switch is trickier, because days may have already ticked under the old mode's rules while the request sat pending — say three calendar days passed, one of them logged as missed under `CALENDAR_BASED` rules, before an admin approves the switch to `VISIT_BASED`. Does that missed day still count once the package is `VISIT_BASED`, where "missed" isn't even a concept? I'd propose the simplest resolvable rule: **the switch takes effect from the moment of approval, going forward — whatever already happened to the package under the old mode stays exactly as recorded.** No retroactive re-interpretation of the past. This needs to actually be decided, not left implicit, but I think that's the right default and it's simple enough to explain to anyone looking at a package's history later.

**The invoice display — yes, straightforward, and it should show up in one more place too.** The invoice can show the current governing terms (mode, expiry) plus, if there's a pending request, a "pending approval — if approved, becomes: X" note underneath. That's a simple conditional render off whatever status field the request carries. The one thing I'd add to your instinct: it's probably not *just* the invoice. Staff looking at that patient's package later — on their profile, days after the sale — would want to see "pending override requested" too, not just at the moment of purchase. Same display logic, one more place it needs to show up, not a new mechanism.

**Blocking actions on a pending package — I'd split this rather than apply one rule everywhere, because your instinct is right for one case and doesn't quite land for another.**

- *Cancel/Refund pending:* agreed, block new visits from drawing against this package while it's pending. If the cancellation gets approved, retroactively un-doing "we already gave two free sessions to a package that just got cancelled" is genuinely messy — better to just not let that happen in the first place.
- *Hold pending:* this is where blocking doesn't actually do what you want it to. The whole point of a hold request is protecting upcoming days from being marked missed — but "block new visits" doesn't touch day-*counting* at all, since days get marked missed by the calendar ticking, not by someone trying to log a visit. Blocking visits here doesn't protect anything; it just adds friction with no actual effect on the thing you're trying to prevent. There's a genuine tradeoff underneath this one, and I'd rather hand it to you straight than pick quietly: either (a) a hold request takes *immediate, provisional* effect the moment it's submitted — which does protect the days in real time, but means a receptionist can unilaterally pause a package's clock pending review, a looser standard than "nothing happens until an admin acts," or (b) nothing happens while pending, and if approved, the admin's approval retroactively backdates the waiver across however many days were pending — which keeps the strict "receptionist can't change anything alone" rule intact, at the cost of the protection not being real-time. I'd lean toward (b), mainly because it's consistent with the exact precedent Section 7 already set for the fee waiver — but this one's genuinely close enough that it's worth you actually deciding rather than me picking for you.

**The visit-type toggle — worth flagging directly: this isn't a new idea, it's one you already looked at and set aside.** README Section 2 already considered exactly this — "a 'lock' concept... was considered and set aside for now... add it later, as a single ordinary RLS policy, if and when a specific clinic asks for it." What you just described is that same lock, phrased as a configurable permission instead of a hard rule. I don't think anything's changed that would argue for building it now — no clinic has actually hit this friction yet — so I'd treat this as the same already-deferred item resurfacing, not a new one, and leave it deferred unless a real need shows up. If it ever does, the shape is simple: one boolean on `clinics` gating whether receptionists can deviate from the default at all.

---

Updating the log with all of this now.