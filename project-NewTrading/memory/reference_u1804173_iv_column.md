---
name: reference-u1804173-iv-column
description: "U1804173.IV column is iv30 of the underlying, NOT the per-option implied vol — use IBKR historical option data for accurate per-strike IV"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 350b1975-d6ea-43f7-9086-8a8012f5cbc2
---

The `IV` column in `U1804173` (and likely sibling account snapshot tables) tracks the **ATM iv30 of the underlying**, not the implied vol of the specific held option contract.

**Why this matters:** for OTM strikes (especially index puts where skew is steep), the option's own IV can be 5-15 vol points HIGHER than the underlying's iv30 at entry, and the crush trajectory on that specific strike is what drives the held P&L. Using the U1804173.IV time series understates vega P&L on out-of-the-money options dramatically.

**Concrete example (2026-05-14, ESTX50 18SEP26 5200 P):**
- U1804173.IV on 26-Mar entry: 25.3% (underlying iv30)
- Actual 5200P own-IV from IBKR historical: 37.1%
- 11-vol-point gap → understated the vega loss by ~120 EUR/share when I first decomposed the position from U1804173 alone

**How to apply:** for IV / vega decomposition of a held option, pull IBKR historical option data (CSV via `historical_<sym>_<strike><right>_<exp>_<asof>.csv` exports or Tdata reqHistoricalData with option contract) — don't rely on U1804173.IV which is the underlying's surface anchor, useful for context but not for per-option attribution.

See also [[reference-tdata-vol-percentile-helpers]] for the proper iv30/rv30 underlying helpers.
