---
name: reference_savetrades_overwrite
description: "Tdata::saveTrades() drops and recreates the Trades table on every save (dbWriteTable overwrite=TRUE). New columns survive via RSQLite auto-inference, but should be smoke-tested with scripts/smoke_test_savetrades.R before any user clicks Save in RReporting."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4ed7508e-d187-4257-a7e0-5e399d43e3c1
---

`Tdata::saveTrades(trades, force=FALSE)` in `Tdata/R/trades.R` is **destructive** — it calls `safe_db_write(conn, "Trades", ...)` which in turn calls `DBI::dbWriteTable(... overwrite=TRUE, append=FALSE)`. That **drops and recreates** the Trades table from the input data frame on every invocation.

**Practical implications:**
- New columns added via `ALTER TABLE Trades ADD COLUMN` survive the round-trip ONLY because:
  1. `Tdata::getAllTrades()` reads ALL columns via `dbReadTable(conn, "Trades")`.
  2. `validate_trades_data()` in `Tdata/R/db_validation_functions.r:132` passes unknown columns through (only standardizes specific named columns).
  3. `dbWriteTable(overwrite=TRUE)` auto-infers SQL types for columns not listed in `field.types` (`saveTrades` only specifies types for TradeNr / TradeDate / DateTime / TimeZoneSource / Pos / Prix / Comm. / Total / Risk / Reward / PnL).
- Auto-inference: R `character` → SQLite `TEXT`; R `numeric` → SQLite `REAL`; R `integer` → SQLite `INTEGER`. Verified working for `EventType` (TEXT) and `Return` (REAL) added 2026-05-18.

**Smoke test before letting users click Save with new schema:**
`scripts/smoke_test_savetrades.R` — backs up the DB, reads → saveTrades(force=TRUE) → reads, diffs schema + counts + values + fixture rows. 30 seconds, no data change in a no-op round-trip. Run after any ALTER TABLE on Trades.

**For long-term robustness:** add new columns explicitly to `saveTrades`'s `field.types` argument and to `validate_trades_data()`'s type-standardization loops. Requires a Tdata patch release; not strictly needed if you're confident in the auto-inference path.

Related: [[project_trades_signed_delta_risk]] is the migration that triggered the discovery of this behavior.
