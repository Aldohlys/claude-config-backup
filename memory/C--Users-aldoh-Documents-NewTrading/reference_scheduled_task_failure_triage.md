---
name: reference_scheduled_task_failure_triage
description: "'Database locked' after PC wake = missed RApplication tasks all start at once (StartWhenAvailable); triage from logs/activity.log FAIL lines, per-task logs, t-YYYYMMDD.log"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9e70c95e-b289-4caa-ad20-76f71acd2008
  modified: 2026-10-20T15:02:41.520Z
---

On 2026-10-19 the PC woke at 20:51 and Task Scheduler started ~12 missed `\RApplication\` tasks within the same second (most have StartWhenAvailable=True). They contended for `mydb.db`; Tdata logged "Database locked on connection attempt 2/2" and DailyPortfolioUpdate's `getIBKR('U1804173')` failed (snapshot missing until the 22:00 run). Other tasks retried and succeeded.

Where to look, in order:
- `RApplication\logs\activity.log` — one line per run, `date;task;start;end;min;OK|FAIL;;reason`. `grep ';FAIL;'` gives the list.
- Per-task logs in `RApplication\logs\` (`<TaskName>.log` from run_task.bat, `daily_portfolio_update.log`, `collect_option_surface.log`, `backup_database.log` since 10-19). macro_context (RunScanner) logs to `flow_scanner.log`, not a macro_context log.
- `logs\t-YYYYMMDD.log` — Tdata log; grep `locked` for lock warnings.
- `Get-ScheduledTaskInfo` LastTaskResult: 0x41301 = still running, 0xC000013A = console closed (bot_daily still logged OK), 0x800710E0 = refused/not started.

BotDaily catch-up (2026-10-15, after a crash reboot): result 0x41306 = killed by the task's 30-min ExecutionTimeLimit. Only the .bat wrapper dies; the Rscript child keeps running orphaned and still writes the xlsx (that run took ~58 min vs ~3 normal), but the wrapper's bot_fwd step and log_activity never run — start `python RStudies\reports\bot_fwd\main.py all` by hand. bot_daily itself does NOT need TWS (Yahoo only; TWS only fills missing last bars); bot_fwd does (exit 3 = TWS unreachable, intake/marks skipped, caught up next run).

Recurring non-lock FAILs seen 10-13..16: collect_option_surface / UpdateHistoricalOptions "TWS API not available" when TWS is closed in the evening.

**How to apply:** when the user reports a lock message or asks what failed, read activity.log first, then the failing task's own log; say which failures are lock-caused (rerun) vs. code/data (fix). Staggering wake catch-up (random delay on triggers) would prevent the pile-up — not implemented. Related: [[reference_rscript_home_git_identity]], [[reference_data_integrity_check]].
