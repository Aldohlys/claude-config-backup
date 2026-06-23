---
name: Minimum viable dashboard for directional futures trade
description: Once in a directional position, the edge is path+catalyst not vol mispricing; dashboard should track the live edge, not the pre-trade analysis. Watch only what gates decisions.
type: feedback
originSessionId: 0544fcb3-a0ac-4de6-93f8-9e759ecc9335
---
**Rule:** Once a directional futures position is open, the dashboard must track the *live edge* (path + catalyst), not the pre-trade analysis (vol mispricing, smile, multi-DTE structure). Watch only signals that gate a decision.

**Why:** User flagged cognitive overload risk on the MCL Jul'26 Iran-war trade — temptation to watch spot + Brent + WTI Jun + WTI Jul + smile + term structure + news flow simultaneously. Most of these are redundant or informational-only post-entry. Surplus monitoring degrades decision quality, doesn't improve it.

**How to apply:**

For a directional futures swing trade (1-3 weeks), the **minimum viable set**:

1. **Position chart at execution timeframe** (e.g., 15m for swing). Live.
2. **Pre-set TWS price alerts** at entry/exit/add levels. Don't watch — let TWS interrupt.
3. **One thesis-confirmation cross-check**, once daily. For oil/Middle East: Brent-WTI spread. For equity-rate trades: 2s/10s or DXY. For sector trades: relative sector ETF.
4. **One catalyst-path channel** (news / earnings / data calendar).
5. **One curve/term-structure spread** when the thesis has a duration component. Use the first non-expiring spread (see `reference_futures_expiry_week_dynamics.md`).

**Drop after entry:**
- Spot price when holding the front futures (same series, double-counted)
- The expiring contract (post-roll especially)
- Multi-strike IV / smile slope (vol thesis is moot once directional)
- Sub-frequency price series that don't gate decisions

**Principle:** dashboards should match the live edge, not the analytical artifacts that justified entry.
