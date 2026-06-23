---
name: Daily portfolio update — hardening opportunities
description: Open follow-ups from the 2026-04-24 daily portfolio update incident. Account-list filter has been deduped client-side; broader hardening still pending.
type: project
originSessionId: 8d82e5fa-6d83-469c-9702-1baa81598922
---
The `\RApplication\DailyPortfolioUpdate` scheduled task (10:00/17:00/22:00 daily) had two latent issues surfaced 2026-04-24:

1. **Demo account in live loop**: `daily_portfolio_update.R` iterated `getAccountChoices("ibkr")` which returns DU5221795 (paper) alongside the live U-accounts. Live TWS rejected DU5221795 with a misleading PY ERROR. Already patched in `daily_portfolio_update.R` by mirroring the `managedAccounts()` intersection pattern from `RGetIBKR.R`.

   **Why:** the `^[UD]` regex in `Tdata::getAccountChoices` lumps live and paper together; the daily script needs only what the current TWS session manages.

   **How to apply:** lift the filter into a Tdata helper (e.g. `getAccountChoicesForSession("ibkr")`) so future scripts don't repeat the bug. Two callers already need it (`daily_portfolio_update.R`, `RGetIBKR.R`).

2. **Silent timeout on account fetch**: at 17:01:46 `reqAccountUpdates timeout for U25343478 — TWS unresponsive; skipping` was logged at PY-WARNING level only — no R-level error raised. The R loop continued without that account, and the scheduled task reported success (`LastTaskResult: 0`). Account data for U25343478 was simply absent for the 17:00 run; nothing alerted the user until they noticed.

   **Why:** the Python helper swallows the timeout to avoid aborting the whole run. But for a scheduled batch, "skipped silently" is worse than "failed loudly" — the next scheduled run at 22:00 won't retry the missing account because the task is considered successful.

   **How to apply:** promote the timeout to an R-level error (or at minimum a `logger::log_error` that the script checks via `quit(status = 1)`). The bat file already returns the exit code to Task Scheduler, so a non-zero exit will make the task report failure and Windows will retry per the trigger config.
