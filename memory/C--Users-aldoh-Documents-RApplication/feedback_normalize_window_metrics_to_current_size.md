---
name: feedback_normalize_window_metrics_to_current_size
description: "A metric differenced across a time window must be normalised to today's position size, per leg — and validated by stable-subset parity with the old formula"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1e2ceadf-edaa-4f9c-82d1-d1e2a412227d
  modified: 2026-09-02T04:20:25.662Z
---

Any metric that differences a **quantity-sensitive total** (market value, notional, exposure) against its own history silently assumes the quantity never changed. It did. Normalise each past observation to *today's* size before differencing:

```
metric = mean over sessions of  SUM over legs of  cur_pos * (cur_price - price_that_session)
       = mean over sessions of  SUM over legs of  cur_mktValue - hist_mktValue * cur_pos / hist_pos
```

**Why:** TODO #78. `WeeklyunPnL = MktValue - avg5(MktValue)` reported ABBN at **-13 439.50 CHF** after 200 of 500 shares were sold mid-window — the market value of the shares sold, dressed up as a price move. Worse, AI (Gonet) came out **+1 862.88** when the truth was **-803.60**: a position *increase* had flipped a loss into a reported gain. A wrong sign is not a rounding problem.

**How to apply:**

- **Normalise per leg, never per position.** A vertical's legs net to `pos = 0` (trade 746 BRK B is `+2 / -2`), so a position-level value-per-share divides by zero. The leg key is `Instrument` for IBKR, `symbol` for Gonet (which has no `Instrument` column).
- **Line legs up with an inner join.** A leg opened mid-window then contributes nothing for the sessions it did not exist, instead of inventing a move; a closed leg drops out. It also keeps the comparison on the same basis as whatever the caller aggregates.
- **Don't quietly drop a row class while refactoring.** CASH rows are bound into the display frame *before* the join, so excluding them would have silently removed their weekly FX move.

**The validation that proves it:** partition positions by whether their size actually changed across the window, then assert **the unchanged subset reproduces the old formula exactly** (Gonet 14/14, U1804173 5/5) while every changed one differs. That separates "fixed the bug" from "changed the number", which a spot-check on the one reported case cannot. Same spirit as [[feedback_validate_metric_noise_floor_and_persistence]].

Related: [[feedback_open_positions_from_portfolio_not_trade_status]] (the sibling mistake — deriving a quantity from the wrong source), [[reference_testing_tuser_box_internals]].
