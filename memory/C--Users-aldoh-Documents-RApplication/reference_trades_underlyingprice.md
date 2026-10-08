---
name: reference_trades_underlyingprice
description: "Trades.UnderlyingPrice (REAL, 2026-10-08) = underlying at Close/Adjust fills of option-only trades; who fills it, gaps, backfill script"
metadata:
  node_type: memory
  type: reference
  originSessionId: 3f7efce0-fd27-406b-b771-59dc993e4d3e
  modified: 2026-10-08T09:14:48.629Z
---

`Trades.UnderlyingPrice` (REAL, added 2026-10-08, TODO #106; also on
`TestTrades`) holds the underlying's price at a **Close or Adjust fill**. It
feeds the exit interval of `Tdata::attributePnL` (Greek split of a trade's
last move); without it that interval is all "execution", flagged `noS`.

- **Who writes it:** RReporting `fill_underlying_price()` in `close_trade` /
  `adjust_trade`, via `Tdata::getUnderlyingPriceAt(symbol, datetime_utc,
  day_close)` → Python `tdata_py.get_price_at` (IBKR 1-minute RTH bars,
  `formatDate=2`). Open rows and stock/future trades get NA by design: a trade
  with a stock or future leg uses that leg's own fill price.
- **TWS down or no bar:** NA plus a persistent warning; the user types it in
  later through the Trades-tab **Modify** modal ("Underlying price at fill").
  A value typed on one leg serves every leg with the same `DateTime`.
- **Backfill:** `scripts/backfill_underlying_price.R [ACCOUNT] [YYYYMMDD]
  [--dry-run]` — only rows still NULL, so re-runnable. Run it from a directory
  holding `config.yml` (e.g. RReporting): elsewhere tdata_py finds no Tickers
  DB and resolves every symbol as STK/SMART, so index underlyings (SPX = IND)
  return no bar. ~10 fills/minute.
- **Schema:** declared REAL in `saveTrades` `field.types` (an all-NULL column
  would otherwise be recreated with an inferred type); listed in RReporting
  `trade_fields`, all four row builders, `modify_trade` and `sub_all_trades`.
  A direct SQL fill makes open RReporting sessions refuse Save until reloaded
  (5.20.14 freshness guard).

Related: [[reference_trades_datetime_utc]] (which time the fetch uses),
[[reference_rreporting_trade_fields_schema_sync]], [[reference_savetrades_overwrite]].
