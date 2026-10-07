---
name: reference_tdata_getvalue_batch_all_or_nothing
description: tdata_py getValue returns a bare -1 (0 on no connection) for the WHOLE batch when one contract fails — callers must guard; getGonet does since 5.20.12
metadata:
  node_type: memory
  type: reference
  originSessionId: a3517eda-94f8-48a8-b46e-2095ae048c64
  modified: 2026-10-05T06:27:54.697Z
---

`tdata_py.contract.getValue(list_sym, reqType=...)` (`Tdata/inst/python/tdata_py/contract.py` ~l.50-215) prices a list of symbols in one `qualifyContracts` + `reqTickers` call. Return contract:

- DataFrame `datetime, sym, price` on success. Unsubscribed instruments come back as price 0 or -1 rows (Error 354 / 10168), not as errors.
- Bare `-1` for the ENTIRE batch if any one contract fails to qualify or raises. The typical trigger is a symbol with no Tickers row: the defaults are STK/USD/SMART, so a CHF listing gets Error 200, followed by "can't be hashed because no 'conId'".
- Bare `0` if the IB connection fails.

So one bad symbol silently unprices every other symbol in the call. On 2026-10-05 06:16 a missing DSPF Tickers row left 13 Gonet lines with a NULL price, and no Account row was written ([[reference_gonet_account_history]]).

Guard in R: `gonet_fetch_prices(syms, reqType, fetch)` in `Tdata/R/account.R` (5.20.12). It retries a failed batch one symbol at a time (~15 s per symbol) and NaN-fills any symbol still missing, so `gonet_price_missing` routes it to the Yahoo / last-known fallback. `getValue` itself was left unchanged because it has other callers.

How to apply: any other caller that batches symbols through getValue (RPreTrade, scanners, the Live portfolio) has the same exposure. Check `is.data.frame()` on the result before using it. Before adding a holding to GonetPos.csv, add its Tickers row (with ConId and the correct currency). Related: [[reference_gonet_csv_bookkeeping]], [[reference_tdata_vs_ibkr_mcp]].
