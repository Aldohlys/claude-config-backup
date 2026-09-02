---
name: feedback_price_action_rr_gate
description: "Derive stop/target from swing highs and lows and check the OPTION payoff at target before trusting any scanner R:R column"
metadata:
  node_type: memory
  type: feedback
---

User's rule, stated 2026-08-31 on an AR BOT candidate: **"A price action chart must give clear information about exiting the trade, either at a profit or at a loss."** They read AR's chart as stop \$36 / target \$40 at a \$38.45 price, called it non-asymmetric, and were right — the scanner's `rr_meas = 1.53` had passed it as GREEN.

**Why:** `rr_meas` is (40-session base height) / (distance to 20-session low). It is a ratio of distances with **no clock attached**. On AR it implied a target of \$45.58 reachable 4.5% of the time inside 20 sessions. The scan legend already says "reported, not gated — true reward:risk is option-level"; this is what ignoring that costs. See [[reference_first_touch_barrier_test]].

**How to apply — three steps, in order, before any BOT entry:**
1. **Levels from structure, not from the scanner.** Fractal swing highs/lows (5-bar) over 1-2 years plus days-at-price. Take the nearest real rejection shelf as target and the nearest support shelf as stop. Sanity-check the stop is ≥1.5 ATR away — a stop inside the noise inflates R:R and collapses the hit rate (SCCO: 1.15 ATR stop → 44% hit rate on a 1.10 R:R frame).
2. **First-touch, not distance.** Run [[reference_first_touch_barrier_test]]. A 60-70% hit rate on a 0.6 R:R frame is roughly break-even at the underlying level.
3. **Translate to the option — this is the step that decides.** On AR the Nov 40C returned **+4.0% if the target was hit on schedule** and −61% at the stop: theta plus an 11.5% round-trip spread ate the entire +4% underlying move. A target under ~5% of spot does not pay through a long option inside the \$300 lot cap. Check this BEFORE liking the setup.

**Corollary:** when the option only pays if you hold *through* the charted target, that is a different trade with an invented target — and it conflicts with [[feedback_hard_to_exit_winners]]. Do not let the option math talk you past the level the chart gave you.

Related: [[reference_atr_move_multiples]] (is the target reachable at all), [[project_rr_calibration_result]] (R:R_min=0.5 is a **return on option premium**, NOT an underlying reward:risk ratio — do not compare the two), [[feedback_analyze_lot_definition]].
