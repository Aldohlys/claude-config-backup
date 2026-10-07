---
name: reference_tuser_weekly_unpnl
description: "Tuser Positions WeeklyunPnL = mean over the last 5 snapshot dates of (today's value − that day's LAST snapshot value restated to today's size); Gonet keys on symbol"
metadata:
  node_type: memory
  type: reference
  originSessionId: a3517eda-94f8-48a8-b46e-2095ae048c64
  modified: 2026-10-05T06:28:01.156Z
---

Defined in `Tuser/portfolio/logic/portf.R`, `compute_weekly_unpnl` (~l.354-455), tests in `Tuser/tests/test_weekly_unpnl.R`.

```
WeeklyunPnL = mean over the 5 most recent snapshot DATES before today of
              SUM over legs of (cur_mktValue − hist_mktValue × cur_pos / hist_pos)
hist = the last snapshot of each date (MAX(heure))
```

- It is an average gap against the last five sessions, not a 5-day change. A steady decline therefore reads at roughly half the 5-day move: SDZ was −680 CHF over 5 days but shows −375.
- The last snapshot of a day can be intraday, e.g. a 17:01 run when no evening run happened. Its price feeds the figure as-is, so a repaired snapshot must carry the price at that time, not the close.
- Gonet keys on `symbol` (TradeNrs are reused across instruments); IBKR keys on TradeNr with Instrument legs. A leg absent from a session contributes nothing for that session.
- The Positions tab "Etat des comptes (N).csv" export is Tuser's own output (columns `t_WeeklyunPnL_base`, `t_is_total`), not a bank file.

To reproduce a figure from the DB, compute per-date last-snapshot mktValue for the symbol and average (today − each). Related: [[reference_gonet_account_history]].
