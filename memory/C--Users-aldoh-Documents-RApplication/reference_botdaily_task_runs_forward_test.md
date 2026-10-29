---
name: reference_botdaily_task_runs_forward_test
description: "The BotDaily scheduled task also runs the BOT forward test (bot_fwd) — a missing \"Run finished\" in bot_daily.log is usually bot_fwd marks, not bot_daily"
metadata:
  node_type: memory
  type: reference
  originSessionId: b1df7bc4-c85e-4ae6-818f-337366774300
  modified: 2026-10-22T08:23:59.917Z
---

`run_bot_daily.bat /task` (scheduled task **BotDaily**, 17:10) runs `bot_daily/main.R`, then `RStudies/reports/bot_fwd/main.py all`, and only then writes `===== Run finished` to `logs/bot_daily.log` and the `activity.log` row. bot_fwd has its own log: `logs/bot_fwd.log`.

- With TWS **down**, bot_fwd skips intake + marks and exits 3 in ~20 s.
- With TWS **up**, marks needs a BID + ASK 1-hour history request for every live option, every session: measured 2026-10-22 at ~4.5 s per contract (668 contracts ≈ 50 min). It grows with open forward-test positions.
- Until 2026-10-22 the task had a 30-min ExecutionTimeLimit, so every TWS-up run was killed (result 0x41306) mid-marks or just before the finish echo, and marks wrote nothing until all requests returned. Fixed: limit PT3H; marks.py qualifies / requests / commits in batches of 20, so a killed run keeps its progress and resumes from `last_mark`.
- Confirmed 2026-10-22 17:10: finished in 10.4 min, forward test exit 0, activity.log row written.
- Sleep trap (2026-10-24): a missed 17:10 slot starts as a night catch-up (StartWhenAvailable, 01:05); the PC then slept, the 3h wall-clock limit expired, Task Scheduler killed the .bat (0x41306) but the child Rscript survived and wrote the workbook 45 h later (25.10 22:25). Result: no forward test, no 'Run finished'. Not the marks length. The task is at \RApplication\BotDaily (Get-ScheduledTaskInfo needs -TaskPath \RApplication\).
- Task Scheduler's Operational event log is disabled on this machine; the bot_daily/bot_fwd logs are the only run record.

Diagnosis order for "BotDaily hung": look at `bot_fwd.log` start/finish markers before suspecting R or reticulate. Related: [[reference_launcher_bats_and_scheduled_tasks]].
