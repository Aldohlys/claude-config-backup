---
name: project_position_analysis_iv_skew_conventions
description: "RPreTrade Position Analysis options-projection model — projected IV = entry + vol-level offset (all legs) + skew (OTM only, puts & calls); per-IV chart anchors to the ATM leg"
metadata: 
  node_type: memory
  type: project
  originSessionId: d09853ea-92e1-47c8-89e9-895d5407bcf1
---

RPreTrade Position Analysis (Tuser/analysis: `pos_anaUI` + `analytics.R`) projects option positions with these conventions — confirm against them before touching IV/skew handling:

- **Projected IV per leg** = `trade_IV + vol_level_offset` (uniform, ALL legs) `+ vol_skew_offset` (added ONLY to OTM legs, `|delta| <= 0.30` at the projected price — applies to OTM **puts AND calls**, not puts only). This is the IV used to compute MktPrice and shown in the comparison table's **IV column** (which displays the *projected* IV, not the entry IV; 1 decimal = IBKR IV precision; blank on TOTAL/stock legs).
- The "Vol Skew" input is named just **"Vol Skew (%)"** (was "Vol Skew - OTM Puts") and applies to both OTM puts and calls.
- **"Projected Position P/L per IV" chart**: x-axis anchored to the position's OWN IV — Position 1's option legs when present, else Position 2's (same precedence as the Projected Underlying default). Within the anchor, the reference is the non-OTM ("ATM", `|delta| > 0.30`) legs — "the ATM leg, if it exists, is always the reference." Dashed reference line = `mean(reference-leg trade_IV) + vol_offset`, **plus the skew kick only when EVERY anchor leg is OTM** (single OTM call → 35+2+1=38%; bull call spread → ATM-leg projected IV, no skew). The percentile band stays anchored to the reference-leg `trade_IV`.
- The per-IV **curve keeps each leg's skew** (inter-leg vol-point difference, incl. the OTM kick, stays constant) and applies a **uniform** vol-point shift as IV sweeps — it does NOT re-derive OTM status as x moves. P/L at the reference line matches the comparison table / per-date / per-price charts.
- The tab has three P/L charts decomposing the projection axes: **per date** (theta), **per underlying price** (delta/gamma), **per IV** (vega). Projected Underlying defaults to Position 1's underlying (else Position 2, else 100).

**Why:** the per-IV anchoring/skew rules took several clarification rounds (2026-06-17); the user's mental model is specific. For the user's universe (sub-100% IV breakout trades) vega is a second-order P/L driver vs delta/theta. Related: [[project_asymmetric_trade_thesis]], [[project_bs_calculator_design]].
