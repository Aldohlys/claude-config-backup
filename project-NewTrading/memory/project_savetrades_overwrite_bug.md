---
name: saveTrades overwrite bug (2026-04-24)
description: RReporting's saveTrades historically overwrote the entire Trades table from in-memory snapshot — stale snapshots silently destroyed concurrent writes. Now guarded.
type: project
originSessionId: 8d82e5fa-6d83-469c-9702-1baa81598922
---
On 2026-04-24, two freshly-added trades (QQQ TradeNr 713, MARUZEN 5982) were silently destroyed by a `saveTrades` call from a stale RReporting in-memory snapshot. Root cause: `Tdata::saveTrades` used `safe_db_write(..., overwrite=TRUE)` on the whole Trades table, with no concurrency check.

**Why:** RReporting loads the full Trades table into a reactive (`all_trades()`) at Load time. Any other writer that touches Trades between Load and Save (a second RReporting session, fix_tradenr, manual edit) gets silently overwritten when the stale snapshot is saved back. There is no per-row UPSERT — it's a wholesale table replacement.

**How to apply:**
- The fix in `Tdata/R/trades.R::saveTrades` adds a freshness check: if DB has TradeNrs absent from the input, abort via `Tbasics::display_message` + `return(invisible(NULL))` and log the missing TradeNrs. `force = TRUE` parameter bypasses the check for legitimate deletions.
- This is a Tdata package change — must rebuild + redeploy via `build_tdata.R` for it to take effect in RReporting.
- The Delete button in RReporting (server.R:145 `ModalDelete_ok`) only mutates in-memory; it relied on overwrite-Save to persist. With the freshness check, deletes now hit the abort path. Long-term: add an explicit `deleteTrades(tradenrs)` API and rewire the Delete button.
- Recovery path when this happens: check `mydb_backup_*` and `mydb_remote_snapshot_backup_*` files in `data/` for a pre-incident snapshot; if too old, manually re-enter via RReporting (and use `fix_tradenr.bat <account> <date>` to backfill portfolio TradeNr NULLs).
- Re-inserting trades that don't come through Flex Query (e.g., TSE Japanese stocks like 5982 MARUZEN): use the convention from `scripts/insert_5982_maruzen.R` — `Total = -(Pos*Prix + Comm.)` for buys, `Risk = abs(Total)`, `TimeZoneSource = "America/New_York"`, Strategy="Daubasses" for net-net plays.
- Reconciling portfolio TradeNr after re-entry: match by `Instrument` against `Trades` for active rows (`scripts/reconcile_qqq_715.R` template), not by hard-coded TradeNr. Stocks need `portfolio.symbol = Trades.Ssjacent` instead (Instrument strings differ between portfolio and Trades for stocks).
- Diagnostic pattern: when investigating a "lost trade" incident, cross-check the user's reported time against `mydb.sql` mtime, `mydb_remote_snapshot*` mtimes, and the `getTradeDates ERROR: inexisting trade` log line — those nail the actual deletion window better than the user's recollection. Two simultaneous losses point at table-level overwrite, not single-row deletion.
- Backup hygiene: the git-tracked `mydb.sql` dump (BackupDatabase task, 21:00 daily) is only useful if it ran BEFORE the incident. Today's 16:26 manual dump was post-deletion and would have committed the corrupted state to git. Consider either (a) sanity-gating the daily commit on `COUNT(*) FROM Trades >= yesterday`, or (b) keeping rolling N-hour dumps during heavy edit periods.
- Single-writer assumption: the freshness check enforces it as a contract, but the human protocol matters too — only one RReporting session at a time, and close RReporting before running fix_tradenr / manual SQL / any other writer. The code guard catches violations rather than enabling safe concurrent edits.
- UX gap exposed by the guard: RReporting's Delete button silently breaks (relies on overwrite-Save to persist). Until `deleteTrades(tradenrs)` API is added, Delete + Save will hit the abort dialog. Consider disabling the button or tooltip-routing the user to DB Browser for now.
