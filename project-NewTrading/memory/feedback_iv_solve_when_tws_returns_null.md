---
name: Solve IV from bid/ask mid when TWS returns null
description: getOptValue often returns delta+bid+ask but iv=null; solve IV via BS bisection from the mid instead of treating the surface as missing
type: feedback
originSessionId: 791d7ea0-3b98-46b6-b585-5209bd9337c8
---
When `tdata_py.contract.getOptValue` returns rows with populated `bid`, `ask`, `delta` but `iv=None` (and `mkt_price=None`), the surface is NOT missing — solve IV from the bid/ask mid via Black-Scholes bisection. This happens routinely on TSLA and other names; do not stall the vol-funnel section because of it.

**Why:** The /analyze TSLA run had every IV cell coming back null in the live pull, which initially looked like the surface was unusable. Switching to a BS bisection on the mid produced the full IV surface across 31d/51d expiries with results consistent with the persisted IV30/IV90 in the Prices table — close enough for term-structure / skew / RR reads.

**How to apply:** Use these inputs: spot from `Prices.price` (or live ticker), risk-free `r` via **`Tdata::getLastRate(currency, DTE)`** (defined in `Tdata/R/interest_rate_utils.R`). It maps DTE to the right tenor automatically: <18d→1w, 18-59d→1m, 60-134d→3m, 135-269d→6m, 270-544d→1y, ≥545d→2y. Pass the option's currency (USD/EUR/CHF/JPY), not always USD. **Never use the 10Y** — options at 7d-180d don't share a 10-year discount factor. Macro 10Y is for regime context only. Dividend yield `q` (0 for TSLA; look up for dividend-payers). Bisect over [0.001, 5.0] for 80 iterations. Verify by spot-checking ATM IV against `Prices.iv30` — should be within ~1 vol point.

```python
def bs(S,K,T,r,q,sig,right):
    if T<=0 or sig<=0: return 0
    d1=(math.log(S/K)+(r-q+0.5*sig*sig)*T)/(sig*math.sqrt(T))
    d2=d1-sig*math.sqrt(T)
    cdf=lambda x: 0.5*(1+math.erf(x/math.sqrt(2)))
    return S*math.exp(-q*T)*cdf(d1)-K*math.exp(-r*T)*cdf(d2) if right=='C' else K*math.exp(-r*T)*cdf(-d2)-S*math.exp(-q*T)*cdf(-d1)
def iv(price, S,K,T,r,q,right):
    lo,hi=0.001,5.0
    for _ in range(80):
        m=(lo+hi)/2
        if bs(S,K,T,r,q,m,right) > price: hi=m
        else: lo=m
    return m
```
