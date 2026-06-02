---
name: TWS drops the whole client connection on malformed requests, not just the one request
description: Sending bad field values (NaN strike, None conId, missing tradingClass) causes TWS to tear down the API session, corrupting subsequent calls in the batch
type: feedback
originSessionId: efdcf942-8e75-4a7f-a442-14c3e668ac27
---
When TWS receives a syntactically unparseable field (e.g. Error 320 "Unable to parse field: 'Strike' for input string: 'nan'"), it does **not** just error that request — it **closes the entire client connection** (`Peer closed connection`, `WinError 10054`). Every subsequent call on that client fails until the Python layer reconnects (typically ~3 minutes), and any request mid-flight at the moment of the drop can trigger the reticulate-asyncio hang (see `feedback_reticulate_asyncio_uninterruptible.md`).

**Why:** On 2026-04-20 during a `getVolMetrics` batch, the NaN-strike fallback path for DG (EUREX) sent `strikes=[nan, nan, nan, nan]` to IBKR. TWS responded with Error 320 followed immediately by `Peer closed connection`. The batch that was processing the next ticker (DHR) had its in-flight IBKR call silently dropped and never returned — this was one of the root causes of the 5-hour hang. Similar pattern seen on 2026-04-20 with SAF (NA iv180 fallback) and previously with SOFR3 (missing contract expiration, TODO #37).

**How to apply:**
- Treat malformed-field errors (320, 200 "no security definition", 366 "no historical data query found") as **connection-corrupting events**, not isolated per-request failures. Validate inputs on the Python side before calling `ib.reqXxx()`.
- Every code path that builds an IBKR request must defensively filter out NaN/None/empty fields (strikes, expirations, conIds, tradingClass) upstream — don't rely on TWS to validate.
- When a batch job sees a Peer-closed right after an Error 320/200 line in the log, assume subsequent tickers in the same loop are unreliable until a fresh connection is confirmed.
- Symptom in logs: `Error <N>, reqId X: ...` followed within a second or two by `Peer closed connection.` and `ConnectionResetError: [WinError 10054]`.
