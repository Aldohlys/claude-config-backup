---
name: feedback_test_tws_first
description: Any change touching ib_async / TWS API behaviour must be validated against the live API first; if TWS is not active, halt and warn the user instead of editing blindly
type: feedback
originSessionId: 110f4600-4558-489d-88b2-cdf4eb853b4a
---
Any change in Tdata code (Python or R) that depends on how TWS / ib_async behaves — qualifyContracts, reqSecDefOptParams, reqAccountUpdates, reqHistoricalData, contract construction conventions for FUT/FOP/STK/CASH, etc. — must be **tested against a live TWS connection FIRST**, before the source edit is written.

**Why:** TWS API behaviour is not fully trustworthy or fully documented. Field semantics shift between contract types (e.g. `symbol` for STK is the underlying, but for FUT it's the root and `localSymbol` is required for unique identification). End-events arrive for some account types and not others. Subscriptions populate state silently while their await-handles never resolve. The only reliable validation is a live test against the user's actual TWS session — reading the docs or reasoning from the code is not enough. Multiple bugs in this codebase originated from making "obvious" changes that were never live-tested.

**How to apply:**
1. Before editing any Python/R code that issues a TWS call, write a small standalone test script (e.g. `scripts/test_<thing>.py`) that exercises the EXACT change against ib_async + a live TWS.
2. Use a high clientId (e.g. 99) so it doesn't clash with the user's launcher.
3. Run via `C:/Users/aldoh/AppData/Local/r-miniconda/envs/r-reticulate/python.exe` (the conda env reticulate uses — confirm via `reticulate::py_discover_config()$python`).
4. Verify the response shape, not just "no error" — qualifyContracts can succeed with conId=0 for instance.
5. **Only after the live test confirms behaviour, edit the source.** Then test again post-edit if practical.
6. **If TWS is not currently active**: STOP. Warn the user that the change cannot be validated without an active TWS connection, and ask them to launch TWS or defer the change. Do not edit blindly hoping it works.

**Examples from past failures:**
- chains_manager.py FUT: `Contract(symbol="MCLN6", secType="FUT", ...)` returns Error 200; needs `localSymbol="MCLN6"` instead. Caught by live test 2026-05-10 only after a deploy attempt with the wrong field.
- account.py reqAccountUpdates timeout: end-event doesn't fire for paper sessions; abandoning the read after timeout drops live data that's already been streamed in. Two iterations (5.10.14, 5.10.15) before the right behaviour was identified — both could have been a single iteration with a TWS test up-front.
