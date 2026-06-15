---
name: feedback_dplyr_summarize_self_reference
description: dplyr summarize() evaluates args in order — a new col shadows an input col of the same name for later args
metadata: 
  node_type: memory
  type: feedback
  originSessionId: be45d53a-b30f-4b5b-bcb7-7f9cb9f57ba7
---

In `dplyr::summarize()` arguments are evaluated top-to-bottom, and a summary column shadows any input column of the same name for **subsequent** expressions. So `summarize(Total = sum(base_Total), Total_native = sum(Total))` makes `Total_native` read the just-created scalar `Total` (base currency), NOT the native per-leg `Total` — both end up equal.

**Why:** Caused a real RReporting bug (2026-06-15). `summary_open`'s realizedPnL showed ≈ the whole cost basis because a value reading native `Total`/`Pos` was placed *after* `Total` was redefined to `sum(base_Total)`. Confirmed in isolation: buggy order → the two columns are equal; reordered → distinct.

**How to apply:** Any summarize column that reads an input column must be computed from the raw per-row vector BEFORE you redefine that name in the same `summarize()` call (or rename one of them). Order matters. Same trap is flagged in [[reference_trades_column_rename_playbook]] and bit [[project_rreporting_summary_realized_pnl]].
