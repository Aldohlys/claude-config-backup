---
name: feedback_verify_price_move_vs_close
description: "A press \"+X% on the day\" is usually the intraday print, not the close — pull OHLC before repeating any reported market reaction"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: bcd2f874-bc6c-429b-a57e-f3de0e852923
  modified: 2026-09-22T00:59:01.222Z
---

Never repeat a reported price reaction ("shares rose 5% on the announcement") without pulling the
OHLC bar for that date. Financial press quotes the intraday extreme, often from an intraday wire
filed hours before the close.

**Why:** the two can point in opposite directions, and the close is the one that carries
information. Sandoz Capital Markets Day, 2026-09-08: coverage said shares gained "as much as 5.1%".
The actual bar — prior close 68.24, open 70.90, high 71.70, low 66.38, close 68.20 — is a complete
reversal of a +5% gap on a 7.8% range. "Rose 5%" says the market liked the 2030 targets; the bar
says the market faded them. That distinction was the whole conclusion on whether the plan was
already priced in.

**How to apply:**

- When a source gives a percentage move, fetch the daily bar (`yfinance` history, or IBKR
  `get_price_history`) and quote open/high/low/close plus the prior close. State the close move as
  the reaction; mention the intraday print only as context.
- Reversal bars are the informative case: a faded gap on heavy volume after a guidance or
  strategy event says the news was already in the price. Do not let a press headline overwrite that.
- Check the volume on the bar too, so a rebalance or expiry cross is not mistaken for conviction —
  see [[reference_index_rebalance_volume_signature]].
- Same discipline as [[feedback_anchor_net_claims_on_reconciliation]]: the claim gets anchored on
  the primary series, not on the secondary description of it.
