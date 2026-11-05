Final checks are done: the repo is still at the same commit and every citation matches. I'm sending the prototype and test-results file now, and the full audit write-up follows right after it.

## PKGUPGRADE-02 — Package upgrade audit and test results

**Verdict.** The direction is sound, but the proposal can't be built as written. The core shape holds: a second `packages` row linked to the original, a `package_kind` column, 10 + 16 visits, one upgrade per original, and the price kept on the invoice line instead of the package.

The weak part is the handover from the original to the upgrade. You confirmed early purchase in the log, where the upgrade waits in a "Queued" status until the tenth visit. That choice reverses a rule you settled on 2 November. It also produces most of the hard problems I found: the stranded upgrade, the closure case and the ambiguous review date. Separately, the ₹0 floor and the pricing formula each collide with rules already recorded in the repo.

I audited only the upgrade proposal, as you asked, and changed nothing in the repo. Everything marked "recommend" is my recommendation, not one of your decisions. The file `package_upgrade_prototype_and_test_results_2026-11-04.md`, which I just sent, holds the SQL and the verbatim output of every test mentioned here.

### 🔴 Structural

**1. Early purchase contradicts a settled rule.**
- `packages.md` lines 122–125 (S28 D13/D14) say queued advance sales and delayed auto-activation are formally rejected. They also say buying a second package for the same complaint is strictly blocked while one is Active.
- A Queued upgrade is exactly that: a second package bought while the first is Active, switching on automatically at the tenth visit. The log doesn't mention the conflict.
- You have two clean choices. Keep early purchase and record a narrow, upgrade-only exception to D13/D14. Or sell the upgrade only once the original is Completed.
- The second choice costs little. A patient who decides at visit 6 waits four more visits, and a daily patient finishes ten visits in roughly eleven days, well inside the 30-day window.
- Selling only after completion removes the Queued status and, with it, the stranded-upgrade race, the closure problem and the review-date problem described below. The double-save on the last visit and the daily-cap rule still need handling.
- Recommending against early purchase reverses something you confirmed, so I'm putting it to you as a choice.

**2. The tenth-visit handover is the dangerous moment, and the log gives a requirement with no mechanism.** The log says to "protect concurrent attempts and the tenth-visit handover" but not how. I staged each case with two real concurrent database sessions:
- **Double save.** Two tabs save the 10th visit together. Both report "covered 10/10", and the original ends with 11 covered visits.
- **Stranded upgrade.** An upgrade is bought while the 10th visit is being saved. The original goes Completed and the new upgrade is stranded as Queued under it. The patient's next visit is billed at the ordinary rate even though they just paid for 16 more.
- **Flip order.** The "one Active package per complaint" index is checked row by row and can't be deferred. The original must leave Active before the upgrade enters it. A single statement that flips both rows passes or fails depending on which row the scan reaches first.

What worked, all tested:
- A per-complaint lock, taken at the start of both "record a visit" and "buy an upgrade". It is `SELECT … FOR NO KEY UPDATE` on the complaint row, a lock type that doesn't block normal foreign-key checks. The package is re-read after the lock.
- Table-level guards, so a caller that forgets the lock fails with a clear error instead of corrupting data. A visit guard on the package row stops the double save on its own.
- Optionally, a deferrable exclusion constraint in place of the partial unique index, so flip order stops mattering. It still rejects two Active rows at commit.

The file also shows which single guard stops which race. The visit guard alone does not stop the stranded upgrade. The commit-time check alone does, but it makes the tenth visit fail instead of the upgrade. The double-save is not specific to upgrades: `packages.md` line 173 already lists last-allowance race locking as an open item.

**3. The ₹0 upgrade collides with three recorded rules.** You confirmed the floor, so a negative difference becomes ₹0. But:
- Q30 (`packages.md` line 165) says there is no zero-price package sale.
- The checkout page (line 23) allows a zero-total invoice only for a covered-only checkout.
- The payments page (line 11) defines PAID only as actual receipts or that same covered-only exception.

The log notices the zero-total rule (its line 60) but not Q30 or the PAID definition. In the database, a ₹0 package line looks identical to a free standard-package sale. With only the log's constraints, a STANDARD package with a ₹0 line was accepted.

The guard I tested permits a zero line only for an UPGRADE. The rules need a deliberate, narrow amendment: a zero-total invoice is allowed when its only lines are covered visits and/or an upgrade line whose computed charge is zero. It would be closed with "Mark as paid" and no payment row.

**4. Three versions of the price formula are in circulation, and each breaks somewhere.**
- `packages.md` line 193 (S33) says 26-visit catalog price minus 10-visit catalog price.
- Your first wording was minus what the patient paid.
- The log's working formula is current 26-visit price minus the original invoiced charge. The log itself records that you haven't signed off on billed versus paid.

With the original at ₹1,800 and the 26-visit package at ₹5,000, a fully paid original gives ₹3,200 under all three. They diverge here:

| Situation | Log formula | Cash paid | S33 catalog difference | What to notice |
|---|---|---|---|---|
| ₹1,000 of the ₹1,800 paid | ₹3,200 | ₹4,000 | ₹3,200 | Cash-paid counts the unpaid ₹800 twice: ₹5,800 billed for a ₹5,000 course |
| Original complimentary, fully waived | ₹3,200 | ₹5,000 | ₹3,200 | Log formula carries the gift forward: patient pays ₹3,200 in total for 26 visits |
| 10-visit price later raised to ₹2,000 | ₹3,200 | ₹3,200 | ₹3,000 | S33 rule: patient pays ₹1,800 + ₹3,000 = ₹4,800, under the ₹5,000 price |
| Original paid in full, then refunded (Q23) | ₹3,200 | ₹5,000 | ₹3,200 | Log formula credits ₹1,800 that was handed back |
| 26-visit price later cut to ₹1,500 | ₹0 | ₹0 | ₹0 | Floor applies, no refund, as you directed |

I recommend keeping the log's invoiced-charge formula, because it is the only one that survives partial payment and later price changes. It is also the number the checkout page already treats as authoritative: original billed amount is the sum of the immutable issued line snapshots (line 17).

Two precautions follow from the table:
- Read the amount from the original package's own purchase line, not the invoice total, because a counter invoice can also hold an exam fee.
- Block the upgrade if the original's invoice has any refund recorded (once the refund module exists).

Whether a waiver on the original should carry into the upgrade is a business call. If you want the clinic to collect the full ₹5,000 in total, the basis would be billed minus waived minus refunded.

**5. The daily cap and start date at the handover are undefined, and the cap breaks.** The log says the one-covered-visit-per-day rule must continue (its line 124). `packages.md` lines 33–37 define `max_visits_per_day` per package.
- In the test, the original's tenth visit was saved on 10 Nov and the upgrade became Active the same day. A second visit that day was then covered by the upgrade: two covered visits on one date with the cap at 1.
- Both fixes were tested. First, apply the repo's existing T/T+1 start-date rule (`packages.md` lines 130–134) to the upgrade, so coverage starts tomorrow if any package already covered a visit today. Second, count the cap per complaint per day across both package rows. The second visit then fails with "daily covered-visit cap 1 reached for this complaint".
- The log is also silent on whether the upgrade inherits `max_visits_per_day`. I recommend copying it from the original.

**6. The review clock for the chain is undefined (analysis only, not tested).** The log's field sketch stores a review date on each row. The original keeps its short-package date, and the upgrade gets original start + 29.
- Example: the original starts Sun 1 Nov, so its own review date is Wed 11 Nov. The patient buys the upgrade early, on Fri 6 Nov, and by Wed 11 Nov has used only 6 of the 10 visits.
- On Thu 12 Nov the original is still Active and past its own review date. Under `packages.md` lines 46–55 the app would show Review Overdue and a confirmation modal on every visit. A future receptionist would also face a mandatory justification note on each visit.
- All of that would apply to a patient who is legitimately inside a 26-visit course until Mon 30 Nov.
- The rule needed: once a non-closed upgrade exists, the chain's review date is the original's start + 29, and everything that shows Due or Overdue must use it. Derive it; don't store a second copy.
- This case can't occur if upgrades are sold only after the original is Completed.

### 🟡 Safety nets

**7. The log's "backend-enforced" requirements can't be met by its constraints alone.** With no extra guards, the database accepted all of these:
- an upgrade under a STANDARD 26-visit parent;
- an upgrade of an upgrade;
- an in-place edit of a package from 10 to 26 visits;
- a STANDARD package sold with a ₹0 line;
- a package with no purchase line at all;
- a visit attached to a Queued package;
- an upgrade bought 64 days after the original started;
- an upgrade with a client-supplied, backdated `purchased_at`.

A set of small table-level triggers (six in total, also covering the races above) closed all but the last. That one slipped through my own guard, and the fix is to make `purchased_at` server-set.

The declarative parts do work as the log hopes. Composite foreign keys on id + patient + clinic + complaint make "same patient, clinic and complaint as the parent" automatic. The CHECK on kind, size and parent holds, and UNIQUE(parent) blocks a second upgrade.

Enforce at table level because the v11 schema draft's row-level security is allow-all on its tables, including `packages` (`supabase_migration.md` line 1066). Unless direct writes are revoked and everything goes through database functions (RPCs), a rule living only inside an RPC can be bypassed. The repo hasn't decided that yet.

**8. Manual closure of the original while an early-bought upgrade is Queued** (log open item 4).
- Tested: closing the original left the paid upgrade stranded, and the next visit was billed at the ordinary rate.
- Fix: closing the original closes its Queued upgrade in the same transaction under one reason. A commit-time trigger rejects any other path.
- The upgrade's invoice is untouched (S28 D05: closure never silently alters finances). The mistaken-package flows Q20–Q23 handle the money.
- The closure dialog (`packages.md` line 66) should list the upgrade's 16 forfeited visits too.

**9. UNIQUE(parent) creates a dead end.** After a mistaken upgrade is closed, the original can never be upgraded again (tested). That constraint is also what caps the chain at 26, so loosen it carefully. Allow a replacement only if the closed upgrade redeemed zero visits. Otherwise the chain could reach 10 + 5 + 16 = 31.

**10. A Closed original should not be upgradeable.** This is log open item 1, and I agree with its recommendation, for a stronger reason than the log gives. A Closed package may already have been refunded under Q23 while its invoice line still says ₹1,800, so the formula would credit money already returned. That is the refunded row of the table.

**11. Dates and timezone.** The window arithmetic is right:
- Original start 1 Nov gives Day 30 = Mon 30 Nov.
- Purchases at 10:00 and 23:59:59 IST on 30 Nov were accepted, and 00:30 IST on 1 Dec was rejected.
- However, on a database whose session timezone is UTC (the Supabase default), `current_date` still says 30 Nov until 05:30 IST on 1 Dec. An implementation using it would accept a purchase 5½ hours after the window closed.
- Compute "today" in the clinic's timezone; v11 has `clinics.default_timezone`.
- The log's `purchase_date` as a DATE is fine only if computed that way. The repo's `purchased_at` timestamp is safer.
- I did not check the clinic's real database timezone.

**12. Smaller items.**
- **Gap scenario.** Log open item 2 covers visits invoiced on 15 and 18 Nov, then an upgrade on 20 Nov. Existing rules already answer it: no retroactive coverage (`packages.md` line 113) and immutable issued lines and redemptions (checkout page lines 16–18). Line 113 is worded around correcting a mistake, so one confirming sentence from you would make the general principle explicit.
- **Receptionist.** "Receptionist may initiate" describes a future role. The current phase is admin-only (`packages.md` line 90, payments page line 7). Either way the price must be computed on the server (Q29).
- **Combined invoice.** Q19 says a counter checkout issues one combined invoice with distinct lines, while the log speaks of an "upgrade invoice". An upgrade bought alongside a visit and an exam fee should go on that same invoice.
- **Names.** The log's `coverage_start_date` and `purchase_date` are the repo's `start_date` and `purchased_at`. The log omits `max_visits_per_day`.
- **Price snapshot.** S28 D01 (`packages.md` line 9) says a purchase snapshots quantity and price. With price only on the invoice line, that holds if every package has exactly one purchase line, which a unique constraint enforces.
- **Tracker workbook.** It has no upgrade rows, so there was nothing to reconcile there.

### 🟢 Performance
No concern. An upgrade adds one package row, one invoice line and a few small indexes on tiny tables. The cost is complexity, not speed.

### Decisions I need from you
Each has my recommendation. None is a decision until you make it.
1. **Early purchase.** Keep it with an upgrade-only exception to S28 D13/D14, or sell the upgrade only after the original is Completed? I recommend after Completed for the MVP.
2. **₹0 floor.** Approve the narrow amendment to Q30, the zero-total rule and the PAID definition, or block the ₹0 case and handle it manually? I recommend the amendment, since you directed the floor.
3. **Pricing.** Confirm the invoiced-charge basis. Decide whether a waiver on the original (complimentary or partly waived) carries into the upgrade. Approve blocking an upgrade when the original's invoice has a refund.
4. **Closed original.** Confirm it is not upgradeable.
5. **Replacement upgrade after a closed one.** Accept the dead end, or allow a replacement only when the closed upgrade redeemed zero visits?
6. **Daily cap.** Does the upgrade inherit `max_visits_per_day` from the original? I recommend yes, copied at purchase.
7. **Chain review date.** Confirm it is the original's start + 29 once a non-closed upgrade exists.
8. **Closing the original (only if early purchase stays).** Should closing it also close its Queued upgrade in the same step, with the invoice left to the Q20–Q23 flows?
9. **MVP inclusion.** With after-completion-only, the remaining work is a smaller guard set plus the lock on the last-allowance save that every package needs anyway. That fits your "reasonably contained" condition. With early purchase it is a larger job. Does that change your call?

### What I verified, and what I didn't
**Verified by running it.**
- I built a throwaway model of only the upgrade on PostgreSQL 16.15, with a UTC session timezone and default isolation. I ran ten test groups, staging the races with two real concurrent sessions.
- I then extracted the code from the sent file into a fresh database and re-ran it. The output matched the file apart from random IDs. The price table was computed in SQL.
- Repo claims were checked against the current tip of main, commit 9670880, which includes your 3 Nov call log. Line numbers throughout are from that commit.
- The model follows the current decision pages, not the July-era v11 schema. v11 still has expiry fields and no payments or invoice-line tables.

**Not verified.**
- The clinic's real database, its timezone and its RLS roles.
- The refund module, beyond its arithmetic effect.
- The frontend.
- The review-date rule in finding 6, which is analysis only.
- The "sell only after Completed" variant as its own configuration. That it removes the Queued problems follows from there being no Queued rows.
- Anything deployed.

The reference functions are illustrative, not production code. Nothing was staged, committed, or written to the repo or the project.

This chat now holds a decision queue, several corrections and some open gaps. Want me to start a running log, for example `package_upgrade_audit_log_2026-11-05.md`? It would have one entry per finding, each recording Source (PKGUPGRADE-02), Old position, New position, Status, Where it lands and an IST timestamp. I haven't created it, and I haven't saved the prototype file to the project; say the word and I will.

Sources:
- [packages.md](https://github.com/omsrivastava512/clinic-demo-be/blob/9670880cdbd7be86fa20dba16130f6abad986532/study/module_2/decisions/packages.md)
- [checkout-and-invoice-lines.md](https://github.com/omsrivastava512/clinic-demo-be/blob/9670880cdbd7be86fa20dba16130f6abad986532/study/module_2/decisions/checkout-and-invoice-lines.md)
- [payments-and-settlement.md](https://github.com/omsrivastava512/clinic-demo-be/blob/9670880cdbd7be86fa20dba16130f6abad986532/study/module_2/decisions/payments-and-settlement.md)
- [S33 call transcript discussion log](https://github.com/omsrivastava512/clinic-demo-be/blob/9670880cdbd7be86fa20dba16130f6abad986532/study/module_2/decisions/history/package_call_transcript_discussion_log_2026-11-03_2231_IST.md)
- [supabase_migration.md (v11 schema draft)](https://github.com/omsrivastava512/clinic-demo-be/blob/9670880cdbd7be86fa20dba16130f6abad986532/docs/schema/supabase_migration.md)
- The attached upgrade log, `package_upgrade_design_discussion_log_2026-11-04_0220_IST.md` (your file; no URL)