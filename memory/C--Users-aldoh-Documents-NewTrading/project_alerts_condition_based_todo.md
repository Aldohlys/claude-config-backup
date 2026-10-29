---
name: project-alerts-condition-based-todo
description: "Condition alerts live since Tdata 5.23.0 (2026-10-26): Alerts.TriggerType/Condition JSON, getTriggeredAlerts on last Yahoo close, e-mailed by CheckAlerts 07:30; option own-IV conditions still open"
metadata:
  node_type: memory
  type: project
  originSessionId: dd8f0bea-ff17-47d3-80cd-59333ebf1b0c
  modified: 2026-10-26T18:07:21.780Z
---

Since 2026-10-26 (Tdata 5.23.0, TODO #65 partial) an alert can fire on a price condition, not only on its date.

- Write: `addAlert(theme, asset, alert_date, description, trigger_type = "Condition"|"Both", condition = list(yahoo = "NUCL.L", close_lte = 41.9))`. Grammar: `close_lte`/`close_gte` on a Yahoo symbol, `date_gte`, composed with `all_of`/`any_of`. Validated at insert. Themes: `ALERT_THEMES` (13, incl. Stop/Momentum/Volatility).
- Evaluate: `getTriggeredAlerts()` uses the **last Yahoo daily close** (no TWS) — at 07:30 that is the previous session. Unevaluable alerts come back in attr `"unevaluated"` and get their own e-mail section. A met condition is e-mailed every morning until dismissed.
- `scripts/check_alerts.R --dry-run` prints the e-mail without sending.
- 5.24.0: leaf `{"gonet_weight":"PM_15606539","weight_lte":0.09}` = share of NLV in the last Gonet snapshot (alert 88, gold rebalance at 9%). Theme `Allocation`.
- Friday task `\RApplication\CheckAlertsPreclose` 16:30 runs `check_alerts.R --preclose`: condition alerts only, on live Yahoo quotes (`quantmod::getQuote`), e-mail only if something is met. It exists because 07:30 uses the previous close, so a weekly close would otherwise reach the user on Monday. Paris, Zurich and London close 17:30 CET every weekday.
- Gonet exit alerts (26.10 review + `Reports/Gonet_action_plan_20261026.md`):
  - Small-gain lines are absolute: NUCL 81, GTT 82, CNYA 83, DSPF 84 (year-end window to 8.1.2027), AMRZ 94 (dismiss after the sale).
  - Gold weight: 88. TTE results reminder: 87.
  - Core holdings, 5.25.0 relative leaf: `all_of [close ≤ level, stock − beta × index ≤ −5 pts]`, alerts 102–114.
    - Indices: SMI for the Swiss names, ESTX50 for AI/OR/TTE, SPY for QQQ (beta 1.12; the 0.82 down-week slope with intercept was wrong for cumulative moves).
    - Bases are the 26.10 closes; reset them with the monthly re-computation.
- **Why relative:** user rule — light corrections are neither hedged nor sold; deep falls are hedged with ESTX50 put spreads (Jun-27 5700/4600 priced at 142.6 pts on 26.10). With absolute levels, a −10% market fall fired 11 of 13 exits.

**Open:** option own-IV conditions (alerts 60/61 shape) need TWS; Tuser Alerts tab still offers 4 themes and no TriggerType/Condition fields (`alertUI.R:29`).

**How to apply:** for a price trigger, write a Condition alert instead of a dated reminder. Yahoo symbol must be the listing actually held (NUCL.L, not US NUCL). Related: [[reference-alerts-email-date-window]].
