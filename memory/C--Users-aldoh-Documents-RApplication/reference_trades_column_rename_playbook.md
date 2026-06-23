---
name: reference_trades_column_rename_playbook
description: How to rename a Trades/TestTrades column safely (flag-day) — learned doing
metadata: 
  node_type: memory
  type: reference
  originSessionId: 47893e77-8f73-4364-903a-3678497da0fa
---

Renaming a `Trades`/`TestTrades` column is an all-or-nothing flag-day: deployed Tdata's SQL must match the DB columns, or every app errors. Proven recipe (#35 Phase 1, 2026-06-14):

1. **Check target-name collisions FIRST.** `grep -rcE "\bTarget\b"` across Tdata/RReporting/Tuser. `Symbol` (189×) and `Price` (175×) were already heavy market-data identifiers — but turned out to be *display aliases* of `Ssjacent`/`Prix` (`summarize(Symbol=first(Ssjacent))`, `Price=fmt(Prix)`) in blocks that never touched the Trades frame, so the rename unified them. Real within-frame collisions are rare; verify per file.
2. **sed scope MUST include test dirs.** `grep -rlE "\bOld\b" Tdata RReporting Tuser` (whole trees, not `Tdata/R`). Missing `Tdata/tests`/`RReporting/tests` in Phase 1a left hardcoded `WHERE Statut` SQL in test-trades.R → 30 errors.
3. **No `saveTrades` field.types edit for TEXT renames** — `field.types` only lists numeric/int/date cols; character columns (and a new `Underlying`) infer to TEXT automatically; `getAllTrades` uses `dbReadTable` (all cols). So a TEXT-column add/rename needs no Tdata code change beyond the references themselves.
4. **summarize() self-reference trap:** if you rename a Trades column to a name also used as a flex SOURCE column in a `summarize()` (e.g. `Comm.`→`Commission` where the flex input is `Commission`), a LATER expression (`Total = ... + sum(Commission)`) reads the just-created output (sign flip), not the input. Reorder so the dependent expr (`Total`) comes BEFORE the renamed-output line.
5. **DB rename:** `ALTER TABLE Trades RENAME COLUMN "Old" TO "New"` (+ TestTrades); quote dotted names like `"Comm."`. Back up first: `cp data/mydb.db data/mydb_pre_*.db`.
6. **Then rebuild/deploy Tdata** (`/build Tdata auto`) — until then deployed Tdata ↔ DB are inconsistent and apps break.
7. **Verify beyond unit tests:** `symf.R` stats functions (`stats_one_position`/`stats_one`/`stats_all`) have no unit tests — smoke-test them on real open trades via `devtools::load_all(Tdata)` (NOT `library(Tdata)`, which loads the deployed/old version) + `box::use(symbol/logic/symf)`. The portfolio↔trade symbol-branch join in `stats_one_position` must be gated `filter(type=="Stock")` (report carries `type`).

See [[project_trades_symbol_underlying_model]] and docs/TRADES_REFACTORING_PLAN.md. Pairs with [[feedback_build_package_git_add_all]] (concurrent work + /build) and [[feedback_dplyr_filter_masked_standalone]].
