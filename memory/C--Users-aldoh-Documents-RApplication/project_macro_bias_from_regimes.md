---
name: project_macro_bias_from_regimes
description: "Macro report headline bias is derived from the regime model since 2026-10-05; LONG BIAS cannot occur with current REGIME_WEIGHTS"
metadata:
  type: project
---

Since 2026-10-05 (RStudies 83e9e3d) the macro_context headline bias comes from `synthesize(scenario_scores)` in `scenarios.R`: dominant regime >= 40% and >= 15 pts ahead -> Liquidity Stress = DEFENSIVE, Directional Flow = LONG BIAS, else NEUTRAL. The old points system (VIX level, VIX9D/VIX3M, 10Y band, breadth band, mismatches) is gone; `macro_context_results.long_pts/short_pts` are NA from that date.

**Why:** the points sat at 3/3 through the whole 2026-09-24..10-05 bond selloff while Liquidity Stress dominated the regime panel on the same page; calm vol counted twice and cancelled the 10Y > 5% signal.

**How to apply:**
- Directional Flow never exceeded 33.6% on 41 stored days, so LONG BIAS is unreachable until `REGIME_WEIGHTS` are re-tuned; `rates_press` is not a Liquidity Stress input, so the 10Y never shows as a driver. Both are open for the TODO 105 weight review — don't treat a NEUTRAL in a bull market as a finding.
- The bias has no programmatic consumer left (swing scanner removed; its orphan `reports/swing_scanner/main.R` is still tracked).
- Related: [[project_bpt_vs_macro_scanner]], [[reference_cboe_vx_settlement]].
