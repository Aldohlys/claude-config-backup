---
name: reference_box_spread_cash_parking
description: "Box spreads to park idle cash — SPX/XSP not SPY, and why FX-hedging collapses the yield to the home-currency rate (covered interest parity)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: bd9ad6e2-6358-4db8-8add-e9ee8e2cfee1
---

Box spread ("long box" = buy K1/K2 call+put spreads) to earn interest on idle cash.
Full write-up: `Trades/box_spread_cash_parking_20260710.md` (2026-07-10).

Key points:
- Use SPX or XSP (European, cash-settled, no early assignment). NEVER SPY (American → early assignment breaks the risk-free property).
- Held to one expiry the box = (K2−K1) exactly; no pin/directional risk; minimal margin.
- FX wall: a box is a USD money-market asset. Hedging the FX (sell USD fwd, or borrow USD to fund) collapses the return to the CHF rate — covered interest parity. Unhedged = USD rate but long USD/CHF, whose vol swamps a short-horizon coupon.
- Equity analogy differs: hedging FX on a USD stock keeps the stock return (separate from the hedge's rate-differential cost); on a pure-interest instrument the kept return IS the rate differential, so they cancel. No USD rate on CHF cash risk-free.
- For this account, unhedged USD box blows past the CHF 0–+6k USD net-liq band ([[ibkr_fx_exposure]]).
- Only worth it: large matched-currency balance, longer horizon (LEAPS-dated), earning below box implied rate.
- CHF SMI box NOT viable (checked live 2026-07-10, 100k to Sep): CHF 1-3mo rate ≈ −0.04% (negative; SNB ~0) so no yield to harvest; OSMI legs ~20pts wide → near-money 500-wide box costs ~531pts natural vs 500 face = ~290 CHF/box entry loss, thin book (some strikes 0 OI). Use index OSMI not Swiss single-stock (American). USD 3mo 3.73% = yield lives in USD only.
