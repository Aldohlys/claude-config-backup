---
name: reference_tdata_getyahoodata_name_mapping
description: Tdata::getYahooData maps only 3-letter names to Tickers.YahooName and returns rows under the Yahoo name (FXC -> FXC.SW); longer names (EUR.CHF) go to Yahoo unmapped and come back empty
metadata:
  node_type: memory
  type: reference
  originSessionId: 699892b4-1a21-4a3f-a5a9-7224c9a599df
  modified: 2026-10-08T21:37:48.745Z
---

`Tdata::getYahooData(tickers)` with a character vector treats only names matching `^[A-Z]{3}$` as candidates for `getYahooName()` (Tickers.YahooName). Consequences found 2026-10-08:
- `FXC` is fetched as `FXC.SW` and the returned rows carry `ticker = "FXC.SW"`, so a caller that checks "is FXC in my data" never finds it (macro_context's cache re-appended it on every run, four copies).
- `EUR.CHF` (YahooName `EURCHF=X`) is sent as is and always returns empty closes.
- Passing a data.frame with a `YahooName` column uses those names directly.

**How to apply:** callers that start from universe Symbols should resolve Tickers.YahooName themselves and rename rows back (macro_context `fetch.R::fetch_yahoo_as_universe()`, RStudies 4dd1438). Tdata itself is unchanged. Related: [[reference_tdata_vs_ibkr_mcp]], [[reference_ticker_rename_trap]].
