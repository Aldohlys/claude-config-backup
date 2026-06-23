---
name: reference_historical_option_module_in_rpretrade
description: "The \"Implied Volatility History\" / historical-option charts live in RPreTrade, NOT RReporting"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6dcde008-846b-4bea-b088-7083498ced49
---

The "Implied Volatility History", "Price History", "Volume History", "Bid-Ask Spread" charts are rendered by `Tuser/studies/view/historicalOptionUI.R` and wired into **RPreTrade's Historical Options tab** via `RPreTrade/R/historical_options.R` (`historicalOptionUI$server(..., enable_iv_plot = TRUE)`), imported in `RPreTrade/server.R`.

**RReporting does NOT use this module** — it only loads `journalUI` + `accountUI`. If a screenshot of these charts is attributed to RReporting, it's actually RPreTrade.

It's a box module read live from `R_BOX_PATH=Tuser`, so edits need only an app restart — no build/deploy. Module exposes both `ui()` and `compact_ui()`. See [[feedback_plotly_lines_sort_by_x]].
