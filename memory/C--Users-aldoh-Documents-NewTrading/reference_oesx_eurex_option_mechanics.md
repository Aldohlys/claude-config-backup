---
name: reference_oesx_eurex_option_mechanics
description: "OESX (ESTX50/EUREX index options) — points×10 multiplier, December = quarterly-only, quarterly most liquid"
metadata: 
  node_type: memory
  type: reference
  originSessionId: ee50ec03-6504-49a0-a229-909d602ca8db
---

OESX = EUREX options on the EURO STOXX 50 (ESTX50 / SX5E cash index). Ticker DB: TradingClass OESX, Exchange/OptExchange EUREX, Currency EUR, Type IND, **Multiplier 10**, Div_yield ~2.86%.

**Quoting & sizing:** premiums are quoted in **index points**; the contract multiplier is **10 EUR per point**. So a 125.0-point spread = 1,250 EUR/lot; a 169.7 put = 1,697 EUR. There are no "shares" — don't say "/sh". Multiply any quoted premium by 10 for EUR/lot.

**Expiries (verified 2026-06-02 via `chains_manager.getExpirationDates`):** ~6 months out, EUREX lists only standard monthly expiries (3rd Friday) — **no weeklies that far out**. December has a *single* expiry, the **quarterly Dec 3rd-Fri** (Mar/Jun/Sep/Dec). The quarterly carries the institutional/LEAP open interest and is the **most liquid**: at the 5,700P, bid/ask spread was 1.7 pts (Dec quarterly) vs 2.0 (Nov monthly) vs 3.1 (Jan monthly). Term IV was nearly flat (~18.7/18.6/18.2% Nov/Dec/Jan) → rolling out a quarter costs ~nothing on a vol basis; pick the quarterly for liquidity.

**Pulling quotes:** `contract.getOptValue('ESTX50', expiry, strikes, 'P'/'C', currency='EUR', force_refresh=True)` resolves OESX/EUREX from the ticker DB. **IV comes back null on OESX** — BS-solve from the mid when needed (r via [[reference_tdata_interest_rate_utils]], q≈2.86%). See [[reference_closed_market_option_marks_stale]] for the EUREX-closed stale-mark trap. Related: [[project_estx50_hedge_roll_20260601]], [[reference_cl_options_multiplier]] (futures-option multipliers vary: CL 1000, MCL 100, ES 50, OESX 10).
