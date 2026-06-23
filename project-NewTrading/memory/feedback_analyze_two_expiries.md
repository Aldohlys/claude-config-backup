---
name: /analyze — enumerate spreads at two expiries (~30d + ~50-60d)
description: Run the Phase D.4 spread enumerator across two expiry buckets so the report can show the time-premium / R:R trade-off
type: feedback
originSessionId: 791d7ea0-3b98-46b6-b585-5209bd9337c8
---
In `/analyze` Phase D.4, run `compute_spread_risk_reward` for at least two expiries: a ~30 DTE bucket and a ~50-60 DTE bucket (both clean of the next earnings event when possible). Report top spreads from both, with the time-premium trade-off explicit.

**Why:** A single 30 DTE enumeration shows only the cheap-debit / high-R:R / lottery profile. Adding the longer expiry surfaces the higher-EV / higher-Δ-prob alternative — same cap, different shape of edge. The user wanted the full risk-budget picture across the time axis to choose between "cheap lottery" and "more time, more probability".

**How to apply:** Set `EXPIRIES = [front-30, mid-50]` in the Phase D.4 driver. Concat results, tag with `expiry`, sort by R:R desc and EV desc separately. Report at minimum: top-by-R:R combined, top-by-EV (usually 51-60d), and a few near-ATM 5pt rows for the high-prob/low-R:R band. Skip the longer expiry if it crosses the next earnings.
