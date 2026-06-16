---
name: reference_daily_update_subprocess_db_contention
description: daily_portfolio_update spawns subprocesses right after parent DB writes → transient SQLite lock; add PRAGMA busy_timeout to child reads + honor exit-code contracts
metadata: 
  node_type: memory
  type: reference
  originSessionId: 897e1968-7f9d-444e-8d31-015253997431
---

`scripts/daily_portfolio_update.R` spawns child Rscript subprocesses via `system2`
immediately after the parent does TWS-backed DB writes (Step 0 `getIBKRActiveCurrencyValues()`,
then Step 0b `check_fx_freshness.R`; Step 3 `sync_stop_risk.R` similarly). The parent's
sub-second write can still hold the SQLite DB when the child opens its own connection, so the
child's query errors out (SQLITE_BUSY) before doing any real work — surfacing as a non-standard
exit code (saw **5**, not the script's documented 0/1/2).

Two failure modes this produced, and the fixes (commit 15049ff, 2026-06-16):

1. **Transient lock** → add `DBI::dbExecute(conn, "PRAGMA busy_timeout = 5000")` right after
   `dbConnect` on the child's **read-only** connection. It retries up to 5s instead of erroring
   immediately; safe because the connection only SELECTs. Wrap in `invisible()` — `dbExecute`
   returns rows-affected (0 for a PRAGMA) which Rscript auto-prints into stdout otherwise.

2. **Mislabeled error** → the caller mapped ANY non-zero child exit to one domain message
   ("FX rates STALE"), so a crash read as "stale rates" when rates were actually fresh. Honor the
   child's documented exit-code contract: exit 1 = genuinely stale; other non-zero = "check did
   not complete (exit N) — staleness unverified". Don't collapse error vs domain-signal into one.

**Why:** a once-daily background script that runs read-only checks should tolerate a momentary
lock, and a subprocess error must not masquerade as a domain verdict.
**How to apply:** if another `daily_portfolio_update` step (or any spawn-after-write script) shows
a spurious failure, suspect parent/child DB contention first; add busy_timeout to the child read
connection and check the caller isn't conflating crash-exit with a meaningful non-zero exit.
Related: [[project_stop_based_risk_sync]], [[feedback_sourced_script_on_exit]].
