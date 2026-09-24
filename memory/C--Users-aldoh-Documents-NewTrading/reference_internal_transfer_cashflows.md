---
name: reference_internal_transfer_cashflows
description: Every IBKR internal transfer (cash or shares) between U1804173 and U25343478 needs a CashFlow row in BOTH accounts, or the Tuser TWR shows a fake loss/gain; the IBKR activity statement is the reference
metadata:
  type: reference
---

Tuser's TWR (`twr()`, via `accountf$account_extend_data`) is driven by NetLiquidation plus Account.CashFlow rows. The CashFlow rows use NLV = 0, heure `00:00:0n`, one row per currency, and are converted to base by AccountWithConversionRate. Position transfers count as flows too (e.g. 6797.T in at JPY 245,000 on 2026-04-27).

Case 2026-09-23: a JPY 245,000 internal transfer out from U25343478 to U1804173 on 2026-06-12 was never booked. U25's TWR showed −2.2% YTD with a −5.4% step on 12 Jun. After adding the row (−245000 JPY in U25, +245000 JPY mirrored in U1804173), Tuser shows +3.18% against IBKR's statement TWR of +2.83% to 21 Sep. The remaining gap is IBKR's dividend and interest accruals, which are not in our NLV.

**How to apply:** when a TWR shows a step with no market reason, pull the IBKR activity statement CSV (sections "Deposits & Withdrawals", "Transfers", "Change in NAV"; statement TWR under "Net Asset Value") and match it against `SELECT ... FROM Account WHERE CashFlow<>0` for both accounts. The IBKR MCP connector here only covers U1804173, not U25343478.

Related: [[project_u25_fx_hedge_policy]], [[reference_u25_margin_cushion]].
