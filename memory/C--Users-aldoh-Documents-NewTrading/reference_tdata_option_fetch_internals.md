---
name: reference_tdata_option_fetch_internals
description: "How Tdata/tdata_py option-chain fetch primitives behave — cost, per-expiry validity, DTE floors — for lean option fetching"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 65083a85-2e71-48fe-97b7-db8b02bd59a8
---

Hard-won during the 2026-06-09 /analyze leaning ([[project_vol_module_refactor]]). Verify before relying — but these held across QQQ/MT/J/NVDA/F.

- **`getStrikesInRange(sym, expiration=, center_strike=, range_pct=)`** (tdata_py) — returns strikes **qualified valid for THAT specific expiry**. Use this for per-expiry strike selection. Footgun: `trading_class` is the 2nd positional arg → pass `expiration=`/`center_strike=` as **keywords**. Qualification is cached per (sym, trading_class, expiry) — first call slow, then free.
- **`getAllStrikes(sym)`** (no expiration) — returns the **union theoretical grid across ALL expirations**, no qualification (cheap). DANGER: includes strikes that don't exist on a given expiry → `getOptValue` → IBKR **Error 200 "No security definition"** + 0 pairs. Do NOT use to pick strikes for a specific expiry. (This is why the getAllStrikes approach failed; getStrikesInRange is correct.)
- **Strike grids vary by expiry on the same name.** QQQ had \$1 strikes on 20260616 but \$5 on 20260623; MT/J/NVDA are \$5 grids; QQQ near-ATM \$1. So % bands give wildly different strike counts; a fixed `moneyness_pct` can't serve both dense \$1-grid high-priced names and low-priced names where the spread *width* dominates → size bands as `max(base_pct, width/spot + room)`.
- **`getIV_DTE(sym, ccy, spot, DTE)`** — returns NA for `DTE ≤ 10` or `≥ 730`. Only considers expiries ≥ today+7, then interpolates variance between the two bracketing expiries (parabola IV-vs-strike fit). So short horizons (<~10 cal days) can't get a horizon-matched IV — use the nearest usable expiry. It fetches near+next expiry, ~4 strikes each (after the [[project_vol_module_refactor]] fix, via getStrikesInRange not the whole chain).
- **`getVolMetrics(sym)`** (R wrapper) is HEAVY: calls python `get_volatility_metrics` (252d hist IV/HV bars → current_iv/current_hv/iv_percentile/hv_percentile) AND computes iv15/30/90/180 via 8 `getIV_DTE` option-chain fetches AND writes the Prices DB. If you only need **rvp / rv30**, call `tdata_py$get_volatility_metrics(sym, lookback_days=252L, hist=TRUE, price=FALSE)` directly and read `hv_percentile` / `current_hv` — skips the 8 option fetches + DB write.
- **`getOptValue(... force_refresh=TRUE)`** bypasses the 30-min parquet quote cache → a live snapshot per strike (~15s/call). Returns NaN bid/ask/last (null marks) when the market is **closed** (pre-open); a cache hit can mask this. So pre-open option-derived metrics legitimately come back NO DATA. See [[reference_closed_market_option_marks_stale]].
- **`get_chain_oi(sym, expiration, strike_min, strike_max)`** — per-strike OI via reqMktData genericTickList=101; qualifies/prices the whole range, so keep `strike_min/max` tight (±~12%, not ±25%).
