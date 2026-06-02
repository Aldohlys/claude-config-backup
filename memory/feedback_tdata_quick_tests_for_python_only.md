---
name: Use quick_tests=TRUE for Tdata builds touching only inst/python
description: Tdata's full test suite hits live IBKR via reticulate and can hang 15+ min; quick_tests skips it for Python-only changes
type: feedback
originSessionId: 29cdc5d5-050d-4ed2-a819-c20c72b37017
---
When `/build Tdata` only touches files under `Tdata/inst/python/...` (the
Python `tdata_py` module), always pass `--quick-tests` (or `quick_tests =
TRUE` to `build_package()`). Python-only changes have no matching R test
files (`R/foo.R → test-foo.R`), so quick-tests runs zero tests — fast and
deterministic.

**Why:** Tdata's full `devtools::test()` includes `test-har-volatility.R`
which calls `Tdata::fitHAR("SPY", source = "ibkr")` → Python
`getHistoricalBars(SPY, 200 D, 15 mins)`. Against a slow or busy IBKR
session that call can hang for 15+ minutes, and per
`feedback_reticulate_asyncio_uninterruptible.md` R cannot bound it with
`tryCatch`/`withTimeout`. Build #1 of v5.10.11 on 2026-04-30 spent 30+
minutes hung on exactly that call before I killed it; the v5.10.11 retry
with `quick_tests=TRUE` finished cleanly in ~12 minutes (deploy time only).

**How to apply:**
- Bash form: `... build_package('Tdata', auto_version=TRUE, version_type='patch', quick_tests=TRUE)`
- Slash command form: `/build Tdata auto --quick-tests`
- The change must include at least one corresponding R file (e.g. an exported
  wrapper) to *force* test coverage; otherwise rely on the manual Python test
  workflow before building.
- This is specific to Tdata. Tbasics/Tlogger have no live-IBKR tests, so the
  hazard doesn't apply there.
- Related: `feedback_reticulate_asyncio_uninterruptible.md` (root cause),
  `feedback_no_nan_to_tws.md` (recently added Python-only fix that triggered
  this lesson).
