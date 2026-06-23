---
name: feedback_plotly_lines_sort_by_x
description: "plotly mode=\"lines\" connects points in ROW order — sort by the x column before add_trace"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6dcde008-846b-4bea-b088-7083498ced49
---

plotly `add_trace(mode = "lines"/"lines+markers")` draws the connecting line in **data-frame row order**, not by the x value. If rows aren't chronological the line backtracks and draws long flat/diagonal segments between dates.

**Why:** historical data built with `bind_rows()` over multiple sources — especially `get_or_retrieve_option_historical(..., include_archived = TRUE)` which splices archived/cached rows with fresh ones — is NOT sorted by datetime. The artifact looks like a data/IV bug but it's purely a plotting-order bug.

**How to apply:** sort each per-series subset by the x column immediately before `add_trace`, e.g. `sub <- sub[order(sub[[time_col]]), ]`. Bug fixed 2026-06-17 in [[reference_historical_option_module_in_rpretrade]] (`iv_plot`/`price_plot`/`spread_plot`); `arrange` had been imported since inception but never used. Bars (volume) are immune. Related: [[feedback_ggplotly_autosize]].
