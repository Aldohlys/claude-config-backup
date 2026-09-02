---
name: reference_testing_tuser_box_internals
description: "Testing a non-exported box-module function: reach it via environment(mod$exported_fn), and point safe_db_connect at a fixture DB with a temp R_CONFIG_FILE"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1e2ceadf-edaa-4f9c-82d1-d1e2a412227d
  modified: 2026-09-02T04:20:38.360Z
---

Two mechanics for testing Tuser logic, both used by `Tuser/tests/test_weekly_unpnl.R`.

**Reaching a module-internal function.** `box::use(portfolio/logic/portf)` exposes only `#'@export`ed names, so an internal helper is unreachable through `portf$name`. But an exported function's enclosing environment *is* the module namespace, which holds everything:

```r
box::use(portfolio/logic/portf)
weekly <- environment(portf$prepare_portf_data_table)$compute_weekly_unpnl
```

Preferable to adding an `@export` purely for testability — the module's public surface stays honest.

**Pointing DB code at a fixture.** `Tdata::safe_db_connect()` resolves the path with `config::get("DB")`, **not** `Sys.getenv("R_DB_PATH")` — so setting `R_DB_PATH` does nothing. Write a temp config and point `R_CONFIG_FILE` at it:

```r
writeLines(c("default:", paste0("  DB: ", db)), cfg)
Sys.setenv(R_CONFIG_FILE = cfg, R_CONFIG_ACTIVE = "default")
```

Then `dbWriteTable()` the fixture snapshots into a throwaway SQLite file. This is how to test history-reading logic without pinning to live dates that rot — cf. [[feedback_stale_tests_are_live_data_not_dates]].

**Running them.** Tuser is not a package, so `devtools::test()` fails and `tests/testthat.R` errors with "Cannot find DESCRIPTION for installed package Tuser" (its existing `test-*.R` files only run under a harness that assumes a package). Standalone scripts are the working convention: `Rscript tests/test_weekly_unpnl.R`, exiting non-zero on failure. `R_BOX_PATH` normally comes from Renviron.site; fall back to the Tuser root so the script runs from a bare session.
