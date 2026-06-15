---
name: feedback_readcsv_typeconvert_integer
description: "read.csv/type.convert coerces all-numeric columns to <integer>, breaking bind_rows against character columns"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a2854ddf-d4f4-4f36-b9b1-d5d843760743
---

`read.csv(check.names=TRUE)` runs `type.convert`, which parses a column whose values are **all numeric** as `<integer>` — even when it's logically a string (e.g. IBKR `UnderlyingSymbol` for numeric-ticker markets like Japan: 1976, 5658). Downstream `dplyr::bind_rows()` then fails to combine that integer column with a character column of the same name:
`Can't combine ..1$Ssjacent <integer> and ..2$Ssjacent <character>`.

**Why:** RReporting `read_trade()` splits a flex CSV into regular + CASH rows and binds them; CASH sets `Ssjacent="CASH"` (character) while numeric tickers came in as integer.

**How to apply:** Force ID/symbol/description columns to character right after `read.csv`, alongside the existing all-empty→logical guard (`trade_operations.R::read_trade()` coerces `Strike`/`Put.Call`/`Expiry`/`UnderlyingSymbol`/`Description`). Same trap also turns all-empty columns into `<logical>` NA. Related: [[feedback_apply_df_row_coercion]], [[r-partial-matching]].
