---
name: reference_isIBAvailable
description: Use Tdata::isIBAvailable() — not a custom TCP socket probe — when an R script needs to check whether TWS is reachable before issuing live IBKR requests
type: reference
originSessionId: 67df7b93-72b2-4e92-a5f8-7541ce5b225c
---
`Tdata::isIBAvailable()` (defined in `Tdata/R/ibkr.R:15`, calls `tdata_py$isIBAvailable()` in `Tdata/inst/python/tdata_py/IB_connection.py:88`) opens a connection, checks `ib.isConnected()`, disconnects, returns TRUE/FALSE. It honours the `R_CONFIG_FILE` config-yml `ibkr.api_port` (default 7496) and `ib_async`'s native 4-second connect timeout, so a refused/closed port returns FALSE within ~4s without hanging.

**Use it when** an R script (e.g. `/analyze`, scheduled portfolio updates, scanner runs) does live IBKR fetches and you want to skip them gracefully — rather than letting each per-call ConnectionRefusedError accumulate (4s per attempt × N strikes × M expiries).

**Don't** re-implement a `socket.connect_ex(('localhost', 7496))` probe — that catches port-closed but not "TWS process up but rejecting API". `Tdata::isIBAvailable()` does the real handshake.

**Caveat:** `isIBAvailable()` returning TRUE only proves TWS accepted a connect. It does NOT prove TWS will respond to subsequent `reqContractDetails`/`reqSecDefOptParams` etc. — those can still hang if TWS is degraded mid-session (1100/1102 connectivity blip aftermath). For that residual risk, `IB.RequestTimeout` is the fix (see `project_ib_request_timeout.md`).
