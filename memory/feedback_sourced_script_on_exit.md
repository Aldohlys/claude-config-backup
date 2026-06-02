---
name: Don't use on.exit() at top-level of sourced R scripts
description: on.exit() inside functions is fine, but at top-level of a sourced .R script it causes "Invalid or closed connection" errors; use explicit dbDisconnect at end
type: feedback
originSessionId: 1347083d-d9f2-4281-99ce-26cad155e9e3
---
When writing one-shot R scripts that the user sources (e.g. `source("~/scripts/migrate_X.R")`), do NOT use `on.exit(DBI::dbDisconnect(conn), add = TRUE)` at top level. Use explicit `DBI::dbDisconnect(conn)` at the end of the script.

**Why:** `on.exit()` is designed to register cleanup against an enclosing function frame. At the top level of a sourced script there is no function frame, and on some R versions / harness setups the registered expression fires prematurely — resulting in subsequent DB calls failing with `Erreur : Invalid or closed connection`.

**Observed twice in 2026-04-20 session:**
- `scripts/restore_macro_tables.R` v1: used on.exit → "Invalid or closed connection" on ATTACH. I misdiagnosed as ATTACH incompatibility and rewrote without both. v2 worked (removed on.exit among other things).
- `scripts/migrate_earnings_schema.R` v1: used on.exit on a trivial ALTER TABLE script → "Invalid or closed connection". v2 removed on.exit → worked.

**How to apply:**
- Inside an exported package function: `on.exit(DBI::dbDisconnect(conn), add = TRUE)` — correct, keep using it. See `Tdata/R/ticker.R` for many examples.
- In a top-level `scripts/*.R` one-shot that the user sources: explicit disconnect at end:
  ```r
  conn <- DBI::dbConnect(RSQLite::SQLite(), Sys.getenv("R_DB_PATH"))
  # ... work ...
  DBI::dbDisconnect(conn)
  ```
- For scripts that need cleanup on error, wrap in `tryCatch` + explicit disconnect in both `finally` and happy path, OR wrap the script body in a function and call it at the end.
