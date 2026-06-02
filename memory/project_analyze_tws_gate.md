---
name: /analyze TWS reachability gate (isIBAvailable)
description: How /analyze handles TWS down vs TWS-stuck states; live fetches gated on isIBAvailable; residual hang risk requires TODO #52
type: project
originSessionId: 67df7b93-72b2-4e92-a5f8-7541ce5b225c
---
`/analyze` (RStudies/reports/analyze/main.R) probes TWS once at startup via `Tdata::isIBAvailable()` and writes the result into `CONFIG$tws_reachable`. Every live-fetch helper in funnel.R and structures.R checks that flag and short-circuits to `"FETCH FAILED: TWS not reachable"` when FALSE — so the run completes in ~15s with a warning, instead of issuing per-call requests that block reticulate's asyncio loop.

**Why:** the user requested "exit gracefully and warn user that TWS is not available" instead of letting individual TWS calls retry connection-refused 4s at a time across many strikes/expiries.

**How to apply:**
- Don't add new live-IBKR calls to /analyze without checking `config$tws_reachable` first
- The `isIBAvailable()` probe handles the *connection-refused* case (TWS not running, port closed, firewall block)
- It does **NOT** handle the *TWS-stuck-mid-request* case — when TWS is up but the request hangs forever (e.g. Error 1100/1102 mid-fetch). R-side `tryCatch` cannot bound an asyncio call (memory `feedback_reticulate_asyncio_uninterruptible.md`). PSX 2026-04-30 hit this twice during the rework — chain fetch hung 8+ minutes after a connectivity blip
- Full hang-immunity for /analyze depends on TODO #52 landing (Python-side `asyncio.wait_for` wraps on every `ib.req*`). Until then, `/analyze` can still hang if TWS gets stuck mid-call. Don't claim "no hang" without that caveat
- The residual hang surface is acceptable for `/analyze` because it's an interactive tool — user can Ctrl+C. Scheduled tasks need defense-in-depth (Task Scheduler kill-tree per TODO #53)
