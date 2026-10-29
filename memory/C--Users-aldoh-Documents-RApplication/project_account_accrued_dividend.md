---
name: project_account_accrued_dividend
description: "account_identity residual on U25343478 = IBKR dividend accruals (in NLV, not stock/cash); Account.AccruedDividend column + backfill from Activity Statement (Tdata 5.22.0, 2026-10-22)"
metadata:
  node_type: memory
  type: project
  originSessionId: e98095da-1bf1-4f7f-800d-dd01e11b9f4e
  modified: 2026-10-26T10:06:59.960Z
---

IBKR's NetLiquidation includes **dividends past their ex-date but not yet paid**. They are neither position value nor cash, so `NLV − StockMarketValue − OptionMarketValue − TotalCashBalance` leaves them over. On U25343478, Japanese dividends (ex-dates 27.08 and 28.09, paid 2–3 months later) left 100–280 CHF over. `scripts/check_data_integrity.R` account_identity flagged 20261008 00:11 (279.8 CHF against a 1%-of-NLV threshold of 278.6).

**Ruled out:** FX. The account is JPY-hedged (about 3.6M JPY of stock against −3.6M JPY of cash), but revaluing the stock at IBKR-implied rates gave the same residual. The remaining ±60 CHF is ordinary timing noise between when NLV and position prices are read.

**Evidence source:** IBKR Activity Statement CSV. It has a `Net Asset Value,Data,Dividend Accruals` line (203.12 CHF at 06.10) and a `Change in Dividend Accruals` section listing posting (Po) and reversal (Re) rows by posting date. In a MULTI file, each account's section starts at `Account Information,Data,Account,<acct>`.

**Fix (Tdata 5.22.0, started 2026-10-22):**
- `getIBKRData` (account.py) reads the `AccruedDividend` tag from `ib.accountValues(account)` after `reqAccountUpdates`.
- New `Account.AccruedDividend` column. It is NULL for Gonet, Live, cash-flow and pre-5.22 rows.
- account_identity subtracts `IFNULL(AccruedDividend,0)`.
- `scripts/backfill_accrued_dividend.R <statement.csv> <account> [--commit]` fills NULL rows inside the statement period. To cover dates after 06.10, it needs a statement that runs past them.
- It also reads the user's **Flex "Dividends" query** export (`Downloads\Dividends (N).csv`). Its "Change in Dividend Accruals" section has the header `"Date","ClientAccountID","AssetClass","Symbol","Description","NetAmount",...`; a posting is +NetAmount and its reversal is −NetAmount. The export names no period, so pass `--from/--to YYYYMMDD` (RApplication 0539692, 2026-10-26). A short-period export has an empty accrual section, which is why `Dividends (4).csv` was useless and `(5)` (Jan–Oct) worked.

**State on 2026-10-26:** 07.10–20.10 filled from `Dividends (5).csv` (17 rows, constant 38,616 JPY open accruals ≈ 201–203 CHF). The 08.10 00:11 residual went from 279.8 to 77.1; October U25343478 has no NULL accruals left. Cross-check: 38,616 JPY × ConvertToCHF equals the live-recorded value on 06.10 (203.16) and 22.10 (201.61), so the export had every posting.

**State on 2026-10-22:**
- Tdata 5.22.0 is built and deployed (fd43727). RApplication d66e4f9 holds the check and the backfill script.
- The column was added with ALTER TABLE to Account and TestAccount. DB backups: `mydb.db.bak_accrued_*`.
- The backfill wrote 191 U25343478 rows up to 06.10. The 07.10–21.10 rows are still NULL, so the 08.10 00:11 row is still flagged. Filling them needs a statement through 21.10.
- Verified live on 2026-10-22 09:51. `getIBKR("U25343478")` stored AccruedDividend 201.59 CHF, close to the statement's 203.12 on 06.10. The row's residual went from 161.0 to −40.6. The empty result from the earlier bare `accountValues()` probe was a probe artefact; the production path is fine.

**Why:** the residual is a real NLV component, not bad data. Widening the tolerance would have hidden real errors of the same size.

**How to apply:** treat a positive NLV residual that steps up on ex-dates as accruals, not a data error. Check that the tag's live value matches the statement's NAV accrual line (it had not been checked against live TWS when this was written).

Related: [[reference_ibkr_activity_statement]], [[feedback_schema_change_regression_tests]], [[reference_account_cashflow_conventions]].
