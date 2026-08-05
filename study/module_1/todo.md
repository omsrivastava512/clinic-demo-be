No visuals, staying with text as before.

## The missing clinician selector in the procedure logger — confirmed, and your assessment is sound

You're right that I hadn't explicitly clocked this from the earlier screenshot, and now that you've spelled it out directly: yes, this is real, and it's the same underlying gap as `patients.clinician_id` having no UI field — just showing up in a second place. Your reasoning for why it's fine *right now* is also correct, and it lines up with something already decided in this conversation: since we settled on deriving "who's currently treating this patient/course" from `visits` on demand (the `DISTINCT ON` query) rather than storing it in a dedicated column, the cost of expanding to multi-clinician later is almost entirely a **frontend** problem — adding a clinician-selection control to this exact screen, per complaint course — not a backend one. The schema already supports per-visit clinician attribution (`visits.clinician_id` exists, unused right now because everything defaults null or to the one clinician). So "overkill for the current setup" is the right call — you're not accumulating backend debt by deferring this, only frontend work you'll do when you actually need it.

## The registration form — what I can and can't tell you

I need to be straightforward about my limits here: I don't have access to your live frontend code — only the four reference documents and the two screenshots you've shared directly. `codebase-context.md`'s description of the intake form (Pass 2: referral mode, demographics, clinical notes builder) doesn't mention any clinician-assignment field, which lines up with what you're describing — but that document explicitly flags itself as a possibly-outdated base reference, not a live source of truth. I'd trust a quick grep of your actual intake component (something like `clinicianId` or `clinician_id`) over this document either way.

What I *can* tell you with certainty from the schema itself: there's no backend cost to this gap existing today, and none to closing it later. `patients.clinician_id` is nullable with zero enforcement — it's simply sitting there unused until a form actually populates it. Adding that field later is a pure frontend change (a dropdown, wired to an existing column) — no migration required.

## Retroactive editing — this deserves the deep answer you asked for

You're right that this creates no migration issue in the narrow sense — an `UPDATE` statement changing `clinician_id` on an existing row isn't a schema change, and nothing today blocks it. I checked specifically: neither `patients.clinician_id`, `complaint_courses.clinician_id`, nor `visits.clinician_id` has any trigger or constraint watching that column at all — no derivation, no validation, nothing. Technically, you could build that correction form tomorrow with zero backend work.

But going and checking that carefully surfaced something you should know about before you build it — a real, concrete gap, not a hypothetical one.

**`visits` and `invoices` have no `updated_at` column at all.** Compare this to every other clinical table: `patients`, `complaint_courses`, `packages`, `clinics`, `profiles`, `clinic_service_prices` all have `updated_at timestamptz` plus the `trigger_set_timestamp()` trigger you worked through in Module 1's hands-on exercise. `visits` and `invoices` don't. This isn't an oversight — it matches the MVP philosophy stated directly in the SQL comments: *"Row existing = session complete and paid. No status lifecycle."* The schema was deliberately built assuming these two tables are write-once — created, then never touched again.

That means: if you build a retroactive-edit form for `visits.clinician_id` today, exactly as currently designed, a correction would leave **zero trace anywhere** — not even a timestamp saying the row was ever touched after creation, let alone what it used to say, who changed it, or when. This is the concrete shape of the "data integrity" concern you're asking about — not referential integrity (foreign keys and `CHECK` constraints still apply to every `UPDATE` exactly as they do to every `INSERT`, that part's unaffected either way) but *historical* integrity — the ability to trust that what you're looking at reflects reality, and to reconstruct what it looked like before a correction.

This also connects directly back to one of your own still-open Module 8b questions from the original study plan: *"Does 'a visit row existing = session complete and paid' really match how your front desk works, or are there real in-between states?"* Retroactive editing is really the same question wearing different clothes — the moment you allow correcting a visit after the fact, you've quietly admitted "row exists = done, permanently" isn't strictly true anymore. Worth deciding this deliberately rather than backing into it through a small correction form.

**Does allowing this open the door to editing everything, and should it?** No — and this doesn't need to be an all-or-nothing policy. It's worth deciding per field, based on what kind of fact each one represents:

- **Attribution/metadata fields** — `clinician_id`, the free-text `complaint` snapshot — are low-risk to allow direct correction on. Nothing about them being wrong then fixed threatens billing accuracy or auditability in a way that matters much for a small clinic.
- **Financial fields** — `consultation_fee_in_paise`, `services_total_in_paise`, `invoices.amount_in_paise`, `payment_status`, `payment_mode` — are a different category entirely, and the schema itself already tells you how seriously to take them: `grand_total_in_paise` is a `GENERATED` column Postgres will *physically refuse* to let you edit directly, and `invoice_number` is deliberately generated once, at insert only, specifically so it can never silently drift — Iteration 10 calls this out explicitly, warning that extending the invoice trigger to fire on update would "corrupt the sequential numbering audits." That's the schema's own design philosophy already telling you: money-related facts, once written, should not be silently overwritable. I'd extend that same philosophy to any retroactive-edit feature you build, rather than treating "allow editing clinician_id" as a general precedent that money fields should get the same treatment.
- **Tenancy fields** — `clinic_id`, `owner_id` — already have real trigger-enforced protection (`validate_visit_clinic_id`, `validate_owner_is_admin`, etc.) and should stay exactly as locked-down as they are regardless of what you decide here.

Concretely, if you do build this: the cheapest real safety net, before or alongside the correction UI, is simply adding `updated_at` + `trigger_set_timestamp()` to `visits` and `invoices` — a genuinely safe, low-cost change in Module 9's terms (nullable-with-default column, existing trigger function, no data migration needed). That alone doesn't tell you *what* changed or *who* changed it, but it stops corrections from being completely invisible. A full audit trail (an `updated_by` column, or a proper history table logging old/new values) is the more complete answer if you ever need real accountability — but that's a heavier addition, worth deferring until you actually have multiple clinicians and a real incentive for someone to misattribute a session, which isn't your situation yet.

```sql
-- 1. Two branches, same owner
insert into clinics (name, owner_id) values ('Branch A', '<admin_id>') returning id;
insert into clinics (name, owner_id) values ('Branch B', '<admin_id>') returning id;

-- 2. Your existing patient's id
select id from patients where mrn = 'TEST01';

-- 3. Complaint course at Branch A — auto-provisions patient_clinic_access
insert into complaint_courses (clinic_id, patient_id, complaint_name, start_date, last_date)
values ('<branchA_id>', '<patient_id>', 'Test Complaint', current_date, current_date)
returning id;

select * from patient_clinic_access where patient_id = '<patient_id>'; -- confirm the row appeared

-- 4. Visit at Branch B — should FAIL, no access row there
insert into visits (clinic_id, patient_id, complaint_course_id, date, complaint,
                     visit_type, consultation_type, consultation_fee_in_paise, services_total_in_paise)
values ('<branchB_id>', '<patient_id>', '<complaint_course_id>', current_date, 'Test Complaint',
        'CONSULTATION', 'FIRST', 50000, 0);

-- 5. Visit at Branch A — should SUCCEED
insert into visits (clinic_id, patient_id, complaint_course_id, date, complaint,
                     visit_type, consultation_type, consultation_fee_in_paise, services_total_in_paise)
values ('<branchA_id>', '<patient_id>', '<complaint_course_id>', current_date, 'Test Complaint',
        'CONSULTATION', 'FIRST', 50000, 0);
```
