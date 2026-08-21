# Handover — Packages (Attendance Tracking & Package Derivation)

**Cross-file dependencies — read these before deep-diving here:**
- The four-bucket money-column names (specifically why `therapy_fee_in_paise` is the field being zeroed by the package trigger) → `handover-fee-computation.md`
- The override column shape (`computed_refund_in_paise`, `final_refund_in_paise`, override_status, etc. on packages) → `handover-pricing-integrity-overrides.md`

**Full-detail reference:** `README.md` §1 (Package Management Features), §8 (Packages — Attendance Tracking), §9 (Package Derivation Mechanism)

---

## The Core Finding — Verified Against the Real Migration File

`packages` has exactly two triggers: one stamps `updated_at`, the other (`packages_clinic_id_guard`) validates `clinic_id`/`patient_id` against `patient_clinic_access` — the standard tenancy check. Neither has anything to do with attendance.

`duration_days`, `attended_days`, `missed_days`, and `expiry_date` are all **plain stored values** — not generated, not trigger-derived. `visits` has **no `package_id` column at all** — no FK, no trigger, nothing anywhere connects a specific visit to a specific package. `attended_days` / `missed_days` / `day_log` are entirely app-maintained counters with zero structural connection to real visit records. Nothing in the database would catch two visits logged against one package while `attended_days` only got bumped once, or a deleted visit whose count never decremented.

---

## Refinement Priorities

### Priority 1 — Add `visits.package_id` (The Core Fix)

```sql
ALTER TABLE visits
ADD COLUMN package_id uuid REFERENCES packages(id);
```

Nullable — plenty of visits aren't against a package at all.

Once it exists, `attended_days` stops being a number the app has to remember to update correctly, and becomes something a trigger keeps correct.

**Why not derive transitively through `complaint_course_id`:** worth being precise about, because the instinct is reasonable. `visits` and `complaint_courses` are NOT one-to-one — they're many-to-one (many visit rows accumulate under one complaint course over its lifetime, which is the entire point of a complaint course). A transitive join (`packages → complaint_courses → visits`) is technically possible, but a single complaint course can outlive more than one package over time (one expires, a second gets purchased for the same ongoing complaint). So the join alone would credit every visit ever logged under that complaint course to whichever package is being queried — including visits from before that package existed. It would need an additional date-range filter to be correct — more query-time work than a direct `WHERE package_id = X`, not less.

### Priority 2 — The `package_id` Derivation Trigger (Server-Side)

In a `BEFORE INSERT` trigger on `visits`, whenever a visit would otherwise carry a nonzero `therapy_fee_in_paise`:

```sql
SELECT id INTO v_package_id
FROM packages
WHERE linked_complaint_id = NEW.complaint_course_id
  AND status = 'Active';
```

Given the partial unique index below (at most one active package per complaint course), this is a clean, unambiguous single lookup, not a guess.

**If found:**
- Set `NEW.package_id` to that package's id
- Zero out `NEW.therapy_fee_in_paise` — the package already paid for this session; charging the per-visit rate on top would be double-billing
- Increment that package's `attended_days` in the same trigger (same discipline as `clinics.invoice_counter` — one write, not a separate step to remember)

**If not found:**
- `package_id` stays NULL
- `therapy_fee_in_paise` gets its normal computed value — an ordinary pay-per-visit encounter

**Exam fee and lapse penalty are unaffected by package status** — they are independent flat charges, not something a package covers. A patient with an active package still pays the 350 exam fee or the 150 lapse penalty as applicable.

**Why backend, not frontend:** if a client could submit a `package_id`, nothing would stop it pointing at a package belonging to a different complaint entirely — free therapy that should have been charged, the same class of hole as trusting a client-sent price. Deriving it mechanically from `complaint_course_id` (already trusted via existing tenancy checks) closes that off completely. The client never supplies `package_id` at all.

### Priority 3 — Active-Package Uniqueness Constraint

Nothing currently stops two `packages` rows with `status = 'Active'` existing for the same complaint course simultaneously. Fix:

```sql
CREATE UNIQUE INDEX idx_one_active_package_per_complaint
ON packages(linked_complaint_id)
WHERE status = 'Active';
```

Unlimited historical (Completed/Expired) packages per complaint course allowed over time; at most one Active at once. This is what makes the Priority 2 lookup a clean single-row result rather than an ambiguous multi-row one.

### Priority 4 — `missed_days` (Deferred — Needs Workflow Clarity)

A missed day has no visit row to count — it can't be trigger-derived the same way as `attended_days`. Better computed at read time (elapsed eligible days since `purchase_date`, adjusted for `exclude_sundays` if applicable, minus `attended_days`) than stored and incremented.

**Blocker:** depends on exactly how "missed" gets decided in the real workflow — only once a day is unambiguously past with nothing logged, or can staff mark a day missed in advance? Not clear enough yet to commit to a design. Parked until the `visits.package_id` link is in place and the real workflow is confirmed.

### Priority 5 — `day_log` (Deferred — Pick One Source of Truth)

`day_log` is a JSONB column on `packages` meant to hold a day-by-day calendar (attended / missed / upcoming). No trigger populates it. Three separate things (`attended_days`, `missed_days`, `day_log`) all claim to represent the same underlying fact — which days this patient actually attended — with nothing keeping them in agreement.

Long-term resolution: pick one source of truth (`day_log` as the real record, the two counters becoming cached aggregates derived from it) rather than three peers that can silently disagree. Not enough detail yet on how `day_log` actually gets populated and read in the frontend to propose a concrete fix responsibly. Deferred until the `visits.package_id` link is established and attendance tracking is being actively built.

### Priority 6 — Backstop Constraints (Low Priority, Add After Derivation Is Settled)

```sql
-- Prevents attended_days from exceeding duration_days or going negative
ALTER TABLE packages ADD CONSTRAINT chk_attended_days_valid
CHECK (attended_days >= 0 AND attended_days <= duration_days);

-- Catches an obvious data-entry mistake
ALTER TABLE packages ADD CONSTRAINT chk_expiry_after_purchase
CHECK (expiry_date >= purchase_date);
```

A backstop, not a fix on its own — only useful after the trigger is in place so these can't be hit by normal operation.

---

## Additional Constraint Gaps (Same Spirit)

- **A complaint course's `status = 'Active'` should be checked before a new visit can be logged against it.** Already flagged in the earlier module gaps log. Nothing currently stops a visit being added to an already-`'Completed'` complaint course.
- **`patient_rate_overrides` uniqueness** (lives in pricing-integrity domain but related): `CREATE UNIQUE INDEX ON patient_rate_overrides(patient_id, override_type) WHERE is_active;` — stops a patient from having two simultaneously-active overrides of the same type. See `handover-pricing-integrity-overrides.md` for the full `patient_rate_overrides` design.

---

## Package Management Features

### Hold

Admin/owner-only. The record should not carry an explicit "on hold" flag or duration — attendance reads as separate date-range segments in the history (e.g., "first 6 days: Jul 1–6," "remaining 4 days: Aug 9–12"), and the gap between segments is how a hold reads implicitly. Current `packages` columns don't represent multiple discontinuous active segments today — this needs its own design pass, not solved here. Deferred.

### Cancel + Refund

Admin/owner-only.

**Formula (resolved):** refund = package total − (days used × the **regular, non-discounted** rate). Confirmed by Om directly — deducts at the regular rate, not the package's discounted per-day rate. The source transcript's ₹810 worked example (which had deducted at the discounted rate) was incorrect; the "discount forfeited on early exit" rule text was the accurate read.

**Button label:** "Cancel Package" — matches the owner's own term, and reads more accurately than "Refund" alone since the action is terminating the package, of which the refund is a consequence.

**Schema prerequisite:** `'Cancelled'` must be added to `packages.status CHECK (... 'Active'/'Completed'/'Expired')` before a cancellation state can be recorded.

**Override columns on `packages`** (the refund equivalent of the four columns on `visits` — see `handover-pricing-integrity-overrides.md` for the full pattern):
- `computed_refund_in_paise` — backend-derived from the confirmed formula
- `final_refund_in_paise` — nullable override (if the admin adjusts the refund amount)
- `override_reason`, `override_by`, `override_status` — identical to the visits pattern

**Refund preview:** clicking "Cancel Package" calls the same backend function that finalizes the cancellation, purely to preview (writing nothing). Confirming calls that identical function for real. Same code path both times — preview and reality can't drift apart.

**Edge case — reassigning remaining days to a different complaint:** rare (patient wants their last few package days applied to a different body part). Recommended: cancel the original package honestly, ₹0 refunded — money didn't leave the business, it was reallocated, which is different from "nothing happened" and different from an actual cash refund. For the reallocated days: open a real new complaint course, log real visits, with those specific visits' charges waived via the override mechanism. Keeps the record accurate. No dedicated UI flow for this — manual admin steps when it comes up, given rarity.

### Flag / Red Zone

**Not a packages concern** — this feature stays in `README.md` as-is (dedicated `patient_flags` table, not folded into `patient_alerts`). Referenced here only to note that: when reassigning package days is handled via a real new complaint course + visible waived visits, a flag next to that visible pattern in the history is far more useful to future staff than a flag backed only by a text note.

---

## Specialty Services — Package Exclusion

Specialty services (laser, cupping, ISTM, dry needling, others) are explicitly excluded from package coverage — a package never covers these, regardless of what's active.

**Existing hook:** `services.category CHECK (category IN ('STANDARD','PREMIUM'))` already exists. Seed specialty services as `'PREMIUM'`; whatever logic decides "is this line item covered by an active package" simply filters `WHERE category != 'PREMIUM'`. This is a filter on an existing column, not a new mechanism.

This also favors the `visit_service_complaint_links` extension approach (for multi-complaint specialty representation) over a fully separate `special_treatment_events` table — if specialty stays inside `visit_services` tagged by category, package exclusion is one WHERE clause; in a wholly separate table, package logic has to know to actively ignore an entire other structure. See `handover-fee-computation.md` for the full specialty-services multi-complaint design.
