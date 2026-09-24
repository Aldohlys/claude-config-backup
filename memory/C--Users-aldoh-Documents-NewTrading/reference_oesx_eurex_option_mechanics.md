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

**Pulling quotes:** `contract.getOptValue('ESTX50', expiry, strikes, 'P'/'C', currency='EUR', force_refresh=True)` resolves OESX/EUREX from the ticker DB.

**Correction (verified 2026-09-21, 11:33 CET, EUREX open):** the earlier note that *"IV comes back null on OESX"* was a **market-closed artifact, not a property of OESX.** With the exchange open the pull returns `mktdata_type=1` and a fully populated frame — bid, ask, `impliedvol` and `delta` on every leg, including 2027 quarterlies (Jun-18-27 5,900P: 210.3/212.8, IV 18.80%, delta -0.304). The 2026-09-01 "mark prices only, no bid/ask on the 2027 legs" caveat had the same cause. So **pull during EUREX hours (09:00-17:30 CET) and you get executable quotes plus IV; BS-solve only if you are stuck pulling outside hours** (r via [[reference_tdata_interest_rate_utils]], q ~2.86%). See [[reference_closed_market_option_marks_stale]] for the stale-mark trap.

**Index spot:** there is no `contract.getStockPrice` in tdata_py. Pull the cash index with `contract.getValue(['ESTX50'], secType='IND', exchange='EUREX', currency='EUR')` -> a frame with `datetime`/`sym`/`price`.

**VSTOXX is not available on Yahoo** — both `^V2TX` and `^VSTOXX` return "delisted / no data" (checked 2026-09-21), so the vol-regime triggers written against "VSTOXX >= 25-30" have no automatic feed. **Proxy from the OESX surface itself**: the IV printed at the hedge strikes (e.g. ~9-10% OTM Dec put at 20.8%, long-dated 2027 put at ~18.8%) tells you whether there is a panic bid; VIX is a weaker cross-check for European stress. Related: [[project_estx50_hedge_roll_20260601]], [[reference_cl_options_multiplier]] (futures-option multipliers vary: CL 1000, MCL 100, ES 50, OESX 10).
