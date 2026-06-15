---
name: project_rreporting_summary_realized_pnl
description: "Summary-tab open-trade realizedPnL comes from the trade's OWN legs, not broker avgCost (avoids FX + fee leaks)"
metadata: 
  node_type: memory
  type: project
  originSessionId: be45d53a-b30f-4b5b-bcb7-7f9cb9f57ba7
---

RReporting Summary tab, Opened Trades (`reactives.R::summary_open` + `trade_operations.R::compute_open_realized`, shipped 2026-06-15 stable/prod commit 4000dcc): realizedPnL for an open trade = `sum(Total) + own_avg_price × net_pos` in native currency, converted once at Last.Date. `own_avg_price` is derived from the opening-direction legs.

**Why:** The earlier `Total − curCost` (with `curCost` from broker `avgCost × pos`) showed non-zero realized P&L on accumulate-only trades from two leaks: (1) **FX** — `Total` was converted per leg at each leg's date while `curCost` used one Last.Date rate, so they didn't cancel in base currency; (2) **cost basis** — IBKR `avgCost` folds in broker fees the recorded `Total` omits. Trade 702 (CRST, GBP): the £2.06 residual was exactly 0.5% **UK stamp duty** baked into `avgCost`. UK stamp duty, French FTT, corporate actions all do this.

**How to apply:** Accumulate-only positions now read realizedPnL = 0; genuine partial closes report the realized gain on the closed volume. Broker `avgCost` feeds only the displayed `curCost`; FX exposure stays in `unPnL`. Don't reintroduce a realized formula based on broker avgCost. Beware the [[feedback_dplyr_summarize_self_reference]] trap (it bit the native-Total here). Risk shown in this view is ceilinged at per-strategy MaxRisk — see [[project_strategies_maxrisk_cap]]. Related: [[project_trades_signed_delta_risk]], [[project_fx_risk]].
