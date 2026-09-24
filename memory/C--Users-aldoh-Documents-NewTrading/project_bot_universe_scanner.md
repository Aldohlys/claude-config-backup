---
name: project-bot-universe-scanner
description: One merged tradable universe + the Python BOT breakout scanner; technicals only, options deferred to a second step
metadata:
  type: project
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

**BOT daily workbook (2026-09-23).** `NewTrading/scripts/bot_daily_to_xlsx.py` turns `Reports/bot_daily_<date>.csv` into `Trades/bot_daily_<date>.xlsx`, styled like the old bot_scan workbooks. Sheet "BOT daily" has a tier column, rows shaded by tier, a frozen header plus tier/date/name columns, and an autofilter. Sheet "legend" has the tier rules and counts, plus column definitions parsed from `RApplication/docs/BOT_TOOLS_DESIGN.md` §3. `run_bot_daily.bat` calls it after the R step (the task run and the manual run; the manual run opens the XLSX).

Tiers are the user's choice, "trend first", and are highlighting, not ranking:
- BOT: tradable, trend ≥ 4/6, asym ≥ 1.5
- BOT-: tradable, trend ≥ 4/6, asym 1–1.5
- COUNTER-TREND (blue): trend ≤ 3/6, asym ≥ 2
- WATCH: asym ≥ 1
- LOW: the rest of the tradable rows
- VETO: tradable = 0

**Why:** high-asym rows are mostly pullbacks (trend 0–1/6), not BOT continuation. Rows keep the CSV reading order. `--all` rebuilds every CSV.
