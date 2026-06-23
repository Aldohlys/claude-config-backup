---
name: reference_timeframe_missions_bot
description: "Multi-timeframe entry framework for BOT — each timeframe owns one decision, capped by the option's 2-4wk clock"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 22ea52dc-242f-4d9a-8122-f80db5e6977c
---

Endorsed 2026-06-10. Top-down nesting where each timeframe owns a DIFFERENT decision (not all scanning for the same thing), and the option's fixed 2-4 week life caps every mission:

- **Macro / sector** = permission + size (veto layer). No sector flow → no liquidity → no trade. Flow must persist across the whole hold window. Sizing/veto, never the trigger (regime has no BOT timing edge — see [[project_regime_backtest]]).
- **Weekly** = (a) room: target must be reachable in 2-4wk — tie to ATR envelope ([[reference_atr_move_multiples]], 10-20d pays ~1.5× ATR typical, up to ~4× strong); (b) not fighting a weekly down-structure. Too slow to be the trigger.
- **Daily** = the thesis: setup, asymmetry, invalidation, AND the clock. By arithmetic the thesis timeframe (10-20 daily bars = the whole position). A weekly setup needing 6-8 weekly bars is irrelevant to a 3wk option.
- **Intraday** = trigger/timing only, tight leash. Refines entry to tighten the stop, but never vetoes the daily thesis. Waiting 3 extra days on a 15-day option = ~20% of life to theta, so a day or two max.

Conflict rule: higher TF wins direction, lower TF wins timing; weekly/daily directional disagreement = stand aside (range). Decide which TF owns which decision BEFORE looking (avoids timeframe-shopping for confirmation).

Patterns (accumulation/distribution etc.) are fractal but NOT equally reliable: weekly/daily = thesis-grade; sub-daily = timing texture only.

The duration-as-clock point is what justifies the time stop in the exit rules — see Step 4 in `Strategies/Breakouts/bot_strategy_checklist.md` (vehicle rule; stock has no clock, uses capital-efficiency stop instead).
