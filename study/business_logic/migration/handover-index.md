# Handover Index — Business Logic Domain Map

**Paste this file into a new chat first.** It tells you — and any fresh instance of me — which of the three domain files to pull in for whatever you want to work on in that session.

**Underlying reference that never goes away:** `README.md` (the business rules log) is still the authoritative, full-detail record underneath everything here. The three domain handover files are distilled, context-carrying summaries derived from it — not replacements.

---

## The Three Domain Files

### 1. [`handover-fee-computation.md`](./handover-fee-computation.md)
**What it covers:** The additive billing engine — the three visit states (first-time / follow-up / gap-return), the four-bucket money-column redesign (`exam_fee`, `lapse_penalty`, `therapy_fee`, `services_total`), per-complaint therapy scaling, the `PER_VISIT_DAY` vs `PER_COMPLAINT` scope configurability, the same-day dedup trigger, the `consultation_fee_scope` column, and examination-fee-as-a-service. Also covers specialty-service pricing modes and the `clinic_tier_rates` architecture.

**What it assumes you already know (lives elsewhere):**
- The computed-vs-actual override pattern → `handover-pricing-integrity-overrides.md`
- The `package_id` derivation (how therapy visits link to packages) → `handover-packages.md`
- The session/encounter gap → see the **standing cross-cutting note** below

**What is still genuinely open (needs Om's direct answer before this domain can finalize):**
- The joining-fee-vs-per-complaint-exam-fee tension: a returning patient with a brand-new complaint *currently* pays a fresh exam fee for that complaint. Om floated a "true once-ever joining fee" concept that would NOT re-charge them. These are two different real behaviors — Om needs to say explicitly which one holds going forward before the exam-fee trigger can be built.

---

### 2. [`handover-pricing-integrity-overrides.md`](./handover-pricing-integrity-overrides.md)
**What it covers:** The computed-vs-actual pattern and why the backend must derive fees from its own rate tables (not trust client-submitted values). The four override columns (`final_amount_in_paise`, `override_reason`, `override_by`, `override_status`) and their CHECK constraint. The three override scopes: transactional (one-off), standing per-patient (`patient_rate_overrides` with component breakdown), and promotional (manual one-off for now). Why request-and-approve beats TOTP and the shared-code-word idea. The `visits_with_effective_charge` view and the `security_invoker = true` requirement for any view built from this point.

**What it assumes you already know (lives elsewhere):**
- The four-bucket money-column names they override → `handover-fee-computation.md`
- The `computed_refund_in_paise` / `final_refund_in_paise` shape on `packages` → `handover-packages.md`

**What is still genuinely open:**
- None — this domain is fully specified. The only pending item is *building* the trigger and view, not *deciding* anything.

---

### 3. [`handover-packages.md`](./handover-packages.md)
**What it covers:** The attendance-tracking gap (no `visits.package_id`, three ungoverned counters: `attended_days`, `missed_days`, `day_log`). The `package_id` derivation mechanism (server-side BEFORE INSERT trigger on `visits`, lookup by `complaint_course_id + status = 'Active'`). How a matched package zeroes `therapy_fee_in_paise` and increments `attended_days` in the same trigger. The active-package uniqueness constraint (partial unique index). The `missed_days` read-time computation question. Cancellation + refund override columns on `packages`. The `patient_rate_overrides` partial-unique-index. How specialty services are excluded from package coverage via `services.category = 'PREMIUM'`.

**What it assumes you already know (lives elsewhere):**
- The full four-bucket money-column redesign (why `therapy_fee_in_paise` is the field being zeroed) → `handover-fee-computation.md`
- The override column shape (`computed_refund_in_paise`, `final_refund_in_paise`, etc.) → `handover-pricing-integrity-overrides.md`

**What is still genuinely open:**
- `missed_days`: depends on exactly how "missed" gets decided in the real workflow — only once a day is unambiguously past with nothing logged, or can staff pre-mark a day missed? Not clear enough yet to commit to a design. Parked until the rest settles.
- `day_log`: no trigger populates it; three separate things (`attended_days`, `missed_days`, `day_log`) claim to represent the same fact with nothing keeping them in sync. Long-term: pick one source of truth, make the others derived. Deferred until the `visits.package_id` link exists first.

---

## Standing Cross-Cutting Note: The Session/Encounter Gap

**This is the single decision that has shown up inside all three domain files for different reasons.** It is not three separate footnotes — it is one architectural gap wearing different hats depending on where you're standing.

The schema has no `session` or `encounter` concept. `visits.complaint_course_id` is a single FK, so a visit-row is always scoped to one complaint. This creates friction in four distinct places:

1. **Invoicing** — `invoices.visit_id` being singular made batching multi-visit invoices awkward (the original Option A/B/C gap).
2. **Billing correctness** — the flat exam fee (350) and lapse penalty (150) are supposed to fire once per session, but two independent visit rows each deciding on their own whether to add them risks charging twice. The four-bucket redesign's same-day dedup trigger is the current workaround; a real `session_id` would make the same-day lookup trivially simple.
3. **Constraint conflicts** — "one visit per complaint per day" as a hard rule conflicts with confirmed real usage (exam morning, therapy two hours later, same complaint, same day = two visit rows). The right boundary is session, not day.
4. **Specialty-service multi-complaint representation** — a single cupping application covering neck+back+shoulder in one sitting has no clean home when visit rows are each bound to one complaint. The `visit_service_complaint_links` junction is the workaround; a real session row would be the natural anchor.

**Decision status:** Not resolved. The current workaround designs (four-bucket trigger, `visit_service_complaint_links`) are buildable without it and are the right next step. But this is a real fork, not something to defer indefinitely — it gets harder to add a session concept retroactively the more the workarounds compound. Worth a dedicated decision conversation once the three domain builds are underway and the practical friction becomes concrete.

---

## Suggested Reading Order for Common Scenarios

| What you're working on today | Load these files |
|---|---|
| Fee computation trigger / money-column redesign | Index + `handover-fee-computation.md` |
| Override columns, CHECK constraint, view | Index + `handover-pricing-integrity-overrides.md` |
| `visits.package_id`, attendance trigger | Index + `handover-packages.md` + `handover-fee-computation.md` (for the four-bucket column names) |
| `patient_rate_overrides` redesign | Index + `handover-pricing-integrity-overrides.md` |
| Specialty services / `visit_service_complaint_links` | Index + `handover-fee-computation.md` + `handover-packages.md` |
| The session/encounter decision | Index (read the standing note above) + all three domain files |
