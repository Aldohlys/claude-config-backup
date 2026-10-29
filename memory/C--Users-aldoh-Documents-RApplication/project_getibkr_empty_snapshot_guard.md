---
name: project_getibkr_empty_snapshot_guard
description: "reqAccountUpdates timeout can return NLV/cash but no positions → Account row with stock 0; getIBKR retries once then refuses (Tdata 5.22.1, 2026-10-26)"
metadata:
  node_type: memory
  type: project
  originSessionId: 71e55639-752f-4ac0-abc5-c54fed516806
  modified: 2026-10-26T10:07:07.815Z
---

When `reqAccountUpdatesAsync` hits its 60 s timeout, `ib.portfolio(account)` can come back **empty** while NetLiquidation, cash and AccruedDividend have arrived. `getIBKRData` (account.py) then leaves StockMarketValue/UnrealizedPnL at their initial 0, and `getIBKR` used to append that Account row (and skip the position snapshot). Seen on U25343478: 22.10 10:01:21 and 10:19:00 (timeout warnings 10:02:21, 10:20:00 in `logs/t-20261022.log`) and 26.10 10:03:35. The rows read as a ~22.7k CHF drop; check_data_integrity account_identity flags them. Not a sale: cash didn't move, positions were back at the next run. All three rows were deleted (backups `logs/mydb_pre_delete_empty_snapshot_20261026_093551.db`, `data/mydb.db.bak_accrued_20261026_101257`).

**Fix (Tdata 5.22.1, commit 56e9794, user's rule):** with an empty portfolio and NLV − stock − options − cash − AccruedDividend > max(100, 1% NLV) (account_identity's tolerance), `getIBKR` logs WARN and calls `getIBKRData` once more; still incomplete → log ERROR, store nothing, `stop()`. An empty portfolio whose NLV ≈ cash + accrual ("everything sold, dividend pending") is stored normally. `stop()` rather than `return(0)` because `daily_portfolio_update.R` counts a step failed only on error. `scripts/RGetIBKR.R` now wraps each account in tryCatch and exits 1 (RApplication 0539692).

**Why:** a zero-stock snapshot is silent data corruption; skipping costs only one snapshot, the next scheduled run takes a full one.

**How to apply:** a stock-0 Account row with normal cash/NLV = this timeout, not a trade; look for "reqAccountUpdates end-event timeout" in the t-YYYYMMDD.log at that time. A "getIBKR ... no positions twice" ERROR in daily_portfolio_update is this guard working.

Related: [[project_account_accrued_dividend]], [[feedback_tws_subscription_outlives_end_event]], [[project_ib_request_timeout]].
