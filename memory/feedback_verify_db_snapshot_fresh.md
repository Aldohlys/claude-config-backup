---
name: Force fresh VM snapshot download to verify post-upload state
description: diff_db.ps1 reuses a cached local snapshot; after any sync/merge/upload, delete the cached file to force a fresh download before trusting the diff
type: feedback
originSessionId: 20890e27-e237-4b5b-ad56-028d4f1ab618
---
After any `sync_db.ps1 Push`, `merge_db.ps1 -Apply + upload`, or manual scp to the VM, do NOT trust `diff_db.ps1`'s output without first deleting the cached snapshot.

`diff_db.ps1` checks `Test-Path $TEMP_DB` and prints "Using existing VM snapshot" when the file exists, skipping the gcloud download entirely. The cached file can be hours stale — it reflects whatever the VM had the last time something downloaded it (often a pre-merge state), NOT the post-upload state you just created.

**Why:** During the 2026-04-24 PC/VM reconciliation session, multiple rounds of "looks like it worked but the diff shows it didn't" were caused by this. The snapshot had been modified locally by merge_db.ps1 INSERTs, then uploaded to VM, but the local cached file got reverted to a pre-merge state somewhere in the sync_db.ps1 Push cycle (which re-downloads at the start). Re-running diff_db.ps1 without deleting the snapshot produced confusing "pre-merge" readings that led to false panic.

**How to apply:**
- After any write path to VM, explicitly:
  ```powershell
  Remove-Item "C:\Users\aldoh\Documents\RApplication\data\mydb_remote_snapshot.db"
  .\diff_db.ps1
  ```
- If `diff_db.ps1` output contradicts what you know just happened (e.g. shows rows missing that you just inserted), snapshot staleness is the likely cause — delete and re-run before investigating further.
- Backup files `mydb_remote_snapshot_backup_*.db` are NOT the same as the main snapshot; `diff_db.ps1` only reads `mydb_remote_snapshot.db` (no suffix).
- Potential script improvement: add a `-Refresh` flag to `diff_db.ps1` that forces re-download. Not implemented yet; delete-then-rerun is the current workaround.
