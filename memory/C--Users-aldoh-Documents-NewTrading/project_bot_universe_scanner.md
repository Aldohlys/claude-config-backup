---
name: project-bot-universe-scanner
description: "One merged tradable universe + the Python BOT breakout scanner; technicals only, options deferred to a second step"
metadata:
  node_type: memory
  type: project
  originSessionId: e13833e0-75b1-4cd3-b4b2-9a6d425e2a9d
  modified: 2026-10-08T17:46:23.209Z
---

`Strategies/tradable_universe_20260827.csv` is the single canonical universe (137 rows).
Two eligibility columns rather than two files: `book_BOT` (Y / catalyst / blank) and
`book_SELL`. Derived purely from `atr_med5y` + liquidity — no hand-asserted exclusions.
`Strategies/tradingview_universe_20260827.txt` is the TradingView import.

`Strategies/Breakouts/bot_scan_universe.py` scores it and writes
`Trades/bot_scan_<date>.xlsx` (colour-coded rows + legend sheet) plus a gitignored HTML.
`--refresh-atr` pulls 5y and rewrites `atr_med5y`/`book_BOT` — do this every month or two,
otherwise the Gate 4 band freezes.

**Why:** the scanner is deliberately technical-only. Prices from yfinance, no TWS/DB, so no
IV and no VRP. Vehicle choice (outright vs spread) and affordability need a live chain and
are a SECOND step. Two static exclusions were tried and both were wrong — see
[[feedback-no-static-option-judgements-in-screens]].

**How to apply:** add nothing option-derived to the entry screen. When a name looks
untradable on premium, that is a step-2 finding, not a universe exclusion.

Related: [[project-bot-book-design]], [[reference-rstudies-indicators-source]]

**BOT daily workbook.** Since 2026-10-08 main.R writes `Reports/bot_daily_<date>_<hhmm>.xlsx` directly (`RStudies/reports/bot_daily/bot_daily_xlsx.R`): sheet Data = tier + 14 daily columns (ema21_disp_atr added 2026-10-20, see [[project-ema21-extension-column]]), shaded by tier, filter on row 1; Detail = all 95 fields; Legend = tier rules + counts + self-contained definition of every column (`BOT_DAILY_LEGEND`, run stops on an undefined column). No CSV any more. 2026-09-23 → 10-08: CSV in Reports + `NewTrading/scripts/bot_daily_to_xlsx.py` → `Trades/bot_daily_*.xlsx` (script kept for those old files, no longer called by `2_run_bot_daily.bat`).

Tiers are the user's choice, "trend first", and are highlighting, not ranking:
- BOT: tradable, trend ≥ 4/6, asym ≥ 1.5
- BOT-: tradable, trend ≥ 4/6, asym 1–1.5; OR (2026-10-08) asym ≥ 1, daily trend ≤ 3/6 but w_trend_state ≥ 4/6 and close above a rising daily EMA50 ("daily trend paused in intact trend", any asym ≥ 1 capped at BOT-). User's case: NET, flag breakout drifting above the broken level, was mislabelled COUNTER-TREND; user rejected a separate BREAKOUT-HOLD tier ("overcomplicates, need a synthetic view over 100+ names"). Forward test records tier_reason = weekly_trend_hold.
- COUNTER-TREND: REMOVED 2026-10-08 — those rows (daily trend ≤ 3/6, asym ≥ 2, not paused-trend BOT-) are now WATCH. User: "don't catch a falling knife", a BOT long needs positive price action (SAF/MS/KO 10-08); user rejected keeping it as its own tier and chose WATCH over VETO/LOW (VETO = mechanical no-trade with reason, LOW = asym < 1).
- WATCH: asym ≥ 1, none of the above — a level to watch, not an entry
- LOW: the rest of the tradable rows
- VETO: tradable = 0

**Why:** high-asym rows are mostly pullbacks (trend 0–1/6), not BOT continuation. Rows keep the reading order (tradable, asym desc, sess_target_p75 asc). Same tier rule in bot_daily_xlsx.R::bot_tier and bot_fwd common.py::tier_of.
