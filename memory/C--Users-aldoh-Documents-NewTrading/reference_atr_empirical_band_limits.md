---
name: reference_atr_empirical_band_limits
description: Statistical limits of the Tuser/vol ATR empirical expected-move band — what the displayed n and CI actually support
metadata: 
  node_type: memory
  type: reference
  originSessionId: f74ba752-af98-461b-a9dd-da6fba63cfa6
---

The ATR "Empirical" expected-move row (`Tdata/R/atr_move.R`, surfaced in `Tuser/vol/view/xmoveUI.R`) is a **location-scale model**: move = C(q) · ATR%_today · √N, where the shape vector C is the asset's own forward N-session moves each standardized by the ATR% prevailing at that day's start. Heteroscedasticity (time-varying scale) is handled correctly; the load-bearing assumption is that the **standardized shape is stationary** — same kind of move once devolatilized.

Two limits NOT documented in the code comments (the comments only flag ATR 1-5%, horizon 3-25, equities):

1. **Overlapping windows → effective sample ≈ n_obs / horizon_sessions, not n_obs.** Forward moves are computed on every start day, so consecutive obs share N−1 days and are heavily autocorrelated. J 10-session showed n_obs≈1986 but only ~200 independent draws. The displayed n and the `< 250 obs` flag **overstate precision**, badly so in the tails. Rule of thumb: each tail needs ≥~5 effective obs, i.e. `((1-conf)/2) · (n_obs/N) ≥ 5`, else the band edge is essentially unestimable (e.g. 95% CI at 20 sessions on a few years of data).

2. **Shape is pooled across vol regimes → band is average-regime, optimistic in stress.** Devolatilizing removes most but not all fat tails; crash regimes have a fatter, more negatively-skewed standardized shape than calm ones. The 70-80% middle of the band is trustworthy; it grows progressively optimistic toward the tails and in a regime unlike the 8-year average.

Practical read: trust the 70-80% band, distrust ≥90% / long-horizon edges. Regime-conditioning the shape would be a redesign — not worth it. Related: [[reference_atr_move_multiples]], [[project_vol_module_refactor]].
