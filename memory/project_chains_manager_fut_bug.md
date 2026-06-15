---
name: project_chains_manager_fut_bug
description: Three sequential bugs in chains_manager prevented FUT/FOP option-chain fetching. All fixed and live-tested. Kept as historical record.
type: project
originSessionId: 435ffc45-0ae7-46ab-8d17-978675432004
---

**STATUS: RESOLVED** in Tdata 5.10.16 (2026-05-10) and 5.10.17 (2026-05-10). Verified end-to-end against live TWS via `RGetVolMetrics.R MCLN6` returning populated iv15/iv90/iv180. Three distinct bugs in `Tdata/inst/python/tdata_py/chains_manager.py` were stacked on top of each other:

### Bug 1 (5.10.16): qualifyContracts uses wrong field for FUT
`getChains` built `Contract(symbol=sym, secType='FUT', ...)` where `sym` was the local symbol (e.g. `"MCLN6"`). IBKR rejects this with Error 200 — for futures, the local symbol goes in the `localSymbol` field, not `symbol`. **Fix**: branch on secType, use `Contract(localSymbol=sym, ...)` for FUT.

### Bug 2 (5.10.16): reqSecDefOptParams unreliable for FUT
Even after qualifying the underlying, `reqSecDefOptParams(root, '', 'FUT', conId)` returned Error 322 ("no derivatives") with `futFopExchange='NYMEX'` and **hung indefinitely** with `futFopExchange=''`. **Fix**: for FUT, use `reqContractDetails(Contract(symbol=root, secType='FOP', exchange=opt_exchange, tradingClass=options_tc))` instead, then aggregate the (expiration, strike) pairs into the same shape `reqSecDefOptParams` would have returned. The options-class TradingClass comes from the Tickers DB row (e.g. `MCO` for MCL).

### Bug 3 (5.10.17): strike qualification used wrong Contract type for FOP
`_qualify_strikes_with_states` built `Option(symbol=sym, ...)` per strike. For FUT, this:
- Defaulted secType to `OPT` (stock option) instead of `FOP` (futures option)
- Passed `sym="MCLN6"` (local future) instead of root `"MCL"`
Result: every strike returned Error 200; `getOptionStrikes` reported "0 newly qualified" for all 439 candidates after wasting ~25s. **Fix**: branch on secType. For FUT, resolve underlying root via `Contract(conId=underlying_conId)` once, then build `Contract(secType='FOP', symbol=root, currency=currency, tradingClass=options_tc, ...)` per strike. Caller (`getOptionStrikes`) now passes secType, underlying_conId (from `chain[1]`), and currency down.

### Workflow rule that emerged
The user established `feedback_test_tws_first.md` during the resolution: any change touching IBKR/TWS API behaviour must be live-tested against TWS BEFORE the source edit. Each of the three bugs above was confirmed via `scripts/test_*.py` scripts before patching. This caught the second bug (reqSecDefOptParams) which would have shipped silently after only the first fix.

### Reference data (verified 2026-05-10 against live TWS)
- MCLN6 (Micro WTI Crude Oil Jul'26 future) — conId=661016559, exp=20260618, root=MCL, options TradingClass=MCO
- MCL Jul'26 92 Put — conId=860132805, localSymbol="MCON6 P9200", multiplier=100
- Tickers row stores: Name=MCLN6, Type=FUT, TradingClass=MCO (the OPTIONS class, NOT the future's own trading class)
