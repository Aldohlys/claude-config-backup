---
name: project_analyze_spread_module_fetch_todo
description: TODO — tdata_py.spread re-fetches its whole strike band per width; fetch once per expiry + quiet DEBUG. Last /analyze option-fetch lever.
metadata: 
  node_type: memory
  type: project
  originSessionId: 65083a85-2e71-48fe-97b7-db8b02bd59a8
---

The only remaining option-fetch inefficiency in `/analyze` after the 2026-06-09 leaning pass (all R-side levers done — see [[project_vol_module_refactor]], RStudies 36bfa1b). It's in **Python**, so deferred.

**Problem:** `analyze/structures.R::enumerate_structures()` loops `for (exp in expiries) for (w in config$spread_widths)` and calls `tdata_py.spread::compute_spread_risk_reward(expiration=exp, spread_width=w, force_refresh=TRUE)` once **per width per expiry**. Each call independently (a) qualifies + (b) `getOptValue`-force-refreshes the *entire* moneyness band (e.g. QQQ ~71 strikes; the "54-strike getOptValue" in run logs) and (c) enumerates ~130 spreads + logs each at DEBUG. With 2 widths × N expiries that's the same band priced 2N times. The enumeration is cheap arithmetic; the repeated force-refresh band-pricing is the real recurring cost.

**Fix (tdata_py.spread + Tdata rebuild/deploy/R-restart):**
- Fetch the band's option quotes **once per expiry**, then enumerate ALL widths from that single dataframe (move the width loop inside the module, or add a `spread_widths` arg).
- For the analysis path, consider **not** force-refreshing (use the 30-min quote cache) — structures is a report, not order submission; or expose a flag.
- Lower the per-spread `[SPREAD]` log from DEBUG (or gate behind a verbose flag) — hundreds of lines per run.

**Files:** `RApplication/Tdata/inst/python/tdata_py/spread.py` (`compute_spread_risk_reward`); caller `RStudies/reports/analyze/structures.R` (`enumerate_structures`, ~line 503). Validate against QQQ ($1 grid, dense) + MT/F (cheap, width-aware band) per the matrix in [[project_vol_module_refactor]].
