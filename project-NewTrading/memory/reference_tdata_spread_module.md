---
name: Tdata spread analyzer reference
description: tdata_py.spread.compute_spread_risk_reward enumerates verticals — use it instead of one-off R:R math
type: reference
originSessionId: 791d7ea0-3b98-46b6-b585-5209bd9337c8
---
`tdata_py.spread.compute_spread_risk_reward` is the canonical vertical-spread enumerator. /analyze Phase D.4 should call it instead of computing one-off R:R numbers manually.

**Path:** `RApplication/Tdata/inst/python/tdata_py/spread.py` (also installed in 6+ renv libs).

**UI consumer:** `RApplication/Tuser/spread/app.R` is the Shiny app that wraps this function — useful as a reference for input plumbing (sym, trading_class, current_price, multiplier, currency, exchanges).

**Signature highlights:**
```python
compute_spread_risk_reward(
    sym, trading_class, expiration, current_price,
    moneyness_pct=0.20,        # ±20% of strikes around spot
    spread_width=5 or 10,      # iterate both for /analyze
    right='C' or 'P',
    multiplier=100, currency='USD',
    exchangeSec='SMART', exchangeOpt='SMART',
    force_refresh=True,        # MANDATORY in /analyze
)
```

**Returns DataFrame with columns:** `right, spread_type` (CREDIT/DEBIT), `short_strike, long_strike, width, net_premium, max_risk, max_reward, reward_risk_ratio, prob_success_delta, prob_success_market, edge, expected_value`.

**Conventions:**
- Bull call spread → DEBIT row where `short_strike > long_strike` (short the higher strike, long the lower).
- `prob_success_delta` = probability of price > short_strike at expiry (i.e., probability of MAX reward, not any profit).
- `edge = prob_success_delta − prob_success_market`. Positive edge = delta-implied prob exceeds the EV=0 implied prob (potential mispricing).
- `max_risk` is per LOT (full structure), already scaled by multiplier — gate the $300 cap directly on this column.
