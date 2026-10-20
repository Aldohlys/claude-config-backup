---
name: reference_macro_context_same_day_rerun
description: "Re-running macro_context the same day reuses the DB caches (~25 s, no Yahoo re-download); delete today's cache rows to force fresh data (~3.5 min)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 73edccd0-5cb1-4d69-9320-0509f2f70101
  modified: 2026-10-19T18:56:29.890Z
---

`main.R` reads three caches keyed by `cache_date` = today: `macro_context_cache` (prices), `macro_breadth_cache` (S&P 500 breadth), `intermarket_cache` (450-day history, sector map). A second run on the same day logs `Cache hit: ...` for all three and takes ~25 s (verified 2026-10-12, 22-25 s). Only these still go to the network: the events calendar, the VIX futures curve, symbols added to the config since the morning run, and `refetch_empty_closes()` for bars with an empty last Close.

The run overwrites the day's `macro_context_YYYYMMDD.html` and `intermarket_sectors_YYYYMMDD.xlsx`, and deletes/re-inserts today's rows in `macro_context_results`, `macro_intermarket_*`.

**How to apply:** for a layout or render-only change (e.g. testing a branch), just re-run from the RStudies repo root — the cache makes it cheap and the numbers stay identical to the morning report. To force fresh Yahoo data, delete today's rows from the three cache tables first. "Changed since <date>" in the summary band compares against the previous `cache_date`, which on a Monday is Sunday's run (same Friday US close), so Monday reads "no change". Related: [[project_macro_context_incremental_history]], [[reference_r_environment]], [[reference_yahoo_empty_close_bars]].
