---
name: reference_shiny_first_python_call_latency
description: "Slow/'blocked' Shiny action = first tdata_py call of the session (~45 s Python start-up); app logs live in <App>/app/logs, not RApplication/logs"
metadata:
  node_type: memory
  type: reference
  originSessionId: 1985370b-cb80-4ee0-9651-d8f3aa23babe
  modified: 2026-10-20T15:02:18.345Z
---

A Shiny action that "looks blocked" is often the first Python use of the session: the lazy `Tdata::tdata_py` init (see [[project_tdata_py_lazy_init_startup]]) costs ~45 s before any TWS work. Later calls in the same session skip it.

Case 2026-10-20, RReporting Close of trade 758: 53 s total = 43 s Python start-up + 3 s `isIBAvailable()` + 6 s `getUnderlyingPriceAt`. Close/Adjust of options-only trades call TWS for the underlying price at fill time. Fixed by a `withProgress` bar in `fill_underlying_price` (RReporting `0459de5`); eager init at app start was rejected because it slows every launch.

**Where to look:** the app's own log is `RReporting/app/logs/t-YYYYMMDD.log` (app CWD), which has the RReporting-namespace lines (`close_trade`, etc.). `RApplication/logs/t-YYYYMMDD.log` only has Python/other-process lines — reading it alone made the close look hung when it had finished.

**How to apply:** for "app is slow/blocked" reports, read the app-dir log first and time the gap before "Setting Python LC_NUMERIC locale" / "Python environment initialized". For any new first-use TWS call in a Shiny handler, wrap it in `withProgress`.

Also: IBKR Flex Query CSVs can repeat the header line inside the file (multi-section export); `read_trade()` strips it since RReporting `2681259`.
