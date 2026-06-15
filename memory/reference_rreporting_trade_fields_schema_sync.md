---
name: reference_rreporting_trade_fields_schema_sync
description: "When adding a Trades column, RReporting trade_fields + all 4 row-building paths must be updated or New-Trade rbind breaks and a Save silently drops the column"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 9396fe68-7474-48eb-9f7a-c38651ff5dc2
---

RReporting's `trade_fields` (hardcoded column vector in `RReporting/app/server.R`) MUST list every column in the Trades table. Two reasons it's load-bearing:
- `getAllTrades()` uses `dbReadTable` → `all_trades()` carries **all** DB columns. `insert_new_trade`/`adjust_trade`/`close_trade` do `select(all_of(trade_fields))` then `rbind` into `all_trades()` — a column in `all_trades` but not in the new rows (or vice-versa) makes the rbind error ("names do not match").
- Save does `saveTrades(all_trades()[,-1])`, which **drops + recreates** the Trades table from that frame ([[reference_savetrades_overwrite]]). A column absent from the in-memory frame is silently dropped from the DB.

So when a new Trades column is added, update ALL of:
1. `trade_fields` in `server.R`.
2. Every path that builds rows merged into `all_trades()`: `insert_new_trade`, `adjust_trade`, `close_trade` (its backward-compat "add missing columns" block), and `create_manual_cash_trade` (which is then `[, names(current_trades)]`-subset before rbind).

Observed 2026-06-14: `trade_fields` had drifted — missing `Underlying` (added by #35) and `Stop` (added by #67) — so the New-Trade flow's rbind was latently broken and a Save would have dropped both columns. Fixed when wiring [[project_stop_based_risk_sync]] (RReporting 7249fea): added both to `trade_fields` and to all four row builders.

Gotcha: the staged new-trade data (from `readTrades`) names the instrument column `Description` (renamed to `Instrument` only inside the writers) and carries `Strike`/`Right` — so `is_options_block(rows)` crashes on it (its Instrument-string fallback hits a NULL column). For modal/observer stock-vs-option gating use `isTRUE(!is.na(Strike[1]) && Right[1] %in% c("P","C","Put","Call"))` directly instead.
