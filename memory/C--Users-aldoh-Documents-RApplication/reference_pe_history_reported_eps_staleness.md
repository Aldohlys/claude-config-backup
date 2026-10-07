---
name: reference_pe_history_reported_eps_staleness
description: "Yahoo earnings calendars stop silently (AI.PA 2022, RIO 2008, BTI 2006); collect_pe.py now expires/drops them; consensus surprise is NOT a quality test; /analyze shows expired series as n/a"
metadata:
  node_type: memory
  type: reference
  originSessionId: 1c22afef-2f88-43f7-b806-e8014e3af784
  modified: 2026-10-07T15:49:31.408Z
---

Found 2026-10-07. `PE_History` basis `reported_eps` (scripts/collect_pe.py --rebuild) forward-filled
the last trailing EPS forever, so names whose Yahoo earnings calendar had stopped got a current P/E
from years-old reports: AI 47.6 (Yahoo trailing 30.4; calendar ends 2022-04, EPS ~0.43x estimates),
SU 91, BHP 50, RIO / BTI from 2008 / 2006. /analyze's own-history row and adjustment gap read them.

Fix (collect_pe.py): cadence detection (median report gap > 120 d -> 2 half-year reports, else 4);
a TTM window must be consecutive reports (gap <= 140 d quarterly / 250 d half-yearly); the daily
figure expires that many days after its report; newest report > 200 d old -> name dropped and its
reported_eps rows deleted; a rebuild deletes the window before writing. `--dry-run` lists cadence
and drops. 2026-10-07 run: dropped AI, SU, RIO, BHP, BTI, FNV, USAR; series ended early AZN, EQNR, OKLO.

**Consensus surprise is no quality test**: TRV, PLD, UBSG report 1.1-2x estimates every quarter
on real figures; INTC/BA/COIN swing near zero. A |surprise| filter dropped 21 good names.

No universe name is half-yearly on Yahoo today (TTE, NOVN, UBSG show quarterly EPS).
/analyze valuation.R: series whose last row is > 10 days old -> "n/a, series ends <date>";
a Type=STK name with no series is still an equity (not rendered as an ETF).
Related: [[reference_valuation_app_sws]], [[reference_yahoo_ticker_and_oi_traps]].
