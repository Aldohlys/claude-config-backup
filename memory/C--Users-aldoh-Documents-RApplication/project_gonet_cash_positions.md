---
name: project_gonet_cash_positions
description: "Gonet cash is now CASH portfolio positions with realized-FX-on-closed-trades cost basis (Tdata 5.13.0, 2026-07-10)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 8392b3e4-5d8b-457c-9eab-5c60ef858317
---

Tdata **5.13.0** (2026-07-10): Gonet CHF/USD/EUR cash is now real `type=CASH` portfolio positions, not a prompt-entered Account-tab number.

- Maintained in `GonetPos.csv` + `GonetTrades.csv` (cash rows, unique TradeNrs 26/27/28). The **balance prompt was removed** — CSVs are the single source of truth.
- `getGonet`: cash split out of the IBKR price pipeline (like precious metals), priced at FX spot to base; `mktValue = balance*spot`, stored `currency = base`. `unPnL = gonet_realized_fx()` — average-cost lots per instrument, realizing FX only on **closed / partially-closed** legs: `closed_native_cost * (sell_rate - avg_entry_rate)`.
- `getAccountGonet`: derives CashBalanceCHF/USD/EUR + TotalCashBalance from the CASH positions, excludes them from StockMarketValue, includes their realized FX in UnrealizedPnL. Cash shows in Positions (Cash/Forex sectors) and the Trade tab; `stats_all_gonet` blanks the (distorted) per-unit StartPrice for cash.

**Why realized-FX-on-closed-trades (not a full ledger):** recorded Gonet trades net **−97k USD / −86k EUR** but balances are **+522 / +1763** → the cash is ~99% from unrecorded CHF↔foreign conversions/deposits/dividends, so a true cost basis is impossible. Open-position buys are cash→stock transfers (their FX stays in the stock's valuation), so only closed legs' realized FX is attributable to cash. Past FX is accepted as-is (fresh baseline today).

**How to apply:** maintain cash in the two CSVs; don't reintroduce the prompt. Going forward the user books dividends/conversions/deposits as trades. Detail in Tdata CHANGELOG 5.13.0.

See [[reference_gonet_pnl_and_stats]], [[reference_gonet_tradenr_reused]], [[reference_gonet_foreign_etf_pricing]], [[feedback_user_edits_db_directly]], [[reference_cash_fx_pnl_in_totals]].
