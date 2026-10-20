---
name: reference_rscript_home_git_identity
description: "Rscript sets HOME to Documents when HOME is unset (Task Scheduler, PowerShell), so git calls from R miss ~/.gitconfig (\"Author identity unknown\"); Git Bash hides the bug. BackupDatabase failed 2026-10-09..10-19 from this."
metadata:
  node_type: memory
  type: reference
  originSessionId: fee82fc0-0744-4df5-88d5-279a61fab8b5
  modified: 2026-10-20T13:07:15.595Z
---

R on Windows sets HOME to `C:\Users\aldoh\Documents` when HOME is unset. Task Scheduler and plain PowerShell leave it unset; Git Bash sets it. Any `system2("git", ...)` from Rscript under a scheduled task then misses `C:\Users\aldoh\.gitconfig` → `git commit` exits 128 "Author identity unknown". A manual test from Git Bash passes, so reproduce scheduler bugs with `Start-ScheduledTask`, not by hand.

Case: \RApplication\BackupDatabase exit 1 every night 2026-10-09 → 10-19 (first run past the weekly rule in [[project_rapplication_repo_policy]]), no dump commit after 10-02. Fixed 2026-10-19 in RApplication c5439b7: `backup_database.R` does `Sys.setenv(HOME = USERPROFILE)` before git calls; `backup_database.bat` now logs to `logs\backup_database.log` (run markers, like run_task.bat) and exits with the R code.

**How to apply:** any new R script run by Task Scheduler that calls git (or anything reading ~) needs the same HOME line. To test a commit+push script without pushing: `git config remote.origin.pushurl /nonexistent/no-push`, run, then `git config --unset remote.origin.pushurl`.

Unexplained, harmless: the dump has 723 `unistr()` lines only under the scheduler (same chocolatey sqlite3 3.47.2 by hand gives none); the script rewrites them, so the content is equivalent.
