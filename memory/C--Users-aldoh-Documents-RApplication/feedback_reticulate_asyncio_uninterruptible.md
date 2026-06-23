---
name: feedback_reticulate_asyncio_uninterruptible
description: R-side error handling is useless once execution enters a blocking async Python call via reticulate — the timeout must live on the Python side
type: feedback
originSessionId: efdcf942-8e75-4a7f-a442-14c3e668ac27
---
R `tryCatch` / `withCallingHandlers` / `setTimeLimit` **cannot interrupt a Python coroutine** that is awaiting a future via reticulate. Once R enters `tdata_py$<anything>()` and the Python side is inside `asyncio.wait_for(...)` (or any awaitable) that never resolves, R is blocked inside the reticulate bridge and no R-level supervision can free it.

**Why:** On 2026-04-20 to 2026-04-21, `update_missing_iv_safe.R` wrapped every `getVolMetrics(sym)` call in `tryCatch`. TWS hit a brief Error 1100/1102 server-switch outage mid-request for DHR. The Python `ib_async` call for DHR never received a response after the reconnect. The R process hung for **5+ hours** on that one ticker, producing no progress, no error, no tryCatch fire — until manually killed. Verified the same Python process was still alive and logging connection events while R was stuck.

**How to apply:**
- Never rely on R-side `tryCatch` to bound latency of a reticulate call that delegates to `asyncio`. Assume the call is uninterruptible once control crosses into Python.
- Any batch job that iterates over IBKR calls must ensure the Python side has per-request `asyncio.wait_for(..., timeout=N)` (or equivalent) before the coroutine is awaited. Without that, one dropped connection can hang the entire loop.
- When diagnosing a "silently frozen" batch, check whether execution is stuck inside a reticulate call before suspecting R-level logic bugs.
- Applies to all `tdata_py$*` functions that talk to IBKR (`get_volatility_metrics`, `getValue`, `getOptValue`, chain/strike manager calls, etc.), not just volatility.
