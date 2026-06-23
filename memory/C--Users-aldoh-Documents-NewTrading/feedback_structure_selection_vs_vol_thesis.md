---
name: Structure selection must match the vol component of the thesis
description: If the surface thesis has a vol edge (negative VRP, cheap skew, long-gamma setup), a vertical spread throws that edge away — only outright longs capture it
type: feedback
originSessionId: 6777ad2b-a236-467c-b8db-01c856609c09
---
## Rule

When a trade thesis includes a **vol component** (negative VRP, cheap call skew, long-gamma favorable, IV percentile low and expected to expand), the structure choice must be able to *capture* that component. Verticals cannot — they are directional instruments by construction and throw vol exposure away.

**Order of structures ranked against a "long vol + bullish" thesis:**
1. Outright long OTM call — full gamma, full vega, full skew capture
2. Risk reversal (buy call / sell put) — captures skew explicitly but takes downside wing risk
3. Debit vertical — *directional only*, vol edge discarded
4. Calendar/diagonal — different vol exposure (term structure, not level), use only if term-structure-specific thesis

## Why

On JPM 2026-04-24, the analysis (§6 of `Trades/JPM_vol_analysis_20260423.md`) correctly ranked outright long call #1, vertical #2, risk reversal #3. Trader chose vertical for capital-sizing reasons (balancer role, not conviction bet) — a legitimate trade-off, not an error.

What next-day data confirmed: IV rose 0.4 vp on a down move (put skew bidding, VRP narrowing toward realized as thesis predicted). Vertical spread benefit from that vol tick: ~$4 on 2 lots (0.4 vp × ~0.05 net vega × 2 × 100). An outright long call would have captured 3–5× more vega offset. The vol thesis was *validating*, but the chosen structure couldn't express it.

This isn't a mistake — it's a calibration point. The trade-off was conscious. But the rule needs to be named so future structure choices are made with eyes open to what's being given up.

## How to apply

Before recommending or endorsing a structure, decompose the thesis:

| Thesis component | Outright long | Debit vertical | Risk reversal | Calendar |
|---|---|---|---|---|
| Directional (delta) | ✓ | ✓ | ✓ | partial |
| Long gamma / realized > implied | ✓ | ✗ (small) | ✓ | ✗ |
| Cheap call skew | ✓ | partial (cap caps the skew capture) | ✓✓ (best) | ✗ |
| IV expansion expected | ✓ | ✗ (small) | ✓ | depends on term |
| Term structure (front vs back) | ✗ | ✗ | ✗ | ✓✓ |

If the thesis has ≥2 vol components, flag explicitly to the user that a vertical discards them — do not silently endorse a vertical as "efficient" when vol edge exists. User may still choose it for capital reasons (as with JPM), but should make that trade-off consciously.

**Corollary:** when reviewing a losing trade with an intact thesis (see `feedback_losing_trade_decisions.md`), check whether the structure was the right expression in the first place. Sometimes the path issue isn't timing — it's that the chosen vehicle couldn't carry the thesis.
