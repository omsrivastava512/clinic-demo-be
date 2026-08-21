# Handover — Pricing Integrity & Override Architecture

**Cross-file dependencies — read these before deep-diving here:**
- The four-bucket column names being overridden (`exam_fee_in_paise`, `lapse_penalty_in_paise`, `therapy_fee_in_paise`, `services_total_in_paise`) → `handover-fee-computation.md`
- The refund override shape on `packages` (`computed_refund_in_paise`, `final_refund_in_paise`) → `handover-packages.md`

**Full-detail reference:** `README.md` §7 (Pricing Integrity & Override Architecture)

---

## The Core Principle

**The backend must compute every rule-governed amount from its own rate tables, every time, regardless of what the client request contains.** Never trust a fee value submitted by the frontend.

**Live precedent in the current schema:** `derive_owner_id_from_patient()` — runs on `patient_alerts`, `patient_vitals`, `clinical_notes`, and `timeline_events`. It silently overwrites whatever `owner_id` a client submits with the value it derives fresh from the parent patient row, every time. The same discipline needs extending to fee columns.

**IMPORTANT correction:** an earlier version of the discussion cited `derive_clinic_id_from_patient()` as a second example of this pattern. That function does not exist in the current schema — it was live briefly in v3, got gutted to a no-op the moment `patients.clinic_id` was removed in v4, and was replaced (not restored) in v5. Citing it as live was a mistake. `derive_owner_id_from_patient()` is the only real, live precedent.

**Current gap:** `visits.consultation_fee_in_paise` and `services_total_in_paise` are plain columns today, trusted from whatever gets inserted. `grand_total_in_paise` can't be tampered with (it's generated from those two), but the two things it sums are currently unprotected. Closing this requires a BEFORE INSERT trigger deriving fees from the rate tables directly — once those tables (`clinic_tier_rates`, etc.) exist.

**Why a trigger, not RLS WITH CHECK:** RLS's `WITH CHECK` clause can only accept or reject a write already proposed — it's a yes/no gate, not something that can compute a value and substitute it. A `BEFORE INSERT`/`BEFORE UPDATE` trigger is the only mechanism that can reach in and overwrite a column before the row is saved. It also fires no matter how the write arrives — app, script, direct database access — the same bet `grand_total_in_paise` being GENERATED already makes.

---

## The Four Override Columns

**On `visits`:** leave `consultation_fee_in_paise`, `services_total_in_paise`, and `grand_total_in_paise` exactly as they are — `grand_total_in_paise` keeps its current job, always the honest sum of the four-bucket columns, generated, untouchable. Add four new columns:

| Column | Type | Meaning |
|---|---|---|
| `final_amount_in_paise` | `integer`, nullable | NULL = nothing overridden; real charge is `grand_total_in_paise` as normal |
| `override_reason` | `text`, nullable | Why the deviation was requested/approved |
| `override_by` | `uuid` references `profiles(id)`, nullable | Which admin approved it |
| `override_status` | `text CHECK IN ('none','pending','approved')` default `'none'` | Current state of the override request |

**What actually gets billed:** `COALESCE(final_amount_in_paise, grand_total_in_paise)`

**On `packages`:** same shape, different names (for refunds). Also requires adding `'Cancelled'` to `packages.status` first — it currently only allows `'Active'/'Completed'/'Expired'`, no cancellation state exists yet. Then add:
- `computed_refund_in_paise` — backend-derived from the confirmed formula
- `final_refund_in_paise` (nullable) — override
- `override_reason`, `override_by`, `override_status` — identical pattern

---

## Why Nullable, Not Defaulted

**Rejected approach:** defaulting `final_amount_in_paise` to equal `grand_total_in_paise` at insert time, so every row always carries a concrete number.

**Concrete failure this creates:** a visit logs Monday, `grand_total_in_paise` computes to 550, nobody overrides anything — `final_amount_in_paise` gets written as 550. Tuesday, someone catches a genuine data-entry mistake (wrong tier selected) and corrects it. `grand_total_in_paise` recalculates automatically (it's generated). But `final_amount_in_paise` stays frozen at 550 — now the row shows 550 and 650 disagreeing for a reason that has nothing to do with a waiver. Any query looking for overridden visits by checking `final_amount != grand_total` picks this up as a false positive, indistinguishable from a real waiver.

**Staying nullable:** `COALESCE(final_amount_in_paise, grand_total_in_paise)` always reflects the live `grand_total_in_paise` for any row that was never actually overridden, automatically, with zero extra work to keep them in sync.

---

## The CHECK Constraint

Same style as `chk_referral_doctor_info` / `chk_consultation_type` elsewhere in the schema — several columns required to move together:

- `status = 'none'` → `final_amount_in_paise`, `override_reason`, `override_by` all NULL
- `status = 'pending'` → `override_reason` filled in, `final_amount_in_paise` and `override_by` still NULL (nothing charged differently yet, just a pending request)
- `status = 'approved'` → all three filled in

This only proves the DATA is internally consistent — not that the right PERSON wrote it. A separate trigger handles authorization.

---

## Authorization — The Trigger Layer

The CHECK constraint proves some admin's id is sitting in `override_by`. But it doesn't prove the person performing this specific approval IS that admin. Two things the trigger must do:

1. Check `auth.uid()` (from the verified JWT — never trusted from client input) actually has `role = 'admin'` in `profiles` before allowing `override_status` to move to `'approved'`.
2. Derive `override_by` from `auth.uid()` itself — do NOT accept whatever id the client submits for that column. Reason: two admins A and B exist; A is logged in and approving; if the trigger trusted a client-submitted `override_by`, a bug or stale dropdown could record "B approved this" when A actually did it — misattributing a real action to the wrong person. Deriving from `auth.uid()` closes this: whoever is logged in and clicking approve is who gets recorded, always.

This reuses `validate_owner_is_admin()` — the same check already performed elsewhere in the schema — rather than inventing a new kind of permission.

---

## Authorization Mechanism — Why Request-and-Approve

Three options were compared:

**Shared code word:** ruled out. Structurally identical to trusting the price itself — anyone who learns the word can trigger it from dev tools, and it can't distinguish which admin approved anything.

**Live TOTP (Google Authenticator-style, verifying a specific admin without switching sessions):** real and buildable — Supabase's admin-level tooling includes a way to verify a named user's MFA factor from the backend, separate from that user's own active session, so a server-side function could take "admin X, code 123456" and check it directly against admin X's enrolled authenticator without ever touching the receptionist's browser session. Genuine step up: needs backend code deployed *outside* the database, the first piece of this project that wouldn't live entirely in SQL. Worth reconsidering if instant, in-the-moment approval turns out to matter enough to justify it.

**Request-and-approve (chosen):** receptionist flags `override_status = 'pending'` with a reason at the computed price. The visit stays billed at the full computed amount — a complete, valid, closed transaction on its own. The admin, from their own separate login whenever they next look, updates that exact row. Permitted purely because their profile's `role = 'admin'`. Nobody authenticates as anyone else, in anyone else's session. Cost: nothing beyond the columns and an RLS rule already being built anyway. And since the doctor here is the admin and is usually right there anyway, "asynchronous" in practice probably means seconds, not a real wait.

**A pending request never blocks anything.** While `override_status = 'pending'`, the visit is billed at the full computed amount — a complete, valid transaction. The waiver, whenever approved, retroactively adjusts an already-closed transaction. This is an ordinary business practice (revise an invoice after the fact), not a broken state.

---

## Query Convenience — The View

**Typing `COALESCE(final_amount_in_paise, grand_total_in_paise)` everywhere is error-prone.** A query that forgets the COALESCE and reads `grand_total_in_paise` directly doesn't error — it silently returns the wrong number for any overridden visit. No warning, just quietly incorrect.

**Solution:** a view — `visits_with_effective_charge`:
```sql
CREATE VIEW visits_with_effective_charge WITH (security_invoker = true) AS
SELECT
  *,
  COALESCE(final_amount_in_paise, grand_total_in_paise) AS effective_charge_in_paise
FROM visits;
```

**`security_invoker = true` is not optional.** Views bypass Row-Level Security by default in Postgres — a view normally runs with the permissions of whoever created it, not whoever's querying it, so a plain view can silently return every row from every clinic regardless of who's asking, once real RLS policies exist. `security_invoker = true` (available in Postgres 15+, which Supabase runs) makes the view respect RLS exactly as if the query had hit `visits` directly. Doesn't matter today (everything is still `v1_allow_all`), but fails silently the instant real policies land. **Treat this as a standing habit: any view built from this point forward should include `security_invoker = true`.**

**Writes still go directly to `visits` unaffected.** Only reads that want "what did we actually charge" switch to the view. Reads that specifically want "what would this cost under standard pricing" still correctly read `grand_total_in_paise` from `visits` directly — the view doesn't replace that.

**Why not a second generated column:** checked directly against Postgres documentation, confirmed impossible. A generated column's formula cannot reference another generated column, and `grand_total_in_paise` is itself generated. The view is the only real option.

**Downstream blast radius:**
- `daily_ledger` currently reads `grand_total_in_paise` directly — arguably should show what was actually collected once overrides exist.
- Any future revenue dashboard: same concern.
- `invoices.amount_in_paise`: independent of `visits` today, so an override doesn't automatically flow through to an eventual invoice — a related downstream question to resolve when invoicing is built, not now.

---

## The Three Override Scopes

### 1. Transactional (one visit or refund gets a one-off adjustment)
The four-column design above, applied to a single row. This is the baseline case — needed regardless of anything else.

### 2. Standing Per-Patient (`patient_rate_overrides`)

**Redesigned from a single flat rate.** One `override_amount_in_paise` can only represent "the total is always exactly X, no matter what." It can't represent a partial override (consultation waived but normal daily rate; normal consultation but discounted daily rate). Om confirmed real cases of both shapes exist.

**One row per component being overridden, not one row per patient:**

| Column | Notes |
|---|---|
| `patient_id` | |
| `override_type` | `text CHECK IN ('CONSULTATION_FEE','THERAPY_RATE','FLAT_TOTAL')` — same fake-enum convention used throughout the schema |
| `override_amount_in_paise` | |
| `reason` | Free text — "longtime patient," "financial hardship," etc. |
| `granted_by` | |
| `granted_at` | |
| `expires_at` | Nullable — blank for indefinite, a date for something meant to lapse |
| `is_active` | So it can be turned off later without erasing that it ever existed |

A patient can have up to two component rows active at once (consultation + therapy independently), or one `FLAT_TOTAL` row. Mutually exclusive by convention for now (enforcing cross-row constraints with a plain CHECK is awkward, and this edge case hasn't come up yet).

Rows over columns-on-patients specifically because each grant deserves its own reason and history — a consultation waiver from two years ago and a three-month therapy discount granted later are genuinely different decisions that shouldn't share (or triplicate) reason/granted_by/expires_at fields.

Whatever computes a visit's baseline price checks this table first, per component, before falling back to normal tier rates — set once, automatic after that.

**Confirmed — Om's own ₹180 case:** scales per complaint (₹180 × however many body parts get treated, not a flat total regardless of count). This rules out `FLAT_TOTAL` for his case. The correct model is:
- `CONSULTATION_FEE = 0` — covers whichever flat one-time fee would otherwise have applied (exam fee OR lapse penalty — only one of those can ever fire on a given visit-day, so one override_type row covers both)
- `THERAPY_RATE = 180` — replaces the normal 200/300 tier rate, scaling per complaint exactly like the standard rule

`FLAT_TOTAL` stays available in the design for a patient who genuinely wants a true single-number-no-matter-what arrangement.

**Constraint:** `CREATE UNIQUE INDEX ON patient_rate_overrides(patient_id, override_type) WHERE is_active;` — stops a patient from having two simultaneously-active overrides of the exact same type.

### 3. Promotional (blanket discount for a day or event — the Mother's Day example)

Not worth dedicated structure yet. Treat as the ordinary transactional override, applied by hand to each qualifying visit as it happens. A real "promotions" concept (date ranges, eligibility rules, overlap handling) is genuine complexity for something that's happened once so far — build if and when it becomes a recurring pattern.

---

## Refund Preview (Cancel Package)

**Clicking "Cancel Package" should call the exact same backend function that finalizes the cancellation** — purely to preview (writing nothing), then for real when confirmed. Same code path both times, so preview and reality can't quietly drift apart if the formula ever changes. Not two copies of the math, not frontend-computed.
