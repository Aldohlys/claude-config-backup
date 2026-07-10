---
name: ""
metadata: 
  node_type: memory
  originSessionId: 8392b3e4-5d8b-457c-9eab-5c60ef858317
---

The "Live" account is assembled at read time by binding the three real account tables (getportfUI: `bind_rows` of readPortfolio("U1804173"/"U25343478"/"Gonet")). There is **no `Live` table** in the DB.

Any function that runs `SELECT ... FROM '<account>'` with the account name (e.g. `compute_weekly_unpnl`) throws **"no such table: Live"** for the virtual account, which the routine Positions render tryCatch turned into "Portfolio data incomplete or unavailable" (bit 2026-07-10). Guard with a `sqlite_master` existence check → return NULL / skip.

`getAccountLive()` aggregates the three sub-accounts by date with `multiple="any"` (non-deterministic pick of a matching row) and only recomputes **today** (`date >= s_date`), so it can't rebuild historical Live rows and can pick a stale sub-account snapshot. It also **appends** (not idempotent) — calling it twice creates duplicate Live rows.
