---
name: project_vov_percentile_is_cross_sectional
description: "/analyze's vol-of-vol percentile ranks a ticker against the universe, NOT against its own history; the historical variant was never validated"
metadata: 
  node_type: memory
  type: project
  originSessionId: fed3d834-ab45-425c-b8ba-bcfec60d789d
  modified: 2026-08-26T17:34:34.408Z
---

Since Tdata 5.14.2 / RStudies commit 4589c2f (2026-08-26), `/analyze` reports
vol-of-vol as a percentile rather than an absolute level. Two limitations are
**not** recorded in either CHANGELOG:

1. **The percentile is cross-sectional.** It answers "how unstable is this
   ticker's vol *versus other tickers*", read off `VolOfVolBreakpoints`
   (quantiles over ~208 scanner-universe names). It does **not** answer "versus
   this ticker's own past". A time-series percentile was never tested — its
   rank persistence is unknown, so do not present the current number as a
   historical reading.
2. **Raw values are not comparable across versions.** The lookback moved
   252 → 504 days, so pre-5.14.2 figures don't reproduce (MT: 2.529 → 2.419).
   Any stored/quoted vol-of-vol from an older report is on a different scale.

3. **The documented +0.76 persistence is overstated.** Measured 2026-08-31 over
   the two most recent disjoint 504-day blocks on 205 names, close-to-close
   rank persistence is **+0.445**, not +0.76. Random 24-name subsamples (the n
   behind the docstring) have median +0.445 and reach +0.76 only 1.4% of the
   time. The +0.76 is quoted verbatim in `compute_vol_of_vol` docs,
   `vov_to_percentile` docs, and `scripts/refresh_vov_breakpoints.R`, and is the
   sole cited justification for the percentile approach. Not yet corrected in
   the source.

4. **Estimator alternatives were tested and rejected (2026-08-31).** Yang-Zhang
   and Rogers-Satchell were built and scored against the shipped 10-day
   close-to-close estimator on the full universe:
   - **YZ re-ranks hard but adds nothing.** Agreement with shipped ranking only
     +0.624; 30% of names move >25% of the field. Out-of-sample skill against a
     low-noise target (non-overlapping 21-day RV, block B -> block A):
     **YZ +0.391 vs C2C +0.386**, paired bootstrap difference +0.005,
     95% CI [-0.094, +0.101]. A coin flip.
   - **YZ is contaminated by overnight-gap share** (+0.45 to +0.51 in both
     blocks; C2C ~+0.10, RS ~0). Its `var(o)` term is a 10-obs sample variance,
     as inefficient as C2C, so its noise floor RISES with gap share
     (0.681 -> 1.214 across terciles, Spearman +0.978) while C2C's stays flat.
     That systematically lifts foreign listings (.SW/.PA) and 24h commodity
     ETFs (IAU, SLV) — SLHN moved 165->20, SAP 149->43.
   - **RS is strictly worse**: out-of-sample -0.133 vs C2C, CI [-0.274, +0.003].
   - **YZ's higher persistence (+0.625 vs +0.445) is repeatability, not skill** —
     it survives per-block gap-share adjustment (+0.640) yet buys zero
     out-of-sample improvement. Persistence alone was too weak a criterion.
   - Out-of-sample ceiling is +0.371 (target predicting itself), and C2C already
     sits at it. **Verdict: leave `.vov_core` alone.**

Refresh cadence: `scripts/refresh_vov_breakpoints.R`, quarterly is ample.
Re-run after a material change to `ScannerUniverse`. Missing table degrades to
"percentile unavailable", never an error.

**Why:** the metric was shipped after this session proved the absolute level
uninformative; the cross-sectional/historical distinction is exactly the sort of
thing a future reader would assume the other way round.

**How to apply:** if asked "is this ticker's vol-of-vol high *for it*", say the
current metric doesn't answer that and offer to test time-series persistence
first — per [[feedback_validate_metric_noise_floor_and_persistence]]. See also
[[project_scanner_universe]] for what the basket contains.
