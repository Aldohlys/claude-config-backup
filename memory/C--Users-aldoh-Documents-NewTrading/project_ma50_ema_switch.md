---
name: project_ma50_ema_switch
description: "BOT trend gate MA50 switched SMA->EMA (2026-06-06, committed); backtests intentionally left on SMA"
metadata: 
  node_type: memory
  type: project
  originSessionId: bd976eeb-0f33-429a-889f-ea458256c068
---

On 2026-06-06 `calc_ind()` in `RStudies/reports/shared/indicators.R` had `ma50` changed from `TTR::SMA(Close,50)` to `TTR::EMA(Close,50)`. Drives the BOT trend gate (S1 price>MA50, S2 MA50 slope, stage, alignment) in **both** `/analyze` (phases.R) and `swing_scanner` (flow_score.R) — they share `calc_ind`. **Committed** to RStudies `main` as `0223a0c` (alongside a VRP/Term static-legend removal in report.R). Verified: PARR EMA50 = 58.51 = exact TradingView value (SMA50 was 61.77).

**Why:** user charts on TradingView with EMA50; SMA gate diverged ~3 pts (~5%) in trends, risking ALIGNED↔MISMATCH flips on MA-type alone near the line. See [[feedback_classify_breakout_vs_meanreversion]].

**Settled scope (do NOT redo):**
- `bot_strategy_checklist.md` S1/S2 updated to EMA50 + dated note (2026-06-06). NewTrading is untracked/no remote so not committed — see [[project_newtrading_remote_todo]].
- `macro_context/breadth.R` (%-above-50DMA) and `bear_market_check.py` (SPX regime MA50) deliberately LEFT on SMA — standard breadth/regime convention, not the BOT entry gate.
- **Backtest correction:** `breakout_backtest_5y.py` (and `_relaxed_s6`, `_tech_health`, `breakout_backtest.py`) have **no price-MA50 gate** (squeeze + volume + near-20d-high only) — the EMA switch does NOT affect them; their forward-return benchmarks stand. The ONLY script gating on MA50 is `bot_retroactive_test.py` (SMA50, lines 92/156/157). User decided 2026-06-06 to **leave it on SMA** (fixed-sample validation, not the headline benchmark). No re-run pending.
