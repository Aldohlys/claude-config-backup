---
name: reference_live_virtual_account_no_table
description: "\"Live\" is a virtual account with no DB table — per-account-table queries must guard; getAccountLive() rewritten in Tdata 5.18.0"
metadata:
  node_type: memory
  type: reference
  originSessionId: 8392b3e4-5d8b-457c-9eab-5c60ef858317
  modified: 2026-09-03T07:55:19.260Z
---

The "Live" account is assembled at read time by binding the three real account tables (getportfUI: `bind_rows` of readPortfolio("U1804173"/"U25343478"/"Gonet")). There is **no `Live` table** in the DB.

Any function that runs `SELECT ... FROM '<account>'` with the account name (e.g. `compute_weekly_unpnl`) throws **"no such table: Live"** for the virtual account, which the routine Positions render tryCatch turned into "Portfolio data incomplete or unavailable" (bit 2026-07-10). Guard with a `sqlite_master` existence check → return NULL / skip. Note the `Account` **table** does hold `account = 'Live'` rows (written by `getAccountLive()`); it is the per-account *portfolio* tables that don't exist.

**`getAccountLive()` — rewritten 2026-09-03 in Tdata 5.18.0 (TODO #80).** The old description no longer holds; it now:

- writes **`Currency`** on every row. It previously wrote NULL, and `AccountWithConversionRate` joins on `Currency`, so a NULL rate turned every metric into NA — 475 of 1281 Live rows, i.e. the whole Live Account tab. Amounts are converted to base with `convert_to_base_date()` before summing, so the row is self-consistent with the currency it declares.
- picks the day's **last snapshot with a non-zero `NetLiquidation`** per sub-account, replacing `inner_join(..., multiple = "any")`. A cash-flow row carries `NetLiquidation = 0` and its own native currency, so an arbitrary pick could understate Live NLV by a whole sub-account and mix currencies. See [[reference_account_cashflow_conventions]].
- sums the day's **cash flows across every row of the date**, each converted from the currency it was booked in, instead of inheriting whatever the picked row carried.
- is **idempotent**: it deletes the dates it is about to write. It used to append, duplicating a date once per run (314 duplicate rows had accumulated).

Still true: it only recomputes **today** (`date >= s_date`), so it cannot rebuild historical Live rows — past days keep whatever shape the old writer gave them. Historical rows were repaired once by `scripts/fix_live_account_currency.R`; 9 duplicated `(date, heure)` groups whose values genuinely differ were left for manual review, plus one legacy row (`Live 20260831 00:00:01`) with an understated NLV that the read side never picks.

**Testing it:** `getAccountLive()` writes, so never run it against the production DB to try something. Copy the DB, point `R_CONFIG_FILE` at a temp config naming the copy, and seed "today" by cloning a past date's sub-account rows — see [[reference_testing_tuser_box_internals]] for the same config-redirect trick.
