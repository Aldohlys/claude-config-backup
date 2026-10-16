---
name: project_rreporting_trade750_status_anomaly
description: Unresolved 2026-10-16 RReporting display anomaly — trade 750 shown "Ajusté" in main Trades tab until Trade # filter toggled; Closed Trades showed close 01.10 vs DB Close 09.10
metadata:
  type: project
---

2026-10-16: the user saw trade 750 (FCX 81 C, BOT) as "Ajusté" in the main Trades tab with no Trade # filter. Filtering to 750 showed Fermé, and going back to All kept Fermé. Closed Trades at the same time showed Close.Date 01.10.2026 / DIT 10, but the DB has 4 rows all Fermé with the Close leg on 2026-10-09 (Close.Date comes from Tdata trades.R `last_datetime = max(DateTime)`).

The main-table code (`server.R` output$trades → `sub_all_trades()`) displays Status as stored in memory and computes nothing. Hypothesis: stale in-memory state left by edits in that session (DB saved 08:56). Not reproduced; the main-tab screenshot never arrived.

**Why:** if in-memory and DB diverge, a Save in RReporting overwrites the DB with the stale copy (see [[feedback_no_db_writes_behind_rreporting]]).
**How to apply:** if it recurs, ask for the main-tab screenshot plus the session's edits on that trade, compare `all_trades()` with the DB rows, and reload before saving.
