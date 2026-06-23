---
name: reference-futures-daily-mtm
description: "Futures daily mark-to-market mechanics — how P&L slices hit cash daily vs. stock's single realization at close, TWS column interpretation, and the margin-call risk that drawdowns trigger immediately"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 27ac7a57-c5fd-45f3-8c7e-d48473decb3e
---

# Futures daily mark-to-market — reference for advisory conversations

User comes from a stocks-only background; treat futures intuition as actively being built. Use this whenever discussing P&L mechanics, position sizing, or margin behavior on futures (CL/MCL, ES, GC, etc.).

## The core mechanic

Final P&L on a futures trade is **arithmetically identical** to a stock: `(exit − entry) × multiplier × pos`. The only difference is the **cash-flow path** to that endpoint.

Every trading day, at the exchange's official settlement time, the exchange:
1. Sets a daily settlement price (last trades / VWAP of close window — not the regular session close print)
2. Computes the day's price change vs. yesterday's settlement (or vs. entry on the day of fill)
3. **Debits or credits the broker account in cash** by `(today_settle − yesterday_settle) × multiplier × pos`
4. Resets the position's cost basis to today's settlement; tomorrow Unrealized P&L starts from zero again

**Settlement times** (relevant ones):
- NYMEX energy (CL, MCL, NG, RB, HO): **14:30 ET = 20:30 CET**
- COMEX metals (GC, SI): **13:30 ET = 19:30 CET**
- CME equity index (ES, NQ): **15:00 ET = 21:00 CET**
- CME currency: **14:00 ET = 20:00 CET**

## TWS column interpretation (IBKR)

For an open futures position, the row shows:

| Column | What it means |
|---|---|
| **UNRLZD P&L** | Total profit since position opened, MTM through current tick. **This is what hits cash if you close right now** (minus commission). |
| **SECTOR DLY P&L** / **Daily P&L** | Today's slice only — change since yesterday's settlement. Subset of UNRLZD. |
| **AVG PRICE** | Blended entry across all adds. May or may not reflect MTM resets depending on TWS display mode. |
| **MARKET VALUE** | IBKR's notional collateral calc — **not your cash**, not your P&L. Ignore for trade decisions. |

Closing → realizes UNRLZD as cash, minus ~$0.85/contract commission on MCL (varies by product).

Holding through settlement → today's DLY portion gets paid into cash anyway; the position carries forward with new cost basis = settlement price.

The UNRLZD figure can differ slightly from naive `(LAST − AVG) × mult × pos` because TWS uses a mark price (often bid-side or last-settle-adjusted) rather than the displayed LAST print. Treat UNRLZD as authoritative for "cash on close."

## The practical consequence: drawdowns are real cash, not paper

**Stocks**: a 30% retracement is paper loss; account cash doesn't move until you sell. You can ride it out.

**Futures**: a 30% retracement debits 30% of notional from your account that same day. The drawdown becomes a **cash margin requirement immediately**.

If accumulated MTM losses erode your account below the maintenance margin threshold, the broker issues a **margin call**:
- Add cash by next morning, or
- IBKR auto-liquidates positions at current price — **forcing you out at the worst possible moment**, even if your stop-loss hasn't been hit

This means futures position sizing has **two constraints**, not one:
1. **Stop-loss risk** — what you'd lose if SL fires (same as stock)
2. **Maintenance margin headroom** — what cumulative MTM drawdowns you can absorb before the broker pulls the plug (unique to futures)

For small positions in a well-funded account (e.g. 2× MCL with $325 SL in a 53k account), constraint 2 is trivially satisfied. For large positions or thin accounts, constraint 2 is what catches people.

## Secondary differences worth knowing

- **Interest mechanics**: cash credits earn USD interest immediately, debits stop earning the moment they leave. Long-held winning futures positions accumulate carry naturally. Minor at retail size but explains why futures prices embed cost-of-carry vs. spot.
- **Tax timing (jurisdiction-dependent)**: many countries treat each MTM settlement as a taxable event, so a position open across year-end has its 31-Dec MTM treated as realized. **Switzerland exempts capital gains for private investors on most instruments** — user is Swiss → not their problem.
- **Currency conversion**: each day's MTM hits the USD sub-account separately. FX exposure accumulates daily vs. one conversion event at close (matters for [[ibkr-fx-exposure]] tracking).

## How to use this in advisory conversations

- When proposing a futures trade, size against **both** SL risk AND margin headroom.
- When user reports an open futures P&L number, confirm whether they mean **UNRLZD** (cash on close) or **DLY** (today's slice).
- When discussing drawdown tolerance, do not import the stock intuition of "ride it out" — the broker will force a decision via margin call before the trader does.
- For trades held across year-end in non-CH jurisdictions, flag the MTM tax-realization effect.

## Related
- [[reference-cl-options-multiplier]] — CL=1000, MCL=100; gets the per-tick MTM math right
- [[reference-tws-futures-term-structure]] — curve plot for futures
- [[ibkr-fx-exposure]] — how USD MTM credits flow through to CHF base
