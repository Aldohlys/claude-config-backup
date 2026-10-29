---
name: bot-daily-sector-verdict
description: "BOT_daily sector_verdict column reads macro_context's sector map from the DB table macro_intermarket_sectors; column name `group` must stay"
metadata:
  node_type: memory
  type: project
  originSessionId: 9b8de78c-0e64-46b9-a8b6-2dc5cd5c14a2
  modified: 2026-10-26T11:46:43.044Z
---

Since 2026-10-26 (commit 5112f98), the BOT_daily Data sheet has `group` (the sector) and `sector_verdict` (LONG/SHORT/WATCH/AVOID) right after `name`. `direction` was dropped from the Data sheet unless `--direction both` is run, but it stays in the Detail sheet. The verdict comes from the latest `cache_date` in the DB table `macro_intermarket_sectors`, which macro_context/main.R writes each run. Its groups are the same ScannerUniverse.Cluster groups as BOT's `group`, so they match by name. Nothing checks for staleness: the map date only appears in the Legend header.

**Why:** the user wanted the macro sector-map recommendations beside each BOT name. The column was not renamed to "sector" because bot_fwd/intake.py reads `group` (and `direction`) from the Detail sheet.

**How to apply:** if macro_context has not run, sector_verdict is empty. Run macro_context before BOT_daily. Do not rename `group` or remove `direction` from Detail without updating bot_fwd. Related: [[bot-monthly-runs]].
