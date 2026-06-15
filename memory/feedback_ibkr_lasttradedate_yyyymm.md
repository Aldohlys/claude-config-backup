---
name: feedback_ibkr_lasttradedate_yyyymm
description: When qualifying a futures Contract by root symbol + 6-digit month, IBKR returns the contract whose last trading date falls in that month — which usually means the *previous* contract-month-label. Use full date or conId.
type: feedback
originSessionId: 435ffc45-0ae7-46ab-8d17-978675432004
---
`Contract(secType='FUT', symbol='MCL', exchange='NYMEX', currency='USD', lastTradeDateOrContractMonth='202606')` returns **MCLM6** (May 2026 contract, last-trade 2026-05-18), NOT MCLN6 (July 2026 contract, label JUL26, last-trade 2026-06-18). IBKR matches on actual last-trade-date, not on the contract-month label printed in TWS.

**Why:** Empirically verified 2026-05-10 against TWS for MCL. The "contract month" label (JUL26) and the "expiration date" (JUN 18 '26) are different fields; the API key `lastTradeDateOrContractMonth` matches the latter. NYMEX energy futures typically last-trade in the prior calendar month vs their contract-month label, so root+YYYYMM looks intuitive but silently picks the wrong contract.

**How to apply:** When building a futures Contract programmatically, never use `lastTradeDateOrContractMonth='YYYYMM'`. Use the full `'YYYYMMDD'` (the actual last trading date), or — preferred — pass `conId=` from the Tickers table. The `Future(localSymbol='MCLN6', lastTradeDateOrContractMonth='20260618', ...)` and `Future(conId=661016559, ...)` patterns both qualify correctly. Tdata's `contract.py:124-129` already does this; chains_manager.py:144 does NOT (see chains_manager FUT bug memory).
