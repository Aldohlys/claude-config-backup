---
name: bot-monthly-runs
description: BOT_monthly run hazards (phase-1 cache always beside live DB, --resume after SYM run) and the 2026-10-22 live run outcome
metadata:
  type: project
---
`reports/bot_monthly/main.R` writes its phase-1 cache to `data/bot_monthly_phase1.rds` beside the **live** `mydb.db`, even with `--db <copy>`. Exporting `R_DB_PATH=<copy>` in the shell does not help: Renviron.site overrides it. So a `SYM ...` or `--limit` run on a copy overwrites the live cache with only those rows, and a following `--resume` would rebuild from those few names and then demote every other eligible name ("not_gated").

**Why:** happened 2026-10-22 (a 3-name test run replaced the 312-row cache); the full run later restored 409 rows.
**How to apply:** after any partial run, never `--resume` until a full run has rewritten the cache. `--dry-run` skips all Tickers writes, including the demotion step, so to test demotion you need a non-dry run with `--db <copy>`. A full run takes ~2.5 h (17:01–19:31 on 2026-10-22) and needs TWS up during US hours.

Live full run of 2026-10-22, on branch `claude/funny-albattani-5nviy7`: 324/409 eligible (322 eligible, 2 record_override = STNG, KNIN), no demotions. Backup taken first: `data/mydb_backup_20261022_pre_bot_monthly.db`. KNIN is in only through the trade-record override, with ADV 38 against the 85 floor; raise `RECORD_MIN_TRADES`/`RECORD_MIN_PNL` if that is unwanted. The override is keyed on `Trades.Symbol`, see [[trades_table_schema]].
