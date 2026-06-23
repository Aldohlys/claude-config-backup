---
name: IVolatility Backtest API Plus — pipeline notes
description: Working knowledge of IVolatility REST API quirks, output layout, and the existing probe script
type: reference
originSessionId: bc7eb8bf-68dc-4f7b-b7cb-a86ad3ffcd75
---
User had a 7-day trial of **IVolatility Backtest API Plus** ($149/mo plan; trial ended ~ early May 2026). If renewed, the existing pipeline still works.

### API quirks (undocumented)

- Base: `https://restapi.ivolatility.com`
- Auth: query param `?apiKey=...`. No HTTP header auth.
- Date format: `YYYY-MM-DD`.
- **Async job pattern**: range queries return `{status:{urlForDetails:...}, data:[]}` immediately. Hit `urlForDetails` to poll a metadata endpoint that lives at `/data/info/<uuid>`. That returns a JSON array `[{meta:{status:"PENDING"|"COMPLETE"}, data:[{fileSize, urlForDownload}]}]`. Poll until `status=COMPLETE` and `fileSize > 0`, then GET `urlForDownload?apiKey=...` for the gzipped CSV. Up to 3 minutes for 450k+ row IVS pulls.
- IVS endpoint is parameterised by **moneyness (OTM%)**, not delta — but each row carries a `delta` column, so 25Δ extraction is a `groupby(date, Call/Put)` of the row whose `|delta - 0.25|` is minimal.
- IVX endpoint columns: `7d IV Call`, `7d IV Put`, `7d IV Mean`, `14d IV Call`, ..., `1080d IV Mean` (with spaces, not underscores).
- **Trial earnings endpoint returns 0 rows** despite Plus marketing claiming earnings data — must hand-curate dates per ticker.

### Existing pipeline files (in `RStudies/`)

- `ivolatility_probe.py --symbol XXX` — fetches all 4 datasets per ticker into `ivol_probe_out/<XXX>/`
- `IVvsSpotLag.R` — multi-ticker event-study harness; defines `EARNINGS` per-ticker, `load_ticker()`, `build_paths()`, `run_study()`, `summarise()`. Guard `SUPPRESS_PANEL` lets it be sourced without auto-running.
- `catalyst_study.R` — post-earnings catalyst variant
- `catalyst_pooled.R` — pooled across tickers, tests both directions
- `fomc_study.R` — FOMC pre/post window with negative horizons
- `robustness_WMT.R` — knob-sweep template

### Data on disk (17 tickers, 2021-01-04 to 2026-04-27)

Single names: AAPL, DOW, GOLD, JNJ, NFLX, OXY, TGT, WMT, XOM
ETFs: SPY, QQQ, IWM, XLK, XLE, XLF, XLV, XLU

Per ticker, ~20MB of parquets including `ivs_full.parquet` (450k rows of moneyness×tenor surface). All under `ivol_probe_out/<TICKER>/`.

### Hard-curated earnings dates

Are inside `IVvsSpotLag.R` at the `EARNINGS <- list(...)` block. Sources verified during April 2026 session:
- WMT: `stock.walmart.com/financials/quarterly-results/`
- TGT, DOW, GOLD, XOM, JNJ, OXY, NFLX: `alphaquery.com/stock/<TKR>/earnings-history`
- AAPL: Apple IR

ETF entries are empty `as.Date(character(0))` since indexes have no earnings exclusion.

### Other notes

- IVolatility's `Options Real-Time API` ($149/mo) provides intraday — needed if pivoting from EOD to intraday flow studies.
- For 25Δ surface alternatives: **Polygon.io Options Advanced** ($79/mo) gives raw chains, requires Black-Scholes inversion + smile fit; **OptionMetrics IvyDB via WRDS** is free for academics and gold-standard.
