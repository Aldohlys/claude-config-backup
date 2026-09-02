---
name: reference_first_touch_barrier_test
description: "First-touch barrier method for testing a BOT candidate's chart levels, with AR/SLB/SCCO calibration from 2026-08-31"
metadata:
  node_type: memory
  type: reference
---

Method built 2026-08-31 to test whether a BOT candidate's price-action stop/target actually pays. Invoked by [[feedback_price_action_rr_gate]].

**Method.** Wilder ATR14. Express the chart target and stop as ATR multiples from spot. Walk forward from every historical bar (5y daily), horizon 20 sessions; whichever barrier the High/Low touches first wins; same-bar both-touched scored as a loss (conservative). Report target-first %, stop-first %, and EV as a % of notional. Run it twice: unconditional, and conditioned on the setup state (proxy: close > EMA50 **and** rng_pct ≥ 70).

**Caveats:** windows overlap heavily, so effective n is far below printed n — read the numbers as directional. The EMA50/rng_pct mask is a proxy for the checklist state, not the full 6-component S score.

**Calibration, 2026-08-31 scan (the three GREEN rows tested):**

| name | frame | R:R | all bars | GREEN state | EV (GREEN) |
|---|---|---:|---|---|---:|
| AR | 40.00 / 36.00 | 0.63 | 60.9% | **69.5%** | +0.99% |
| SLB | 58.51 / 53.06 | 0.27 | 79.4% | **74.4%** | −0.37% |
| SCCO | 220.78 / 199.80 | 1.10 | 52.6% | **44.1%** | −0.30% |
| SCCO | 220.78 / 176.59 (scanner low20) | 0.33 | — | 69.2% | +1.14% |

**Three findings worth keeping:**
- **The GREEN state is not uniformly predictive.** It *improved* AR's odds (60.9→69.5%) and *worsened* SLB's and SCCO's (79.4→74.4, 52.6→44.1). Continuation character is name-specific — see [[reference_bot_name_continuation_character]]. Never assume the checklist state adds edge; test it per name.
- **`rr_meas` fails the clock test.** AR's 1.53 implied target \$45.58: first-touch 4.5% inside 20 sessions vs 19.1% for its implied stop. The frames that make EV positive (SCCO stop 3.83 ATR = −15.7%) are ones no \$300 option lot survives.
- **A stop tighter than ~1.5 ATR destroys the hit rate.** SCCO's 1.15 ATR gap-fill stop looked like R:R 1.10 and delivered 44%.

Scripts (scratchpad, not committed): first-touch `ft.py`, extension buckets `ext.py`. Related: [[reference_atr_move_multiples]], [[reference_atr_empirical_band_limits]].
