---
name: reference-scheduled-task-bat-paths
description: Task Scheduler actions hard-code NewTrading/scripts/*.bat names; renaming a .bat silently breaks BotDaily/BotMonthly/RunScanner. Set-ScheduledTask needs -TaskPath; auto mode blocks task edits.
metadata:
  node_type: memory
  type: reference
  originSessionId: b1932204-088c-4118-9bdf-6a4787b3b75f
  modified: 2026-10-27T12:00:06.493Z
---

Since 2026-10-27 the launchers in `NewTrading/scripts/` are numbered by run order: `0_run_bot_monthly.bat`, `1_run_macro_context.bat`, `2_run_bot_daily.bat`, `3_run_analyze.bat`. The renaming was done outside a Claude session, and the three tasks kept the old names until fixed the same day:

| Task | Execute | Arguments | WorkingDirectory |
|---|---|---|---|
| BotDaily (17:10) | `scripts\2_run_bot_daily.bat` | `/task` | `RApplication` |
| BotMonthly (no trigger) | `scripts\0_run_bot_monthly.bat` | `/task` | `RApplication` |
| RunScanner (09:00) | `scripts\1_run_macro_context.bat` | `/task` | none |

No command or skill calls the `.bat` files; only Task Scheduler does.

**Why:** a renamed or moved `.bat` gives no error until the next trigger fires, so the daily run is lost.

**How to apply:**
- After any rename or move in `scripts/`, list task actions: `Get-ScheduledTask | % { $_.Actions.Execute }` and grep the old names.
- `Set-ScheduledTask -TaskName X -Action ...` fails with "fichier introuvable" (0x80070002) unless `-TaskPath $t.TaskPath` is passed. Rebuild each action with `New-ScheduledTaskAction`, keeping `-Argument` and `-WorkingDirectory`.
- The auto-mode classifier blocks task edits ("Security Weaken"): write the script and have the user run it, then read back the actions and `Test-Path` each Execute.

Related: [[reference-scheduled-task-failure-triage]]
