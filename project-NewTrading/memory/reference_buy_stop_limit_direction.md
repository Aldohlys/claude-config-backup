---
name: reference-buy-stop-limit-direction
description: "For a BUY stop-limit order, LMT price must be ≥ STP trigger (and slightly above to allow slippage). Reversing the inequality fills only on pullback, missing the breakout."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 243c14ae-fe86-4883-bf49-ab3946ae5c8e
---

**Buy stop-limit (entry on upside breakout):**
- **STP** (trigger) = price at which the order activates — above current market for a buy stop
- **LMT** (max fill) = highest price you're willing to pay once activated
- **Rule: LMT ≥ STP** (typically LMT = STP + $0.10-0.20 slippage budget)

**Example (MCL breakout, 2026-05-14):** Buy STP $99.00 / LMT $99.20.
- Triggers when price touches $99 going up
- Submits buy-limit order with max fill $99.20
- Will fill anywhere from market up to $99.20 once triggered

**The trap — reversing them:**
- Buy STP $99.20 / LMT $99 would arm at $99.20, then only fill at $99 or below
- Result: order misses a clean breakout continuation, only fills on a post-trigger pullback (i.e., on the fakeout, not the move)
- Looks like price protection but actually inverts the intent

**Symmetric rule for sell stop-limit (exit on downside break):** LMT ≤ STP. Reversing means only filling on a bounce after the stop trigger — misses the move you're trying to stop out of.

**IBKR TWS note:** For CME futures, trigger method is greyed out — exchange-side triggers fire on a single Last print, no candle-close or wick-filter option (per [[reference-tws-no-candle-close-trigger]]).
