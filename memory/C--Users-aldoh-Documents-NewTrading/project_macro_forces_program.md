---
name: project_macro_forces_program
description: "macro_context redesign program — 7 macro forces chosen (step 1), scenario backtest (step 2) rerun daily 2026-10-20; what held, what is still open"
metadata:
  node_type: memory
  type: project
  originSessionId: cfd01905-af04-48c7-8713-2272d5ea18b3
  modified: 2026-10-20T20:58:25.499Z
---

The macro_context redesign runs forces → scenario → next paths → trades, for 4–20 day BOT trades.
- Reports:
  - `NewTrading/Reports/macro_forces_selection_20261020.md` (step 1, section 8 = daily rerun);
  - `macro_scenarios_backtest_20261020.md` (step 2, section 6 = daily rerun).
- Scripts: `NewTrading/scripts/macro_forces/`.

**Forces (7):** real yields (DFII10), oil (CL=F), dollar (DX-Y.NYB), gold (GC=F), rotation (XLI+XLB)/(XLP+XLV), stress (VIX), leadership (QQQ/RSP). Mean |r| 0.15–0.17 over 2004-26.

**What held on daily data, 2008-26:**
- Goldilocks → QQQ up over 2-4 weeks.
- Liquidity stress → high yield rebounds, stronger with weak credit.
- Reflation → the rotation reverses.
- Stagflation → XLY lags XLP over 4 weeks (few episodes).
- Tightening with a non-steepening curve → SPY weaker over 1 week.

**What did not hold:**
- Debasement → GLD up: an artifact of unadjusted IBKR prices.
- Leadership unwind as a 1-week signal.

**State of play, 2026-10-20:**
- The 7 forces themselves are NOT in the R report. The report still uses the 13 archetype scenarios.
- What did reach the report: the Risk badge (VIX + HY spread), the M7 history figures, and the stagflation ETF fix.
- Rotation and leadership now move together: weekly r +0.38 in 2025, +0.47 in 2026, after −0.3 to −0.5 historically. Recheck quarterly; merge the two forces if r stays above 0.5.
- Still open:
  - the "next paths" step;
  - setting the Risk thresholds from data;
  - a backtest of the 13 R scenarios themselves (M7 uses the 7-force scenarios as a stand-in);
  - checking every scenario's `bot$long/short` ETF lists against its calibrated fingerprint (GDX in stagflation was one mismatch);
  - the VIX ≥ 25 stress banner wording ("Consider waiting…"); the `bot$note` texts were made non-advisory in f17834e.

**Why:** the user is rebuilding macro_context so that each label rests on measured history, not judgment.

**How to apply:** start any next macro_context step from these reports. Treat 7-force numbers as a proxy for the 13 scenarios, not as their measurement.

Related: [[project_macro_context_redesign]], [[project_macro_events_study]], [[reference_macro_proxy_data_traps]], [[feedback_macro_context_no_advisory]].
