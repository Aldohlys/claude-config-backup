---
name: project_bot_gate_redundancy
description: "BOT's 9 gates carry ~5.2 effective dimensions weighted 6:1:1 (structural, solid); but NO scoring scheme predicts realized P&L on 109 real trades, and the vol-expansion proxy is invalid - vol contracted in 65% of BOT trades"
metadata:
  node_type: memory
  type: project
---

Measured 2026-08-31 on 205 scanner-universe names, 9,630 events (24 signal
dates/name, 21d step, 30d forward), using the production `calc_ind()` from
`RStudies/reports/shared/indicators.R`. Bootstrap CIs are date-clustered.

## The nine gates are three clusters, weighted 6:1:1

Effective independent dimensions: **5.2 of 9**, only 3 eigenvalues > 1.

- **trend/position (lambda 3.27, 6 gates)**: S1,S2,S4,BK1,BK2,BK3.
  S1<->S2 correlate **0.80**; underneath, `ma50_disp`<->`ma50_slope` 0.95,
  `ma50_disp`<->`rsi14` 0.92, `rsi14`<->`rng_pct` 0.92.
- **compression (1 gate)**: S5 — orthogonal to everything (-0.04..+0.10).
- **volume regime (2 gates)**: S6, BK4 — also orthogonal, and mildly *opposed*
  to each other (-0.12; they share `vol_ma20` on opposite sides of a fraction).

So `S:5/6 BK:4/4` reads as ten confirmations but is ~three, and the loudest
component is the least specific one.

## What BOT P&L actually is

Greek attribution over 95 closed trades: **delta explains return-on-risk at
+0.797, vega -0.146**. A leveraged directional bet financed by theta - not a
volatility trade. Full detail: [[project_bot_three_class_framework]].

## A vol-expansion event study said "rebalance" - REAL TRADES SAY NO

A 9,630-event study (24 dates/name, 30d forward) scored schemes against *vol
expansion* and found the flat 9-gate score insignificant (+0.047, CI
[-0.009,+0.104]) while S5+volume regime reached +0.234, with the 6/9 trend
cluster contributing nothing (-0.028). **That conclusion did not survive
contact with the 109 closed BOT trades (2022-07..2026-08).**

- **The proxy is invalid.** Realized vol expansion vs return-on-initial-risk:
  **+0.055, CI [-0.162,+0.267]**. Median vol_exp is **0.87** and vol EXPANDED in
  only **35%** of trades - winners 0.91, losers 0.82, both below 1. Despite
  being a debit strategy, realized BOT P&L is *not* a vol-expansion bet.
  Optimizing the score for vol expansion optimizes the wrong thing.
- **No scheme predicts realized P&L.** Spearman vs return-on-risk, n=105:
  flat -0.050, declustered -0.084, S5+volregime -0.063, S5 alone -0.081,
  trend -0.028. Every CI spans zero (width ~+/-0.20). Scanner-era only
  (>= 2025-09-20, n=48): +0.069 / +0.025 / -0.003 / +0.015 / +0.077 - all noise.
- **S5 at entry associated with WORSE outcomes**: S5=1 win 36%, median -0.487;
  S5=0 win 47%, median -0.158 (n=39 vs 66, noisy, but opposite to the study).
- No range restriction to explain it away: entry scores span 0-8, sd 2.10.
  Many trades were entered at score 0-1, so the score was never a hard gate.

**Do not rebalance `breakout.R`.** Confirmed by S-2 (2026-09-01), which re-ran
everything against the *directional* outcome S-1 validated, with gates mirrored
per direction and horizons matched to the realized 12-trading-day median hold:

- 19,260 evaluations, within-direction, date-clustered CIs: **every scheme, every
  horizon, both directions lands in -0.049..+0.023 with every CI spanning zero.**
- Scanner-as-used (score picks the side): **49.8% win rate**, mean signed return
  -0.057% at 12d vs a 0.000% coin-flip baseline. Conviction doesn't help.
- 94 real trades, direction-correct: flat -0.001 [-0.200,+0.195], all null.

**RETRACTED:** the earlier "trend cluster is counter-predictive" (-0.120 on 30-day
MFE) was an artifact of a 30-day horizon (~2x the realized hold) plus MFE, which
over a long window tracks volatility and mean reversion rather than captured
direction. The score is not harmful - it simply has no measurable ranking power.

## Vol-of-vol must stay a universe filter, not a gate

Added as an equal third vote it makes the best score *worse* (+0.190 vs +0.234)
on the vol-expansion proxy - which the real trades then invalidated, so treat
that as weak evidence at best. Consistent
with [[project_vov_percentile_is_cross_sectional]]: it is a slow name attribute
(persistence +0.45), not signal-timescale information.

`atr_pct` is the only existing BOT indicator in that same category
(**ICC 0.646** — two-thirds of its variance is ticker identity). Everything else
is genuine per-signal state (ICC <= 0.112; `adx10` only 0.040). The three name
attributes are mutually near-orthogonal: vov<->ATR% +0.026, vov<->gap share
+0.063 — three different statements (how variable vol is, how large, when it
arrives).

**Why:** the user's concern is focus vs. noise from multiplying indicators. The
dilution is already inside the score; subtraction beats addition here.

**How to apply:** do not propose adding vov/gap-share gates to `breakout.R`, and
do NOT propose rebalancing the trend cluster either — the outcome evidence for
that was a proxy the real trades refuted. What survives is structural and
outcome-independent: `S:5/6 BK:4/4` must not be READ as ten independent
confirmations, and `atr_pct` is a name attribute. Before any future score
change, validate the outcome variable against `Trades WHERE Strategy='BOT'`
FIRST — that step is what caught this. Caveats: one 4-year regime, overlapping 30d
windows, vol-expansion is a proxy (true debit P&L needs entry IV, unavailable
historically). Scripts: scratchpad `bot_audit.R`, `rebalance.R`, `variants.R`.
