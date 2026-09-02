---
name: project_bot_exit_and_timeframe
description: "BOT exits: discretion beats every mechanical rule (targets/trailing/hard stops all reduce expectancy); weekly timeframe adds 3.0 effective dimensions, concentrated in S5/S6/BK4"
metadata:
  node_type: memory
  type: project
---

Measured 2026-09-01 on 90 reconstructed BOT trade equity curves (`unPnL` per leg
+ realized-to-date) and 1,019 daily/weekly observations across 205 names.

## Exits: do NOT mechanize

Actual outcomes: mean ror **+0.301**, win 43.3%, P(>1) 30.0%, P(>2) 17.8%,
payoff ratio 2.29, expectancy +0.26/trade.

**Every simulated rule lost to actual**: breakeven-after-+0.25x +0.207; trailing
give-back-25% +0.102; trailing@1.0x/50% +0.256; hard stop -0.75x +0.265; profit
target +1.0x +0.03. The trailing rule raised win rate 43%->57% while cutting
P(>2) from 17.8% to **3.3%** - the clearest demonstration that optimizing win
rate destroys this payoff.

Two mechanisms:
- **Losers recover**: median loser bottoms -1.10x, closes -0.75x; 28 of 51 went
  below -1.0x then closed above. Hard stops lock in worse than the actual
  average loss.
- **Winners barely dip**: median winner MAE -0.12x vs -1.10x for losers. Little
  for a stop to protect.

A real leak exists (14 trades up >=0.25x closed red, giving back 4,325 ~ half of
net P&L) but every rule that catches it costs more in truncated right tail.

**How to apply:** never propose profit targets or mechanical stops for BOT.
If asked for "clearer exit rules", codify the existing behaviour (don't cap
winners, don't stop mechanically) rather than replace it.

## Multi-timeframe: weekly carries real new information

Weekly gates vs their daily twins (phi): S1 +0.442, BK3 +0.493, S2 +0.387,
BK1 +0.236, BK2 +0.151, S4 +0.109 - but **S5 -0.061, S6 -0.064, BK4 +0.009**,
i.e. the compression and volume gates are *independent across timeframes*.

Effective dimensions: daily 4.9, weekly 5.3, **combined 7.9** - weekly adds 3.0.
Timeframes disagree on the setup ~52% of the time; full confluence is only 4.9%
of observations (1 name in 20).

**Value is in S5/S6/BK4, not the trend cluster** (already 0.39-0.49 correlated
across timeframes - adding weekly trend gates thickens the cluster that is
already over-weighted, see [[project_bot_gate_redundancy]]). Do not ship 18
flags; present a per-cluster confluence state.

**Not validated for profitability** - entry effects are unmeasurable at n=95.
