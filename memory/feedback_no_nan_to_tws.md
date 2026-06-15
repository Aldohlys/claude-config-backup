---
name: feedback_no_nan_to_tws
description: Defensive validation rule — filter NaN strikes/values before any TWS API call to avoid Error 320 + peer-closed cascade
type: feedback
originSessionId: 29cdc5d5-050d-4ed2-a819-c20c72b37017
---
Never let NaN values reach the IBKR TWS API. Filter them out at the call-site
boundary, log a warning, and either substitute a valid value or return None.

**Why:** A NaN in a contract field (e.g. `strikes=[nan, nan, nan, nan]`) causes
TWS Error 320 — "Error reading request. Unable to parse field: 'Strike' for
input string: 'nan'" — which is immediately followed by `Peer closed
connection` (asyncio `ConnectionResetError WinError 10054`). This poisons the
socket and corrupts any subsequent request in the same batch. We saw it for
IBIT vol metrics on 2026-04-28: three term-structure calls (15d/90d/180d) each
sent `[nan, nan, nan, nan]`, each got Error 320, each tore down the
connection. Result: `iv15`, `iv90`, `iv180` all `NA`. Pattern matches
`feedback_tws_drops_connection_on_malformed.md`.

The upstream cause was usually missing ticker metadata (no row in the
`Tickers` DB) so the strike-picker returned NaN. That should be fixed at the
data-source level too, but the *guard* belongs at the TWS boundary because
we cannot trust every upstream caller.

**How to apply:**
- In any function that issues a TWS request (`getOptValue`, `getStrikesfromExpDate`,
  contract qualification, historical data fetches), validate numeric inputs
  immediately before constructing the request.
- Use `math.isnan(x)` (Python) / `is.na(x)` (R) to detect.
- For list inputs (e.g. strikes): drop NaN entries, log how many were dropped,
  and if the resulting list is empty return `None` / `NA` instead of calling
  TWS with an empty or malformed payload.
- Existing guard lives in `Tdata/inst/python/tdata_py/contract.py::getOptValue`
  (added 2026-04-28, just after `requested_strikes = [float(s) for s in strikes]`).
  Use it as a template when adding similar guards elsewhere.
- Don't rely on `tryCatch` / Python `try/except` to recover — by the time the
  exception fires, the connection is already gone (see
  `feedback_reticulate_asyncio_uninterruptible.md`).
