---
name: tdata-iv-rv-percentile-helpers
description: "Tdata::getIVPercentileLevels (252d IBKR IV history → p10/p25/p50/p75/p90) and Tdata::getVolMetrics ({iv30, ivp, rv30, rvp, price}). Use the percentile-levels helper for live IVP interp; expect per-ticker gaps in getVolMetrics."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3b70ddd8-ed66-422b-bfce-e9ef777ec479
---

Two Tdata helpers for vol percentile metrics. Different data sources, different reliability profiles.

## `Tdata::getIVPercentileLevels(sym)` (added 5.8.10, 2026-02-04)

Fetches 252-day OPTION_IMPLIED_VOLATILITY history from IBKR via `tdata_py.get_iv_percentile_levels`. Returns:

```r
list(current = 0.284,   # current IV30 as fraction
     p10 = 0.220, p25 = 0.250, p50 = 0.300, p75 = 0.350, p90 = 0.420,
     days_covered = 252L)
```

Used by `/analyze::resolve_ivp` to compute a live IVP when the DB Prices.ivp row is NA/stale. Linear-interp `current` between adjacent percentile breakpoints yields a single 0-100 IVP value:
- `current < p10` → scaled down from 10
- `current between p25 and p50` → 25 + frac*(50-25)
- etc.

**Reliable:** has worked on every ticker tried in the May 2026 redesign smoke tests.

## `Tdata::getVolMetrics(sym_list)` (existing)

Wraps `tdata_py.get_volatility_metrics`. Returns one row per ticker:

```r
data.frame(datetime, sym, iv30, ivp, rv30, rvp, price)
```

Source: IBKR aggregate IV (current_iv), IBKR aggregate HV (current_hv), with yfinance fallbacks for RV30. IVP/RVP are 252-day percentile ranks.

**Inconsistent per-field availability:** UPS short 2026-05-12 returned `rv30=41.4%` but `rvp=NA` (no realized-vol percentile from IBKR). GM same date returned both. Cause: IBKR doesn't always have the HV-percentile field populated for every symbol.

**How to use:**
- `resolve_rvp()` in `live_sources.R` falls back to `getVolMetrics` for RVP after DB Prices.rvp.
- Always check `is.na()` on individual returned fields, not just on the overall result.

## When to pick which

| Need | Use |
|---|---|
| Live IVP only (percentile rank) | `getIVPercentileLevels` + linear-interp |
| All of {iv30, ivp, rv30, rvp} in one call | `getVolMetrics` |
| Just RV30 (with yfinance fallback) | `Tdata::getVolMetrics` returns rv30 with built-in yf fallback |
| Cross-ticker percentile breakpoints (p10-p90) | only `getIVPercentileLevels` exposes them |

Related: [[reference_tdata_install_topology]], [[reference_tdata_vrp_formula]].
