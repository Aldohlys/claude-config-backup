---
name: reference_dt_basecurrency_sort_pin_row
description: DT pattern — sort currency-formatted columns by hidden base-ccy amount + pin TOTAL row
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1c542077-4f43-4c81-8b96-d63da2d47d58
---

DataTables pattern used in `Tuser/symbol/logic/datatablef.R` `portf_datatable()` (Positions table, routine Portfolio tab), shipped 2026-06-08.

Currency-formatted display cells (e.g. `currency_format(MktValue, currency)`) are strings, so DT sorts them alphabetically — 100 USD ranks above 90 CHF. Fix without changing what's displayed:

- In the data prep (`portfolio/logic/portf.R` `prepare_portf_data_table()`, BOTH the options/IBKR and Gonet branches), emit hidden numeric columns `t_MktValue_base` / `t_WeeklyunPnL_base` / `t_unrealizedPnL_base` = `convert_to_base_date(<raw>, currency, portf_date)`, plus a `t_is_total` flag (1 for the TOTAL row). `convert_to_base_date` is identity for the TOTAL row since it's already base ccy.
- In `portf_datatable()`, build `columnDefs` by 0-based index (`which(names(dt)==col)-1`, rownames=FALSE): hide the helper cols, and for each display col add `list(orderData = <hidden idx>, targets = <display idx>)` so the cell sorts by the base-ccy amount.
- Pin the TOTAL row to the bottom with `orderFixed = list(pre = list(<t_is_total idx>, "asc"))` — `orderFixed.pre` is always the PRIMARY sort, so the flag groups TOTAL last while the user's clicked column is the secondary sort. (`post` would NOT work — it's only a tiebreaker.)

Logic is generic: helpers/orderFixed only apply when the columns are present, so the shared function serves both account types. Per-position cells still display in trade currency; only the TOTAL cell displays in base currency. See [[reference_account_cashflow_conventions]] for base-ccy conversion conventions.
