# IBKR FX Exposure & Daily P&L Mechanics

## Daily P&L Composition (base currency: CHF)
The DAILY P&L header in TWS Portfolio includes ALL of the following:
1. **Position P&L** — security price changes, converted to CHF at today's rate
2. **FX translation P&L** — positions unchanged but FX rate moved, revaluing in CHF
3. **Cash FX P&L** — FX rate changes revalue non-CHF cash balances
4. **FX derivatives P&L** — e.g. 6S (CHF futures) options structures

The individual DLY P&L column shows per-position moves (possibly in native currency), so summing them will NOT match the DAILY header which includes FX effects.

## FX Cash Column (Real FX Balance)
- **NOT actual cash** — it's a virtual/accounting entry
- Represents the implicit currency funding from holding positions in that currency
- Created automatically when trading non-base-currency instruments
- Disappears as you close the positions that created it
- **Does NOT trigger debit interest** — IBKR charges interest on Settled Cash, not FX Cash
- Only real cost is **FX risk** (exposure to exchange rate movements)

## User's EUR Positions (as of 2026-02-27)
- EUR FX Cash created by: AI (AIR) diagonal spread + SAF bull call spread
- UBSG is CHF-denominated (not EUR despite being on EUREX)
- User converted ~6,000 CHF to EUR to neutralize the FX risk

## User's USD Exposure
- Long USD via FX Cash (~15,764) from US-listed positions (QQQ, AMD, PFE, HAL, EWY, etc.)
- Short USD via 6S risk reversal: Long 1.30 Call / Short 1.215 Put / Short 1.33 Call (Jun05'26, CME)
- 6S notional delta: ~31,808 CHF
- Net USD exposure: ~-16,000 (net short USD)
- USD Settled Cash: +25,448 — earns credit interest (~4.3%)

## Flattening FX Exposure
- "Close Currency Balance" in TWS only closes real cash, not virtual exposure
- After conversion, FX Cash stays negative but real cash offsets it — net exposure near zero
- FX Cash goes to zero only when underlying positions are closed

### FXCONV vs IDEALPRO (confirmed via IBKR docs 2026-06-02)
- **FXCONV** = currency *conversion*: changes cash balances but **does NOT create/affect the Virtual FX Position** in the FX Portfolio. No trade record there. ("when the order fills your virtual position in the FX Portfolio section will not be affected")
- **IDEALPRO** = FX pair *trade*: **creates/nets a Virtual FX Position** (avg cost + daily P&L tracking). Geared to ≥~25K units; smaller fills as odd-lot.
- Both produce the *same* real cash effect; only the FX-Portfolio recording differs.
- **To REDUCE an existing virtual FX position (e.g. trim an oversized USD.CHF short): use IDEALPRO** — a buy nets against the existing short and shrinks it on-screen. FXCONV would leave the virtual short untouched and just add offsetting cash (net-neutral but display stays confusing).
- TWS "Let TWS make this determination automatically" → defaults FX orders to FXCONV destination. Source: ibkrguides.com FX Portfolio + Traders' Academy "Converting Currency".

## Reading NET currency exposure (the recurring confusion — resolved 2026-06-02)
- **Two windows, two views, must COMBINE them:** `Nt Lqdtn Vl` per currency (Market Value – Real FX Balance) = what you HOLD (cash + securities), EXCLUDES FX-pair positions. FX Portfolio (Virtual FX Position) = open FX-pair trades, marked daily.
- **True net exposure = Nt Lqdtn + FX Portfolio legs.** Neither window alone is the answer. `Nt Lqdtn` looking long does NOT mean net long if a virtual short sits in the FX Portfolio.
- **The daily P&L is the arbiter:** virtual FX positions carry REAL daily P&L (e.g. EUR.CHF long gains when EUR rises; USD.CHF short loses when USD rises). If a virtual position moves your DAILY header, it's a live exposure — count it.
- **User's actual goal:** FX-NEUTRAL on USD trades (long SPY/NVO/etc. create unwanted long-USD), NOT a USD/CHF bet. Wants GROSS hedge, low-churn — does NOT want to trade FX per position (commission drag).
- **DIRECTION (corrected 2026-06-02 — I had it backwards twice):** user is net **LONG** USD ≈ USD `Nt Lqdtn` (13,115 CHF ≈ 16,700 USD as of 2026-06-02). The −32,500 USD.CHF virtual short is NOT additive — it's the record of conversions already inside the low USD cash, hedging PART of the long. Account is still long → to neutralize, **SELL/convert USD→CHF** (~16,700 USD). Buying USD.CHF makes it MORE long (wrong way). Proof you're long: Net Liq 58,248 = Σ per-ccy Nt Lqdtn with NO virtual term; total Unrealized −2,451 = securities only, excludes virtual FX −259.
- **Hedge to MARKET VALUE (USD NAV), NOT delta-notional.** Delta is wrong for FX (it's price sensitivity, not USD held) AND churny. Delta-adjusting would let the long SPY put's ~−$25-30k delta-notional falsely flip you to "short USD". Currency risk = liquidation value of USD assets = `Nt Lqdtn USD`. Premiums are small/slow → low drift.
- **Operating rule:** keep a fixed USD cash buffer; trading cash↔option within it doesn't move USD NAV (no re-hedge). Re-hedge only when `Nt Lqdtn USD` drifts past the band (fresh CHF→USD funding or big realized USD gain). Quarterly glance, not per-trade. **One metric: `Nt Lqdtn USD`; ignore the virtual short's displayed size / FX Cash column.**

### STANDING FX POLICY (set 2026-06-02, user-confirmed)
- **Target: `Nt Lqdtn USD` band = 0 to +6,000 CHF** (small deliberate long = working reserve, not zero — user wants ~3.5–5k USD cash dry powder for new long-USD option trades).
- **Maintenance:** trade out of the buffer freely (cash↔premium doesn't move NAV). Act ONLY if `Nt Lqdtn USD` climbs > ~6k (winning USD trades or fresh CHF→USD funding) → sell the increment USD→CHF. Rebuild buffer from CHF only when dry powder actually runs low (~monthly).
- **Vehicle:** small sells (<25k) via Trade → Convert Currency (FXCONV); leaves the virtual short cosmetically intact but moves `Nt Lqdtn USD`, which is all that matters.
- **EXECUTED 2026-06-02:** sold 12,000 USD→CHF @ ~0.7858. `Nt Lqdtn USD` 13,115 → **3,635 CHF** (~4,625 USD long), USD cash buffer ~2,870 CHF (~3,650 USD). USD/CHF exposure cut ~72% (1% move ≈ 36 CHF now vs 131 before). Account NAV unchanged (~58,148) = clean conversion.
