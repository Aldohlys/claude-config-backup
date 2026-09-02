---
name: reference_bot_name_continuation_character
description: "Per-name breakout continuation stats by extension bucket — SLB and EQT mean-revert from the BOT setup state, SCCO continues, AR is conditional"
metadata:
  node_type: memory
  type: reference
---

Measured 2026-08-31, 10y daily. Within the setup state (close > EMA50 and rng_pct ≥ 70), median forward 20-session move in ATR, bucketed by how extended the name already is (`prior20_atr`). Percentage = P(+2 ATR before −2 ATR). Method: [[reference_first_touch_barrier_test]].

| | <1.5 ATR | 1.5–3 | 3–4.5 | ≥4.5 |
|---|---:|---:|---:|---:|
| AR | −0.40 (55.7%) | +0.20 (53.5%) | +0.57 (67.2%) | +0.43 (51.4%) |
| SLB | +0.40 (52.1%) | −0.26 (46.7%) | −0.63 (42.1%) | −0.42 (49.4%) |
| SCCO | +1.47 (57.9%) | +0.90 (60.4%) | +1.29 (56.9%) | +0.56 (49.8%) |
| EQT | −0.07 (52.8%) | −0.41 (41.3%) | −0.27 (41.9%) | −0.55 (48.6%) |
| FCX | +0.05 (62.8%) | +0.38 (61.0%) | +0.30 (52.2%) | +0.73 (49.8%) |

**Reads:**
- **SLB and EQT do not continue.** Negative median forward move in 3 of 4 buckets, win rate never above ~53%. They are mean-reverting from the very state the checklist calls GREEN. Do not buy SLB breakouts; if a SLB long is already working, this is the exit case, not the add case.
- **SCCO has the best character** — every bucket positive, 50-60%, median MFE 2.0-3.1 ATR. Its practical problem is price (\~\$210) and IV (47-50%), which push it to spreads whose debit (\~\$680-690 for a 10-wide) is >2x the \$300 lot cap. Good name, doesn't fit the book.
- **FCX and AR are conditional** — they pay in specific extension buckets, not uniformly.
- **Extension degrades nearly every name at ≥4.5 ATR.** Buying after a 20-session run of 4.5+ ATR is buying the top of the move distribution — consistent with the p90 coefficient in [[reference_atr_move_multiples]].

Re-measure before reusing: these are per-name regime characteristics, not constants. Related: [[feedback_classify_breakout_vs_meanreversion]], [[feedback_price_action_rr_gate]].
