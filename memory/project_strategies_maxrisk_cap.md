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

**MaxRisk = per-trade capital POLICY BUDGET (decided 2026-06-16).** The display cap
`pmin(Risk, MaxRisk)` hits ALL strategies. Treated as a hard per-trade budget: a
debit/long/stock/FX trade (BOT/LTO/VALUE/FOREX) whose TRUE risk exceeds budget displays
*capped* on purpose — that clamp is an oversize FLAG, not an error to hide by raising the
ceiling. (Earlier 2026-06-15 framing said "set those HIGH so they never clip" — superseded.)
Edges: MaxRisk `NULL`→600 fallback; `0`/negative→cap disabled for that strategy.

**Sizing base: 1% of the IBKR sleeve, NOT total NLV.** All strategy-tagged Trades live in
the IBKR accounts — overwhelmingly U1804173 (~58.6k CHF), VALUE in U25343478 (~22.2k); the
402k Gonet stock book is untagged (CSV, stocks-only) and does NOT back these budgets. So the
absolute per-trade ceiling = 1% of U1804173+U25343478 (~80.7k) ≈ **800 CHF**. Telling: 1% of
U1804173 alone ≈ 586 ≈ the old 600 default. **Applied values (DB, 2026-06-16, CHF):** OFI 800,
BOT 800 (full 1%, core/proven); WHEEL 700, BPT 700, VALUE 700; CS 600, LTO 600, FOREX 600;
Sharpe2 500, Perso 500; CAL 400, A14 400, Dan 400; TBILL 100, Erreur 100 (near-zero/mistake
floors). Differentiated by conviction × risk-definition. (Stale VALUE/FOREX peak-risk of
~100k/~130k in old audits = legacy full-notional, not real for a 22k account → VALUE is
stop-based.)
