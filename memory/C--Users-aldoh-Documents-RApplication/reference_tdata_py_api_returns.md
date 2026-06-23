---
name: reference_tdata_py_api_returns
description: tdata_py Python API return structures — list_historical_config / update_historical_data
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6521c13e-ba17-4066-985c-d891cb840ed3
---

Return-key conventions for the `tdata_py` historical-data API (easy to get wrong):

**`list_historical_config(return_dict=TRUE)`** returns:
- `active_contract_details` (NOT `contracts`) — list of contract dicts
- `collection_settings` (nested) — `historical_duration`, `historical_frequency`, etc.
- `contract_summary` — `total_contracts`, `active_contracts`, `expired_contracts`
- each contract dict: symbol, trading_class, expiration, strike, right, exchange, active, last_updated (as of v5.8.22)

**`update_historical_data()` / `collect_data_for_active_contracts()`** return:
- `contracts_processed`, `contracts_errors`, `files_updated` (ints)
- `errors` (PLURAL, list of strings) — NOT `error` (singular)
- `error` (singular string) appears only on a top-level exception

Pairs with the R `$` partial-matching trap ([[r-partial-matching]]): always read these with `result[["errors"]]`, never `result$error`.

**Spot + option-value fetchers (used for the IV-surface capture, TODO #50):**
- `tdata_py$getValue(sym)` → data frame with columns **`datetime, sym, price`** — spot is `price`, NOT `value`. (`getLastPrice` does not exist.)
- `tdata_py$getOptValue(sym, expiration, strikes, right)` → per-strike data frame **`strike, value, bid, ask, impliedvol, delta`** (delta signed: calls +, puts −). Cache-backed (quotes TTL ~30 min); the right place to source per-strike IV/delta. `getStrikesfromExpDate(sym, expdate)` lists available strikes; `getExpirationDates(sym, min_date)` returns the sorted expiry list. `get_chain_oi(...)` returns OI/volume only — NOT greeks.
- Reference consumers in `Tdata/R/volatility.R`: `captureOptionSurface`, `getOptionPrices`.

