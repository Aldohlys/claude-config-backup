---
name: project-bot-book-design
description: BOT book design validated on 2.5y of trades — risk cap, ATR barbell, hold range, entry factors
metadata:
  type: project
---

`Strategies/Breakouts/bot_book_design_20260827.md` — built on 93 BOT trades, 2024-01-09 to
2026-07-02 (39W/54L, R unit 241 USD, +0.461R = +111 USD/trade). Trailing 12m (51 trades) is
the better half and is NOT the baseline.

Headline settings, all evidenced in the doc:
- **Risk cap USD 300 per trade** (was CHF 600 ~= USD 745, ~2.5x what was actually taken).
  Changed across plan/checklist/quick-ref. Realised avg loss -0.78R vs a -0.50R target —
  cutting losses sooner is the single highest-value change.
- **ATR barbell confirmed**: 1.7-3.0% veto holds (45% of trades, 10% of R). Both ends pay.
  Low arm is event-gated — AAPL +11.0R at 1.68 ATR vs CSCO -1.9R at 1.46.
- **Hold 2-4 weeks**, floor ~10 sessions. <=7 calendar days ran 32% WR.
- **F1 ATR percentile >= 75** (own trailing 2y) — strongest single filter, 20.7R of 42.9R.
- **F2 `|Close_t - Close_t-20| >= 2 x ATR14`** — unsigned; direction comes from S1/BK3.
- **VRP does NOT separate entries** (0.28R cheap vs 0.42R rich). Collinear with F1.
- **Extension helps, does not exhaust** — prior travel >3 ATR beat <1 ATR by 0.5R/trade.
- **MFE/MAE**: winners 3.08 / 0.37 ATR, losers 0.77 / 1.48. Trades declare themselves early.
- Two clocks: price moves on SESSIONS, premium decays on CALENDAR days (5 per 7). Each
  session of movement costs 1.40 calendar days of theta.

Book size is signal-limited, not capital-limited: 3.13 trades/mo = 1.64 concurrent. 5
concurrent needs 3x the rate. 10 dropped as unrealistic.

Sections 9a-9d record claims I made and then retracted — read them before repeating any.

Related: [[project-bot-universe-scanner]], [[feedback-no-static-option-judgements-in-screens]]
