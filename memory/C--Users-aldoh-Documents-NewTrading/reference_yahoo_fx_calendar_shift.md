---
name: reference_yahoo_fx_calendar_shift
description: "Yahoo =X FX bars from Tdata::getYahooData are dated one day early during UK summer time (Sunday rows, no Friday rows) — inner joins with futures/stocks drop ~1 day in 5"
metadata:
  node_type: memory
  type: reference
  originSessionId: b21f6538-9f8c-45e5-a7da-87859978dacb
  modified: 2026-10-02T06:57:49.429Z
---

Yahoo FX daily bars (`EURUSD=X`, `CHFUSD=X`, ...) are stamped at London midnight. During BST that is 23:00 UTC on the previous day, so the dates in `intermarket_cache` come out one day early: Sunday rows exist and Friday rows are missing. Futures and stocks (`GC=F`, `^GSPC`) carry normal Mon–Fri dates. Seen 2026-10-02.

**Consequence:** `merge(..., by = "date")` of an FX leg with a non-FX leg drops about 1 row in 5, so a "21-row" 1M change spans about 6 weeks. The first gold-in-EUR ratio showed −7.0% 1M when the correct value was −1.4%.

**How to apply:** for mixed FX/non-FX ratios, use an as-of join (latest FX on or before each date). `get_close()` in `RStudies/reports/macro_context/intermarket.R` does this since 2026-10-02. The root cause is in Tdata `getYahooData` date handling for =X tickers and is not fixed. Check it before trusting any FX-vs-asset daily alignment elsewhere, e.g. ConvertToCHF ([[reference_fx_table_outliers]]).
