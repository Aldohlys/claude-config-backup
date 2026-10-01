---
name: reference_bot_monthly_add_names
description: Filling BOT metrics for newly added tickers - symbol-subset run cuts terciles on the subset; dry run + merge cache + --resume; bid/ask only in US hours
metadata:
  type: reference
---

`RStudies/reports/bot_monthly/main.R SYM ...` cuts the GapShare / VoV terciles across the names of that run only, and overwrites `data/bot_monthly_phase1.rds` with them. For new names, do instead:
1. `main.R --dry-run --out X.csv SYM ...` (phase 1 for the new names; Tickers untouched; cache now holds only them);
2. merge those rows into a copy of the last full phase-1 cache (keep the old rows whose name is not in the new set);
3. `main.R --resume` (terciles across the whole set, Tickers written for all).

The ATM bid/ask probe (`resolve_atm_spread`) needs US market hours: at night quotes are stale or empty and BOT_VehicleHint (options / stock_only) would be wrong. Daily-price metrics (ATR, touch coefs, ADV, VoV) can run any time; `--no-tws` skips the probe.

First used 2026-10-02: one-off task `BOT_monthly_new_names` (15:45) running `RApplication/logs/bot_monthly_new_names_20261002/run.bat` on the 62 names of the cluster review; its `run.log` holds the outcome — check it if not yet reported. Related: [[reference_bot_three_tool_architecture]], [[feedback_no_edit_running_rscript]], [[project_scanner_correlation_groups]].
