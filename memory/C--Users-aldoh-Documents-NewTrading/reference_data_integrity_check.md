---
name: reference_data_integrity_check
description: Weekly data-integrity regression check scripts/check_data_integrity.R (task \RApplication\DataIntegrity, Sat 09:00) — what it checks, how to read/extend it, --all/--db for audits
metadata:
  type: reference
---

`RApplication/scripts/check_data_integrity.R` was built on 2026-09-23 at the user's request ("a regression test run once a week that checks overall data integrity"). It runs as the scheduled task `\RApplication\DataIntegrity` every Saturday at 09:00 (StartWhenAvailable), through run_task.bat, with `--email`.

Checks, all aimed at bug classes found during the 2026-09-22/23 repairs:
- ERROR: FX one-day outliers (±5% vs both neighbours) and stale currencies (>10 days).
- ERROR: account identity NLV − SMV − options − cash, for Gonet and U25 only (IBKR accounts carry accruals and are skipped).
- Account moves above ±4% that neither UPnL nor a CashFlow explains: ERROR for base-currency rows ≤5 days apart; WARN for USD-era rows, longer gaps and the DU paper account.
- ERROR: snapshot mktPrice ≤ 0. WARN: mktPrice NA, and >25% jumps between snapshots (catches renamed tickers, wrong listing or currency).
- ERROR: Gonet ledger vs GonetPos, negative shares or cost-sign errors, Gonet price vs Yahoo close outside 0.85–1.18.
- WARN: exact duplicate Trades rows. ERROR: dividend rows on a trade that doesn't hold the stock.

Output goes to `NewTrading/Reports/data_integrity_YYYYMMDD[_dbname].csv`; the exit code is 1 on any ERROR. Options: `--all` (whole history), `--days N`, `--db PATH` (read-only audit of a backup), `--no-yahoo`. Validation: on `mydb_before_gonet_reprice_20260922.db` it flags 84 errors covering every repaired bug; the live DB is clean.

Email works: `ALERT_EMAIL_PASSWORD` lives in `RApplication/Renviron.site`, which is a HARD LINK to `R-4.4.3/etc/Renviron.site`, so every R session loads it. It is not a Windows env var: check from R (`Sys.getenv`), not from PowerShell. A test email was sent on 2026-09-23. The file also sets R_BOX_PATH, R_CONFIG_FILE, R_CONFIG_ACTIVE, R_DB_PATH, R_LOG_DIR and RETICULATE_PYTHON.

**How to apply:** when fixing a new data bug, add a check for its class here so it cannot recur silently. Run `--all` after any bulk DB repair.

Related: [[reference_gonet_account_history]], [[reference_fx_table_outliers]], [[reference_internal_transfer_cashflows]].
