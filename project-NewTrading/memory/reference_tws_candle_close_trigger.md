---
name: TWS has no native candle-close trigger; CME futures Trigger Method is locked
description: IBKR TWS triggers all fire on tick; for CME futures the Trigger Method field is greyed out (exchange handles stops on single Last print). Workarounds = STP LMT wick filter, alert+manual, API watcher.
type: reference
originSessionId: 0544fcb3-a0ac-4de6-93f8-9e759ecc9335
---
**TWS has no native "trigger on N-minute candle close" order type.** All built-in triggers (stops, conditional orders, buy-stops, alerts) fire on a tick or aggregate of recent ticks.

**For CME futures specifically, the Trigger Method dropdown is greyed out at "Default"** — empirically confirmed 2026-05-11 on MCL Jul'26 order ticket. Stops on CME products are held *at the exchange*, not at IBKR, and trigger on a **single Last print** at the stop price. TWS cannot override this. "Double Last" / "Bid-Ask" trigger methods (which work for equities) are unavailable for CME futures.

### Workarounds (CME-futures-applicable, ranked simplest first)

1. **STP LMT with tight limit (the wick filter that works)**
   - Order type: STP LMT
   - Stop price = trigger level + $0.20-0.30 buffer (e.g., $96.30 for a "real $96 reclaim" intent)
   - Limit price = Stop + $0.10 (e.g., $96.40)
   - Time in Force: GTC, allow outside RTH
   - Behavior: wick to stop snaps back → triggered but no fill (limit not met) → you miss the bad print. Genuine breakout walks through → fills near stop.
   - Cost: max ~$0.10 slippage on the add. Risk: a genuine fast move can leapfrog the limit and leave you unfilled — acceptable for adds (you miss the trade, not lose money).
   - **This is the practical answer for CME futures.**

2. **Price alert + manual entry**
   - TWS alert on "Last >= X", popup/email/mobile push, then manually confirm candle quality and place order.
   - Best for liquid-session trades where user is available.

3. **Alert with attached order**
   - Alerts → Modify Alert → Action: "Submit order". Pre-built order fires on alert trigger.
   - Can layer conditions (volume, time-of-day) to filter thin-book overnight wicks.
   - Still tick-triggered at heart.

4. **IBKR API watcher (ib_insync / ibapi)**
   - `reqRealTimeBars` or `reqHistoricalData(keepUpToDate=True)` on 15-min bars; on bar close, evaluate condition and send order.
   - ~30 lines of Python. Matches semantic precisely. Worth building once for reuse.
   - User has Tdata/Python infra already.

### Decision heuristic
- One-off CME-futures pyramid add: use #1 (STP LMT + buffer). Accept ~$10-30 slippage cost.
- Recurring pyramid-add strategy: build #4 once, reuse.
- Liquid-session discretionary trade: #2 is fine.

### Where to actually find the (locked) field
File → Global Configuration → Orders → Stop Triggering Method. Visible but inert for CME products. Not in the Order Ticket's Miscellaneous section (where it's also greyed out).
