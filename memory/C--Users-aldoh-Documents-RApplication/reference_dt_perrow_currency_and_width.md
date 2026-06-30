---
name: reference_dt_perrow_currency_and_width
description: DT idioms — per-row multi-currency cell formatting + shrinking a DT to content width
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3f4e5a5e-17d7-445c-8433-e74f630f76ef
---

Two reusable idioms in `Tuser/symbol/logic/datatablef.R` `basic_portf_datatable()` (shipped 2026-06-30, Tuser commit 1a5d3d0):

**Per-row trade currency.** `DT::formatCurrency()` applies ONE currency symbol to a whole column, so it can't render a column whose rows are in different currencies. Instead pre-format each cell to a string with `Tdata::currency_format(value_vec, currency_vec)` — `currency` IS vectorized. But its `accuracy` arg is **scalar, not per-row** (the `label_currency` closure captures a single accuracy), so to mix decimals (e.g. sub-1 prices → 4dp `0.0001`, else 2dp) split rows into accuracy groups and call `currency_format` once per group. The data must carry a per-row `currency` column; the function consumes it then drops it (`dt$currency <- NULL`) so it isn't displayed. All four callers (Trade tab `tr`, Symbol tab `sym`, `an_portfUI`, `an_symUI`) now select a `currency` column; when absent it falls back to the old scalar `formatCurrency`.

**Shrink a DT to content width.** A DT `<table>` is `width:100%`, so with `autoWidth=TRUE` columns stretch to fill the panel and read poorly. To make it compact: give each column a content-tuned `width` (px) via `columnDefs` AND wrap the `DTOutput` in `div(style="display: inline-block;")` — the inline-block container collapses to the table's content instead of the full panel width. The Trade-tab stats tables already used this inline-block wrap; the `tr` table got it too. See also [[reference_dt_basecurrency_sort_pin_row]].
