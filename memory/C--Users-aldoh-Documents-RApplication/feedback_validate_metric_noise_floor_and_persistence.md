---
name: feedback_validate_metric_noise_floor_and_persistence
description: "Before trusting any derived statistical metric, measure its noise floor under a null and test rank persistence across disjoint periods; a constant verdict is the tell"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: fed3d834-ab45-425c-b8ba-bcfec60d789d
  modified: 2026-08-26T17:34:22.657Z
---

When the user reports that a computed metric "always says the same thing", do
**not** start by re-reading the formula or re-tuning thresholds. Two cheap
experiments settle it:

1. **Noise floor under a null.** Feed the estimator synthetic data where the
   true value is known to be zero (e.g. iid normal returns for a volatility
   statistic) and see what it returns. If the reported thresholds sit at or
   below that floor, the metric cannot discriminate and no re-thresholding
   rescues it. This result is model-free and therefore the strongest evidence.
2. **Rank persistence.** Compute the metric on two *disjoint* periods for ~25
   names and take the Spearman correlation of the rankings. Report the SE
   (~`1/sqrt(n-1)`). This decides whether a percentile/ranking presentation is
   legitimate, independent of whether the level is.

**Why:** on 2026-08-26 `/analyze` printed "Vol-of-vol — Very high, extreme vol
instability" on every report. The formula looked reasonable. The null test
showed the estimator returns ~1.94 when the truth is exactly 0 (10-day
*overlapping* windows share 9 of 10 returns, so `diff(log(rv))` is mostly
sampling error, then annualized by `sqrt(252)`) — i.e. the "> 2.0" cutoff sat on
its own noise floor. The persistence test then overturned my first
recommendation: ranking was **+0.22** at the shipped 252-day lookback (useless)
but **+0.76** at 504 days, and widening the rolling window — which I had
proposed as a fix — made ranking *worse* (+0.27) while making the number look
tidier. Cosmetics and information pointed in opposite directions.

**How to apply:**
- Overlapping rolling windows plus differencing plus annualization is the
  classic noise amplifier; suspect it first.
- Never recommend a fix for a statistical metric before testing it — my
  "widen the window" advice was wrong and only the persistence test caught it.
- Prefer reporting a **percentile against a stored reference basket** over an
  absolute level whenever the level is uncalibrated but the ranking persists;
  keep the raw figure alongside as provenance.
- Applies to any derived metric here (Flow_Score, BOT_Score, IVR/ivp/rvp, VRP),
  not just vol-of-vol. Related: [[project_vov_percentile_is_cross_sectional]],
  [[feedback_analyze_neutral_stance]].
