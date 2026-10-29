---
name: zone-engine-replay-2026-10-26
description: "Replay of BOT_daily zone variants (bar_order, breakout_n) — R-neutral; asym>=1 filter not predictive"
metadata:
  node_type: memory
  type: project
  originSessionId: a056d24e-aa71-4187-879f-be8862ed517f
  modified: 2026-10-26T11:08:38.220Z
---

2026-10-26: added `bar_order` and `breakout_n` flags to reports/shared/zones.R (made defaults the same day: bar_order = TRUE, breakout_n = 2; RStudies 9d43a51) after MMM showed asym 0.42 against a chart reading of ~1.5. Cause: the 2026-10-20 breakout bar was booked as a pivot high (ZigZag assumed high-before-low within the bar), plus a broken band of highs still "contained" spot through its padding. Each fix alone left MMM unchanged or worse; both together gave asym 1.53.

Replay (324 names, every 10 sessions over ~2y, long and short, 20-session first touch, unconditioned entries): both2 changed 28% of rows. Paired dR +0.002 R, name-bootstrap CI [-0.012, +0.017], so no effect. Farther targets were hit less often (67% → 61%), as expected. Rows with asym>=1 had mean R of -0.07, worse than all rows (-0.04), in both versions.

**Why:** reported asymmetry is not a predictive filter by itself, so level-engine changes are correctness fixes, not edge.
**How to apply:** judge future level-engine changes by a paired replay; don't present higher asym as an improvement. See [[research-style-falsification-first]].
