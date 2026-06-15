---
name: feedback_tws_subscription_outlives_end_event
description: A wait_for(reqXxxAsync, timeout) timing out doesn't mean no data arrived — the subscription is often active and ib.portfolio() / ib.accountValues() / etc. already have the streamed data. Don't abandon — read after the timeout.
type: feedback
originSessionId: 110f4600-4558-489d-88b2-cdf4eb853b4a
---
Several ib_async async APIs (`reqAccountUpdatesAsync`, `reqMktDataAsync`, `reqHistoricalDataAsync`) await an "end-event" that signals "all initial data has been received". This end-event is **not reliably emitted by TWS for paper / single-account / certain contract types** — but the underlying SUBSCRIPTION still goes through, and TWS still streams data into the relevant ib_async state stores (`ib.portfolio()`, `ib.accountValues()`, `ib.tickers()`, etc.).

**Why:** Hit twice on 2026-05-10:
- `reqAccountUpdatesAsync(DU5221795)` timed out at 60s while TWS had already streamed the full portfolio + currency balances. First fix dropped portfolio entirely (5.10.14); correct fix reads it anyway after the timeout (5.10.15).
- `reqSecDefOptParams(MCL, '', 'FUT', conId)` hangs forever on FUT — but actually, this one is genuinely broken for futures (Error 322 with explicit exchange, hang with empty); use `reqContractDetails` workaround instead.

**How to apply when adding a new TWS-API call wrapped in `wait_for`:**
1. Always treat the `asyncio.TimeoutError` as a soft signal, not a fatal error.
2. After the timeout: `ib.sleep(2)` to let late updates flush, then **proceed with the read** (`ib.portfolio()`, `ib.accountValues()`, etc.).
3. If the subscription truly delivered nothing, the read returns empty and the rest of the function naturally produces an empty/partial result — no special handling needed.
4. Log a warning that distinguishes "end-event timeout" (likely partial / fine) from "request error" (genuinely broken).

**Anti-pattern to avoid:**
```python
try:
    util.run(asyncio.wait_for(ib.reqAccountUpdatesAsync(account), timeout=60))
except asyncio.TimeoutError:
    return 0   # WRONG — discards data that's already in ib.portfolio()
```

**Correct pattern:**
```python
try:
    util.run(asyncio.wait_for(ib.reqAccountUpdatesAsync(account), timeout=60))
    ib.sleep(2)
except asyncio.TimeoutError:
    logger.warning("end-event timeout — reading partial state from active subscription")
    ib.sleep(2)
# Continue: ib.portfolio() / ib.accountValues() are populated for whatever did arrive.
```
