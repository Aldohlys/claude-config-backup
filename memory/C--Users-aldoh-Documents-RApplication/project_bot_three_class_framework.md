---
name: project_bot_three_class_framework
description: "BOT methodology: P&L is DIRECTIONAL (delta +0.797, vega -0.146); indicators sort into 3 classes; vehicle = outright/vertical/stock on price+liquidity; debit/width sets max payoff and needs no inference"
metadata:
  node_type: memory
  type: project
---

The user's own taxonomy, recorded 2026-09-01. Full detail in `docs/TODO.md` #82
section F. **Strategy profile:** net debit 200-300 USD typical / 400 max,
1-4 weeks, convexity long, win rate under 50% with **avg win / avg loss > 2**,
momentum after a pullback / base / cup-and-handle.

## Foundation: BOT P&L is DIRECTIONAL, not volatility

Greek attribution over 95 closed trades (position snapshots carry IV + Greeks +
`TradeNr`): **delta explains return-on-risk at +0.797**, gamma +0.071,
**vega -0.146**, theta -0.023. Gross: delta +11,154, gamma +9,247, vega +1,931,
theta **-11,355**. Winners and losers differ *only* in delta (median +330 vs
-127); their vega is indistinguishable (+9 vs +14).

**A leveraged directional bet financed by theta.** Theta is the price paid for
the leverage, not a return on it (user proposed the opposite phrasing; the
attribution is unambiguous - theta is negative in winners and losers alike).
This is why the vol-expansion proxy failed - see [[project_bot_gate_redundancy]].

## The three classes - each indicator belongs to exactly one

- **Class 1 - universe.** Names whose behaviour is *consistent*, so the same
  strategy produces no surprises. Slow, name-level, quarterly refresh.
  Members: spread feasibility (below), **gap share** (persistence +0.494; high
  tercile gaps *through* stops), **`atr_pct` band** (ICC 0.646), and the
  **missing option bid-ask filter**. Vol-of-vol is the weakest, a badge at most.
- **Class 2 - coarse screen.** Worth-reviewing names, tuned for **recall over
  precision**: false positives are cheap, false negatives are not. Design
  consequence: **few AND-conditions, one gate per independent dimension**.
  Natural home for the weekly gates ([[project_bot_exit_and_timeframe]]).
- **Class 3 - relative ranking.** Which survivor is better, relatively. Was
  EMPTY before debit/width; everything the score measures is price/volume
  technicals with nothing about what the option costs.

## Vehicle layer (pre-existing methodology, not a finding)

Vehicle is **outright option / vertical spread / stock**, chosen on
**underlying price** and **option liquidity (bid-ask spread)**. It decides which
criteria even apply:

| vehicle | payoff ratio set by |
|---|---|
| vertical | **debit / width** |
| outright | uncapped upside; cost is full premium + full theta (decay -2.93%/day of premium vs -0.70% for spreads) |
| stock | stop distance vs target; no convexity, no theta |

**So the option bid-ask filter GATES WHICH VEHICLES EXIST**, it is not merely a
cost: a vertical crosses the spread on two legs, four times round-trip. Wide
quotes remove the vehicle that makes expensive underlyings affordable.

## debit / width - the one criterion needing no inference

`max payoff = (width - debit) / debit`. **2:1 needs debit <= 33% of width;
3:1 <= 25%; 4:1 <= 20%.** Measured over 22 clean verticals: debit/width median
**26.4%**, max payoff median **2.80:1**, **82% clear 2:1**.

**Why it outranks everything else:** every entry-side *statistical* criterion
tested over three days failed, because n=95 with this outcome variance cannot
resolve effects of ~0.3 (SE ~0.29). debit/width is arithmetic on the quote,
known before entry. It spans class 1 (feasibility: can this name produce a
<=33% spread given its strike spacing?) and class 3 (ranking: best payoff per
dollar among survivors).

## Entry behaviour vs stated thesis

Gate firing at real entries vs universe: S1 79.8% vs 61.8%, S2 72.3% vs 62.5%,
**S5 38.3% vs 38.7% (no selection)**, S6 **26.6% vs 44.5%**, BK4 **16.0% vs
20.8%**. Entries are **trend-following, not base-breakout** - the compression
and volume-expansion signature is absent or actively avoided.

Yet outcome by entry range position is **monotonic** across mean / P(>1) / win:
rng 0-40 -> **-0.02**, 40-70 -> **+0.24**, 70-100 -> **+0.50** (n=26/31/37).
**Buying strength beat buying pullbacks**, contrary to the stated thesis. ~1.4
SE so not significant alone, but three consistent bins x three metrics is the
strongest directional hint found.

**How to apply:** do not propose entry-scoring tweaks as performance
improvements - they are unvalidatable at this n. Deterministic criteria
(debit/width, bid-ask, budget) are where the leverage is.
