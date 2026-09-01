# Packages & Subscription Model — Design Log

**Scope:** Everything discussed about `packages`, package attendance tracking, and the package-mode configurability system, across the "Packages" branch conversation and this session. Chronological, append-only in spirit — same as `LOGS.md`: an item appearing here means "discussed and reasoned through," not automatically "final." Items Bro/Om have explicitly confirmed are marked as such in the summary at the bottom; everything else is proposal-stage, same status as README.md Sections 5–7.

**Why this file exists:** requested directly — chat history isn't reliably scrollable or persistent; this is.

---

## 1. Starting point — confirmed against the live migration file, not memory

- `packages` carries both `clinic_id` (location scoping, validated against `patient_clinic_access` — the standard tenancy pattern every location-specific table gets) and `linked_complaint_id` (FK to `complaint_courses`) — different jobs, both needed.
- `duration_days`, `attended_days`, `missed_days`, `expiry_date` are all plain stored values. Not generated, not trigger-derived.
- `packages` has exactly two triggers: one stamps `updated_at`, the other (`packages_clinic_id_guard`) validates `clinic_id`/`patient_id` against `patient_clinic_access` and has nothing to do with attendance.
- `visits` has **no `package_id` column** — no FK, no trigger, nothing connects a specific visit to a specific package.
- Consequence: nothing today would catch two visits logged against one package while `attended_days` only got bumped once (it's never actually incremented by anything), or a deleted visit whose count never decremented.

## 2. Why `visits.package_id` earns its place, despite the existing `complaint_course_id` link

Challenge raised: visits and packages already both point at `complaint_courses`. Why add a third link?

Confirmed: `packages.linked_complaint_id` has **no `unique` constraint** — a single `complaint_course_id` can have more than one `packages` row, today, with zero schema change. That's the concrete reason `complaint_course_id` alone can't safely stand in for a direct link:

- **Sequential packages.** A long-running complaint can have Package 1 (used up), then Package 2, both against the same `complaint_course_id`. Deriving "which package does this visit count against" from `complaint_course_id` alone becomes ambiguous the moment that happens.
- **Historical reconstruction.** Even resolving the ambiguity live, answering "which visits actually counted against Package 1, specifically" later, after status has moved on, can't be reliably reconstructed — especially once the package's effective boundary is a *live computation* involving closures/waivers logged after the fact (§5), not a fixed stored date. A derived join re-asks the question later using whatever data exists *then*, which can have shifted underneath it.

**Resolution:** keep `linked_complaint_id` (the clinical relationship, a different fact). Add:
```sql
alter table visits add column package_id uuid references packages(id);
```
— nullable, the durable record of "this visit was decided, at logging time, to consume a day of this specific package."

**`attended_days`:** recommend computing at read time (`select count(*) from visits where package_id = $1`) rather than a trigger-maintained counter. The `invoice_counter` pattern doesn't actually transfer here — that pattern exists specifically for atomicity under concurrent writes (two invoices can't get the same number), which a plain count has no need for. A trigger-maintained counter would also need to correctly handle insert, update (package reassigned), and delete to avoid drifting — more surface area, in a project that's deliberately deferred a full test suite. A live count is correct by construction and self-corrects automatically.

## 3. Two new integrity constraints

**(a) One visit per complaint per day:**
```sql
alter table visits add constraint uq_one_visit_per_complaint_per_day
  unique (complaint_course_id, date);
```
Not a new assumption — formalizes something Section 2's fee-restructure design already implies (one shared exam fee per day, one per-complaint therapy charge per day — already assumes one touchpoint per complaint per day). Resolves "does getting treated twice in a day increase the visit-count or the day-count" by construction: with this constraint, there can never be more than one visit per complaint per day, so those two numbers can never disagree.

**(b) At most one active package per complaint at a time:**
```sql
create unique index uq_one_active_package_per_complaint
  on packages (linked_complaint_id)
  where status = 'Active';
```
Plain-English version: Postgres will never allow two `packages` rows to share a `linked_complaint_id` while both currently say `Active`. Retired (Completed/Expired) packages don't count against it — five old packages against one complaint is fine. It only blocks a *second currently-active* one from coexisting with a first.

**Does (b) replace `visits.package_id`?** Substantially shrinks the need, doesn't eliminate it. With at most one Active package per complaint, concurrent ambiguity (which of several simultaneously-active packages) mostly disappears. What it doesn't solve: reconstructing history *after* a package's status has moved on, given the package's effective end is a live computation (§5), not a fixed date. Both constraints are worth adding regardless, on their own merits — `visits.package_id`'s job narrows from "resolve which package" (now mostly handled by (b)) to "preserve an accurate record of the decision once made."

## 4. Two package modes; naming collision caught and fixed

- **`CALENDAR_BASED`** (was informally "date-based"): the clock runs on elapsed clinic-open days, attendance or not. A missed open day is a loss, on the patient.
- **`VISIT_BASED`** (was informally "session-based"): the clock only advances on actual attendance. No calendar pressure.

Current schema (`duration_days`, `expiry_date`, `exclude_sundays`, `missed_days`) is built entirely for `CALENDAR_BASED`. Zero existing support for `VISIT_BASED`.

**Naming collision caught mid-conversation:** "session-based" was being used for two unrelated things — the package attendance-mode above, and `visits.visit_type = 'CONSULTATION'` (doctor-directed, clinician's choice of machines, bundled fee — already resolved in Section 2), which also got called "session based" informally. Renamed the package-mode values away from "session" entirely to kill the collision at the source, rather than leave two different concepts sharing a name.

**Expiry for `VISIT_BASED` packages:** confirmed — should stay *optional*, not absent. Real example (a paralysis case, weekly attendance, no hard limit) had none, but the feature should let a clinic set one if it wants a ceiling. Null = indefinite either way.

## 5. Days, not dates; the closure mechanism; `exclude_sundays` re-examined

**Resolved direction:** compute days-used live rather than storing/mutating an `expiry_date`. A mutable stored `expiry_date` would force a bulk update across every active package whenever the clinic closes, and forces that update to care about *timing* — was a given patient already seen before the closure was declared? A live computation sidesteps this: for each clinic-open day, ask two independent, order-agnostic questions:

1. Was there a visit logged that day? → **attended.** Counts, no matter what happens afterward that day.
2. If not — is that date marked as a closure? → **doesn't count at all.** Otherwise → **missed**, on the patient.

Neither question depends on *when* during the day anything happened, so a same-day "doctor had to leave early" case resolves correctly without any special-case logic: patients already seen that morning keep their attended day regardless of an afternoon closure; patients not yet seen get waived by the closure regardless of what time it was declared.

Consequence worth naming explicitly: under this design, a package is never "extended." The day-budget (say, 10 clinic-open days) never changes. What changes is only how many *calendar* days it takes to use up that budget, since closed days never draw from it in the first place — two Sundays inside a 10-open-day window just means it naturally takes 12 calendar days, not 10, with nothing being pushed or mutated anywhere.

```sql
create table clinic_closures (
  id uuid primary key default gen_random_uuid(),
  clinic_id uuid not null references clinics(id),
  closure_date date not null,
  reason text,
  created_by uuid references profiles(id),
  created_at timestamptz not null default now(),
  unique (clinic_id, closure_date)
);
```
Reactive, not a maintained forward calendar — a row exists only for a day *actually* declared closed, inserted whenever that happens (9am or 2pm, doesn't matter). Covers full closures, "new patients only today," and any other clinic-wide reason — the `reason` field documents which.

Patient-level version, for individual waivers ("on the owner's mercy"):
```sql
create table package_day_waivers (
  id uuid primary key default gen_random_uuid(),
  package_id uuid not null references packages(id),
  waived_date date not null,
  reason text,
  granted_by uuid references profiles(id),
  created_at timestamptz not null default now(),
  unique (package_id, waived_date)
);
```
Recommended admin-gated (`granted_by` must be `role = 'admin'`), same discipline as Section 7's override philosophy.

`status` transitions (Active → Expired) don't need a background/cron job — check and correct lazily, at the moment someone tries to log a visit against the package or opens its detail view. Matches the "row exists = truth, checked when it matters" MVP philosophy already used for `visits`/`invoices`.

**`exclude_sundays`, traced:** comes from `PackageRecord.excludeSundays` in the pre-existing frontend TypeScript type (per `schema_cross_reference.md`) — inherited because the schema-design process matched what the frontend mock data already had, not because Om asked for it in a business conversation. Legacy carry-over, not a deliberate decision.

**Recommendation: generalize, don't remove.** The actual problem isn't Sunday — it's that Sunday is hardcoded as the *only* day a clinic can have a standing weekly closure on. A clinic closed Wednesdays instead currently has nowhere to express that. Proposed: replace the boolean with a small recurring-weekday set per clinic (`clinics.recurring_closed_weekdays`, e.g. empty for a 7-day clinic, `{0}` for Sunday-only as today's effective default), kept separate from `clinic_closures`. Two complementary mechanisms — "always, every week" vs. "just this one date" — rather than one hardcoded special case standing in for both.

**Scenario check (all confirmed to already resolve correctly under the two-question model above):**
- Clinic runs Sundays, no standing closure → Sunday is an ordinary open day, nothing special happens.
- "Open only for new patients today" → a `clinic_closures` row for that date, `reason` documents why; functions as a closure for anyone with an active package.
- Doctor closes early after limited hours → already-seen patients keep their attended day; not-yet-seen patients get waived — resolved by fact 1 vs. fact 2 above, no timing/ordering logic needed.
- Individual patient emergency → `package_day_waivers`, one row, admin-granted.

## 6. Admin-only vs. approval-workflow — which operations, and why

Question: for package-mode/expiry (chosen at sale), Hold, and Cancel/Refund — strictly admin-only, or eligible for the same request-then-approve pattern already built for visit fee waivers (README §7)?

**The actual test:** does the decision have to be made *right now* to let an in-progress transaction complete, with no safe state to sit in while waiting — or can it sit "requested, not yet acted on" without blocking anything or defaulting to something wrong?

- **Package mode/expiry at the moment of a *new* sale — no safe pending state, structurally.** The sale is happening at the counter now; either it waits for an admin (bad for business) or it's created under a guessed mode, running under the wrong clock immediately. Stays admin-only-final, not just as a conservative default but because there's no safe "pending" version of this to hand a receptionist. *(Superseded by §11 below — the default itself turns out to be exactly that safe pending state; corrected there.)*
- **Hold and Cancel/Refund — do have a safe pending state.** While a request sits unactioned, the package just continues exactly as it was — nothing forces an immediate wrong decision. Same shape as the fee waiver. Currently admin-only per Om's existing call (README §1), staying that way — but structurally capable of an approval-workflow later if that restriction is ever loosened. Separate question from whether Om wants to loosen it now (he hasn't).
- **New: a visit-type per-patient default hint** (mirrors `clinics.default_visit_type`) — lowest stakes of all of these, since `visit_type` is already freely re-picked per visit regardless of any default (§2, already resolved). A wrong pre-fill costs one click, nothing downstream commits to it.

## 7. Three-layer configurability for package mode

```sql
alter table clinics add column default_package_mode text
  check (default_package_mode in ('CALENDAR_BASED', 'VISIT_BASED'))
  not null default 'CALENDAR_BASED';

alter table patients add column default_package_mode_hint text
  check (default_package_mode_hint in ('CALENDAR_BASED', 'VISIT_BASED'));

alter table packages add column package_mode text not null
  check (package_mode in ('CALENDAR_BASED', 'VISIT_BASED'));
```
Same pattern already used for `clinics.default_visit_type` (pre-fill, no enforcement) and Section 7's per-patient fee overrides (component-scoped, admin-granted, reason tracked) — not a new mechanism.

`packages.package_mode` is the only place the decision actually binds — snapshotted once, at sale time, matching the schema's existing "snapshot at time of decision" convention (`consultation_fee_in_paise`, `complaint_name`, every denormalized snapshot column here). The patient-level hint is deliberately *not* a standing binding rule — a mode chosen for one hard-to-schedule complaint shouldn't silently apply to an unrelated later package for the same patient.

Conditional CHECK, same shape as `chk_consultation_type`:
```sql
alter table packages
  alter column duration_days drop not null,
  add column visits_promised integer,
  add constraint chk_package_mode_fields check (
    (package_mode = 'CALENDAR_BASED' and duration_days is not null)
    or
    (package_mode = 'VISIT_BASED' and visits_promised is not null)
  );
-- expiry_date stays nullable under both modes (§4) — optional ceiling either way, not mode-gated
```

## 8. `package_day_waivers` — full mechanics

Solves a different problem than `clinic_closures`: an exception about *one patient's* circumstances, not the clinic's operating status. Slots in as a third question in the per-day check, in priority order: (1) was a visit logged that day? → attended, wins regardless of anything else; (2) if not, is there a waiver row for this exact package+date? → waived, doesn't touch the budget; (3) otherwise → missed, on the patient. Only ever relevant on days the patient didn't attend — a waiver is never needed or checked on a day they showed up.

Deliberately not automatic — always requires someone to actively decide it, same "app never auto-flags" principle already established for the Flag/Red Zone feature (README §1). Admin-gated (`granted_by` must be `role = 'admin'`, enforced the same way `validate_owner_is_admin()` already works) specifically because this is discretionary and, left ungated, could be quietly abused.

Distinct from `clinic_closures` on purpose: a closure row protects *every* active package at a branch that date; a waiver protects *one* package. Using a closure to solve a one-patient problem would over-protect everyone else; the two tables exist because they answer genuinely different questions.

## 9. `clinic_closures` — full mechanics, and how the status flip actually works

One row per (clinic, date) declared closed, for any reason. Second question in the per-day check, at the clinic level rather than per-package — if a date has a row here, every currently-active package at that branch excludes that date from its budget, no exceptions.

**The `status` flip (Active → Expired), made concrete.** `packages.status` isn't kept continuously correct by a background process — it's checked and corrected at exactly two touchpoints:
1. **The moment someone tries to log a new visit against a package.** The live days-used gets computed right then; if the budget's already exhausted, `status` gets corrected to `'Expired'` in that same moment, and linking the new visit to this package gets rejected (open question: does that visit then bill standalone, or does the flow force a new package sale first — not decided).
2. **The moment anyone opens the package's detail view.** Same live computation; if it disagrees with the stored value, the corrected value is what displays, and the stored column gets opportunistically updated to match.

`status` is effectively a cache of the last time anyone checked, not a continuously-accurate value — the live computation is always the real authority. Known, accepted gap: a package nobody ever revisits can sit at a stale `'Active'` status indefinitely. Harmless for anything that actually depends on it (both touchpoints above self-correct exactly when it matters), but could make an "active packages right now" report overcount slightly. Not worth solving until that kind of reporting is an actual need.

## 10. `exclude_sundays` / recurring closures — compared against two new alternatives, one rejected, one adopted

Three approaches evaluated head-to-head (usability, reliability, new infrastructure, scalability, build complexity):

- **Pure manual, no tracking mechanism at all** — rejected. Fails outright at 3 branches with real package volume; every affected package needs a separate manual edit every single time, with nothing catching a missed one.
- **`clinic_closures` + `recurring_closed_weekdays`** (§5/§9 above) — confirmed as the schema-level mechanism. One action protects every affected package; scales cleanly; no new infrastructure category.
- **Auto-detect closure from absence of any staff login that day** — evaluated and rejected. "Nobody logged in" is a weak proxy for "clinic was closed" — an outage with paper records and backdated entry, a forgotten password on an otherwise normal day, or delegated next-day data entry would all misfire, incorrectly protecting packages on days real sessions happened. Would also require the one piece of infrastructure this design has otherwise avoided (a scheduled job to evaluate "was there activity yesterday"), plus reconciliation logic for backdated entries arriving after an auto-closure already fired. Sounds automatic, is actually the most fragile and most work of any option considered.
- **Daily onboarding prompt ("clinic running today?" / "taking package patients today?") + a close-button showing affected active packages before confirming** — adopted as a UI layer on top of `clinic_closures`, not a different data model. Fires the same insert, just with a friendlier trigger than remembering a button exists. The "taking package patients" question is sharp enough to warrant a structural column rather than leaving it in free text:

```sql
alter table clinic_closures add column closure_type text
  check (closure_type in ('FULL', 'NO_PACKAGE_PATIENTS')) not null default 'FULL';
```

`reason` stays as supplementary free text on top of this. `recurring_closed_weekdays` stays too, specifically because the daily prompt only fires on days someone logs in — a predictably-closed-every-Sunday clinic shouldn't need a login just to confirm what's already known.

## 11. Approval workflow, refined — create-at-default resolves the "blocks the sale" problem

**Correction to §6:** admin-only applies to *deviating from the default*, not to package creation itself. Selling a package at the clinic's default mode is an ordinary, fast, receptionist-only action, never blocked, exactly like today.

**Adopted workflow for mode/expiry overrides:** the package is created immediately at the default — a fully valid, complete transaction on its own. An override request sits as a separate pending fact layered on top, changing one or two columns if and when approved. This is the same shape as the Section 7 fee waiver, and it resolves what §6 incorrectly flagged as an unsolvable "no safe pending state" — the default itself is that safe state.

**Reconciliation rule needed for mode changes specifically** (not expiry-only changes, which are trivial to adjust retroactively): a mode switch takes effect from the moment of approval, going forward. Whatever already happened to the package under the old mode during the pending window stays exactly as recorded — no retroactive re-interpretation of days that already ticked.

**Display:** invoice shows current governing terms plus, if a request is pending, "pending approval — if approved, becomes: X." Same conditional display also belongs on the package's view within the patient profile, not just the invoice, since staff will look at it again after the sale.

**Blocking while pending — differentiated by case, not a single blanket rule:**
- *Cancel/Refund pending:* block new visits from drawing against the package. Retroactively undoing "already gave free sessions to a since-cancelled package" is worse than just not allowing it.
- *Hold pending:* blocking visit-logging doesn't actually protect anything, since days get marked missed by the calendar ticking, not by visit attempts. Real open tradeoff, not resolved here: (a) hold takes immediate provisional effect on request — protects days in real time, but lets a receptionist unilaterally pause a clock pending review, a looser standard than the rest of this system uses; or (b) nothing happens until approved, and approval retroactively backdates a waiver across the pending window — keeps the strict "receptionist can't change anything alone" rule intact, at the cost of no real-time protection. Leaning toward (b) for consistency with the Section 7 precedent, but this is close enough to be Bro/Om's call, not decided here.

**Visit-type lock/permission toggle — not a new item.** Same "lock" concept README §2 already considered and explicitly set aside ("add it later... if a specific clinic asks for it"). No new need has actually surfaced — treating this as the same already-deferred item resurfacing, still deferred.

---

## Resolved (Bro/Om explicitly confirmed)
- `visits.package_id`, nullable — add it.
- `attended_days` computed at read time, not trigger-maintained.
- One visit per complaint per day — add the unique constraint.
- At most one active package per complaint at a time — add the partial unique index.
- Days-based (not stored-date-mutation-based) tracking; `clinic_closures` reactive table, not a maintained calendar.
- `VISIT_BASED` packages can optionally carry an expiry date too — null means indefinite.
- `exclude_sundays`: keep the concept, generalize into `clinics.recurring_closed_weekdays`.
- `package_day_waivers`: three-question priority model (attended > waived > missed), admin-gated, never automatic.
- `status` flip (Active → Expired): lazy, corrected at two touchpoints — attempted visit-linking, and the package detail view — not a background job.
- Closure auto-detection from login absence: rejected — unreliable signal, would need a scheduled job and backdating reconciliation this design otherwise avoids entirely.
- Onboarding prompt + close-button-with-consequences: adopted as a UI layer over `clinic_closures`, no new data model needed beyond `clinic_closures.closure_type` (`FULL` / `NO_PACKAGE_PATIENTS`), added to support it structurally.
- Admin-only clarified: applies to *deviating from default*, not to package creation at all — a default-mode sale is always immediate and receptionist-only, never blocked.
- Create-at-default-then-request-override adopted for mode/expiry overrides — resolves the "no safe pending state" problem §6 originally flagged.
- Mode-change reconciliation rule: an approved switch applies from the moment of approval forward; days already recorded under the old mode stay as they were.
- Pending-override display belongs on both the invoice and the package's view on the patient profile, not just one or the other.
- Cancel/Refund pending: block new package-linked visits while unactioned.
- Visit-type lock/permission toggle: not new — same item README §2 already deferred, still deferred.

## Open — needs Bro/Om's input
- Final naming for `CALENDAR_BASED` / `VISIT_BASED` (and `duration_days` vs. `visits_promised`) — Om's call.
- Whether `patients.default_package_mode_hint` should ever be enforced/binding rather than purely advisory.
- New item: a patient-level default hint for `visit_type` — raised in passing, not designed yet.
- **Hold pending — genuine unresolved tradeoff:** immediate provisional protection (receptionist-initiated, admin reviews after the fact) vs. nothing happens until approved, then retroactively backdated (stricter, but not real-time). Leaning toward the latter for consistency with the Section 7 precedent, not decided.
- What happens to a visit whose package-link gets rejected because the package expired at the moment of write — bill it standalone, or force a new package sale first?

## Deferred (explicitly "later," not being designed now)
- Converting an already-logged standalone visit into day 1 of a newly-purchased package.