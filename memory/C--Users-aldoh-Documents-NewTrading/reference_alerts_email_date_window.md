---
name: reference-alerts-email-date-window
description: Alerts are e-mailed only if AlertDate is today/tomorrow at the 07:30 CheckAlerts run — script-written alerts must be dated the next weekday or they are never sent
metadata:
  node_type: memory
  type: reference
  originSessionId: 554d4b0c-30f4-4885-9fe7-bf79e0e6a1ac
  modified: 2026-10-26T16:28:50.110Z
---

The only e-mail path for DB alerts: task `\RApplication\CheckAlerts` (daily 07:30) → `RApplication\scripts\check_alerts.R` → `Tdata::getUpcomingAlerts(days_ahead = 1)` (`Active = 1 AND AlertDate BETWEEN today AND today+1`) → blastula SMTP (config `email`, env `ALERT_EMAIL_PASSWORD`). No other alert/e-mail function exists in the repos (verified 2026-10-07, Tdata 5.20.12).

Consequence: an alert written after 07:30 and dated the same day is never e-mailed — that morning's run has passed and the next one excludes it. `bot_momentum_monitor.py` (17:00) had this gap since creation; fixed 2026-10-07 by dating alerts the next weekday. Fri-Sun alerts dated Monday arrive twice (Sunday window Sun-Mon + Monday).

`Tdata::addAlert()` whitelists themes Earnings/Macro/Expiration/Technical only, so scripts using other themes (Momentum, Volatility, ...) write to `Alerts` with SQL directly.

This window applies to Date/Both alerts only. Since Tdata 5.23.0 (2026-10-26), Condition alerts are e-mailed by the same 07:30 run whenever their price condition is met, whatever their AlertDate — see [[project-alerts-condition-based-todo]].

**How to apply:** any new script that writes to `Alerts` should set `AlertDate` ≥ next 07:30 run date, or send its own e-mail. Doc: `Strategies/Breakouts/bot_momentum_monitor.md`. Related: [[project-alerts-condition-based-todo]] (date-only alert system, theme whitelist cleanup).
