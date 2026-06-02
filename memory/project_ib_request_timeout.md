---
name: IB.RequestTimeout caps stuck ib_async requests; one-line fix for half of TODO #52
description: ib_async exposes IB.RequestTimeout as a class attribute (default 0=no timeout); setting it once at module init puts a global ceiling on every util.run() call without per-callsite asyncio.wait_for wrapping
type: project
originSessionId: 67df7b93-72b2-4e92-a5f8-7541ce5b225c
---
`ib_async`'s `IB` class has a class attribute `IB.RequestTimeout` (default `0` = no timeout). Every `util.run()` / `ib.run()` call inside ib_async wraps awaitables with `timeout=IB.RequestTimeout`, so setting it once at module init applies a global ceiling to every TWS request.

**Why this matters for Tdata:** TODO #52 catalogues unbounded `ib.req*` callsites that hang when TWS is up but degraded (e.g. `reqSecDefOptParams` after a 1100/1102 connectivity blip — no `securityDefinitionOptionParameterEnd` ever arrives). R-side `tryCatch` cannot interrupt those (memory `feedback_reticulate_asyncio_uninterruptible.md`). The lib's own `IB.RequestTimeout` is the simplest fix — one config line in `tdata_py/_core.py` after the `CONFIG` global, reading `CONFIG.get("ibkr", {}).get("request_timeout", 60)` and setting `IB.RequestTimeout = float(...)`.

**Shipped in Tdata 5.10.12 (2026-04-30):** `_core.py` sets `IB.RequestTimeout = float(CONFIG.ibkr.request_timeout)` (default 60s) right after the `CONFIG` global loads. Built and deployed to all 7 active Tdata install locations (RLibrary + 6 renv apps). Runtime confirmed: `ib_async.IB.RequestTimeout = 60` after `library(Tdata)`. Out-of-scope locations: R-4.5 Tuser sandbox + system-library `Program Files/R/R-4.4.3/library` (per `project_tdata_install_locations.md`).

To verify on a fresh session: `Rscript -e ".libPaths('C:/Users/aldoh/Documents/RLibrary'); library(Tdata); cat(reticulate::import('ib_async')\$IB\$RequestTimeout)"` — must print `60` (or whatever override `config.yml > default > ibkr > request_timeout` carries). Without forced libPaths, an active renv may shadow the global library and load an older Tdata that doesn't have the patch — so version + libpath always check before claiming verification.

**How to apply:**
- For new ib_async-based Python code, prefer setting `IB.RequestTimeout` once at init over wrapping each `ib.req*` call in `asyncio.wait_for`. It's lib-native and applies uniformly
- Per-callsite `asyncio.wait_for` is still needed for the `reqContractDetailsAsync` / `reqSecDefOptParamsAsync` cases that lack a callback-end timeout in ib_async — TODO #52's residual scope after the global ceiling lands
- `IB.RequestTimeout` is a class attribute, so set it before constructing `IB()` instances. Setting it after instantiation works (instances read the class attr live) but make it a startup invariant
- Pacing (`ib.sleep(N)` between requests) is orthogonal — solves rate-limit / peer-close, not missing-callback hangs. Both are needed
