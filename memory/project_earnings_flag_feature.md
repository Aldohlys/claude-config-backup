---
name: project_earnings_flag_feature
description: How the NextEarnings + EarningsInDays feature is wired — uses yfinance (not WSH), refreshed at scanner start
type: project
originSessionId: 1347083d-d9f2-4281-99ce-26cad155e9e3
---
Added 2026-04-20. Feature surfaces "earnings within N days" signal in the swing scanner output so user can avoid TRADE-rated tickers reporting imminently.

## Data source: yfinance (NOT IBKR WSH)
WSH was the initial design target (user has fee-waived WSH subscription) but IBKR returns Error 10276 "News feed is not allowed" — event-data API requires a separate News entitlement the account doesn't have. yfinance (free) works for all US + most European tickers via YahooName resolution.

## Schema
`Tickers.NextEarnings TEXT` (YYYYMMDD) — the next earnings date
`Tickers.EarningsLastUpdate TEXT` (YYYYMMDD) — last refresh attempt

## Python module
`Tdata/inst/python/tdata_py/earnings_utils.py` — `getNextEarningsDate(symbol, ...)`:
- Resolves symbol → YahooName from ticker_db (handles .SW / .PA / .DE)
- Tries `yf.Ticker(yn).calendar` first, falls back to `get_earnings_dates(limit=8)`
- Returns YYYYMMDD string or None

## R functions (Tdata/R/earnings.R)
- `getNextEarningsDate(name)` — single-call wrapper
- `updateEarnings(names)` — batch UPDATE of Tickers rows
- `updateStaleEarnings()` — refresh only where NextEarnings IS NULL OR past. Called at scanner start.

## Scanner integration
`RStudies/reports/swing_scanner/main.R` — after the `out` dataframe is built, left-joins `NextEarnings` from Tickers, derives `EarningsInDays = as.Date(NextEarnings) - Sys.Date()`. Exposes both as columns in CSV/HTML output.

## Known gotchas
- ETFs always return `(no data)` from yfinance (expected — no earnings). Currently retried on every scan (wasteful ~30s but harmless). Fix: add "skip if EarningsLastUpdate < 7 days" to staleness check.
- European stocks with missing/wrong YahooName can't be resolved. Fix per-ticker by updating `Tickers.YahooName` (e.g. `SAF.PA`, `SU.PA`, `OR.PA`, `RO.SW`) and rerunning `updateEarnings()` for those.
- Requirements: `yfinance>=0.2.40` in `Tdata/inst/python/requirements.txt` (added same day). Installed via `reticulate::py_install("yfinance", pip=TRUE)` into r-reticulate env.

## HTML badge (NOT YET DONE)
Visual highlight in scanner report for earnings ≤7d (red) / ≤14d (amber) is deferred — requires edits to `render_html.R` + `template.html`.
