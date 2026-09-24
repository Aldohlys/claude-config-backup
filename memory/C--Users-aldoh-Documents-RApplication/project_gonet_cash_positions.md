---
name: project_gonet_cash_positions
description: "Gonet cash: CASH positions whose balance is rolled forward from the 10.07.2026 ledger baseline; cash events book to a trade's realized PnL (Tdata 5.20.0)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 8392b3e4-5d8b-457c-9eab-5c60ef858317
---

Gonet CHF/USD/EUR cash is real `type=CASH` portfolio positions (Tdata **5.13.0**), and since **5.20.0** (2026-09-22) the balance comes from the `GonetTrades.csv` ledger, not from `GonetPos.csv`.

**The ledger model.** A cash ledger row is one where `sym_ibkr == currency`. Two kinds, told apart by TradeNr alone — never by date:

- **Baseline** (TradeNr 26/27/28, struck 10.07.2026 off the bank statement): TradeNr matches no trade. `init_cost = -init_position` (cash at zero gain).
- **Attributed cash event**: TradeNr matches a non-cash leg — a dividend, coupon, tax refund. `init_position` = cash balance delta, `init_cost` = the amount attributable to the trade as P&L; equal for a dividend (all profit, no basis). Example `15;17.09.2026;USD;USD;117.09;1;117.09;USD`. The QQQ dividend sits **on the baseline date itself** and still counts, which is why attribution ignores dates.

`gonet_cash_balances()`: `balance(ccy) = Σ init_position over cash rows + Σ init_cost over non-cash legs dated after the baseline`. Fixed the hole where a sale cut stock value and cash never rose (SLHN 17.09.2026, 10 816 CHF). `GonetPos.csv` CASH rows now only declare which currency books exist; their `position` is ignored. Re-baselining = replace the baseline rows, don't add a second set.

`gonet_cash_events()` + `gonet_lots()`: income adds to `realized` **only** — basis/shares/avgCost/unPnL untouched, so a dividend never reads as an unrealized gain. Cross-currency events convert at the event date (AMRZ booked CHF, pays USD). `gonet_lots()` returns the `income` component separately.

Cash rows' `unPnL` is still `gonet_realized_fx()` (realized FX on closed legs) and `realizedPnL` stays NA — Convention A, see [[reference_cash_fx_pnl_in_totals]]. 5.20.0 also restored their `TradeNr` (NA since 5.14.0), which had hidden every cash row from the Trade tab.

**How to apply:** book dividends/conversions/deposits as ledger rows; give one a stock's TradeNr to attribute it. Don't reintroduce the balance prompt or read cash from `GonetPos.csv`.

See [[reference_gonet_pnl_and_stats]], [[reference_gonet_tradenr_reused]], [[reference_gonet_foreign_etf_pricing]], [[feedback_user_edits_db_directly]].
