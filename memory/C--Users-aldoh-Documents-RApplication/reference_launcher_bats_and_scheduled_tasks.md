---
name: reference_launcher_bats_and_scheduled_tasks
description: "Launcher .bat files are named 00_*.bat since 2026-09-15; registered scheduled tasks call .bat paths directly, so renaming one breaks the task silently — and the repo task XML is NOT the registered schedule"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7b2126a4-19e3-44e8-8acf-3feca1d38bea
  modified: 2026-09-15T09:38:25.513Z
---

**Launchers (RApplication/scripts, renamed 2026-09-15, content unchanged):** `00_routine.bat`, `00_reporting.bat`, `00_ligne.bat`, `00_spread.bat`, `00_ticker.bat`, `00_daily_portfolio_update.bat` (formerly `Tuser-routine.bat`, `RReporting.bat`, `Tuser-ligne.bat`, `Tuser-spread.bat`, `Tuser-ticker.bat`, `daily_portfolio_update.bat`).

**Registered tasks that run .bat files** (`Get-ScheduledTask`): `\RApplication\DailyPortfolioUpdate` → `scripts\00_daily_portfolio_update.bat` (repointed 2026-09-15); `BackupDatabase` → `scripts\backup_database.bat`; `CollectOptionSurface` → `scripts\collect_option_surface.bat`; `RunScanner` → `Desktop\run_scanner.bat`.

**Why it matters:** the rename left DailyPortfolioUpdate pointing at a deleted file; its next run (17:00 the same day) would have failed with no visible error in the apps.

**How to apply:**
- Before renaming or moving any `.bat`/script, query `Get-ScheduledTask` for tasks whose action names it, and grep `scripts/*.xml`.
- To fix a task's path, change ONLY its action: `Set-ScheduledTask -TaskName X -TaskPath '\RApplication\' -Action (New-ScheduledTaskAction -Execute <new> -WorkingDirectory <same>)`. Do **not** re-register from the repo XML: `scripts/DailyPortfolioUpdate.xml` has a single 08:00 trigger, while the registered task runs daily at 10:00, 17:00 and 22:00 — importing it would silently drop two runs.
- Tasks live under a task folder: `Get-ScheduledTaskInfo` needs `-TaskPath '\RApplication\'` or it reports "file not found".

Related: [[reference_schtasks_xml_registration]], [[reference_daily_update_subprocess_db_contention]].

## UPDATE 2026-09-15 — long "hung" runs were the PC sleeping; the surface collector now logs

- **Check sleep before calling a scheduled run hung.** In `logs/daily_portfolio_update.log` (81 runs, 2026-07-28 → 09-15) the runs of 222 and 248 minutes, and evening runs that finished hours later, sat exactly on Windows sleep/wake events: `Get-WinEvent -FilterHashtable @{LogName='System'; StartTime=...; EndTime=...}` filtered to providers Microsoft-Windows-Kernel-Power / Power-Troubleshooter, ids 506/507 (also 1, 42, 107). Tasks run on wake (StartWhenAvailable, no WakeToRun). The 30-minute ExecutionTimeLimit did not kill those runs — probably time asleep does not count (not verified). No genuine hang was found (TODO #53 lowered to LOW; #54 closed).
- **CollectOptionSurface** (daily 18:00, `scripts/collect_option_surface.bat`) exits 1 when TWS is not reachable. Since 2026-09-15 the `.bat` appends each run to `logs/collect_option_surface.log` (start/finish markers with the exit code, Rscript path, symbol list, R output). The OptionSurface table had only 24 capture days from 06-10 to 09-04, none after (TODO #50).
- The daily update log also shows quick Step 3 failures on evening runs (`No value returned from IB!`, TWS likely closed at 22:00) and Gonet/Live errors on 08-27 and 09-04 (TODO #87).

## UPDATE 2026-09-21 — every task writes one line to logs/activity.log

`logs/activity.log` is the cross-task record: `date;task;start;end;duration_min;status;detail`,
appended by **`scripts/log_activity.ps1`** (via a `.bat` front end). Read it with
`Rscript scripts/activity_report.R [--days N] [--task NAME]` — runs, failures,
median vs latest duration with a drift ratio, and repeat offenders.

- **All 26 launchers log themselves**, so it works however they are started:
  scheduler, the Tkinter launcher, a shortcut, or by hand.
- **`scripts/run_task.bat <name> <cmd> [args]`** wraps the five scheduled tasks
  that call Rscript/python directly (`CheckAlerts`, `RefreshCOT`,
  `RefreshInterestRates`, `UpdateHistoricalOptions`, `BOT_Momentum_Monitor`) —
  all repointed through it. They could **not** simply be aimed at the existing
  `.bat` launchers: several end in `pause`, which hangs an unattended run.
- `RPreTrade.bat` → `00_pretrade.bat` (renamed outside this work).

Tasks now registered under `\RApplication\`: BackupDatabase, BotDaily (17:10
Mon-Fri), BOT_Momentum_Monitor, CheckAlerts, CollectOptionSurface,
DailyPortfolioUpdate, RefreshCOT, RefreshInterestRates, RunScanner (was pointing
at a deleted `Desktop\run_scanner.bat`; now `NewTrading\scripts\run_flow_scanner.bat`),
UpdateHistoricalOptions.

See [[feedback_batch_arg_and_exitcode_traps]] for the four cmd traps this wiring
had to survive — each found by testing, none by reading.
