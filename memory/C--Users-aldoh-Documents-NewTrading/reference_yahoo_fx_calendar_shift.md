---
name: reference_yahoo_fx_calendar_shift
description: "Yahoo =X FX bars were dated one day early in UK summer time (Sunday rows, no Friday, empty 'yesterday' bar); fixed in Tdata 5.21.5 (redate_fx_london) — rows stored before 2026-10-16 still shifted"
metadata:
  node_type: memory
  type: reference
  originSessionId: b2c1dc04-7dd9-4dc9-a48d-437eadf5cf6b
  modified: 2026-10-16T17:34:01.855Z
---

Yahoo FX daily bars (`EURUSD=X`, `CHFUSD=X`, ...) are stamped at London midnight, 23:00 UTC the day before during BST; quantmod dates bars by UTC day. Before Tdata 5.21.5 every summer FX series from `getYahooData` was one day early (Sunday rows, Friday missing), and today's unfinished bar (empty close) was dated yesterday, which triggered macro_context's "NO CLOSE FROM YAHOO" banner for all 13 FX pairs on every run.

**Fix (2026-10-16, Tdata 5.21.5):** internal `redate_fx_london()` in `Tdata/R/prices.R`, applied to `=X` tickers right after `getSymbols`. Rows dated today-or-later UTC = live quote → London's current date; earlier rows +1 day when the next London midnight is BST; same-date rows keep the last with a close. Tests: `tests/testthat/test-redate-fx.R`. Note quantmod also appends a separate live-quote row (e.g. 17:13 UTC) — a blanket +1 shift would push it to tomorrow.

**Still shifted:** data stored by date before the fix — `ConvertToCHF` rows from `RReporting/scripts/update_currency_rates.R` / `set_CHF_in_DB.R` in summer months sit one day early (not migrated). Daily caches (`intermarket_cache`, `macro_context_cache`) refetch fully, so they are clean.

**How to apply:** if FX dates look off again, check the Tdata version first. The as-of join in `get_close()` (RStudies `intermarket.R`, added 2026-10-02 for mixed FX/non-FX ratios) is still correct and harmless. Remaining banner symbols after the fix are genuine Yahoo gaps (IS0L.DE, GLTL.L, NUCL.L), see [[reference_yahoo_empty_close_bars]].
