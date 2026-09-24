---
name: feedback_batch_arg_and_exitcode_traps
description: "Four Windows .bat traps that silently corrupt task logging: %TIME%'s locale comma splits cmd arguments, pause resets %ERRORLEVEL%, shift moves %0 so %~n0 breaks, and calling a .bat without `call` never returns"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f5ad26f1-60c7-4344-bb10-9710e5cdd89f
  modified: 2026-09-21T17:09:58.124Z
---

Wiring `logs/activity.log` into 26 launchers hit four traps. **Every one passed
code review and failed the first real test** — none is visible by reading.

1. **`%TIME%` carries a locale comma.** On this machine it is `18:37:54,21`, and
   cmd treats a comma as an argument separator, so `"%TIME%"` arrives split
   across two parameters and everything after it shifts — the exit code lands in
   the reason field and the reason is lost. Capture it stripped:
   `for /f "tokens=1 delims=,." %%a in ("%TIME%") do set "T0=%%a"`.
2. **`pause` resets `%ERRORLEVEL%` to 0.** The 14 launchers ending
   `if %ERRORLEVEL% NEQ 0 pause` would have logged every failure as OK. Capture
   the code into `RC` *before* anything else runs, and test the copy.
   `call`-ing another `.bat` resets it too, so the log call must come after the
   capture, never before.
3. **`shift` moves `%0` as well as the arguments.** After an argument-collection
   loop, `%~n0` returns the first remaining argument, not the script name —
   `run_bot_daily.bat` logged itself as `AAPL`. Capture `set "TASKNAME=%~n0"` at
   the top, next to the start time.
4. **Invoking another `.bat` without `call` transfers control and never
   returns**, so everything after it — including the log line — silently does
   not run. Harmless for an `.exe` target, fatal for a `.bat` one, which is why
   a wrapper tested only against `cmd /c exit N` looks fine.

**Testing note:** Git Bash mangles a leading-slash argument into a path
(`/task` → `C:/Program Files/Git/task`), so a `/task` mode looks broken when
driven from bash while working fine under Task Scheduler. Use `//task`.

**How to apply:** when a logging or wrapper change touches `.bat` files, run it
once for a success *and* once for a non-zero exit and read the record. The
failure mode of all four is a plausible-looking line with the wrong content, not
an error.

Related: [[feedback_shell_quoting_traps]],
[[reference_launcher_bats_and_scheduled_tasks]].
