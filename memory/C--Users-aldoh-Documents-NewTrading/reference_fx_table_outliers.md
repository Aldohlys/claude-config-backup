---
name: reference_fx_table_outliers
description: ConvertToCHF can hold one-day wrong rates (another currency's rate, or inverted); on U25 they corrupt the computed StockMarketValue; detection rule and fix
metadata:
  type: reference
---

On 2026-09-23 four one-day outliers were found in ConvertToCHF and replaced with Yahoo `<CCY>CHF=X` closes (for weekend dates, the Friday close):
- GBP 2026-04-21 0.577 (≈ the CAD rate)
- GBP 06-13 1.683
- GBP 09-05 1.671
- JPY 08-09 195.96 (inverted)

ConvertToUSD was clean.

**Cause (found 2026-09-23, fixed in Tdata 5.20.9):** the IBKR branch of `getIBKRActiveCurrencyValues()`. It runs in the daily portfolio update but writes only when Yahoo has no rate for the day yet, mostly at weekends. `tdata_py$retrieveCurrencyPairs` already returns USD per unit; R inverted the inverted pairs again and DIVIDED by CHF-per-USD. The log "Result {...}" lines match each bad value exactly.
- 2026-04-21 came from the Yahoo path instead (a bad intraday quote, not reproducible).
- Since 5.20.9: `ibkr_fx_rows()` does the correct math, and `fx_drop_implausible()` refuses any FX write more than 5% from the last stored rate, in both paths and both tables.

**Why it matters:** for U25343478 the Account NetLiquidation comes from IBKR, but StockMarketValue/UnrealizedPnL are computed in Tdata with these rates. A bad rate therefore shows up as NLV − SMV − OptionMV − cash ≠ 0. The 2026-08-09 06:59 row carried SMV 637.7M. On 2026-04-20 (15:30–22:00) three rows had Japanese holdings left in JPY (SMV 1.23M) even though the rates were fine. The Gonet rebuilds also use ConvertToCHF.

**How to apply:**
- Detection: a rate that is >5% off both its neighbours while the neighbours agree within 3%.
- Repair of an Account row: recompute SMV/UPnL from the position snapshot written in the SAME run. The Account row is written ~2 s BEFORE that snapshot, so pair with the next snapshot within ~15 s, not the previous one.
- Check: the residual NLV − SMV − cash should fall to tens of CHF. If a row is already consistent, it used a good live rate, so leave it (as for 2026-09-05).

Backups: `data/mydb_before_u25_apr20_fix.db` (before the rate fix), `mydb_before_fx_outlier_fix.db` (after the rate fix, before the row recompute).

Related: [[reference_internal_transfer_cashflows]], [[reference_gonet_account_history]].
