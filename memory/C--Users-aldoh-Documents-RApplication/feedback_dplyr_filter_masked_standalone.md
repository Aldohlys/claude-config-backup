---
name: feedback_dplyr_filter_masked_standalone
description: "RReporting standalone Rscript tests — unqualified filter() resolves to stats::filter (mod:stats on search path), not dplyr"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 47893e77-8f73-4364-903a-3678497da0fa
---

When running RReporting standalone tests via `Rscript tests/test_X.R` (sourcing `app/R/*.R`), unqualified `filter(df, Col == x)` resolves to **`stats::filter`**, not `dplyr::filter` — `find("filter")` returns `package:stats`, and a **`mod:stats`** entry sits at search position 2 (above `dplyr`), shadowing it. The symptom is a dplyr column name reported as "object not found" (`objet 'TradeNr' introuvable`, `'TotalPos' introuvable`) inside `close_trade`/validation, because stats::filter evaluates the bare column symbol.

**Why:** the running Shiny app attaches dplyr so app code works in production, but the standalone Rscript context (renv + box) leaves `mod:stats` on top. So these failures are a **test-harness artifact, not a logic bug** — verify by running the same test against `HEAD~1` (fails identically).

**How to apply:** don't trust/triage RReporting standalone-test failures of the form "column not found in a filter()" as code regressions until you've ruled out the masking. The robust fix is to qualify `dplyr::filter` in the app code (matches the project's explicit-import standard, immune to search-path order) — that's why `trade_operations.R`'s filter calls are `dplyr::filter`. Note these old standalone tests (test_close_trade_validation, test_manual_validation) were also drifted from the current Risk/EventType close logic and were removed; regenerate fresh ones if needed. Related: [[feedback_namespace_masking]] if present.
