---
name: reference_dt_idioms
description: "DT idioms in Tuser/symbol/logic/datatablef.R — per-row multi-currency cell formatting, shrinking a table to content width, sorting currency-formatted columns by a hidden base-ccy column, and pinning the TOTAL row via orderFixed.pre"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T02:24:15.535Z
---

Merged 2026-08-27 from two separate memories. All idioms live in
`Tuser/symbol/logic/datatablef.R`.

## 1. Per-row trade currency in one column

`DT::formatCurrency()` applies ONE currency symbol to a whole column, so it can't
render a column whose rows are in different currencies. Instead pre-format each
cell to a string with `Tdata::currency_format(value_vec, currency_vec)` —
`currency` **is** vectorized.

But its `accuracy` arg is **scalar, not per-row** (the `label_currency` closure
captures a single accuracy), so to mix decimals (sub-1 prices → 4dp `0.0001`, else
2dp) split rows into accuracy groups and call `currency_format` once per group.

The data must carry a per-row `currency` column; `basic_portf_datatable()`
consumes it then drops it (`dt$currency <- NULL`) so it isn't displayed. All four
callers (Trade tab `tr`, Symbol tab `sym`, `an_portfUI`, `an_symUI`) select a
`currency` column; when absent it falls back to the old scalar `formatCurrency`.
Shipped 2026-06-30, Tuser commit 1a5d3d0.

## 2. Shrink a DT to content width

A DT `<table>` is `width:100%`, so with `autoWidth=TRUE` columns stretch to fill
the panel and read poorly. To make it compact: give each column a content-tuned
`width` (px) via `columnDefs` **AND** wrap the `DTOutput` in
`div(style="display: inline-block;")` — the inline-block container collapses to
the table's content instead of the full panel width. The Trade-tab stats tables
already used this; the `tr` table got it too.

## 3. Sort currency-formatted columns by a hidden base-ccy column

Used in `portf_datatable()` (Positions table, routine Portfolio tab), shipped
2026-06-08. Currency-formatted display cells are **strings**, so DT sorts them
alphabetically — 100 USD ranks above 90 CHF. Fix without changing what's displayed:

- In data prep (`portfolio/logic/portf.R` `prepare_portf_data_table()`, in BOTH
  the options/IBKR and Gonet branches) emit hidden numeric columns
  `t_MktValue_base` / `t_WeeklyunPnL_base` / `t_unrealizedPnL_base` =
  `convert_to_base_date(<raw>, currency, portf_date)`, plus a `t_is_total` flag
  (1 for the TOTAL row). `convert_to_base_date` is identity for the TOTAL row,
  which is already base ccy.
- In `portf_datatable()`, build `columnDefs` by **0-based** index
  (`which(names(dt)==col)-1`, `rownames=FALSE`): hide the helper cols, and for each
  display col add `list(orderData = <hidden idx>, targets = <display idx>)` so the
  cell sorts by the base-ccy amount.

## 4. Pin the TOTAL row with orderFixed.pre

`orderFixed = list(pre = list(<t_is_total idx>, "asc"))` — `orderFixed.pre` is
always the **PRIMARY** sort, so the flag groups TOTAL last while the user's clicked
column becomes the secondary sort. **`post` would NOT work** — it's only a
tiebreaker.

The logic is generic: helpers/orderFixed apply only when the columns are present,
so the shared function serves both account types. Per-position cells still display
in trade currency; only the TOTAL cell displays in base currency.

**2026-06-30 (Tuser 1a5d3d0):** the same pattern was applied to the routine
**Trade tab** "All trades" stats table — `symf$stats_all()` emits hidden
`t_<col>_base` numerics (StartPrice/Price/Cost/Value/Total/realizedPnL/
UnrealizedPnL/PnL/Risk) + `t_is_total`, and `datatablef$stats_all()` got the
idx0/sort_map/orderFixed treatment (replacing a misplaced `rownames=FALSE`
*inside* `options`, which had left rownames on). `datatablef$stats_all` is shared
with the **Symbol tab** (`displaysymUI`), so it gained the fix for free. Both edits
are additive and back-compatible. The sort+pin logic is factored into
`datatablef$basecurrency_dt_opts(table, base_opts, sort_map)`, reused by
`stats_all` and the Gonet `stats_all_gonet` ([[reference_gonet_pnl_and_stats]]).

See [[reference_account_cashflow_conventions]] for base-ccy conversion conventions.
