# Package upgrade: prototype and test results (scratch PostgreSQL, not the clinic database)

**Date:** 4 November 2026. **Engine:** PostgreSQL 16.15, `TimeZone=UTC` (the Supabase default), default READ COMMITTED isolation.
**Written for:** the audit of `package_upgrade_design_discussion_log_2026-11-04_0220_IST.md` (response PKGUPGRADE-02).

## What this is, and what it is not

A throwaway model of ONLY the upgrade proposal, built from the log's proposed fields and the current decision pages
(`start_date`, `purchased_at`, `max_visits_per_day`, `invoice_line_items.package_purchase_id`, four invoice states).
It is **not** the clinic's schema, was **not** run against any real database, and changed nothing in the repository.
The reference functions (`record_visit`, `purchase_upgrade`, `close_package`) are illustrative, not production code.
Races were staged with two real concurrent database sessions and `pg_sleep` to make the timing repeatable.
Not tested: RLS, Supabase-specific behaviour, the frontend, the refund module (only its arithmetic effect in T9).

**Safety:** everything lives in a schema called `upgrade_lab` and never touches `public`. Even so, run it only in an
empty scratch database, never in the clinic's real Supabase project.

## How to reproduce

1. Create an empty scratch database. 2. Run, in order: `01_schema.sql`, `02_functions.sql`, `02b_invariants.sql`.
3. Run the driver `04_tests.sh` (edit the `PG*` variables at its top). It installs `03_guards.sql` part-way through, on purpose,
so that each hole is shown first without the guard and then with it.
Results below are the verbatim output of that run (random UUIDs differ between runs).

## Which test supports which audit finding

| Test | What it shows |
|---|---|
| T1 | The log's declarative rules work: composite foreign keys force same patient, clinic and complaint; CHECK fixes the shape (UPGRADE = 16 visits + a parent); UNIQUE(parent) allows one upgrade per original. |
| T2 / T2' | Holes the log's own constraint list leaves open, and the table-level guards that close them. One hole remains in the guard itself: a client-supplied `purchased_at`. |
| T3 | Handover order: child entering Active before the parent leaves it fails immediately. A DEFERRABLE exclusion constraint makes the order irrelevant. |
| T4a-c | Two tabs saving the 10th visit: 11 covered visits on a 10-visit package, then the fix (lock, or table guard). |
| T5a-e | Upgrade bought while the 10th visit is saved: a Queued child stranded under a Completed parent, then which guard prevents it and which does not. |
| T6 / T6' | Daily cap of 1 broken across the parent-to-child handover, then fixed. |
| T7 / T7' | Manual closure of the parent while a Queued upgrade exists; the dead end after a closed upgrade. |
| T8 | Day-30 window arithmetic, inclusive 30th date, UTC versus clinic-local date. |
| T9 | Price arithmetic for three candidate pricing bases across seven situations. |
| T10 / T10' | The zero-charge upgrade: how the database sees it and how to tell it from a free standard sale. |

## SQL, part 1: tables and declarative constraints (`01_schema.sql`)

~~~~sql
-- ============================================================================
-- PACKAGE-UPGRADE PROTOTYPE  (scratch Postgres 16, NOT the clinic's database)
-- Purpose: test the design in package_upgrade_design_discussion_log_2026-11-04
--          against a real engine. Table/column names follow the CURRENT decision
--          pages where they exist (start_date, purchased_at, max_visits_per_day,
--          invoice_line_items.package_purchase_id, payments, 4 invoice states),
--          NOT the stale July/Sept docs/schema/supabase_migration.md v11.
-- Anything marked  -- [LOG]  is exactly what the upgrade log proposes.
-- Anything marked  -- [AUDIT] is an addition/change the audit recommends.
-- ============================================================================
\set ON_ERROR_STOP on
-- SAFETY: everything below lives in its own schema, `upgrade_lab`. It never touches `public`.
-- Still: run it only in a scratch/empty database, never in the clinic's real Supabase project.
drop schema if exists upgrade_lab cascade;
create schema upgrade_lab;
set search_path = upgrade_lab;

create table clinics (
  id uuid primary key default gen_random_uuid(),
  tz text not null default 'Asia/Kolkata'            -- v11 has clinics.default_timezone
);

create table patients (
  id uuid primary key default gen_random_uuid(),
  name text not null
);

create table complaint_courses (
  id uuid primary key default gen_random_uuid(),
  clinic_id  uuid not null references clinics(id),
  patient_id uuid not null references patients(id),
  category   text not null check (category in ('ROUTINE','REHAB')),   -- immutable per S33 s3
  status     text not null default 'Active' check (status in ('Active','Closed')),
  -- [AUDIT] supports composite FKs below: "same patient + clinic" becomes declarative
  constraint uq_cc_identity unique (id, patient_id, clinic_id)
);

-- price catalogue: one row per (clinic, category, size)  (S31 Q27: four configurable prices)
create table package_catalog (
  clinic_id uuid not null references clinics(id),
  category  text not null check (category in ('ROUTINE','REHAB')),
  visits    smallint not null check (visits in (10,26)),
  price_in_paise bigint not null check (price_in_paise > 0),
  primary key (clinic_id, category, visits)
);

create table packages (
  id uuid primary key default gen_random_uuid(),
  clinic_id  uuid not null,
  patient_id uuid not null,
  linked_complaint_id uuid not null,
  package_kind text not null check (package_kind in ('STANDARD','UPGRADE')),            -- [LOG]
  parent_package_id uuid,                                                                -- [LOG]
  visits_promised smallint not null,                                                     -- [LOG] this row's own contribution
  max_visits_per_day smallint not null default 1 check (max_visits_per_day >= 1),        -- repo: S30 Q03 (log omits it)
  status text not null check (status in ('Queued','Active','Completed','Closed')),       -- [LOG]
  purchased_at timestamptz not null default now(),                                       -- repo name (log says purchase_date)
  start_date date,                                                                       -- repo name (log says coverage_start_date)

  -- [AUDIT] same-complaint integrity, declarative (no trigger needed).
  constraint fk_pkg_complaint foreign key (linked_complaint_id, patient_id, clinic_id)
     references complaint_courses (id, patient_id, clinic_id),

  -- [LOG] "UPGRADE implies 16 and has a parent; STANDARD has no parent"
  constraint chk_pkg_shape check (
       (package_kind = 'STANDARD' and parent_package_id is null     and visits_promised in (10, 26))
    or (package_kind = 'UPGRADE'  and parent_package_id is not null and visits_promised = 16)),

  -- [LOG] "start date may be NULL while Queued"; [AUDIT] and only then
  constraint chk_pkg_start check (
       (status = 'Queued' and start_date is null)
    or (status in ('Active','Completed') and start_date is not null)
    or  status = 'Closed'),

  -- [AUDIT] Queued is upgrade-only (the settled "queuing rejected" rule survives for everything else)
  constraint chk_pkg_queued_only_upgrade check (status <> 'Queued' or package_kind = 'UPGRADE'),

  constraint uq_pkg_identity unique (id, patient_id, clinic_id, linked_complaint_id),
  -- [LOG] "one upgrade per original"
  constraint uq_pkg_one_upgrade_per_parent unique (parent_package_id),
  -- [AUDIT] parent must be the same patient/clinic/complaint: declarative, no trigger
  constraint fk_pkg_parent foreign key (parent_package_id, patient_id, clinic_id, linked_complaint_id)
     references packages (id, patient_id, clinic_id, linked_complaint_id)
);

-- [LOG + repo] "at most one Active package per complaint". NOTE: a partial UNIQUE INDEX
-- cannot be DEFERRABLE, so it is checked row-by-row, immediately. See test T4.
create unique index uq_one_active_pkg_per_complaint
  on packages (linked_complaint_id) where status = 'Active';

create table visits (
  id uuid primary key default gen_random_uuid(),
  clinic_id  uuid not null,
  patient_id uuid not null,
  complaint_course_id uuid not null,
  date date not null,
  package_id uuid,
  created_at timestamptz not null default now(),
  constraint fk_visit_complaint foreign key (complaint_course_id, patient_id, clinic_id)
     references complaint_courses (id, patient_id, clinic_id),
  -- [AUDIT] "a visit can only be covered by a package of the SAME complaint" (S31 Q24), declarative
  constraint fk_visit_pkg foreign key (package_id, patient_id, clinic_id, complaint_course_id)
     references packages (id, patient_id, clinic_id, linked_complaint_id)
);
create index idx_visits_package on visits (package_id);

create table invoices (
  id uuid primary key default gen_random_uuid(),
  clinic_id  uuid not null references clinics(id),
  patient_id uuid not null references patients(id),
  status text not null default 'UNPAID' check (status in ('UNPAID','PARTIALLY_PAID','PAID','WAIVED')),
  issued_at timestamptz not null default now()
);

create table invoice_line_items (
  id uuid primary key default gen_random_uuid(),
  invoice_id uuid not null references invoices(id),
  visit_id uuid references visits(id),
  package_purchase_id uuid references packages(id),
  amount_in_paise bigint not null check (amount_in_paise >= 0),
  constraint chk_one_source check (num_nonnulls(visit_id, package_purchase_id) = 1),
  -- [AUDIT] exactly one purchase line per package: the "original invoiced charge" lookup is then unambiguous
  constraint uq_line_per_package unique (package_purchase_id),
  constraint uq_line_per_visit   unique (visit_id)
);

create table payments (
  id uuid primary key default gen_random_uuid(),
  invoice_id uuid not null references invoices(id) on delete restrict,
  amount_in_paise bigint not null check (amount_in_paise > 0)
);
~~~~

## SQL, part 2: reference functions and test fixtures (`02_functions.sql`)

~~~~sql
\set ON_ERROR_STOP on
set search_path = upgrade_lab;

-- ---------------------------------------------------------------------------
-- Clinic-local "today". [AUDIT] never use current_date for business dates:
-- on Supabase the session TimeZone is UTC, the clinic is Asia/Kolkata (UTC+05:30).
-- ---------------------------------------------------------------------------
create or replace function clinic_today(p_clinic uuid, p_now timestamptz default now())
returns date language sql stable set search_path = upgrade_lab as $$
  select (p_now at time zone c.tz)::date from clinics c where c.id = p_clinic
$$;

-- 10-visit review date: start date = Day 1 (even a Sunday); afterwards skip Sundays; 10th counted date.
create or replace function review_date_short(p_start date) returns date language sql immutable set search_path = upgrade_lab as $$
  select d from (
    select g::date as d, row_number() over (order by g) as rn
      from generate_series(p_start::timestamp, (p_start + 40)::timestamp, interval '1 day') s(g)
     where g::date = p_start or extract(dow from g) <> 0
  ) x where rn = 10
$$;

-- 26-visit review date: 30 consecutive calendar dates, start = Day 1, so Day 30 = start + 29.
create or replace function review_date_long(p_start date) returns date language sql immutable set search_path = upgrade_lab as $$
  select p_start + 29
$$;

-- ---------------------------------------------------------------------------
-- Test fixtures (fixed UUIDs so the test script stays readable)
-- ---------------------------------------------------------------------------
create or replace function lab_reset() returns void language plpgsql set search_path = upgrade_lab as $$
begin
  truncate payments, invoice_line_items, invoices, visits, packages, complaint_courses,
           package_catalog, patients, clinics cascade;
  insert into clinics(id) values ('00000000-0000-0000-0000-00000000c001'),
                                 ('00000000-0000-0000-0000-00000000c002');
  insert into patients(id, name) values ('00000000-0000-0000-0000-00000000a001','Patient One'),
                                        ('00000000-0000-0000-0000-00000000a002','Patient Two');
  insert into complaint_courses(id, clinic_id, patient_id, category) values
    ('00000000-0000-0000-0000-00000000b001','00000000-0000-0000-0000-00000000c001','00000000-0000-0000-0000-00000000a001','ROUTINE'),
    ('00000000-0000-0000-0000-00000000b002','00000000-0000-0000-0000-00000000c001','00000000-0000-0000-0000-00000000a001','REHAB'),
    ('00000000-0000-0000-0000-00000000b003','00000000-0000-0000-0000-00000000c001','00000000-0000-0000-0000-00000000a002','ROUTINE');
  insert into package_catalog values
    ('00000000-0000-0000-0000-00000000c001','ROUTINE',10, 180000),   -- Rs 1,800
    ('00000000-0000-0000-0000-00000000c001','ROUTINE',26, 500000),   -- Rs 5,000
    ('00000000-0000-0000-0000-00000000c001','REHAB',10,  270000),
    ('00000000-0000-0000-0000-00000000c001','REHAB',26,  700000);
end $$;

-- a standard package + its purchase invoice/line (what checkout would have produced)
create or replace function seed_pkg(p_id uuid, p_complaint uuid, p_visits smallint, p_status text,
                                    p_start date, p_price bigint, p_inv_status text default 'PAID')
returns uuid language plpgsql set search_path = upgrade_lab as $$
declare cc complaint_courses%rowtype; v_inv uuid := gen_random_uuid();
begin
  select * into cc from complaint_courses where id = p_complaint;
  insert into packages(id, clinic_id, patient_id, linked_complaint_id, package_kind, visits_promised, status, start_date, purchased_at)
    values (p_id, cc.clinic_id, cc.patient_id, p_complaint, 'STANDARD', p_visits, p_status, p_start, (p_start::timestamp at time zone 'Asia/Kolkata'));
  insert into invoices(id, clinic_id, patient_id, status) values (v_inv, cc.clinic_id, cc.patient_id, p_inv_status);
  insert into invoice_line_items(invoice_id, package_purchase_id, amount_in_paise) values (v_inv, p_id, p_price);
  return p_id;
end $$;

-- n covered visits on consecutive dates starting at p_from
create or replace function seed_visits(p_pkg uuid, p_n int, p_from date) returns void language plpgsql set search_path = upgrade_lab as $$
declare pk packages%rowtype; i int;
begin
  select * into pk from packages where id = p_pkg;
  for i in 0 .. p_n - 1 loop
    insert into visits(clinic_id, patient_id, complaint_course_id, date, package_id)
      values (pk.clinic_id, pk.patient_id, pk.linked_complaint_id, p_from + i, p_pkg);
  end loop;
end $$;

-- ---------------------------------------------------------------------------
-- record_visit: what the "package_id lookup" step of checkout must do.
--   p_lock               false = the log's design as written (count at read time, no lock)
--                        true  = [AUDIT] serialise on the complaint row first
--   p_child_start_offset 0 = child may cover visits from the same day its parent finishes
--                        1 = [AUDIT] child coverage starts the NEXT day (the T/T+1 rule applied to handover)
--   p_delay              artificial pause between check and insert, only to make races reproducible
-- ---------------------------------------------------------------------------
create or replace function record_visit(p_complaint uuid, p_date date,
                                        p_lock boolean default true,
                                        p_child_start_offset int default 1,
                                        p_delay float default 0,
                                        p_delay_after float default 0)
returns text language plpgsql set search_path = upgrade_lab as $$
declare
  cc complaint_courses%rowtype; pkg packages%rowtype;
  used int; used_today int;
begin
  select * into cc from complaint_courses where id = p_complaint;
  if p_lock then
    -- NO KEY UPDATE: conflicts with other writers of this mutex, but NOT with the KEY SHARE
    -- lock that every FK check (visits.complaint_course_id, packages.linked_complaint_id) takes.
    perform 1 from complaint_courses where id = p_complaint for no key update;
  end if;

  select * into pkg from packages
   where linked_complaint_id = p_complaint and status = 'Active' and start_date <= p_date;
  if not found then
    insert into visits(clinic_id, patient_id, complaint_course_id, date) values (cc.clinic_id, cc.patient_id, p_complaint, p_date);
    return 'UNCOVERED (no package covering this date)';
  end if;

  select count(*) into used_today from visits where package_id = pkg.id and date = p_date;
  if used_today >= pkg.max_visits_per_day then
    insert into visits(clinic_id, patient_id, complaint_course_id, date) values (cc.clinic_id, cc.patient_id, p_complaint, p_date);
    return format('UNCOVERED (daily cap %s reached on package %s)', pkg.max_visits_per_day, pkg.package_kind);
  end if;

  select count(*) into used from visits where package_id = pkg.id;
  if used >= pkg.visits_promised then
    raise exception 'package % already exhausted (% of %)', pkg.id, used, pkg.visits_promised;
  end if;

  perform pg_sleep(p_delay);

  insert into visits(clinic_id, patient_id, complaint_course_id, date, package_id)
    values (cc.clinic_id, cc.patient_id, p_complaint, p_date, pkg.id);

  if used + 1 = pkg.visits_promised then
    -- ORDER MATTERS: parent leaves 'Active' BEFORE child enters it (partial unique index is immediate)
    update packages set status = 'Completed' where id = pkg.id;
    update packages set status = 'Active', start_date = p_date + p_child_start_offset
     where parent_package_id = pkg.id and status = 'Queued';
  end if;
  perform pg_sleep(p_delay_after);   -- hold the transaction open after the flip (race tests only)
  return format('COVERED %s %s/%s', pkg.package_kind, used + 1, pkg.visits_promised);
end $$;

-- ---------------------------------------------------------------------------
-- purchase_upgrade: the "atomic upgrade + invoice" step.
--   p_lock = false : read parent status without any lock (what a straightforward RPC does)
--   p_lock = true  : [AUDIT] take the per-complaint mutex, then RE-READ the parent
-- Price rule is the log's WORKING formula:  max(0, current 26-visit price - original 10-visit LINE amount)
-- ---------------------------------------------------------------------------
create or replace function purchase_upgrade(p_parent uuid, p_now timestamptz default now(),
                                            p_lock boolean default true, p_delay float default 0)
returns text language plpgsql set search_path = upgrade_lab as $$
declare
  par packages%rowtype; cc complaint_courses%rowtype;
  v_today date; orig bigint; target bigint; charge bigint;
  v_status text; v_start date;
  v_child uuid := gen_random_uuid(); v_inv uuid := gen_random_uuid();
begin
  select * into par from packages where id = p_parent;
  if not found then raise exception 'parent not found'; end if;
  select * into cc from complaint_courses where id = par.linked_complaint_id;

  if p_lock then
    perform 1 from complaint_courses where id = cc.id for no key update;
    select * into par from packages where id = p_parent;            -- re-read AFTER the lock
  end if;

  if par.package_kind <> 'STANDARD' or par.visits_promised <> 10 then
    raise exception 'parent must be a STANDARD 10-visit package'; end if;
  if par.status not in ('Active','Completed') then
    raise exception 'parent status % is not upgradeable', par.status; end if;

  v_today := clinic_today(par.clinic_id, p_now);
  if v_today > par.start_date + 29 then
    raise exception 'upgrade window closed: today % is after Day 30 (%)', v_today, par.start_date + 29; end if;

  select amount_in_paise into orig from invoice_line_items where package_purchase_id = par.id;
  if orig is null then raise exception 'parent has no purchase line'; end if;
  select price_in_paise into target from package_catalog
   where clinic_id = par.clinic_id and category = cc.category and visits = 26;
  if target is null then raise exception '26-visit price not configured'; end if;
  charge := greatest(0, target - orig);

  perform pg_sleep(p_delay);

  if par.status = 'Active' then
    v_status := 'Queued'; v_start := null;
  else
    v_status := 'Active';
    v_start := case when exists (select 1 from visits
                                  where complaint_course_id = cc.id and date = v_today and package_id is not null)
                    then v_today + 1 else v_today end;
  end if;

  insert into packages(id, clinic_id, patient_id, linked_complaint_id, package_kind, parent_package_id,
                       visits_promised, status, purchased_at, start_date)
    values (v_child, par.clinic_id, par.patient_id, par.linked_complaint_id, 'UPGRADE', par.id, 16, v_status, p_now, v_start);
  insert into invoices(id, clinic_id, patient_id, status, issued_at)
    values (v_inv, par.clinic_id, par.patient_id, case when charge = 0 then 'PAID' else 'UNPAID' end, p_now);
  insert into invoice_line_items(invoice_id, package_purchase_id, amount_in_paise) values (v_inv, v_child, charge);
  return format('upgrade child status=%s start=%s charge=Rs %s', v_status, coalesce(v_start::text,'NULL'), to_char(charge/100.0,'FM999990.00'));
end $$;

-- ---------------------------------------------------------------------------
-- close_package: manual closure. p_cascade = true closes a still-Queued child with its parent.
-- ---------------------------------------------------------------------------
create or replace function close_package(p_pkg uuid, p_cascade boolean default false)
returns text language plpgsql set search_path = upgrade_lab as $$
declare par packages%rowtype; n int := 0;
begin
  perform 1 from complaint_courses c join packages p on p.linked_complaint_id = c.id
   where p.id = p_pkg for no key update;
  select * into par from packages where id = p_pkg;
  if par.status = 'Closed' then raise exception 'package already closed'; end if;
  update packages set status = 'Closed' where id = p_pkg;
  if p_cascade then
    update packages set status = 'Closed' where parent_package_id = p_pkg and status = 'Queued';
    get diagnostics n = row_count;
  end if;
  return format('closed %s; cascaded to %s queued child(ren)', par.package_kind, n);
end $$;
~~~~

## SQL, part 3: consistency query pack, should always return zero rows (`02b_invariants.sql`)

~~~~sql
\set ON_ERROR_STOP on
set search_path = upgrade_lab;
-- [AUDIT] Monthly consistency query pack. DETECTION, not enforcement: it must return ZERO rows.
-- Read-only. Written against the lab tables; rename tables if you adapt it to a real schema.
create or replace view lab_invariant_violations as
  select 'stranded Queued child (parent is not Active)'::text as problem, c.id::text as package_id
    from packages c join packages p on p.id = c.parent_package_id
   where c.status = 'Queued' and p.status <> 'Active'
  union all
  select 'package covers more visits than it promised', p.id::text
    from packages p
   where (select count(*) from visits v where v.package_id = p.id) > p.visits_promised
  union all
  select 'chain covers more than 26 visits', r.id::text
    from packages r
   where r.parent_package_id is null
     and (select count(*) from visits v
           where v.package_id = r.id
              or v.package_id in (select id from packages where parent_package_id = r.id)) > 26
  union all
  select 'covered visits per complaint/day exceed the daily cap', v.complaint_course_id::text || ' on ' || v.date::text
    from visits v join packages p on p.id = v.package_id
   group by v.complaint_course_id, v.date
  having count(*) > max(p.max_visits_per_day)
  union all
  select 'package without a purchase line', p.id::text
    from packages p
   where not exists (select 1 from invoice_line_items l where l.package_purchase_id = p.id)
  union all
  select 'Completed package that did not use all its visits', p.id::text
    from packages p
   where p.status = 'Completed'
     and (select count(*) from visits v where v.package_id = p.id) < p.visits_promised;
~~~~

## SQL, part 4: table-level safety nets (`03_guards.sql`)

~~~~sql
\set ON_ERROR_STOP on
set search_path = upgrade_lab;
-- ============================================================================
-- [AUDIT] TABLE-LEVEL SAFETY NETS.
-- Principle: every rule that must hold lives on the TABLE, so a caller that
-- forgets a lock (or inserts directly, while RLS is still "v1_allow_all")
-- fails closed instead of corrupting the chain.
-- ============================================================================

-- (1) Identity / entitlement columns are frozen after insert.
create or replace function packages_immutable() returns trigger language plpgsql set search_path = upgrade_lab as $$
begin
  if (new.package_kind, new.parent_package_id, new.visits_promised, new.purchased_at,
      new.patient_id, new.clinic_id, new.linked_complaint_id)
     is distinct from
     (old.package_kind, old.parent_package_id, old.visits_promised, old.purchased_at,
      old.patient_id, old.clinic_id, old.linked_complaint_id) then
    raise exception 'package identity/entitlement columns are immutable';
  end if;
  return new;
end $$;
create trigger trg_packages_immutable before update on packages
  for each row execute function packages_immutable();

-- (2) Cross-row rules for an UPGRADE row that CHECK/FK cannot express.
--     FOR SHARE on the parent: waits for any in-flight change to that row, then reads the LATEST version.
create or replace function packages_child_guard() returns trigger language plpgsql set search_path = upgrade_lab as $$
declare par packages%rowtype;
begin
  if new.package_kind = 'UPGRADE' then
    select * into par from packages where id = new.parent_package_id for share;
    if par.package_kind <> 'STANDARD' or par.visits_promised <> 10 then
      raise exception 'upgrade parent must be a STANDARD 10-visit package'; end if;
    if par.status not in ('Active','Completed') then
      raise exception 'upgrade parent must be Active or Completed (is %)', par.status; end if;
    if new.status = 'Queued' and par.status <> 'Active' then
      raise exception 'a Queued upgrade needs an Active parent (parent is %)', par.status; end if;
    if new.status = 'Active' and par.status <> 'Completed' then
      raise exception 'an Active upgrade needs a Completed parent (parent is %)', par.status; end if;
    if clinic_today(new.clinic_id, new.purchased_at) > par.start_date + 29 then
      raise exception 'upgrade window closed: last day was % (30th calendar date from parent start)', par.start_date + 29; end if;
  end if;
  return new;
end $$;
create trigger trg_packages_child_guard before insert on packages
  for each row execute function packages_child_guard();

-- (3) Q30 "no zero-price package sale": the ONLY legitimate zero package line is an UPGRADE floor.
create or replace function line_zero_guard() returns trigger language plpgsql set search_path = upgrade_lab as $$
declare k text;
begin
  if new.package_purchase_id is not null and new.amount_in_paise = 0 then
    select package_kind into k from packages where id = new.package_purchase_id;
    if k <> 'UPGRADE' then
      raise exception 'zero-price package sale is only permitted for an UPGRADE (Q30)'; end if;
  end if;
  return new;
end $$;
create trigger trg_line_zero_guard after insert or update on invoice_line_items
  for each row execute function line_zero_guard();

-- (4) "No ghost packages": every package must have a purchase line by COMMIT (repo's own deferred-trigger idiom).
create or replace function pkg_needs_purchase_line() returns trigger language plpgsql set search_path = upgrade_lab as $$
begin
  if not exists (select 1 from invoice_line_items where package_purchase_id = new.id) then
    raise exception 'package % has no purchase invoice line (ghost package)', new.id;
  end if;
  return null;
end $$;
create constraint trigger trg_pkg_needs_line after insert on packages
  deferrable initially deferred for each row execute function pkg_needs_purchase_line();

-- (5) Redemption guard on visits: locks the package row, so a double-submit / two tabs cannot over-redeem.
create or replace function visit_pkg_guard() returns trigger language plpgsql set search_path = upgrade_lab as $$
declare pkg packages%rowtype; used int; used_complaint_day int;
begin
  if new.package_id is null then return new; end if;
  select * into pkg from packages where id = new.package_id for update;
  if pkg.status <> 'Active' then
    raise exception 'package % is % and cannot cover a visit', pkg.id, pkg.status; end if;
  if new.date < pkg.start_date then
    raise exception 'visit date % is before package coverage start %', new.date, pkg.start_date; end if;
  select count(*) into used from visits where package_id = pkg.id;
  if used >= pkg.visits_promised then
    raise exception 'package % exhausted (% of %)', pkg.id, used, pkg.visits_promised; end if;
  -- daily cap evaluated across the WHOLE complaint (so it holds across the parent->child handover)
  select count(*) into used_complaint_day from visits
   where complaint_course_id = new.complaint_course_id and date = new.date and package_id is not null;
  if used_complaint_day >= pkg.max_visits_per_day then
    raise exception 'daily covered-visit cap % reached for this complaint on %', pkg.max_visits_per_day, new.date; end if;
  return new;
end $$;
create trigger trg_visit_pkg_guard before insert on visits
  for each row execute function visit_pkg_guard();

-- (6) A Queued upgrade may only exist under an Active parent. Checked at COMMIT, on every path
--     (manual close, race, direct SQL). A legitimate handover (parent Completed + child Active in ONE
--     transaction) passes, because by commit time the child is no longer Queued.
create or replace function no_stranded_queued() returns trigger language plpgsql set search_path = upgrade_lab as $$
begin
  if exists (select 1 from packages c join packages p on p.id = c.parent_package_id
              where c.status = 'Queued' and p.status <> 'Active'
                and (c.id = new.id or p.id = new.id)) then
    raise exception 'a Queued upgrade requires its parent to be Active (stranded upgrade child)';
  end if;
  return null;
end $$;
create constraint trigger trg_no_stranded_queued after insert or update of status on packages
  deferrable initially deferred for each row execute function no_stranded_queued();
~~~~

## Test results (verbatim output of `04_tests.sh`)

~~~~text
PostgreSQL: PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.   TimeZone=UTC

=== T1  Declarative identity + shape rules (composite FKs, CHECKs, UNIQUE). No triggers installed. ===
  child for a DIFFERENT patient than its complaint                 REJECTED  insert or update on table "packages" violates foreign key constraint "fk_pkg_complaint"
  child on a DIFFERENT complaint than its parent                   REJECTED  insert or update on table "packages" violates foreign key constraint "fk_pkg_parent"
  child at a DIFFERENT clinic than the complaint                   REJECTED  insert or update on table "packages" violates foreign key constraint "fk_pkg_complaint"
  UPGRADE with visits_promised = 26                                REJECTED  new row for relation "packages" violates check constraint "chk_pkg_shape"
  UPGRADE with no parent (standalone upgrade sale)                 REJECTED  new row for relation "packages" violates check constraint "chk_pkg_shape"
  STANDARD that carries a parent                                   REJECTED  new row for relation "packages" violates check constraint "chk_pkg_shape"
  Queued STANDARD (queuing stays rejected for sales)               REJECTED  new row for relation "packages" violates check constraint "chk_pkg_queued_only_upgrade"
  visit covered by ANOTHER complaint's package                     REJECTED  insert or update on table "visits" violates foreign key constraint "fk_visit_pkg"
  first upgrade for P1 via RPC:  upgrade child status=Active start=2026-11-12 charge=Rs 3200.00
  second upgrade for P1 via RPC: ERROR:  duplicate key value violates unique constraint "uq_pkg_one_upgrade_per_parent"

=== T2  What the log's constraints do NOT stop (guards NOT installed) ===
  child pointing at a STANDARD 26-visit parent                     ACCEPTED
  (valid upgrade via RPC)  -> upgrade child status=Active start=2026-11-12 charge=Rs 3200.00
  grandchild: an UPGRADE whose parent is itself an UPGRADE         ACCEPTED
  in-place edit: UPDATE visits_promised 10 -> 26 on a STANDARD     ACCEPTED
  sell a STANDARD package with a Rs 0 purchase line                ACCEPTED
  ghost package: a package row with NO purchase line               ACCEPTED
  visit attached directly to a QUEUED upgrade                      ACCEPTED
  upgrade 64 days after parent start, purchased_at = server now()  ACCEPTED
  same, but client SUPPLIES purchased_at = 2026-09-15 (backdated)  ACCEPTED

=== T3  Partial unique index 'one Active per complaint' vs the Queued -> Active handover ===
  start: parent=Active  child=Queued
  child -> Active FIRST (parent still Active)                      REJECTED  duplicate key value violates unique constraint "uq_one_active_pkg_per_complaint"
  parent -> Completed, THEN child -> Active (2 statements)         ACCEPTED
  child -> Active FIRST, parent -> Completed second, ONE transaction (deferred) ACCEPTED
  leave BOTH rows Active at commit (must still fail)               REJECTED  conflicting key value violates exclusion constraint "ex_one_active_pkg_per_complaint"
  (separate small experiment, run by hand: one UPDATE that flips both rows succeeded or failed depending on which row the scan reached first)

=== T4a Two tabs save the 10th visit together. Count-at-read, NO lock (the log's design as written) ===
  session 1: COVERED STANDARD 10/10
  session 2: COVERED STANDARD 10/10
  -> parent=Completed  child=Active start=2026-11-11  covered visits on parent=11
  -> invariant view: package covers more visits than it promised; covered visits per complaint/day exceed the daily cap

=== T4b Same race. [AUDIT] per-complaint lock taken first ===
  session 1: COVERED STANDARD 10/10
  session 2: UNCOVERED (no package covering this date)
  -> parent=Completed  child=Active start=2026-11-11  covered visits on parent=10
  -> invariant view: no violations

=== T5a Upgrade bought while the 10th visit is being saved. RPC reads parent with NO lock ===
  A (saves the 10th visit): COVERED STANDARD 10/10
  B (buys the upgrade):     upgrade child status=Queued start=NULL charge=Rs 3200.00
  -> parent=Completed  child=Queued start=null  covered visits on parent=10
  -> invariant view: stranded Queued child (parent is not Active)
  -> 12 Nov visit: UNCOVERED (no package covering this date)

=== T5b Same race. [AUDIT] upgrade RPC takes the per-complaint lock, then RE-READS the parent ===
  A (saves the 10th visit): COVERED STANDARD 10/10
  B (buys the upgrade):     upgrade child status=Active start=2026-11-11 charge=Rs 3200.00
  -> parent=Completed  child=Active start=2026-11-11  covered visits on parent=10
  -> invariant view: no violations
  -> 12 Nov visit: COVERED UPGRADE 1/16

=== T6  Daily cap (max_visits_per_day = 1) across the parent -> child handover ===
  child coverage starts parent-finish-date + 0 day(s):   1st visit on 10 Nov -> COVERED STANDARD 10/10
                                                          2nd visit on 10 Nov -> COVERED UPGRADE 1/16
  -> invariant view: covered visits per complaint/day exceed the daily cap
  child coverage starts parent-finish-date + 1 day(s):   1st visit on 10 Nov -> COVERED STANDARD 10/10
                                                          2nd visit on 10 Nov -> UNCOVERED (no package covering this date)
  -> invariant view: no violations

=== T7  Manual closure of the parent while an early-bought (Queued) upgrade exists ===
  close parent, cascade=false -> closed STANDARD; cascaded to 0 queued child(ren)
  -> parent=Closed  child=Queued start=null  covered visits on parent=4
  -> invariant view: stranded Queued child (parent is not Active)
     next visit -> UNCOVERED (no package covering this date)
  close parent, cascade=true -> closed STANDARD; cascaded to 1 queued child(ren)
  -> parent=Closed  child=Closed start=null  covered visits on parent=4
  -> invariant view: no violations
     next visit -> UNCOVERED (no package covering this date)
  Re-upgrade after the first child was closed (unconditional UNIQUE(parent)):
     ERROR:  duplicate key value violates unique constraint "uq_pkg_one_upgrade_per_parent"

=== T8  Dates: window = 30th calendar date inclusive; UTC database date vs clinic-local (Asia/Kolkata) ===
  parent start 2026-11-01 -> Day 30 = 2026-11-30
  purchase at 2026-11-30 10:00+05:30    UTC date=2026-11-30  IST date=2026-11-30  -> upgrade child status=Active start=2026-11-30 charge=Rs 3200.00
  purchase at 2026-11-30 23:59:59+05:30  UTC date=2026-11-30  IST date=2026-11-30  -> upgrade child status=Active start=2026-11-30 charge=Rs 3200.00
  purchase at 2026-12-01 00:30+05:30    UTC date=2026-11-30  IST date=2026-12-01  -> ERROR:  upgrade window closed: today 2026-12-01 is after Day 30 (2026-11-30)
  purchase at 2026-12-01 10:00+05:30    UTC date=2026-12-01  IST date=2026-12-01  -> ERROR:  upgrade window closed: today 2026-12-01 is after Day 30 (2026-11-30)

=== T9  Price arithmetic under three bases (Rs). Catalogue P10=1800, P26=5000 unless the row says otherwise ===
A original fully paid                    | log formula 3200 | cash-paid basis 3200 | S33 catalogue-diff 3200 | net cash kept by clinic if log formula: 5000
B original partly paid (1000 of 1800)    | log formula 3200 | cash-paid basis 4000 | S33 catalogue-diff 3200 | net cash kept by clinic if log formula: 4200
C original complimentary (all waived)    | log formula 3200 | cash-paid basis 5000 | S33 catalogue-diff 3200 | net cash kept by clinic if log formula: 3200
D 1000 paid, remaining 800 waived        | log formula 3200 | cash-paid basis 4000 | S33 catalogue-diff 3200 | net cash kept by clinic if log formula: 4200
E catalogue drifted: P10 now 2000        | log formula 3200 | cash-paid basis 3200 | S33 catalogue-diff 3000 | net cash kept by clinic if log formula: 5000
F catalogue fell: P26 now 1500           | log formula    0 | cash-paid basis    0 | S33 catalogue-diff    0 | net cash kept by clinic if log formula: 1800
G original refunded in full (Q23)        | log formula 3200 | cash-paid basis 5000 | S33 catalogue-diff 3200 | net cash kept by clinic if log formula: 3200

=== T10 Zero-charge upgrade (current 26-visit price below what was billed) ===
  upgrade child status=Active start=2026-11-12 charge=Rs 0.00
  invoice: PAID, lines=1, total=0, payments=0

################  INSTALLING TABLE-LEVEL GUARDS (03_guards.sql)  ################
  guards installed: 6 triggers

=== T2' Same holes, guards installed ===
  child pointing at a STANDARD 26-visit parent                     REJECTED  upgrade parent must be a STANDARD 10-visit package
  (valid upgrade via RPC)  -> upgrade child status=Active start=2026-11-12 charge=Rs 3200.00
  grandchild: an UPGRADE whose parent is itself an UPGRADE         REJECTED  upgrade parent must be a STANDARD 10-visit package
  in-place edit: UPDATE visits_promised 10 -> 26 on a STANDARD     REJECTED  package identity/entitlement columns are immutable
  sell a STANDARD package with a Rs 0 purchase line                REJECTED  zero-price package sale is only permitted for an UPGRADE (Q30)
  ghost package: a package row with NO purchase line               REJECTED  package 892cff7f-7e8d-452f-af43-9075813ebed9 has no purchase invoice line (ghost package)
  visit attached directly to a QUEUED upgrade                      REJECTED  package 0551c850-0c98-4a4e-a2bc-a05a7d60bfe7 is Queued and cannot cover a visit
  upgrade 64 days after parent start, purchased_at = server now()  REJECTED  upgrade window closed: last day was 2026-09-30 (30th calendar date from parent start)
  same, but client SUPPLIES purchased_at = 2026-09-15 (backdated)  ACCEPTED

=== T4c Double-submit of the 10th visit: NAIVE record_visit (no lock) + visit guard trigger ===
  session 1: ERROR:  package 00000000-0000-0000-0000-00000000f001 is Completed and cannot cover a visit
  session 2: COVERED STANDARD 10/10
  -> parent=Completed  child=Active start=2026-11-11  covered visits on parent=10
  -> invariant view: no violations

=== T5c Upgrade vs 10th visit, ONLY the child-guard trigger on (FOR SHARE on the parent row) ===
  A (saves the 10th visit): COVERED STANDARD 10/10
  B (buys the upgrade):     ERROR:  a Queued upgrade needs an Active parent (parent is Completed)
  -> parent=Completed  child=none  covered visits on parent=10
  -> invariant view: no violations
  -> 12 Nov visit: UNCOVERED (no package covering this date)

=== T5d Same race, ONLY the commit-time 'no stranded Queued child' trigger on ===
  A (saves the 10th visit): ERROR:  a Queued upgrade requires its parent to be Active (stranded upgrade child)
  B (buys the upgrade):     upgrade child status=Queued start=NULL charge=Rs 3200.00
  -> parent=Active  child=Queued start=null  covered visits on parent=9
  -> invariant view: no violations
  -> 12 Nov visit: COVERED STANDARD 10/10

=== T5e Same race, ONLY the visit-redemption guard on (shows it is NOT enough for this race) ===
  A (saves the 10th visit): COVERED STANDARD 10/10
  B (buys the upgrade):     upgrade child status=Queued start=NULL charge=Rs 3200.00
  -> parent=Completed  child=Queued start=null  covered visits on parent=10
  -> invariant view: stranded Queued child (parent is not Active)
  -> 12 Nov visit: UNCOVERED (no package covering this date)

=== T6' Daily cap across the handover: child starts the SAME day (offset 0) but visit guard counts per complaint ===
  1st visit on 10 Nov -> COVERED STANDARD 10/10
  2nd visit on 10 Nov -> ERROR:  daily covered-visit cap 1 reached for this complaint on 2026-11-10

=== T7' Manual closure with a Queued upgrade, commit-time trigger installed ===
  close parent, cascade=false -> ERROR:  a Queued upgrade requires its parent to be Active (stranded upgrade child)
  -> parent=Active  child=Queued start=null  covered visits on parent=4
  -> invariant view: no violations
     next visit -> COVERED STANDARD 5/10
  close parent, cascade=true -> closed STANDARD; cascaded to 1 queued child(ren)
  -> parent=Closed  child=Closed start=null  covered visits on parent=4
  -> invariant view: no violations
     next visit -> UNCOVERED (no package covering this date)

=== T10' Zero-price STANDARD sale vs zero-charge UPGRADE, guards installed ===
  zero-charge upgrade: upgrade child status=Active start=2026-11-12 charge=Rs 0.00
  STANDARD package with a Rs 0 line                                REJECTED  zero-price package sale is only permitted for an UPGRADE (Q30)

DONE
~~~~

## Test driver (`04_tests.sh`)

~~~~bash
#!/bin/bash
# Package-upgrade prototype tests. Every line of output was produced by running this script against a
# scratch PostgreSQL 16.15 (TimeZone=UTC, like a default Supabase project). Nothing here touches the clinic's data.
export PGHOST=/var/lib/postgresql/pgtest/sock PGPORT=54329 PGUSER=postgres PGDATABASE=upg
export PGOPTIONS='-c search_path=upgrade_lab'
HERE="$(cd "$(dirname "$0")" && pwd)"
TMP="$(mktemp -d -p "$HERE")"

C1=00000000-0000-0000-0000-00000000c001; C2=00000000-0000-0000-0000-00000000c002
A1=00000000-0000-0000-0000-00000000a001; A2=00000000-0000-0000-0000-00000000a002
K1=00000000-0000-0000-0000-00000000b001   # patient 1, ROUTINE complaint
K2=00000000-0000-0000-0000-00000000b002   # patient 1, REHAB complaint
K3=00000000-0000-0000-0000-00000000b003   # patient 2, ROUTINE complaint
P1=00000000-0000-0000-0000-00000000f001; P2=00000000-0000-0000-0000-00000000f002
CH0=00000000-0000-0000-0000-00000000f0c1

q()   { psql -X -q -At -v ON_ERROR_STOP=0 -c "$1" 2>&1; }
qq()  { psql -X -q -At -v ON_ERROR_STOP=0 -c "$1" >/dev/null 2>&1; }
val() { psql -X -At -c "$1"; }
hdr() { echo; echo "=== $1 ==="; }
lbl() { echo "$1$(q "$2" | head -1 | cut -c1-165)"; }
inv() { echo "  -> invariant view: $(val "select coalesce(string_agg(problem, '; '), 'no violations') from lab_invariant_violations")"; }
states() { echo "  -> parent=$(val "select status from packages where id='$P1'")  child=$(val "select coalesce(string_agg(status||' start='||coalesce(start_date::text,'null'),','),'none') from packages where parent_package_id='$P1'")  covered visits on parent=$(val "select count(*) from visits where package_id='$P1'")"; }
try() { local out; out=$(psql -X -q -At -v ON_ERROR_STOP=1 -c "$2" -c "select 'ACCEPTED'" 2>&1)
        if echo "$out" | grep -q '^ERROR'; then
          printf '  %-64s REJECTED  %s\n' "$1" "$(echo "$out" | grep '^ERROR' | head -1 | sed 's/^ERROR:  //' | cut -c1-120)"
        else printf '  %-64s ACCEPTED\n' "$1"; fi; }

ins_child="insert into packages(clinic_id,patient_id,linked_complaint_id,package_kind,parent_package_id,visits_promised,status,start_date)"
seed_active10() { qq "select lab_reset()"; qq "select seed_pkg('$P1','$K1',10::smallint,'Active','2026-11-01',180000)"; [ -n "$1" ] && qq "select seed_visits('$P1', $1, '2026-11-01')"; }

holes_suite() {
  qq "select lab_reset()"
  qq "select seed_pkg('$P1','$K1',10::smallint,'Completed','2026-11-01',180000)"
  qq "select seed_pkg('$P2','$K2',26::smallint,'Completed','2026-11-01',700000)"
  try "child pointing at a STANDARD 26-visit parent"       "$ins_child values ('$C1','$A1','$K2','UPGRADE','$P2',16,'Active','2026-11-12')"
  echo "  (valid upgrade via RPC)  -> $(q "select purchase_upgrade('$P1', timestamptz '2026-11-12 10:00+05:30')")"
  CH=$(val "select id from packages where parent_package_id='$P1'")
  try "grandchild: an UPGRADE whose parent is itself an UPGRADE" "$ins_child values ('$C1','$A1','$K1','UPGRADE','$CH',16,'Closed',null)"
  try "in-place edit: UPDATE visits_promised 10 -> 26 on a STANDARD" "update packages set visits_promised = 26 where id = '$P1'"
  try "sell a STANDARD package with a Rs 0 purchase line"   "select seed_pkg(gen_random_uuid(),'$K3',10::smallint,'Active','2026-11-12',0)"
  try "ghost package: a package row with NO purchase line"  "insert into packages(clinic_id,patient_id,linked_complaint_id,package_kind,visits_promised,status,start_date) values ('$C1','$A2','$K3','STANDARD',26,'Closed',null)"
  # visit attached to a Queued upgrade
  seed_active10 4; qq "select purchase_upgrade('$P1', timestamptz '2026-11-06 10:00+05:30')"
  CH=$(val "select id from packages where parent_package_id='$P1'")
  try "visit attached directly to a QUEUED upgrade"         "insert into visits(clinic_id,patient_id,complaint_course_id,date,package_id) values ('$C1','$A1','$K1','2026-11-06','$CH')"
  # window bypass through a client-supplied purchased_at
  qq "select lab_reset()"; qq "select seed_pkg('$P1','$K1',10::smallint,'Completed','2026-09-01',180000)"
  mk() { echo "begin; insert into packages(id,clinic_id,patient_id,linked_complaint_id,package_kind,parent_package_id,visits_promised,status,start_date $1) values ('$CH0','$C1','$A1','$K1','UPGRADE','$P1',16,'Active','2026-11-04' $2); insert into invoices(id,clinic_id,patient_id) values ('$CH0','$C1','$A1'); insert into invoice_line_items(invoice_id,package_purchase_id,amount_in_paise) values ('$CH0','$CH0',320000); commit;"; }
  try "upgrade 64 days after parent start, purchased_at = server now()"        "$(mk '' '')"
  qq "delete from invoice_line_items where package_purchase_id='$CH0'"; qq "delete from invoices where id='$CH0'"; qq "delete from packages where id='$CH0'"
  try "same, but client SUPPLIES purchased_at = 2026-09-15 (backdated)"       "$(mk ',purchased_at' ",timestamptz '2026-09-15 10:00+05:30'")"
}

echo "PostgreSQL: $(val 'select version()' | cut -c1-40)   TimeZone=$(val 'show timezone')"

hdr "T1  Declarative identity + shape rules (composite FKs, CHECKs, UNIQUE). No triggers installed."
qq "select lab_reset()"
qq "select seed_pkg('$P1','$K1',10::smallint,'Completed','2026-11-01',180000)"
try "child for a DIFFERENT patient than its complaint"   "$ins_child values ('$C1','$A2','$K1','UPGRADE','$P1',16,'Active','2026-11-12')"
try "child on a DIFFERENT complaint than its parent"     "$ins_child values ('$C1','$A1','$K2','UPGRADE','$P1',16,'Active','2026-11-12')"
try "child at a DIFFERENT clinic than the complaint"     "$ins_child values ('$C2','$A1','$K1','UPGRADE','$P1',16,'Active','2026-11-12')"
try "UPGRADE with visits_promised = 26"                  "$ins_child values ('$C1','$A1','$K1','UPGRADE','$P1',26,'Active','2026-11-12')"
try "UPGRADE with no parent (standalone upgrade sale)"   "$ins_child values ('$C1','$A1','$K1','UPGRADE',null,16,'Active','2026-11-12')"
try "STANDARD that carries a parent"                     "$ins_child values ('$C1','$A1','$K1','STANDARD','$P1',10,'Active','2026-11-12')"
try "Queued STANDARD (queuing stays rejected for sales)" "$ins_child values ('$C1','$A1','$K1','STANDARD',null,10,'Queued',null)"
try "visit covered by ANOTHER complaint's package"       "insert into visits(clinic_id,patient_id,complaint_course_id,date,package_id) values ('$C1','$A1','$K2','2026-11-02','$P1')"
echo "  first upgrade for P1 via RPC:  $(q "select purchase_upgrade('$P1', timestamptz '2026-11-12 10:00+05:30')")"
echo "  second upgrade for P1 via RPC: $(q "select purchase_upgrade('$P1', timestamptz '2026-11-13 10:00+05:30')" | head -1 | cut -c1-120)"

hdr "T2  What the log's constraints do NOT stop (guards NOT installed)"
holes_suite

hdr "T3  Partial unique index 'one Active per complaint' vs the Queued -> Active handover"
seed_active10 10; qq "select purchase_upgrade('$P1', timestamptz '2026-11-05 10:00+05:30')"
CH=$(val "select id from packages where parent_package_id='$P1'")
echo "  start: parent=$(val "select status from packages where id='$P1'")  child=$(val "select status from packages where id='$CH'")"
try "child -> Active FIRST (parent still Active)"          "update packages set status='Active', start_date='2026-11-11' where id='$CH'"
try "parent -> Completed, THEN child -> Active (2 statements)" "update packages set status='Completed' where id='$P1'; update packages set status='Active', start_date='2026-11-11' where id='$CH'"
# Alternative: make "one Active per complaint" a DEFERRABLE exclusion constraint (a partial UNIQUE can't be deferrable)
qq "update packages set status='Queued', start_date=null where id='$CH'"; qq "update packages set status='Active' where id='$P1'"
qq "drop index uq_one_active_pkg_per_complaint"
qq "alter table packages add constraint ex_one_active_pkg_per_complaint exclude using btree (linked_complaint_id with =) where (status = 'Active') deferrable initially deferred"
try "child -> Active FIRST, parent -> Completed second, ONE transaction (deferred)" "begin; update packages set status='Active', start_date='2026-11-11' where id='$CH'; update packages set status='Completed' where id='$P1'; commit;"
qq "update packages set status='Queued', start_date=null where id='$CH'"; qq "update packages set status='Active' where id='$P1'"
try "leave BOTH rows Active at commit (must still fail)" "begin; update packages set status='Active', start_date='2026-11-11' where id='$CH'; commit;"
qq "alter table packages drop constraint ex_one_active_pkg_per_complaint"
qq "create unique index uq_one_active_pkg_per_complaint on packages (linked_complaint_id) where status = 'Active'"
echo "  (separate small experiment, run by hand: one UPDATE that flips both rows succeeded or failed depending on which row the scan reached first)"

race_lastvisit() {
  seed_active10 9; qq "select purchase_upgrade('$P1', timestamptz '2026-11-05 10:00+05:30')"
  ( lbl "  session 1: " "select record_visit('$K1', date '2026-11-10', $1, 1, 2.0)" > "$TMP/s1" ) &
  sleep 0.5
  ( lbl "  session 2: " "select record_visit('$K1', date '2026-11-10', $1, 1, 0)"   > "$TMP/s2" ) &
  wait; cat "$TMP/s1" "$TMP/s2"; states; inv
}
hdr "T4a Two tabs save the 10th visit together. Count-at-read, NO lock (the log's design as written)"
race_lastvisit false
hdr "T4b Same race. [AUDIT] per-complaint lock taken first"
race_lastvisit true

race_upgrade_vs_tenth() { # $1 = upgrade uses lock, $2 = visit path uses lock
  seed_active10 9
  ( lbl "  A (saves the 10th visit): " "select record_visit('$K1', date '2026-11-10', $2, 1, 0, 2.0)" > "$TMP/a" ) &
  sleep 0.5
  ( lbl "  B (buys the upgrade):     " "select purchase_upgrade('$P1', timestamptz '2026-11-10 12:00+05:30', $1, 0)" > "$TMP/b" ) &
  wait; cat "$TMP/a" "$TMP/b"; states; inv
  echo "  -> 12 Nov visit: $(q "select record_visit('$K1', date '2026-11-12', true, 1, 0)" | head -1 | cut -c1-100)"
}
hdr "T5a Upgrade bought while the 10th visit is being saved. RPC reads parent with NO lock"
race_upgrade_vs_tenth false false
hdr "T5b Same race. [AUDIT] upgrade RPC takes the per-complaint lock, then RE-READS the parent"
race_upgrade_vs_tenth true true

hdr "T6  Daily cap (max_visits_per_day = 1) across the parent -> child handover"
for off in 0 1; do
  seed_active10 9; qq "select purchase_upgrade('$P1', timestamptz '2026-11-05 10:00+05:30')"
  r1=$(q "select record_visit('$K1', date '2026-11-10', true, $off)"); r2=$(q "select record_visit('$K1', date '2026-11-10', true, $off)")
  echo "  child coverage starts parent-finish-date + $off day(s):   1st visit on 10 Nov -> $r1"
  echo "                                                          2nd visit on 10 Nov -> $r2"
  inv
done

hdr "T7  Manual closure of the parent while an early-bought (Queued) upgrade exists"
t7() {
for casc in false true; do
  seed_active10 4; qq "select purchase_upgrade('$P1', timestamptz '2026-11-06 10:00+05:30')"
  echo "  close parent, cascade=$casc -> $(q "select close_package('$P1', $casc)" | head -1 | cut -c1-150)"
  states; inv
  echo "     next visit -> $(q "select record_visit('$K1', date '2026-11-07', true, 1)" | head -1 | cut -c1-100)"
done; }
t7
echo "  Re-upgrade after the first child was closed (unconditional UNIQUE(parent)):"
qq "select lab_reset()"; qq "select seed_pkg('$P1','$K1',10::smallint,'Completed','2026-11-01',180000)"
qq "select purchase_upgrade('$P1', timestamptz '2026-11-12 10:00+05:30')"
CH=$(val "select id from packages where parent_package_id='$P1'"); qq "select close_package('$CH')"
echo "     $(q "select purchase_upgrade('$P1', timestamptz '2026-11-13 10:00+05:30')" | head -1 | cut -c1-120)"

hdr "T8  Dates: window = 30th calendar date inclusive; UTC database date vs clinic-local (Asia/Kolkata)"
echo "  parent start 2026-11-01 -> Day 30 = $(val "select review_date_long(date '2026-11-01')")"
for ts in "2026-11-30 10:00+05:30" "2026-11-30 23:59:59+05:30" "2026-12-01 00:30+05:30" "2026-12-01 10:00+05:30"; do
  qq "select lab_reset()"; qq "select seed_pkg('$P1','$K1',10::smallint,'Completed','2026-11-01',180000)"
  printf '  purchase at %-24s  UTC date=%s  IST date=%s  -> %s\n' "$ts" "$(val "select (timestamptz '$ts' at time zone 'UTC')::date")" "$(val "select (timestamptz '$ts' at time zone 'Asia/Kolkata')::date")" "$(q "select purchase_upgrade('$P1', timestamptz '$ts')" | head -1 | cut -c1-82)"
done

hdr "T9  Price arithmetic under three bases (Rs). Catalogue P10=1800, P26=5000 unless the row says otherwise"
psql -X -At -c "
with c(case_name, orig_billed, orig_paid, orig_waived, orig_refunded, p10_now, p26_now) as (values
 ('A original fully paid',               1800,1800,   0,   0,1800,5000),
 ('B original partly paid (1000 of 1800)',1800,1000,   0,   0,1800,5000),
 ('C original complimentary (all waived)',1800,   0,1800,   0,1800,5000),
 ('D 1000 paid, remaining 800 waived',   1800,1000, 800,   0,1800,5000),
 ('E catalogue drifted: P10 now 2000',   1800,1800,   0,   0,2000,5000),
 ('F catalogue fell: P26 now 1500',      1800,1800,   0,   0,1800,1500),
 ('G original refunded in full (Q23)',   1800,1800,   0,1800,1800,5000))
select rpad(case_name,40) || ' | log formula ' || lpad(greatest(0,p26_now-orig_billed)::text,4)
    || ' | cash-paid basis ' || lpad(greatest(0,p26_now-(orig_paid-orig_refunded))::text,4)
    || ' | S33 catalogue-diff ' || lpad(greatest(0,p26_now-p10_now)::text,4)
    || ' | net cash kept by clinic if log formula: ' || (orig_paid-orig_refunded+greatest(0,p26_now-orig_billed))
from c" 2>&1

hdr "T10 Zero-charge upgrade (current 26-visit price below what was billed)"
qq "select lab_reset()"; qq "update package_catalog set price_in_paise = 150000 where visits = 26 and category = 'ROUTINE'"
qq "select seed_pkg('$P1','$K1',10::smallint,'Completed','2026-11-01',180000)"
echo "  $(q "select purchase_upgrade('$P1', timestamptz '2026-11-12 10:00+05:30')")"
echo "  invoice: $(val "select i.status||', lines='||count(l.*)||', total='||coalesce(sum(l.amount_in_paise),0)||', payments='||(select count(*) from payments p where p.invoice_id=i.id) from invoices i join invoice_line_items l on l.invoice_id=i.id join packages k on k.id=l.package_purchase_id where k.package_kind='UPGRADE' group by i.id,i.status")"

echo; echo "################  INSTALLING TABLE-LEVEL GUARDS (03_guards.sql)  ################"
psql -X -q -v ON_ERROR_STOP=1 -f "$HERE/03_guards.sql" 2>&1 | head -5; echo "  guards installed: $(val "select count(*) from pg_trigger where tgname like 'trg\_%' and not tgisinternal") triggers"

hdr "T2' Same holes, guards installed"
holes_suite

hdr "T4c Double-submit of the 10th visit: NAIVE record_visit (no lock) + visit guard trigger"
race_lastvisit false

hdr "T5c Upgrade vs 10th visit, ONLY the child-guard trigger on (FOR SHARE on the parent row)"
qq "alter table packages disable trigger trg_no_stranded_queued"; qq "alter table visits disable trigger trg_visit_pkg_guard"
race_upgrade_vs_tenth false false
hdr "T5d Same race, ONLY the commit-time 'no stranded Queued child' trigger on"
qq "alter table packages enable trigger trg_no_stranded_queued"; qq "alter table packages disable trigger trg_packages_child_guard"
race_upgrade_vs_tenth false false
hdr "T5e Same race, ONLY the visit-redemption guard on (shows it is NOT enough for this race)"
qq "alter table packages disable trigger trg_no_stranded_queued"; qq "alter table visits enable trigger trg_visit_pkg_guard"
race_upgrade_vs_tenth false false
qq "alter table packages enable trigger trg_no_stranded_queued"; qq "alter table packages enable trigger trg_packages_child_guard"

hdr "T6' Daily cap across the handover: child starts the SAME day (offset 0) but visit guard counts per complaint"
seed_active10 9; qq "select purchase_upgrade('$P1', timestamptz '2026-11-05 10:00+05:30')"
echo "  1st visit on 10 Nov -> $(q "select record_visit('$K1', date '2026-11-10', true, 0)" | head -1)"
echo "  2nd visit on 10 Nov -> $(q "select record_visit('$K1', date '2026-11-10', true, 0)" | head -1 | cut -c1-120)"

hdr "T7' Manual closure with a Queued upgrade, commit-time trigger installed"
t7

hdr "T10' Zero-price STANDARD sale vs zero-charge UPGRADE, guards installed"
qq "select lab_reset()"; qq "update package_catalog set price_in_paise = 150000 where visits = 26 and category = 'ROUTINE'"
qq "select seed_pkg('$P1','$K1',10::smallint,'Completed','2026-11-01',180000)"
echo "  zero-charge upgrade: $(q "select purchase_upgrade('$P1', timestamptz '2026-11-12 10:00+05:30')")"
try "STANDARD package with a Rs 0 line" "select seed_pkg(gen_random_uuid(),'$K3',10::smallint,'Active','2026-11-12',0)"

rm -rf "$TMP"; echo; echo "DONE"
~~~~
