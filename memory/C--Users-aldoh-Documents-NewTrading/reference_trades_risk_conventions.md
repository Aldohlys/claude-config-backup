---
name: reference-trades-risk-conventions
description: "How Trades.Risk is set per strategy (signed deltas summed per TradeNr) - VALUE stop-based on first row, FOREX 30% of Total, BOT cash-retire on partial close, options max loss at expiry; RReporting defaults that need correcting"
metadata:
  node_type: memory
  type: reference
  originSessionId: e87e0ecc-b1ff-410a-a8f3-4449c3418f84
  modified: 2026-10-08T08:31:08.959Z
---

Trades.Risk is a **signed delta per row**: the trade's risk = sum(Risk) over its TradeNr, and a closed trade nets to 0. Native currency, commissions excluded unless noted.

- **VALUE (U25343478 stocks):** shares × (entry − Stop), with Stop in its own column. On an add, the row's delta = added shares × (fill − stop); e.g. 685: 400 × (874 − 600) + 200 × (766 − 600) = 142,800 JPY. The Stop must equal the resting STP order's trigger. **RReporting's Open dialog proposes the full cost** unless a Stop is typed in. Rows booked without one (755/756 on 2026-10-08) had to be set by hand.
- **FOREX:** Risk = 30% of Total (720: 439,109.8 × 0.3 = 131,732.94).
- **BOT options:** opening debit = +abs(net Total). A partial close of long options retires risk one-for-one with the cash received, never below 0 (`compute_risk_delta_BOT`). So trade-level Risk = cash still invested, i.e. the loss if what remains expires worthless (FCX 750: 296.23 − 32.61 = 263.61). It is not the cost of the remaining lot.
- **Other strategies on options (Perso, LTO, CAL, ...):** since RReporting 6f9a413 (2026-10-08), legs sharing one expiry get their max loss at expiry (`option_max_loss_at_expiry`); trade 758, the SPY 722/712 debit spread, = 85.38. Before that the default was sum(abs(Total)), the gross premium of both legs (585.99). It still falls back to that for calendars, unlimited loss, or a missing strike or expiry.

Related: [[reference-orders-tab-matching]], [[feedback-no-db-writes-behind-rreporting]], [[project-u25-fx-hedge-policy]].
