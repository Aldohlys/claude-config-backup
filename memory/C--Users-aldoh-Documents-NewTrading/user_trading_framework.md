---
name: User Trading Framework
description: Core trading philosophy - liquidity/correlation framework, sector definition, BOT trade mechanics
type: user
---

## Trading Philosophy
- Swing trader: 2-6 week holding period, 10-15 simultaneous trades
- BOT (breakout) strategy: buy calls (bullish) or puts (bearish) on breakouts
- Sometimes bull call spreads / bear put spreads for defined risk
- Targets from Fibonacci zones, exits when target reached or asymmetry gone
- BS-based option target grids for trade planning (compute option price at target spot + vol + time)

## Market Framework: Liquidity & Correlation
Three timeframes of capital flow (user's own framework):
1. **Liquidity crisis** (days to 1-2 weeks) — correlation → 1, all assets sell, "sell what you can, not what you want." Sit out or trade the bounce.
2. **Sector rotation** (2-4 weeks, the swing timeframe) — correlation structure intact, capital moves between correlated groups. This is where trades live.
3. **Structural accumulation** (months+) — central bank gold buying, AI capex, energy transition. Sets backdrop of which sectors have structural bid underneath.

## Sector Definition
- Sector = group of stocks with **high correlation across regimes**, not GICS classification
- Oil services + shipping should be grouped with Energy (high correlation)
- PLTR and AAPL are NOT same group despite both being "Tech" — different correlation structure

## Key Insights from User
- "Narrow leadership" in a sector is ambiguous: could mean flow just started (only leader caught it) OR flow is fading. Distinguish via breadth momentum direction (expanding vs contracting)
- Avoid earnings dates systematically for traded stocks
- FOMC week paralysis doesn't impact user's trading
- Gold/oil running = sector opportunity, spills over multiple sub-sectors
- Report should say "neutral, no clear direction" when that's the case — don't force a view
- Scenarios should reflect **what investors will DO in 2-4 weeks**, not economic states
- Powell (March 2026): doesn't consider stagflation a real issue; K-shaped economy, immigration impact on workforce, AI on labor — these are the real dynamics

## BOT Trade Mechanics (from Trades DB)
- Opening leg: Pos > 0 (buy option)
- Closing leg: Pos < 0 (sell option), same TradeNr
- 223 BOT trades total (2023-2026)
- Top winners: GLD, AAPL, DOW, JNJ, AMGN calls — progressive profit-taking, Fib targets
- Top losers: SPY (IV crush), MCL (stops), ITB — discipline breakdown, lazy exits
