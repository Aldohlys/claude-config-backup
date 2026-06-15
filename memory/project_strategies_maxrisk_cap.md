---
name: project_strategies_maxrisk_cap
description: Strategies.MaxRisk per-strategy Risk budget + signed-delta Risk model generalized to all strategies (TODO
metadata: 
  node_type: memory
  type: project
  originSessionId: 4152f0ec-a745-46da-8951-1e46783a5f4b
---

TODO #38 Phase 5 shipped 2026-06-10 (RReporting commit 74b69b3 on branch
stable/prod). The signed-delta Risk/Return model (per-row Risk = signed change to
live capital; sum(Risk)=0 after close; Return = PnL/MaxOutlay) — previously
BOT-only — now covers OFI/WHEEL/CS/BPT/Sharpe2/Perso. See
[[project_trades_signed_delta_risk]].

**Key facts for future work:**
- New DB column **`Strategies.MaxRisk`** (REAL, base currency = CHF, default 600
  for all 15 rows). Added via ALTER 2026-06-10; captured in data/mydb.sql dump.
- `RReporting/app/R/compute_functions.R`: `cap_strategy_risk()` ceilings the
  per-trade Risk for the six `.GENERIC_RISK_STRATEGIES` — converts native Risk →
  base, compares to the strategy's MaxRisk, clamps back to native. `compute_risk`
  applies it to all six (BOT excluded — exact debit). Budget read via
  `get_max_strategy_risk(strategy)`, **memoised** in `.strategy_risk_cache`;
  call `reset_strategy_risk_cache()` after editing the table mid-session.
- `compute_risk_delta_generic()` classifies Adjust blocks by **position change**,
  not cash-flow sign (these strategies receive premium on open). Worst-case
  default when no clean figure computes = stored MaxRisk
  (`strategy_budget_native()`) — never abs(premium).
- **Going-forward only**: no historical backfill (legacy rows lack reliable
  strike/multiplier). Closed trades keep old Risk/Return.
- **Display-time cap (Summary tab), 2026-06-15 commit 4000dcc**: because the
  entry-time `cap_strategy_risk` is going-forward-only AND excludes some
  strategies (notably LTO + BOT), older/over-budget trades still showed Risk >
  MaxRisk (e.g. LTO 706 = 2806, BPT 573 = 690 vs 600). `reactives.R`
  `summary_open` + `summary_closed` now also ceiling the displayed Risk at the
  per-strategy MaxRisk for **every** strategy: `pmin(Risk,
  vapply(Strategy, get_max_strategy_risk, numeric(1)))`, applied after the
  summarize and BEFORE `compute_return()` so Return + the TOTAL row use the
  capped value. MaxRisk is per-strategy (not hardwired 600). Closed per-row
  Return still prefers stored_return (may predate the cap). See
  [[project_rreporting_summary_realized_pnl]].
- Risk stays **user-editable** (modal pre-fills a proposed value).
- Tests: RReporting/tests/test_risk_resolvers_phase5.R (50) + test_risk_resolvers.R (BOT, 50).

**TODO #75 residuals — #1 + #3 shipped 2026-06-15, only #2 remains:**
- ✅ #1 `modify_trade`: Risk kept editable (not locked); after an edit it re-derives
  the trade's stored `Return` (`recompute_return()`) and warns via showNotification
  when a closed trade's `sum(Risk)≠0` (`check_trade_risk_invariant()`). Both pure
  helpers in `compute_functions.R`, tested by `tests/test_risk_75.R` (18).
- ✅ #3 stock/futures Adjust with a Stop: `adjust_trade()` re-derives absolute
  position Risk from the stop (combined live+adjust legs, via `compute_risk()` stop
  branch) and writes the signed delta `new_total − prior sum(Risk)` — no longer
  waits for the daily `scripts/sync_stop_risk.R`. Adjust modal gained a `Multiplier`
  input (id `adj_multiplier`, non-options; default 1, futures pass contract mult).
- ✅ #2 RESOLVED as "edit in DB Browser" (no in-app editor / Tdata setter built):
  edit `Strategies.MaxRisk` directly in DB Browser for SQLite, then RESTART the app
  (reader memoises in `.strategy_risk_cache`; live edit needs `reset_strategy_risk_cache()`).
  TODO #75 CLOSED 2026-06-15 → archived to docs/TODO_COMPLETED.md.

**MaxRisk = per-trade capital BUDGET, not true risk (key, non-obvious).** The
display cap `pmin(Risk, MaxRisk)` hits ALL strategies, but the cap only does useful
*tolerance* work for the short-premium set (OFI/WHEEL/Sharpe2/CS/BPT) where raw Risk =
large assignment notional. For debit/long/stock/FX strategies (BOT/LTO/CAL/Perso/VALUE/
FOREX) the raw Risk IS the true max loss (premium paid / stop-based / notional), so a low
budget UNDERSTATES it — set those HIGH enough to never clip. Edges: MaxRisk `NULL`→600
fallback; `0`/negative→cap disabled for that strategy. Proposed 2026-06-15 starting
values (CHF, tune to account): OFI 800, WHEEL 1000, Sharpe2 600, CS 600, BPT 600,
Perso 1000, BOT 1500, LTO 2000, CAL 1000, VALUE 5000, FOREX 3000, TBILL/A14/Dan 600,
Erreur 300. Peak-risk audit: VALUE peaks ~100k (stock notional), FOREX ~130k, short-puts
to ~40-77k (uncapped assignment) — these are exactly what the cap tames at display.
