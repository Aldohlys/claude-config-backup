---
name: Trade execution & order-entry discipline
description: Framework for presenting trade entries (price ladder vs single mid) and rules for working limit orders on spreads — avoid chasing, compute leg-level mid, never trust lazy combo quotes
type: feedback
originSessionId: 10fa004f-7f11-4a79-88ea-5b9c8dc47262
---
# Trade Execution Framework

When proposing option trades, present entry pricing as a **decision ladder**, not a single mid price. And when advising on order working, push the user toward disciplined behavior over chasing.

**Why:** learned from JPM May 15 $330/$340 call spread trade (2026-04-24). User saw TWS combo quote at 0.65/1.75 at session open — single "target mid = $1.30" gave no framework for when to walk away or pay up. User also laddered orders $0.90 → $1.00 → $1.10 → $1.30 in $0.10 steps over ~40 minutes before filling at mid. Both issues would be avoided with an explicit price-ladder framework and leg-level mid calculation upfront.

## How to apply

### 1. Entry price ladder (include in every trade proposal)

For each proposed structure, quote four price tiers, not just mid:

| Tier | Definition | Action |
|---|---|---|
| **Target** | mid − 0.05 to −0.10 | Post one patient limit, let MMs come to you |
| **Acceptable** | at mid | Cross to mid if patient limit doesn't fill in 10–15 min |
| **Red line** | price at which a **competing structure becomes more attractive** (economic threshold, not "mid + X%") | Do not pay above this |
| **Hard no** | ask-side | Only if there's an override reason (e.g., tape running away on valid news) |

**Red line is derived, not picked.** Compute it by asking: "At what debit does another structure in my proposal set give better R/R per dollar?" That price is the economic ceiling.

### 2. Compute leg-level implied mid, don't trust the combo quote

```
long_leg_mid  = (long_leg_bid  + long_leg_ask)  / 2
short_leg_mid = (short_leg_bid + short_leg_ask) / 2
spread_mid    = long_leg_mid − short_leg_mid
```

**Why the combo quote lies:** pre-10:00 ET and on illiquid strikes, MMs post "lazy combo quotes" much wider than leg-level arithmetic justifies. Individual legs trade more frequently and reflect actual order-book interest; combo quotes are algorithmic and often stale.

**Cross-check with the vol surface.** If we pulled IV per strike, the BS model price from current IV is a second, independent fair-value anchor. When per-leg bid/ask is itself unreliable (deep OTM, pre-open), model price wins.

**Cross-check against adjacent strikes.** A leg quote that breaks skew smoothness is unreliable — interpolate from neighbors.

**Decision rule:**
- Per-leg math and model price agree → trust the spread_mid, post accordingly.
- They disagree by > 10% → model wins.
- Both look unreliable (pre-open, illiquid) → **don't trade, wait** for quotes to tighten (reliably by 10:30 ET).

### 3. Order-working discipline — don't chain probing bids

**Bad pattern:** laddering in $0.10 steps (0.90 → 1.00 → 1.10 → 1.30). This:
- Wastes 30–40 minutes of attention for zero price improvement
- Signals directional urgency to MMs ("this person will keep paying up")
- Trains MMs to wait you out

**Good pattern:** at most **two orders per session** on a limit spread:
1. **Patient:** implied_mid − 0.05 to −0.10, give it 10–15 min
2. **Fair:** implied_mid exactly, fill or walk away

If neither fills, the trade isn't working today — come back tomorrow with fresh quotes.

### 4. Execution footnotes to include in trade cards

- Morning (pre-10:00 ET) option spreads are **2–3× wider** than intraday. For trades where the red line matters, default to fill attempts after 10:00–10:30 ET.
- **Never market-order a spread** — only limit at a defined debit.
- **Mid fill with no adverse underlying move = textbook good execution.** Users sometimes misread "MM on the other side" as losing — MMs are always the other side of retail spread orders, and a mid fill means neither side captured edge.
