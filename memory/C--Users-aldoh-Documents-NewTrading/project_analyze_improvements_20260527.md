---
name: project-analyze-improvements-20260527
description: "Three /analyze defects surfaced in NVO long walkthrough — vehicle rule cheap_score override, missing slippage in spread EV ranking, BS-mid vs chain-mid divergence on outright grid"
metadata: 
  node_type: memory
  type: project
  originSessionId: 505cfc30-7712-444b-a707-5090e643a079
---

Three /analyze defects surfaced during the 2026-05-27 NVO long walkthrough. None block report generation, but each undermines the data the user reads to decide. Order is by impact, highest first.

## 1. Vehicle rule applies undocumented cheap_score gate

**Symptom:** NVO at \$44.19, IVP 16.8%, cheap_score 6/9 → script said `spread`. Per [[feedback-analyze-vehicle-rule]] the documented rule is "outright if stock<\$150 & IVP<60, spread if stock>\$150 or IVP>\$60" — which says outright.

**Root cause:** vehicle_rule() in shared scoring code includes an extra gate `cheap_score < 7 → spread (vertical for IV cost control)` that isn't in the documented rule.

**Why it matters:** On NVO, spreads carried 30-69% slippage tax on max reward because chain bid/ask was wide on ITM legs (\$1.10 on the \$42 long leg). Documented rule + bid/ask reality both said outright. The script's override pushed toward the structurally worse vehicle.

**Action:** Remove the cheap_score gate from vehicle_rule(), OR scope it tightly (e.g., only override to spread when cheap_score < 4 AND stock > \$80 AND chain liquidity confirms). Audit short-side callers — fix must not silently flip them too.

**Files (RStudies):** likely `reports/analyze/scoring.R` or `reports/analyze/phaseE.R`. Grep for `cheap_score < 7` or `vehicle_rule`.

## 2. Spread enumeration ignores bid/ask slippage in EV ranking

**Symptom:** Script's "top-EV" NVO spread was 42/47 @ 30d with EV −\$17. Slippage cost = (w_long \$1.10 + w_short \$0.07) × 100 = \$117/lot = 40% of max reward. Realistic EV closer to −\$130. The 45/50 spread (w_long \$0.08 + w_short \$0.23 → \$31/lot tax = 9% of reward) was ranked 7th by EV but is the cleanest execution.

**Root cause:** `tdata_py.spread.compute_spread_risk_reward` (see [[reference-tdata-spread-module]]) uses mid prices, doesn't propagate `(ask_long - bid_long) + (ask_short - bid_short)` round-trip cost into EV or surface it as a column.

**Why it matters:** Ranking by EV is misleading when cheap spreads (tight ATM strikes) get penalized vs wide spreads with illiquid ITM legs that look attractively mid-priced but can't be executed near mid.

**Action:** Add `slippage_cost` column to spread output: `(ask_long - bid_long) + (ask_short - bid_short)` per share, scaled ×100 for per-lot. Optionally:
- Show `EV_after_slippage = EV - slippage_cost` as secondary sort column
- Filter or warn-flag spreads where `slippage_cost > 0.25 × max_reward`

**Files:** `Tdata/inst/python/tdata_py/spread.py` for the computation; R-side surfacing in `reports/analyze/report.R` table builder.

## 3. Outright BS grid diverges from chain mid

**Symptom:** \$45 strike call shown with BS entry \$1.69 (R:R 0.18); actual chain mid was \$2.13 — 26% gap. The displayed R:R is computed against a fictional entry price.

**Root cause:** Outright grid uses BS pricing at script's `iv30` estimate; doesn't sanity-check against chain mid when a live quote is available for that (strike, expiry). Likely cause of the gap: per-strike implied vol differs from atm iv30, but the grid uses one IV for all strikes.

**Why it matters:** User compares grid R:R to working orders. A 20%+ entry-price gap means R:R numbers don't correspond to fillable trades.

**Action:** When chain quote available for a (strike, expiry), use `market_mid = (bid + ask) / 2` as the entry price; show BS price as secondary "theoretical" column for reference. When chain quote missing (deep OTM, no liquidity), fall back to BS at chain-implied per-strike IV (not flat iv30).

**Files:** outright grid builder in `reports/analyze/report.R` or `phaseD.R`. Quote pull may need extension to ATM and adjacent strikes (probably already pulled for spread enumeration).

## Implementation order

1. (1) — small change, high impact on vehicle selection. ~30 min code + smoke test on NVO and a short-direction symbol.
2. (3) — moderate refactor of grid builder, eliminates a recurring user-facing trust issue. ~1 h.
3. (2) — Python module change + R surfacing, broader scope. ~1-2 h. Schedule when batching tdata_py changes.

## Validation cases

- **NVO 2026-05-27 long** (this session): rerun should show vehicle=outright, outright grid entries within \$0.10 of chain mid, spread table with slippage column.
- **A high-priced underlying (e.g., MSFT)** to confirm vehicle rule still picks spread for stock>\$150 path.
- **A short trade** to confirm vehicle_rule() change doesn't break short-side classification.
