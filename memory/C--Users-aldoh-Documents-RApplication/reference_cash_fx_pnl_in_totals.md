---
name: reference_cash_fx_pnl_in_totals
description: Tuser Positions/Trades totals include cash-FX P&L (Convention A) and intentionally differ from TWS Unrealized P&L
metadata: 
  node_type: memory
  type: reference
  originSessionId: dec2ae77-46ba-4485-aca5-cc1b13bfadb2
---

IBKR's `Account.UnrealizedPnL` (stored per snapshot, the TWS "Unrealized P&L" figure) = **stock/option positions only**; it EXCLUDES FX P&L on foreign-cash balances (that lands in NetLiquidation). Verified 2026-07-10 on U25343478: stocks 116.5 CHF = IBKR figure exactly.

Tuser's Positions tab (`portf$prepare_portf_data_table`) and Trades-PnL tab (`symf$stats_all`) totals INCLUDE cash FX P&L — the live short-FX exposure of each foreign-currency book (user rebalances/covers a currency short when it drifts). So the app Total intentionally differs from TWS; both tables carry a caption saying so.

Cost basis for a foreign-cash balance = **Convention A**: amount-weighted FX rate across ALL open trades in that currency (FX conversions + foreign-currency stock purchases), via `Tdata::weighted_cash_cost_basis` / `getCurrencyTradesForBasis` (`cash.R`, 5.12.1). Previously anchored to a single explicit FX trade's rate applied to the whole balance (overstated JPY −206 vs correct −85), and left cash borrowed purely via stock purchases (EUR/CAD/KRW — no FX trade) at 0.

CASH rows are built at snapshot-WRITE time in `getIBKR()` (`account.R` ~L599), persisted in the account table — so a cost-basis change only shows after a fresh `getIBKR(account)` snapshot, not on reload. Rows with no explicit FX trade have `TradeNr = NA`; both display functions handle CASH by currency so the NA-TradeNr filter never drops them. See [[project_fx_risk]], [[reference_greeksnet_contract]], [[reference_untagged_cash_hidden_in_summary]].
