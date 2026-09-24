---
name: feedback_test_a_gate_against_what_it_controls
description: "Before recalibrating a threshold, test the criterion against the outcome it claims to control — gap_share gated stop-through risk it does not predict (Spearman 0.073) while the stop distance predicts it at -0.889"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f5ad26f1-60c7-4344-bb10-9710e5cdd89f
  modified: 2026-09-21T17:10:12.593Z
---

When a criterion looks mis-set, the instinct is to argue about the number. Ask
first whether it predicts the thing it exists to prevent — the answer can make
the number irrelevant.

**The case (TODO 88.4, 2026-09-21).** `GapShare_Tercile ≠ high` excluded 93 of
335 names, including FCX and eight megacaps. The stated reason: at high gap
share price gaps *through* a stop instead of trading through it. That is a claim
about a **stop**, so it was testable directly — `gap_vs_stop = p95(|overnight
return|) × px ÷ (px − stop)`, over 169 daily rows:

| pair | Spearman | reading |
|---|---|---|
| `gap_share` ↔ `gap_vs_stop` | **0.073** | inside 1 SE of zero (SE ≈ 1/√(n−1) ≈ 0.077) |
| `gap_share` ↔ gap **size** | **0.477** | it measures what it claims |
| **stop distance** ↔ `gap_vs_stop` | **−0.889** | this is what sets the risk |

`gap_share` was measuring gap magnitude correctly and still failing, because
stop distances vary 0.53–3.0 ATR and swamp gap sizes (0.88–1.25 ATR). All 13
names whose p95 overnight clears the whole stop are **low-to-mid** gap share —
PFE at 0.192, nearly the lowest in the universe.

**Two structural lessons, both reusable:**

- **A name-level attribute cannot gate a trade-level risk.** The stop is
  recomputed every session by the zone engine; membership is recomputed monthly.
  The gate belonged in `bot_daily` (as `gap_vs_stop`), not in `BOT_Eligible`.
  Ask which cadence owns the quantity before choosing where the test lives.
- **A relative criterion (tercile/percentile) excludes a fixed fraction by
  construction and moves with the population.** The same rule measured at
  0.61–0.74 on a 205-name universe landed at 0.429 on 352 rows. See
  [[feedback_relative_criterion_excludes_a_fixed_fraction]].

**Check n before believing ρ.** The first run of this test used 5 names and gave
−0.667, which is inside one standard error of zero at that size and was noise. I
reported it before widening the sample; don't.

Same family as [[feedback_ratio_metric_anticorrelated_with_its_own_thesis]] and
[[feedback_validate_metric_noise_floor_and_persistence]].
