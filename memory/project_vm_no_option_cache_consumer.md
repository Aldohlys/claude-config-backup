---
name: project_vm_no_option_cache_consumer
description: VM does not read chain/strike parquet caches — /analyze never calls getChains/getOptionStrikes
metadata: 
  node_type: memory
  type: project
  originSessionId: 92762a60-9765-437c-baa5-843b8fe004b9
---

The trading VM has **no consumer** for the option chain/strike parquet caches (`NewTrading/{chains,strikes}/*.parquet`), so syncing them to the VM is pointless today.

**Why:** The VM runs `/analyze` only. No file in `RStudies/reports/analyze/` (incl. `funnel.R`, the vol-funnel) calls `getChains()`/`getOptionStrikes()` or reads the parquet caches — the funnel grid is built mechanically from price/ATR. The only strike reference in the reports pipeline is a *comment* in `swing_scanner/universe_filter.R`, and the swing scanner runs LOCALLY (VM has no `scanner_results`; see [[gcloud-vm]]). The VM runs `R_CONFIG_ACTIVE=production`, whose profile in `config.yml` defines **no** `chains_dir`/`strikes_dir` (only `DB` + `cloud`) — it would inherit the Windows `default` paths (`C:/Users/...`), meaningless on Linux.

**How to apply:** TODO #27's premise that caches are "synced to the VM as part of the nightly backup" is aspirational, not current reality. The Cache Janitor (`scripts/janitor_caches.ps1`) cleans LOCAL caches (used by RPreTrade + local scanner) and is wired into `backup_database.bat`. The VM-sync scripts (`sync-caches-to-vm.ps1` / `pull-caches-from-vm.ps1`) exist as manual tools but are deliberately NOT in the nightly task — wire them only after a VM-side cache consumer exists (and add `chains_dir`/`strikes_dir` to the production profile + a Linux path). VM also auto-terminates 22:15 UTC, so nightly pushes risk a powered-off target.
