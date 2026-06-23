---
name: /analyze — pull live OI when cache is empty
description: Never report chain_state=unknown if a live OI pull is feasible; pull it via reqMktData genericTickList=101
type: feedback
originSessionId: 791d7ea0-3b98-46b6-b585-5209bd9337c8
---
In `/analyze` Phase D.3, never report `chain_state: unknown` if TWS is reachable. The `option_chain_oi_history` table is sparse (only 7 symbols populated as of 2026-04-28 — AAPL, CLF, DLR, JNJ, NVDA, TGT, WMT), so most tickers will have NO cached OI. Pull it live.

**Why:** The first TSLA run reported "chain_state: unknown" and skipped naming oi_cap_call because the cache was empty. The user pushed back — there was no reason not to pull live. Live OI took ~2 minutes for 27 strikes × 2 rights × 2 expiries via `reqMktData(genericTickList="101")`.

**How to apply:** Skeleton:
```python
from tdata_py.contract import Option, IB
ib = IB(); ib.connect('127.0.0.1', 7496, clientId=<unique>)
for K in strikes_within_pm_20pct:
    for right in ['C', 'P']:
        opt = Option(sym, expiry, K, right, 'SMART', tradingClass=tc, currency=ccy)
        qual = ib.qualifyContracts(opt)
        if not qual: continue
        tk = ib.reqMktData(qual[0], '101', False, False)
        ib.sleep(2)
        v = tk.callOpenInterest if right == 'C' else tk.putOpenInterest
        if v == v: oi[right][K] = int(v)
        ib.cancelMktData(qual[0])
ib.disconnect()
```
Save to `<ticker>_oi.json`. Run for both expiries used in Phase D.4 (~30d + ~50-60d) since OI distribution differs sharply between weekly and monthly OpEx — June OpEx had 21,452 OI on TSLA $450 vs 31d's small numbers.
