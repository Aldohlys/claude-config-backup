---
name: reference_activity_log_in_calc
description: Open logs/activity.log in LibreOffice Calc with the PowerShell 7 profile function calc-log (';' separator); .log opens in Writer otherwise; never rename the file
metadata:
  type: reference
---

`logs/activity.log` is `;`-separated (`date;task;start;end;duration_min;status;args;detail`). LibreOffice picks the app from the detected file type, so a `.log` opens in **Writer** even when launched from Calc.

**How to open it:** `calc-log activity.log` — a function in the **PowerShell 7** profile (`Documents\PowerShell\Microsoft.PowerShell_profile.ps1`, added 2026-10-26):
`& "C:\Program Files\LibreOffice\program\scalc.exe" --infilter="Text - txt - csv (StarCalc):59,34,76" (Resolve-Path $Path).Path`
(59 = `;`, 34 = `"` text delimiter, 76 = UTF-8). Calling `scalc.exe` from PowerShell needs the `&` call operator, otherwise "Unexpected token".

**Traps:**
- Do not rename to `.csv`: the launchers and `log_activity.ps1` write `activity.log`. A hard link `activity.csv` breaks if a writer ever recreates the file.
- Windows PowerShell 5.1 reads `Documents\WindowsPowerShell\` (conda-only profile) — `calc-log` exists only in `pwsh`.
- If `calc-log` "does nothing", look for windowless `scalc`/`soffice` processes left by earlier attempts; each new launch hangs behind them. `Get-Process soffice*,scalc* | Stop-Process -Force`, then retry.
- Close Calc before scheduled tasks write — see [[reference_launcher_bats_and_scheduled_tasks]] (never save the log from Calc).
