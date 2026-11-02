# Clinic Management System — Packages: Post-Handover Decision & Requirements Addendum

**Addendum ID:** PKG-ADD-2026-10-29-01  
**Baseline:** `clinic_packages_handover_2026-10-28.md` — *Clinic Management System — Packages: User-Authored Requirements & Decision Handover*  
**Baseline cutoff:** 28 October 2026, approximately 10:32 IST  
**Addendum coverage:** Subsequent discussion on 28 October 2026 (approximately 17:20 and 18:41 IST) through 29 October 2026 (approximately 15:38 IST)  
**Prepared:** 29 October 2026  
**Document type:** Chronological, provenance-preserving **addendum**, not a replacement for the baseline and not an implementation specification. Read baseline + this addendum together.

## 0. Evidence standard and precedence

This document records changes **after the baseline cutoff only**, plus necessary references to older rules to identify which are retained, challenged, or superseded. It does not silently convert assistant suggestions into user decisions or treat a described doctor conversation as a fresh owner approval where none occurred.

**Status legend**

- **[USER—DECIDED]** Latest user statement clearly selects a product/workflow direction.
- **[USER—PROPOSED / LEANING]** User suggested or was inclined toward it; further commitment or owner approval is not evidenced.
- **[OWNER—REPORTED / BASELINE]** Doctor's policy reported before baseline cutoff; no new doctor conversation is reported in this addendum window.
- **[ASSISTANT—RECOMMENDATION]** Assistant advice; **not** a confirmed business decision.
- **[DERIVED / VERIFIED]** Mathematical consequence of specified rules, not clinic policy.
- **[OPEN]** Important question not settled, or two statements remain in tension.
- **[SUPERSEDED]** Older statement that the later user position replaces or materially revises.
- **[CARRY-FORWARD]** Existing baseline requirement remains unchanged by the recent discussion.

**Authority order for interpreting this addendum:** later explicit user statements > earlier user proposals when inconsistent; direct owner policy is distinguished from user-proposed changes to that policy; technical conclusions are not inferred to be commercial approval. A payment-status label, an overdue-review badge, and a package lifecycle status must never be conflated.

---

## 1. Executive delta: what changed since 28 Oct ~10:32

1. **Second offering's date window:** User moved away from implementing a literal **one-calendar-month anniversary** and now chooses **a fixed 30-calendar-day review period from purchase**, regardless of 28/29/30/31-day months. [USER—DECIDED as software direction; **doctor has not been reported to approve this revised commercial definition**]. This overrides the baseline's *implementation direction* but not the historical fact that the doctor originally called it "one month, not thirty days."
2. **Single review mechanism:** For the short offering, review after **ten Sunday-excluded eligible calendar days** from purchase; for the long offering, review after **30 consecutive calendar days, Sundays included**. The user wants a simple date-calculation function with two branches and a review date fixed when the package is created. [USER—DECIDED direction].
3. **No hard expiry chosen:** User continued to challenge a strict hard-stop for the long offering and preferred the **same soft review/warning** approach for both. A package overdue for review remains usable, subject to normal remaining allowance and explicit human closure. [USER—DECIDED direction]. Repeated acceptance could extend coverage temporally; the assistant cautioned about this, but the user did not adopt hard expiry.
4. **Long offering's finite quantity:** User then proposed **26 covered complaint-visits** as a ceiling for the 30-day offering, bringing it closer to the 10-visit product's usage/exhaustion model. User said they were **inclined** to 26 and wanted a Sunday-count simulation first. Do not relabel this as owner-approved or unambiguously ratified. [USER—PROPOSED / LEANING].
5. **Sunday simulation:** A consecutive, inclusive 30-day period has **four or five Sundays**, **never three**. Consequently it has **26 or 25 non-Sunday dates**. A uniform 26-visit ceiling is a cap, **not** a promise of 26 non-Sunday treatment opportunities within each 30-day review period. [DERIVED / VERIFIED].
6. **Closure authority evolved twice:** Intermediate position: admin could pre-close only for refund; after-review closure might be delegated to reception. **Latest** user statement: "the admin only can close ... and add just a reason," including *refunded* and *made by mistake*, with refund operations separate. This is strong evidence for **admin-only manual closure**, irrespective of review threshold, with a required selectable reason. It conflicts with the earlier receptionist-after-review permission; treat that earlier permission as overtaken unless user clarifies. [USER—DECIDED latest direction; small role-scope ambiguity flagged below].
7. **Closure decoupled from refund processing:** Selecting a reason and closing a package is **not** itself a refund transaction. Closure should not depend on building a refund module. [USER—DECIDED]. Financial adjustments/refunds remain a separate module and separately unresolved.
8. **Admin visibility for overdue coverage:** User proposed an admin-facing notice whenever a **package-covered visit is recorded after its review date**, not merely an overview of overdue packages. [USER—PROPOSED]. This is different from the pre-existing Needs Attention dashboard proposal.
9. **Twice-per-day monthly coverage:** Baseline doctor-reported rule was one package-covered complaint-visit per day for the long offering; additional same-day genuine treatment is normally billed at the regular rate. **No new owner answer revoked that rule.** Simplifying to 26 total visits does not itself eliminate the daily rule. [CARRY-FORWARD / OPEN whether user seeks to replace it].
10. **No 10/26 visits-price-by-procedure change:** Ordinary complaint-visit remains one flat therapy fee regardless of routine procedure count; unrelated exam/lab patient-level billing, invoices/payments, and separate package redemptions retain their baseline meanings. [CARRY-FORWARD].

---

## 2. Chronological record — preserve the actual evolution

### 2.1 28 October, ~17:20 IST — closure and monthly calendar reconsidered

**Closure brainstorm (not yet decided then):** User asked whether any premature closure should be allowed, whether reception should wait until review, whether admin should be exempt, and whether payment received should control closure. The assistant proposed an admin-anytime / receptionist-post-review model, with a reason and financial separation. That was **assistant advice at this point**, not a decision.

**Monthly calendar definition at this point:** User described the long package as running **from purchase date through the day before the same date of the next month** (e.g., bought on the 4th -> through the 3rd of next month). This represented an intermediate **anniversary-month** model and did not yet settle short-month edge cases. [HISTORICAL / SUPERSEDED by later fixed 30-day decision].

**Why the user opposed hard expiry:** Owner-reported clinic practice includes discretionary concessions when the **clinic itself** closes for a festival or other reason; the user wanted to avoid attendance/holiday/extension machinery. User reasoned that the most salient distinction between the products' time calculation might simply be whether Sundays are counted toward review. User strongly favoured the same soft review pathway for both, not an automated hard stop. [USER—PROPOSED progressing to decision].

**Rare twice-daily scenario:** User challenged the cost of implementing distinct treatment-coverage logic for a rare second same-day visit and asked whether it should be deferred under MVP scope. The assistant argued that two deterministic eligibility policies need not imply a general manual toggle, but the user did **not** expressly ratify that implementation proposal. The reported owner rule from the baseline remained the only explicit policy for a second monthly same-day treatment.

### 2.2 28 October, ~18:41 IST — user specifies fixed calendar-day counting and conditional closure

**Date policy selected:** User said to use **exactly 30 days from purchase** for the long offering even if the month is February or has 31 days; use **ten working/eligible days with Sundays excluded** for the short offering. The distinction “business days” must not accidentally mean Monday–Friday: the user specifically wants **only Sundays excluded** from the 10-day review calculation. [USER—DECIDED].

**Short offering remains count-based:** User recognised that two real visits per day can consume ten allowances in as few as five days (subject to genuine treatment and any other coverage rules). Ten-visit exhaustion therefore remains independent of ten-day review. [USER—DECIDED direction].

**Long offering's count question first reopened:** User explicitly asked whether the long offering should have a visit allowance/counter for consistency. No numerical ceiling had yet been chosen at this point. [OPEN as of this message].

**Intermediate manual closure policy:** User indicated **before-review closure only when refund-related**, admin-only; after review, receptionist should be able to close, using the review date as a trust/access boundary. User wanted minimal receptionist discretion but allowed staff to adjudicate whether absence or clinic closure explains unused visits. This was the then-current rule **before the later revision**. [HISTORICAL / SUPERSEDED in part by 29 Oct].

**Admin alert proposal:** User suggested notifying the admin when a **covered treatment is actually recorded after a package's review date** so the admin can independently inspect or intervene. This supplements, not replaces, the pre-existing overdue-package Needs Attention indicator. [USER—PROPOSED].

### 2.3 29 October, ~15:38 IST — 26-visit proposal and closure simplification

**Quantity proposal:** User suggested adding **26 covered visits** to the 30-day offering to cap redemptions and permit automatic exhaustion/completion, the same as the ten-visit product. They reasoned that this may solve indefinite *quantity* and avoid February/variable-month allowance ambiguity. User explicitly called this an **inclination** and requested the simulation, so record as **strong proposal, not final approval**. [USER—PROPOSED / LEANING].

**Requested simulation:** Number of Sundays within 30 consecutive calendar dates for differing purchase dates/weekdays. Assistant supplied weekday classification and calendar-2026 frequency; implications are documented in section 4. [DERIVED / VERIFIED].

**Latest closure rule:** User simplified the package action: **admin goes into the package, closes it, and selects/records a reason**. Examples given: refunded, made by mistake, and other causes. User explicitly said **closure need not be tied to the refund mechanism** and that refunds can remain separate altogether. [USER—DECIDED]. This is the latest specific closure instruction.

**Assistant replies after this point:** Assistant endorsed fixed 26 as a ceiling for MVP, warned about 25 available non-Sunday days in some periods, recommended maintaining one-month second-same-day ordinary charge, and interpreted closure as admin-only anytime. These remain **assistant recommendations/interpretations** unless confirmed by the user. No additional owner call is reported.

---

## 3. Current offering definitions — separate chosen, proposed, and open

| Dimension | Short offering | Long offering | Provenance / caveat |
|---|---|---|---|
| Clinic's original colloquial name | “10 days” | “One month” | OWNER—REPORTED / BASELINE |
| New implementation-facing description | **10-visit allowance with ten Sunday-excluded days until review** | **30-calendar-day review period with a proposed maximum 26 visits** | Short = USER direction; 30 calendar days = USER decision; 26 = USER proposal |
| Coverage unit | One eligible complaint-visit | One eligible complaint-visit; possibly at most one per day | Flat complaint treatment, not individual procedures |
| Maximum covered visit count | **10** | **26 proposed** | Second cap needs approval |
| Date calculation | Count ten calendar dates *except Sundays* from purchase | Count **30 consecutive calendar dates**, Sundays included | Exact day-one/inclusivity boundary needs specification |
| Standard holiday/special closure handling | No automatic deductions/extension accounting | No automatic deductions/extension accounting | Owner may offer concessions; achieved through soft-review discretion rather than holiday engine |
| Review overdue | Warning, not completion/expiry | Warning, not completion/expiry | USER direction |
| Exhaustion | Complete on 10 redemptions | Complete on 26 redemptions **if cap accepted** | Requires real linked visits, not independently drifting counter |
| Second same-day genuine treatment | May consume another of 10 allowances | Owner originally said second should be charged regularly | Unresolved if user wants to unify/eliminate long-product daily limit |
| Manual closure | Yes, latest admin-only + reason | Same | Latest USER statement; see role ambiguity |

### 3.1 The two calculations are not identical business entitlements

The user correctly wants **one shared review mechanism**, not two different calendar and absence engines. However, **different review formulas alone do not erase the product distinctions**: the short product commits to ten covered treatments; the long product historically committed to a period with only one covered treatment for the same complaint per day. A 26-treatment cap makes both finite in quantity but could retain (or, if separately approved, change) the long product's same-day limit.

### 3.2 Recommended implementation vocabulary (not schema approval)

- `review_date`: the **first date requiring review**, or alternatively store `last_normal_day` + clear comparison semantics. Choose one convention to prevent off-by-one warnings.
- `allowance_limit`: 10; optionally 26 when the long cap is approved.
- `redemptions_used`: derive from actual package-covered complaint-visits linked to a specific purchased package.
- `review_overdue`: derived from package review date, independent of lifecycle status.
- `completed`: allowance exhausted; `closed`: explicit staff action. Neither is implied merely because date passed.

These are domain-level concepts, **not final database column names or DDL**.

---

## 4. Calendar/Sunday simulation — facts established after baseline

### 4.1 General mathematical result

For a **30-consecutive-calendar-day** interval counted inclusively from a purchase date, 30 = 4 × 7 + 2. Every weekday (including Sunday) occurs **at least four times**; the first two weekdays of the interval occur **five times**.

| Purchase weekday | Sundays in 30-day window | Non-Sunday dates |
|---|---:|---:|
| Monday | 4 | 26 |
| Tuesday | 4 | 26 |
| Wednesday | 4 | 26 |
| Thursday | 4 | 26 |
| Friday | 4 | 26 |
| Saturday | 5 | 25 |
| Sunday | 5 | 25 |

- **Three Sundays:** impossible for 30 consecutive calendar dates.
- **Five Sundays:** possible, for Saturday/Sunday starts.
- **Six Sundays:** impossible.
- In **2026**, there are **261 Monday–Friday purchase dates** (each leads to four Sundays) and **104 Saturday/Sunday purchase dates** (each leads to five Sundays), or approximately **71.5% versus 28.5%**. This is calendar arithmetic, not operational attendance evidence.
- Example: **31 October 2026 (Saturday)** purchase, with day one included, has normal 30-day window ending **29 November 2026 (Sunday)** and first overdue date **30 November 2026**. Within that window are five Sundays and only 25 non-Sunday dates.

### 4.2 Consequence for proposed 26-visit allowance

A **uniform cap of 26 redemptions** is neither guaranteed to be used during an initial 30-day window nor automatically earned just because the calendar interval contains 26 non-Sunday days:

1. With a Saturday or Sunday start, the period typically provides only **25 non-Sunday dates**. If the clinic does not treat Sundays and the long product permits **only one covered treatment per calendar day**, even perfect attendance cannot redeem all 26 before the review warning.
2. A Monday–Friday start allows **26 non-Sunday dates**, but this is an upper bound on possible treatment dates under ordinary opening assumptions, **not** a guarantee of attendance or treatment.
3. A 26-visit ceiling limits **quantity**, not **elapsed calendar age**. If redemptions remain after day 30, the proposed soft review could let them be used at substantially later dates until admin closure.
4. An actual clinic holiday reduces possible appointments but **does not automatically change** either the fixed review date or the allowance under the intended MVP.
5. The long product's original owner-reported **one-covered-visit-per-day** constraint is independent of its 26-total cap. Removing the daily constraint would be a **new commercial policy** and is **not established** by this simulation.

**Simulation assumptions:** purchase date counts as the first day; calculations are over dates (not 24-hour timestamps); Sunday-only ordinary non-treatment assumption is used **solely for estimating available non-Sunday opportunities**, not as evidence the clinic never treats on Sunday or that holiday policy is encoded.

### 4.3 February issue resolved only in part

Fixed 30 calendar days removes variability from **month-length arithmetic**, because the review period always covers exactly 30 consecutive dates. A 26-visit ceiling prevents unbounded quantity if adopted. It does **not** prove that 26 treatments are possible within those 30 days, and does **not** turn the owner's earlier 'one month' language into verified consent for an exactly 30-day package.

---

## 5. Review system: consolidated desired behavior

**[USER—DECIDED direction]** A review date is created alongside each package purchase. Both regimes use it for a **soft boundary**: a still-active package can remain available after the date, but proceeding should not be silent.

**Retain from baseline:** show package coverage/age/usage on the relevant complaint; after review, staff see the proposed **Continue/Proceed**, **Back**, and **Go to Packages** navigation. Don't place an arbitrary per-visit "turn package coverage off" toggle into ordinary checkout; to terminate general coverage, use the Packages module. The user previously considered informing staff when another complaint is selected; don't block another legitimate complaint. Exact modal/banner arrangement remains open UX.

**Post-cutoff additional user proposal:** produce an **admin attention/audit signal** when a *package-covered visit* is recorded **after** the review threshold. This is more specific than saying “package is overdue.” It should identify package, complaint/patient, covered visit, actor, and date if implemented. The user has not specified whether this is a push alert, dashboard event, activity log or persistent notification; do not invent delivery or read/unread semantics.

**Consequence:** For a 10-visit package with remaining visits, overdue coverage can continue until the count reaches ten or the package is explicitly closed. For a 30-day package with a proposed 26 cap, overdue coverage can continue until the cap is reached or admin closure. Without a hard date stop, neither product is guaranteed to be closed merely by calendar age. [DERIVED].

**Open:** Is acknowledgement alone enough, or must staff give a reason for each overdue treatment? Who can proceed after review? Whether alerts should be generated once per package or per covered visit? The user spoke of receptionist warnings and admin notifications but did not specify all permission/enforcement details. The doctor has not newly approved indefinitely repeated post-review continuation.

---

## 6. Closure and cancellation: latest position and conflict log

### 6.1 Latest selected operational action

**[USER—DECIDED]** From the Packages module, **administrator explicitly closes the package**, choosing/recording a reason. Reasons mentioned by user include **“refunded”** and **“made by mistake”**, plus other as-yet-unspecified reasons. This is an intentional state change, **not a per-visit on/off switch**.

**Recommended factual audit data (not final DDL):** package ID, closed_at, closed_by, reason code, optional notes, remaining allowance when closed (derived or snapshotted for audit), and whether the closure was before/after review. Previously covered visits remain attached; closing a package affects future eligibility only.

### 6.2 Provenance of supersession

| Earlier state (during post-baseline discussion) | Latest state (29 Oct) | Interpretation |
|---|---|---|
| Before review, closure permitted **only if being refunded** | Reason may also be **created by mistake**, without refund mechanics | **SUPERSEDED:** refund cannot be the sole pre-review justification |
| Admin may close early; receptionist may close after review | “**the admin only can close**” | Latest statement **leans clearly admin-only for any manual closure**. If receptionist-after-review is still intended, this needs explicit correction. |
| Closure/refund potentially connected | Closure **separate from refund processing** | **USER—DECIDED:** close action does not depend on implementing refund engine |
| Review date may control closure access | Latest statement does not explicitly restate a date-based permissions exception | Treat **admin-anytime** as working reading, but confirm the receptionist conflict before enforcing a role matrix |

### 6.3 Financial separation — what is and is not decided

- **[USER—DECIDED]** Closing is not automatically a refund, waiver, invoice cancellation, receipt reversal, or debt write-off. The user explicitly wants refund handling separate.
- **[CARRY-FORWARD]** Invoice balance/receipts do not automatically govern entitlement eligibility. Paying fully/partially/not yet is a billing fact, not a built-in closure permission switch.
- **[OPEN]** If staff choose a closure reason literally labeled **“Refunded”**, where is evidence of money actually returned? An unverified reason must not inflate or reverse financial figures. A neutral `REFUND_RELATED_CLOSURE` reason is an **engineering recommendation**, not a user-approved label.
- **[OPEN]** If a package was issued by mistake, its *issued purchase invoice* may also be mistaken. Package closure alone does not fix immutable invoice charges. That correction/void/refund/accounting process was not settled here.
- **[OPEN]** Does closure automatically prohibit all future redemptions permanently? Reopening, correction, and prior-redemption restoration remain outside the selected MVP surface unless explicitly approved.

---

## 7. Total-allowance counting, repeat visits, and overlap

### 7.1 Short (10) allowance

**[CARRY-FORWARD]** One genuinely distinct, eligible, package-covered visit consumes exactly one allowance. Two same-day genuine complaint visits are not barred merely because of their date. If both covered by a ten-visit package, two allowances are used. Once ten are covered, further treatment should not be charged to that package. Duplicate-entry warning remains appropriate.

### 7.2 Long (26 proposed) allowance

**[USER—PROPOSED]** Count real covered visits and automatically complete when 26 have been consumed. This cap would align the **mechanics** (finite count, automatic completion) with the short offering.

**[OWNER—REPORTED / BASELINE]** Only the first treatment for the same complaint on the same day was to receive long-package coverage; a second genuine treatment would be billed normally. This was not revoked in any subsequent doctor conversation. The latest user did not explicitly accept or reject preserving that rule under a 26-visit cap; **leave it open**, rather than silently applying the ten-visit rule to the long product.

### 7.3 One active package per complaint

The baseline's overlap question remains **OPEN**. The assistant repeatedly recommended **at most one active package per complaint**, with sequential renewals, but the user has not explicitly ratified it after asking whether two packages could be purchased. Distinguish **advance purchase** (not yet usable) from **simultaneous active entitlements** and from a second genuine treatment. An allowance cap alone does not resolve how simultaneous entitlements would be selected.

### 7.4 Engineering invariants worth carrying forward (not owner-level policy changes)

- Preserve a direct, auditable relationship from each covered complaint-visit to the **specific purchased package**, rather than inferring historical redemption from complaint alone.
- Prevent last-allowance races and repeat checkout requests from consuming multiple credits accidentally; read-time counts alone do not fix count-then-insert concurrency.
- For the long product, if one-covered-per-day survives, enforce the daily coverage slot without incorrectly banning a real second visit or double-billing a duplicate request.
- Do **not** install an unconditional unique constraint on `(complaint, date)` for visits; the owner permits genuine repeated same-day treatment.
- Package-covered visit, ordinary therapy invoice line, independent exam/lab fee, and discretionary waiver remain distinct facts.

---

## 8. Finance, examination, billing, and other baseline decisions unaffected

These are **[CARRY-FORWARD]**; they did not change in the addendum period:

1. A package is **complaint-specific**; a visit is an occurrence of treatment for that complaint; multiple routine procedures on the visit do **not** each create a separate therapy charge.
2. Ordinary complaint therapy has a **flat charge per complaint-visit**. Package redemption covers eligible complaint therapy, not arbitrary standalone or patient-level charges.
3. Examination and laboratory fees are **patient-level billing amounts**, not proof an examination occurred; they do not require complaint/visit/service records and do **not consume package credits**.
4. Package **purchase invoice** and actual positive **payment records** are different; no or partial payment does **not** automatically block the user's desired package-coverage workflow. Pending/Due/Paid are invoice/payment states.
5. Existing checkout decisions require real visits and specific coverage provenance, and contemplate an **₹0 covered-visit invoice line**; neither a zero invoice line nor a zero receipt is itself an allowance deduction. Waiver and coverage are not synonyms.
6. Refund mechanisms, payment corrections, late/re-exam fee recurrence, imported existing packages, and cross-module audit details are **not resolved by this addendum**.
7. No complex attendance/leave/closure-day calendar or automatically computed holiday extension engine is being reinstated. Sundays have a narrow use in the ten-day review calculation only.

---

## 9. Delta register against baseline IDs

The IDs below refer to the **P01–P30** table in `clinic_packages_handover_2026-10-28.md`. Entries not referenced here retain their baseline status.

| Baseline ID(s) | Addendum change/status as of 29 Oct | Authority |
|---|---|---|
| P02, P14, P28 | From **owner-reported one calendar month** to **user-selected fixed 30 calendar days from purchase** for review computation; owner approval of changed offering definition **OPEN** | USER—DECIDED software rule; OWNER approval OPEN |
| P07 | Ten Sunday-excluded days to review, irrespective of formal business-day convention; day-one/off-by-one still OPEN | USER—DECIDED direction |
| P08, P15 | No absence or holiday ledger; discretionary clinic closure concessions handled under soft-review policy rather than new automatic extension engine | USER—DECIDED direction; owner need remains reported |
| P10, P16 | One shared **soft-review warning, no automatic hard expiry** for both products | USER—DECIDED direction; clinic acceptance of potentially long continuation still OPEN |
| P06 | Short offering still has finite ten-covered-visit exhaustion | CARRY-FORWARD |
| **New A31** | **26 covered-visit ceiling for 30-day offering** — strong proposal, not final commercial approval | USER—PROPOSED / LEANING |
| **New A32** | Exactly 4 or 5 Sundays in 30 consecutive days -> 26 or 25 non-Sunday dates | DERIVED / VERIFIED |
| P09, P26 | Latest **admin-only manual closure with reason**, any time as working interpretation; separate refund mechanism | USER—DECIDED; conflict with earlier receptionist rule flagged |
| P13 | Add proposed admin event for **post-review package-covered visit**, not just overdue package list | USER—PROPOSED |
| P17–P19 | Genuine twice-daily treatment allowed; long-product second visit ordinarily chargeable per baseline owner; 26-cap effect and possible unification **OPEN** | OWNER—REPORTED / BASELINE; new USER proposal |
| P20 | No ordinary arbitrary checkout coverage toggle reaffirmed; automatic rule-based noncoverage still possible for second monthly same-day visit | CARRY-FORWARD |
| P22–P23 | Unpaid/partial usage non-blocking user policy remains; precise activation trigger still OPEN | CARRY-FORWARD |
| P24 | Simultaneous packages / pre-purchase / overlapping entitlements still OPEN; assistant no-overlap recommendation not user approval | OPEN |
| P25, P29–P30 | ₹0 line representation, existing-package import, same-checkout activation remain **OPEN/unchanged** as in baseline | CARRY-FORWARD |

---

## 10. Unresolved items — priority order for next discussion

### A. Business/owner confirmations required

1. **30 days vs calendar month:** Will the owner adopt **exactly 30 consecutive calendar days** as the standard review window, replacing his explicitly reported “one month” language? Sales wording needs to match what is actually provided. **HIGH**.
2. **26 included treatments:** Is the long offering now a **maximum of 26 package-covered complaint-visits**, even though some 30-day windows contain only 25 non-Sunday treatment dates? Is the cap a commercial promise, a safeguard, or just a review/usage number? **HIGH**.
3. **Second same-day treatment under 26 cap:** Does the owner's prior rule (**only one long-package-covered complaint treatment per day; charge the second normally**) remain, or would the clinic allow the second to deduct another of the 26 allowances? **HIGH**.
4. **Post-review discretion:** Is continuing treatment under a not-yet-closed, overdue package permitted based only on receptionist acknowledgement, including potentially weeks/months later, or must there be admin approval? The same question applies to both products; a finite count doesn't enforce a time limit. **HIGH**.
5. **Who closes after review:** The user earlier permitted receptionists to close after review, then said **admin only can close**. Which rule is final? Latest wording strongly favors admin-only, but an explicit answer would remove permission ambiguity. **HIGH (internal user decision)**.

### B. Product and engineering policy details still open

6. **Date boundary:** Is purchase day the first counted day? On which local date does review warning first appear? What if purchase/issuance and first treatment differ? What is the clinic-local time zone? **MEDIUM**.
7. **Sunday purchase:** If bought on Sunday, do ten eligible days start the next Monday; can a visit on purchase Sunday use the entitlement? Sunday-exclusion for *review-date calculation* does not necessarily mean Sundays are forbidden treatment dates. **MEDIUM**.
8. **Closure reasons and financial truth:** Final reason-code list; how to distinguish refund-related termination from evidence a refund actually occurred; erroneous purchase invoice repair; who can view/audit closed packages. **HIGH integrity / MVP scope**.
9. **Post-review admin event:** When should it be raised (on actual committed covered visit only), how often, what fields, how is acknowledged, and whether it is a dashboard feed or a notification? **MEDIUM**.
10. **Automatic completion and final allowance:** What exactly happens when allowance 10/10 or 26/26 is consumed in a checkout transaction? Need atomic status and eligibility, plus replay/concurrent handling. **TECHNICAL**.
11. **Long same-day ordering:** If there are multiple genuine visits on one day, which is “first covered” and can staff correct an erroneous entry? The clinic currently has restricted post-issuance corrections. **TECHNICAL / BUSINESS intersection**.
12. **Overlap/renewal:** Can the same complaint have more than one currently usable package, or only sequenced ones? How is a later sale handled when an older package remains Active and overdue? **MEDIUM**.
13. **Purchase/payment activation:** Exact event that makes a purchased package usable; same-checkout purchase + first treatment; no-payment invoice use; invoice/visit atomicity. **MEDIUM**.
14. **Data migration:** Historical `duration_days`, `expiry_date`, attendance and status schema are not automatically the new model. Existing packages and code need audit before editing. **TECHNICAL**.
15. **Pricing and category-specific offers:** Routine vs rehabilitation prices, allowance defaults by offering, and configuration ownership were not modified by these discussions and should not be guessed. **CARRY-FORWARD OPEN**.

### C. Explicitly **not** reopened by this addendum

- General complaint lifecycle/auto-closure audit.
- Normalization/denormalization choices.
- General examination/lapse/late-fee source rules.
- Payment receipt immutability and invoice-line source-integrity audit.
- Full refunds, extensions, attendance calendar, new waiver engine, correction framework, or multi-package stacking.

These remain in their respective audit modules unless separately selected for implementation.

---

## 11. Safe next-state summary for an agent continuing the discussion

**Treat as currently selected direction:**

- Same complaint-level flat-fee treatment and automatic package-coverage semantics as baseline.
- Short package: ten covered visits, exhausted on visit ten; review after ten dates **excluding Sundays**.
- Long package: use fixed **30 consecutive calendar dates** for review rather than an actual anniversary month **in the proposed software**; obtain owner confirmation because clinic originally said a calendar month.
- Both: fixed review dates at purchase; overdue means advisory warning; no ordinary per-visit package-off toggle; no automated holiday/absence/extension engine.
- Admin-only package closure through an explicit Packages-module action with reason is the latest working rule; cancellation and refund processing are separate.

**Treat as proposed, not settled:**

- Long package **26 covered treatments** cap; it has not been recorded as a doctor-approved final term.
- Admin post-review covered-visit notification mechanics; exact forms of staff warning and Needs Attention priority.
- Any general move to permit two package-covered long-product visits on the same day.

**Do not silently infer:**

- That 30 consecutive calendar days always contain 26 non-Sunday treatment dates (they may contain 25).
- That 26 visits must or can be consumed before review, or that a count cap prevents indefinite elapsed age.
- That 'one month' was already legally/commercially converted to 30 days by the doctor.
- That receptionist closure after review survived the latest “admin only” instruction.
- That closure reason `refunded` means money was returned or books have been adjusted.
- That package purchase is fully paid, that payment gates treatment, or that waiver equals covered treatment.
- That historical GitHub design logs or earlier assistant recommendations constitute currently deployed/accepted package architecture.

**Use this precedence when resuming:** Original baseline as of 28 October ~10:32 IST **plus** the dated changes and conflicts in this addendum. If a rule appears in both, apply the later explicit user direction; retain unresolved owner confirmations and status labels. Do not present the assistant's recommendations as owner or user decisions.

---

## 12. Reference materials

**Canonical baseline (ChatGPT Library):** `clinic_packages_handover_2026-10-28.md` (original document; do not overwrite).  
**Conversation period:** Current package discussion, user messages after ~10:32 IST on 28 October through the proposal/simulation/closure discussion on 29 October.  
**Backend context:** `https://github.com/omsrivastava512/clinic-demo-be`; particularly `study/module_2/decisions/packages.md`, `checkout-and-invoice-lines.md`, `payments-and-settlement.md` and `study/business_logic/packages_logic/README.md`. These files may contain older or competing rules; this addendum does not assert the repository has been updated or deployed.

**End of addendum.**
