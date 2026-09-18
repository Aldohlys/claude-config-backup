---
name: feedback_ratio_metric_anticorrelated_with_its_own_thesis
description: "A reward:risk ratio built from \"distance to nearest resistance / distance to stop\" is structurally anti-correlated with trend — check a new ranking metric against the setup it is meant to select before shipping it as a sort key"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5f03a6df-1df0-4f76-b348-15024cf64b52
  modified: 2026-09-18T20:21:38.471Z
---

BOT_daily's `asym = (target − px) / (px − stop)` was meant to express tradeable
asymmetry. Measured on the full 69-name run (2026-09-18):
**Spearman(trend_count, asym) = −0.335**, Pearson −0.265.

| trend_state | n | median asym | median res_pct_of_em10 | target from fib_ext | median res_dist_atr |
|---|---|---|---|---|---|
| 0–1/6 | 24 | **0.88** | 57% | 0% | **1.82 ATR** |
| 2–3/6 | 21 | 0.44 | 29% | 5% | 1.02 ATR |
| 4–6/6 | 24 | 0.54 | 26% | 33% | **0.91 ATR** |

**Why:** a name in an uptrend sits near its highs, so its nearest overhead
resistance is close (0.91 ATR) or absent entirely — a third of the trending group
had no overhead zone at all and fell back to the Fibonacci 1.272 extension, which
is near by construction. Small numerator, low ratio. A beaten-down name has
distant overhead supply and scores high. So ranking on the ratio **inverts** the
selection the strategy actually makes ([[project_bot_three_class_framework]]:
S1 fires at 79.8% of real BOT entries vs 61.8% of the universe).

**A reachability filter does not repair it.** Of the five names both reachable
(≤100% of a 10-session expected move) and above 1:1 — NIO 1.95, BABA 1.76,
ATI 1.47, LRCX 1.35, AA 1.03 — every one was at `trend_state 0/6`. Only 8 of 60
zone targets were beyond 100% of a 10-session move (median 32%), so unreachable
outliers were a minority that merely dominated an unbounded ratio; the
anti-correlation is the deeper problem.

**Why:** a ratio inherits the geometry of its denominator and numerator, not the
intent behind it. "Higher is better" felt obvious and was backwards.

**How to apply:** before adopting any derived ratio as a sort key, correlate it
against the condition the strategy is supposed to select. Two cheap checks that
would have caught this: is the metric bounded, and does it rank the names you
would actually have traded? An unbounded ratio also lets one near-zero
denominator dominate the whole ordering — `asym` hit 18.5 when a .618
retracement landed a hair below spot, fixed with a minimum stop distance.
Related: [[feedback_validate_metric_noise_floor_and_persistence]],
[[feedback_normalize_window_metrics_to_current_size]]. Open as TODO #88.3.
