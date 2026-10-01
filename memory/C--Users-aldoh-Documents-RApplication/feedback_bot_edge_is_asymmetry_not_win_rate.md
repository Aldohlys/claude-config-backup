---
name: feedback_bot_edge_is_asymmetry_not_win_rate
description: BOT edge = payoff asymmetry x many bets; win rate is <50% by design and still profitable (EV>0). Daily sheet must stay simple and efficient; never add filters that raise win rate at the cost of fewer bets
metadata:
  node_type: memory
  type: feedback
  originSessionId: 1c26deb6-0a13-4dd0-99ab-91d81259daed
  modified: 2026-09-24T17:17:51.937Z
---

User, 2026-09-24: "Simplicity and efficiency are needed in a daily BOT spreadsheet. Edge will come from asym parameter and trying many bets, not for predictive better win:loss ratio. In fact my win:loss ratio is worse than 50% but the strategy BOT has been profitable, because EV is > 0."

**Why:** the strategy profile is win rate < 50% with avg win / avg loss > 2 ([[project_bot_three_class_framework]]). Rules that raise win rate crush the fat right tail ([[project_bot_exit_and_timeframe]]: P(>2) 17.8% -> 3.3%). Every entry-side predictor tested failed to resolve (n~95, SE~0.29), so filters built on prediction mostly just cut the number of bets.

**How to apply:**
- Judge any BOT_daily change by (a) does it keep the sheet short and fast to read, (b) does it keep the number of candidate bets up, (c) does it sharpen the asymmetry reading. Not by whether it predicts wins.
- Prefer removing a veto over adding one unless the veto protects the loss side per trade (e.g. `gap_through_stop`, a stop that is not a stop). The in-zone veto was dropped for exactly this (TODO #94: 46% of entries removed, no outcome difference).
- Two asymmetries exist and must not be confused: the underlying's chart `asym` = (target-px)/(px-stop), which rests on levels with no measured edge (#94), and the option structure's `(width-debit)/debit`, arithmetic on the quote with no inference.
- Default output = few columns; the rest goes behind `--detail`.
