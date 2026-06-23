---
name: project-analyze-on-vm-deployment
description: /analyze pipeline being ported to the GCP trading VM so Claude Code on the VM can run it. Wrapper + RStudies deployed 2026-05-26; env-var wiring + renv::restore still pending.
metadata: 
  node_type: memory
  type: project
  originSessionId: 5367d3b5-2222-403f-93b3-db52f823f3be
---

# /analyze on the trading VM — deployment status (2026-05-26)

Goal: run `/analyze TICKER DIRECTION` from a Claude Code session on the GCP
VM (gcloud compute ssh aldohlys@trading-vm), with an IBKR Gateway
pre-flight check so Phase D doesn't silently produce chain_state=unknown
when the gateway is down.

**Why:** the local laptop is intermittently offline; VM runs the trading stack
continuously, so running /analyze there avoids needing TWS up on the laptop
and gives faster Phase D pulls (gateway already authenticated).

**How to apply:** when extending /analyze, edit the source-of-truth files
locally and re-push:

- Wrapper:  `C:\Users\aldoh\Documents\RApplication\scripts\analyze.sh`
            → push with `scripts\push-analyze-to-vm.ps1` → `/opt/scripts/analyze.sh`
- Pipeline: `C:\Users\aldoh\Documents\RApplication\RStudies\reports\analyze\*`
            → push with `scripts\push-rstudies-to-vm.ps1` → `/home/aldohlys/RProjects/RStudies/`

## What's deployed

- `/opt/scripts/analyze.sh` (root:root 0755) — reads `production.ibkr.api_port` from
  `/home/aldohlys/config.yml` (= 4001), verifies socket via `ss -tlnp` + `nc -z` (15s
  fast-fail), then runs RStudies/reports/analyze/main.R. Exit codes: 0=ok, 1=usage,
  2=gateway-down, 3=R-failure.
- `/home/aldohlys/RProjects/RStudies/` — seeded with `.Rprofile`, `config.yml`
  (out_dir patched to `/home/aldohlys/reports`), `renv.lock`, `renv/activate.R`,
  `renv/.gitignore`, and `reports/{analyze,shared,macro_context,swing_scanner}/`.
  41 files, 720 KB. Push script skips `renv/library/` and `renv/staging/`
  (Windows binaries) and empties `reports/*/output/` (local artifacts).

## Open follow-ups (next session)

1. **Wire env vars into analyze.sh.** Decided per-invocation pattern (not global
   Renviron.site, not merged config.yml). Add this block to `analyze.sh` just
   before the `Rscript` call:
   ```bash
   export R_CONFIG_FILE="$RSTUDIES_DIR/config.yml"
   export R_CONFIG_ACTIVE=production
   export R_DB_PATH=/home/aldohlys/RProjects/RApplication/data/mydb.db
   export R_LOG_DIR=/home/aldohlys/RProjects/RApplication/logs
   ```
   Rationale: zero blast radius on existing R jobs (daily_portfolio_update.R etc.);
   self-documenting wrapper; aligns with [shared config.yml TODO](project_shared_config_yml_todo.md).
   After editing, `scripts\push-analyze-to-vm.ps1` re-deploys.

2. **`renv::restore` on the VM.** Long-running (10–30 min, compiles from source
   on Linux). Pre-install system deps to avoid mid-restore failures:
   ```bash
   sudo apt-get install -y libxml2-dev libssl-dev libcurl4-openssl-dev \
     libfontconfig1-dev libfreetype6-dev libharfbuzz-dev libfribidi-dev \
     libpng-dev libtiff5-dev libjpeg-dev
   cd /home/aldohlys/RProjects/RStudies
   Rscript -e 'renv::restore(prompt = FALSE)' 2>&1 | tee /tmp/renv_restore.log
   ```
   Note: renv.lock pins R 4.4.3 but VM has R 4.5.2 — version-mismatch warning
   expected, usually harmless. If specific package build fails, suspect it first.

3. **Smoke test** after #1 and #2:
   ```
   gcloud compute ssh aldohlys@trading-vm --zone us-east1-b --project rtrading-basic \
       --command='/opt/scripts/analyze.sh UPS short --no-vol-funnel'
   ```

4. **Optional:** write `bootstrap-rstudies-on-vm.sh` that idempotently runs the
   apt-installs + renv::restore, so the VM is reproducible from scratch. Skipped
   for now — manual run on 2026-05-26.

## Open question parked for later

- The `production:` block in RStudies/config.yml is minimal (cloud + DB paths,
  no `analyze` sub-block). The push script patches the `default.analyze.out_dir`
  to a Linux path; if we ever need divergent VM-vs-laptop analyze settings,
  add a `production.analyze` block and have `load_analyze_config` deep-merge
  it on top of `default.analyze` when `R_CONFIG_ACTIVE=production`. Currently
  `load_analyze_config` only reads `raw$default$analyze`.

## Related memories

- [PowerShell `$Var:` scope-qualified](reference_powershell_variable_colon.md) — bit `push-analyze-to-vm.ps1` first run
- [pscp `--recurse` dest-dir](reference_pscp_recursive_destdir.md) — bit `push-rstudies-to-vm.ps1` first run
- [Shared config.yml TODO](project_shared_config_yml_todo.md) — aligned with per-invocation R_CONFIG_FILE
