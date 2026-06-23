---
name: reference_ibkr_fut_fop_contract_fields
description: Field semantics differ sharply between STK and FUT/FOP; getting them wrong silently fails with Error 200 or hangs reqSecDefOptParams. Verified 2026-05-10 against live TWS with MCLN6 / MCO chain.
type: reference
originSessionId: 110f4600-4558-489d-88b2-cdf4eb853b4a
---
These are the field conventions that have to hold for ib_async to qualify contracts and fetch chains. Every detail was verified on a live TWS session against the MCLN6 (Micro WTI Crude Oil Jul'26) future and its options.

## Futures (FUT)

To qualify a future, IBKR needs to identify a single contract in a series:

```python
# WRONG — Error 200
Contract(symbol="MCLN6", secType="FUT", exchange="NYMEX", currency="USD")

# RIGHT — uses the local symbol field
Contract(localSymbol="MCLN6", secType="FUT", exchange="NYMEX", currency="USD")

# Also right — uses conId if known
Contract(conId=661016559, exchange="NYMEX", currency="USD")
```

After `qualifyContracts()`, the contract gets `symbol="MCL"` (the ROOT, distinct from the local "MCLN6"), `tradingClass="MCL"`, `lastTradeDateOrContractMonth="20260618"`, `multiplier="100"`.

## Futures-Options (FOP)

For futures-options, you need:
- `secType="FOP"` (NOT `secType="OPT"` — that's stock options; ib_async's `Option()` factory defaults to OPT)
- `symbol=ROOT` (e.g. `"MCL"`, NOT `"MCLN6"`)
- `tradingClass=options_class` (e.g. `"MCO"` for MCL options — distinct from the future's own tradingClass `"MCL"`)
- `currency` is required (not optional like for stocks)
- `exchange` must match the actual options exchange (`"NYMEX"` for MCL options)

```python
# Verified-correct shape
Contract(
    secType="FOP",
    symbol="MCL",                              # root, not local
    lastTradeDateOrContractMonth="20260616",   # the OPTION's expiration
    strike=92.0,
    right="P",
    exchange="NYMEX",
    currency="USD",
    tradingClass="MCO",
)
# qualifies to conId=860132805, localSymbol="MCON6 P9200", multiplier=100
```

## Tickers DB row convention for FUT
- `Name` = local symbol (`"MCLN6"`)
- `Type` = `"FUT"`
- `TradingClass` = the **OPTIONS** trading class (e.g. `"MCO"` for MCL futures, `"LO"` for CL futures) — **NOT** the future's own trading class shown in TWS (which is `"MCL"`)
- `Exchange` = futures exchange (e.g. `"NYMEX"`)
- `OptExchange` = where the options trade (often the same as `Exchange` for futures)

## Chain fetching: reqSecDefOptParams is unreliable for FUT

`ib.reqSecDefOptParams(root, futFopExchange, "FUT", conId)` does NOT work reliably for futures:
- With `futFopExchange="NYMEX"`: returns Error 322 ("no derivatives") instantly, despite the chain existing
- With `futFopExchange=""`: hangs indefinitely (no end-event)

**Workaround**: use `reqContractDetails` on the FOP root with the options trading class:
```python
fop_root = Contract(symbol="MCL", secType="FOP", exchange="NYMEX",
                     currency="USD", tradingClass="MCO")
details = ib.reqContractDetails(fop_root)  # ~5s, returns 8000+ ContractDetails
# Aggregate (expiration, strike) pairs from details[*].contract
```

For stocks, `reqSecDefOptParams` still works correctly — keep that path.

## Useful conIds for re-testing
- MCL future Jul'26 (`MCLN6`): conId=661016559, expires 2026-06-18
- MCL Jul'26 92 Put: conId=860132805, localSymbol=`MCON6 P9200`
- MCL Jul'26 92 Call: conId=860132815

These are stable across sessions. Re-test scripts (`scripts/test_mcln6_fix.py` and friends) use them as canaries.
