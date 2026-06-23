---
name: project-alerts-condition-based-todo
description: "Build condition-evaluating alert path in check_alerts.R so triggers fire when ESTX50/SPY/etc. conditions are actually met, not just when AlertDate arrives"
metadata: 
  node_type: memory
  type: project
  originSessionId: 350b1975-d6ea-43f7-9086-8a8012f5cbc2
---

## Gap

The current alert system (`Tdata/R/alert.R` + `scripts/check_alerts.R` + Windows Task `CheckAlerts.xml`) is **purely date-based**. `getUpcomingAlerts(days_ahead)` returns active alerts whose `AlertDate` falls in [today, today+days_ahead]. There is no evaluation of trigger conditions written in the `Description` field — they are free-text instructions for the human, not executable.

This means alerts like "SELL 4700P if ESTX50 <= 5430 AND 4700P own-IV >= 35%" silently sit until their `AlertDate`, regardless of whether the condition is actually met. The user has had to work around this by either: (a) setting `AlertDate` to the option expiry and accepting the alert will surface late, or (b) setting it to a near-term reminder date and manually re-resetting on each review.

**Why:** The original system was scoped for date-bound calendar events (earnings, expiries, FOMC dates), not market-state-conditional triggers. Volatility/hedge-conversion triggers added later (alerts 49/50/51/52 from 2026-03-26, and 60/61/62 from 2026-05-15) don't fit the date-based model.

**How to apply:** When the user adds a market-conditional alert (anything with "TRIGGER" or "if X then Y" semantics), today it must be set up as a manual-reminder date with periodic re-set. A proper fix would let the alert system actually evaluate the condition against live data.

## Implementation sketch (Path B from 2026-05-15 conversation)

1. **Schema extension.** Add columns to `Alerts`:
   - `TriggerType TEXT` — one of `Date`, `MarketCondition`, `Both`
   - `Condition TEXT` — JSON payload, e.g. `{"underlying":"ESTX50","spot_max":5430,"iv_strike":4700,"iv_min":0.35}`
   - Or extract into a normalized `AlertConditions` table keyed by `alert_id`

2. **New function `getTriggeredAlerts()`** in `Tdata/R/alert.R`:
   - Pulls live underlying spot via existing Tdata IBKR helpers (`getOptValue` for option own-IV, `reqMktData` for spot)
   - Parses the JSON `Condition` field for each Active alert
   - Returns rows where condition currently evaluates TRUE
   - Handles missing data gracefully (TWS disconnect → return empty, log warning)

3. **Modify `scripts/check_alerts.R`** to call both:
   - `getUpcomingAlerts()` — date-based reminders (existing behavior)
   - `getTriggeredAlerts()` — condition-met triggers (new)
   - Email body has two clearly separated sections so user can tell them apart

4. **Run cadence.** Current daily 07:30 is fine for date-based reminders but inadequate for live triggers — a market move at 14:00 CET wouldn't notify until next morning. Either:
   - Add a second Task Scheduler entry running hourly during EUREX/NYSE hours
   - Or accept once-daily evaluation as good enough for swing-trade triggers (probably fine — wait-and-convert doesn't need minute-level latency)

5. **Theme cleanup.** `addAlert()` validates Theme against `c("Earnings","Macro","Expiration","Technical")` (alert.R:81) but the table contains Volatility/Pyramid/Stop/Roll/Covenant/Thesis from actual usage. Expand the whitelist when touching this code.

## Concrete first use-case (pilot) — added 2026-06-02

**SPY ≤ 705 level trigger.** During the 2026-06-02 ESTX50 hedge roll, the user wanted a market-driven "act now" alert on the SPY 660P hedge: *if SPY ≤ 705 (–7%), the US hedge is going live → roll 660P up+out to Dec before Sep expiry.* Because the DB can't fire on price, this had to be set as an **external charting alert** (user chose TradingView: SPY crossing down 705, open-ended) while the DB held only the dated calendar reminder (id=61, 14-Aug). This is the canonical case `getTriggeredAlerts()` should absorb: a single DB row `{"underlying":"SPY","spot_max":705}` that auto-fires, instead of splitting the trigger across TWS + a date-based DB stub. The monetize leg (`SPY ≤ 660 AND VIX ≥ 28`) is a second condition on the same hedge — exactly the two-condition shape the JSON `Condition` field is meant to hold. **Use SPY ≤ 705 as the test row when building the path.**

## Existing alerts that would benefit (state as of 2026-06-02)

| id | Asset | Status | What it tries to express |
|---|---|---|---|
| 52 | SPY-SPREAD | **deactivated** | (stale — old SPY 580P, position closed) |
| 60 | OESX-ROLL | **executed 2026-06-02** | roll done; no longer conditional |
| 61 | SPY-ROLL | **active** | SPY ≤ 705 → roll 660P→Dec; monetize if SPY ≤ 660 AND VIX ≥ 28. **Currently date-based (14-Aug) + TWS tick for the level — the pilot above.** |
| 62 | OESX-MONITOR | active | ESTX50 ~5417 zone / VSTOXX ≥ 25-30 → monetise Dec 5700/4600 spread |
| 56-57 | MCL | active | Pyramid/stop levels — could auto-evaluate against live CL price |

## Effort / priority

~1-2 days of work in Tdata + scripts. Touches DB schema (migration needed) and adds a TWS dependency to the daily check (operational fragility — if TWS is down at 07:30, conditions are unevaluated). **Data point from the 2026-06-02 hedge cycle:** the workaround now forces splitting a single logical trigger across two systems (TWS tick alert for the price level + a date-based DB stub for the backstop) — manageable for one hedge, but it's the recurring friction the user keeps hitting. The SPY ≤ 705 pilot above is the clean test case to build against.

See [[reference-u1804173-iv-column]] for the related "DB IV column is wrong layer" finding from the same 2026-05-15 session.
