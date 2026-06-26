---
name: gcloud-vm
description: "Google Cloud trading-vm reference — connection/SSH/SCP, filesystem layout, Shiny Server, R env, SQL-dump compatibility (SQLite 3.37.2), and RStudies//analyze deploy"
metadata: 
  node_type: memory
  type: reference
  originSessionId: b7c72d7e-945c-4cbf-8075-4bd8bf2373ec
---

# Google Cloud VM - trading-vm

## Connection
- Project: `rtrading-basic`
- VM: `trading-vm`, zone `us-east1-b`, machine type `e2-small`
- SSH: `gcloud compute ssh trading-vm --zone=us-east1-b --project=rtrading-basic`
- SCP: `gcloud compute scp <local> trading-vm:/home/aldoh/<file> --zone=us-east1-b --project=rtrading-basic`
- **VM auto-terminates at 22:15 UTC** (cost-saving schedule)
- gcloud CLI path (Windows): `"/c/Users/aldoh/AppData/Local/Google/Cloud SDK/google-cloud-sdk/bin/gcloud.cmd"`
- Auth: `aldohlys@gmail.com`
- **gcloud.cmd from Git Bash mangles `--command="bash /x.sh"`** (splits on the space in the install path → "Google Cloud not recognized"). `scp`/`describe` work from Bash, but for `ssh --command=` invoke via the PowerShell tool: `& $gcloud compute ssh ... --command="bash /tmp/x.sh"`.

## Start/Stop automation & boot wiring (verified 2026-06-26)
- **NOT a Windows task / NOT a startup-script.** Start & stop are two **Cloud Scheduler** jobs in location `us-east1` (project `rtrading-basic`), each a direct OAuth POST to the Compute API (no Cloud Function), as SA `scheduler-sa@rtrading-basic.iam.gserviceaccount.com`:
  - `start-trading-vm` — `45 8 * * 1-5` Europe/Zürich (08:45 Mon–Fri) → `…/instances/trading-vm/start`. **PAUSED by user 2026-06-26** to stop idle billing.
  - `stop-trading-vm` — `15 22 * * 1-5` Europe/Zürich (the 22:15 stop) → still **ENABLED** (safety net; triggers clean IB Gateway shutdown via `ExecStop`).
  - Manage: `gcloud scheduler jobs {pause|resume|run} start-trading-vm --location=us-east1 --project=rtrading-basic`.
- **The scheduler ONLY powers the VM on.** Everything else is `multi-user.target` systemd, so ANY power-on (Cloud iOS app "Start", gcloud, console) reproduces the full stack:
  - `xvfb.service` → `Xvfb :1` (virtual display for headless GUI)
  - `trading-stack.service` ("Trading Stack (IB Gateway + IBC)") → `/opt/scripts/start-trading.sh`, runs as `aldohlys`, `Requires=xvfb`, `After=network-online+xvfb`, `Restart=on-failure`, `TimeoutStartSec=360`. `ExecStop=/opt/scripts/stop-trading.sh`.
  - `shiny-server.service` (independent). `supervisor.service` enabled but `conf.d` empty (unused). No crontab/`@reboot`/`rc.local`.
- **iPhone launch:** Google Cloud iOS app → Compute Engine → `trading-vm` → Start. Allow ~6 min (`TimeoutStartSec=360`) for IB Gateway login before expecting data.
- **iPhone stop is safe:** `stop-trading-vm` and the Cloud app "Stop" hit the SAME `…/instances/trading-vm/stop` API — a graceful ACPI shutdown (GCE allows ~90s) → systemd runs `trading-stack.service` `ExecStop=/opt/scripts/stop-trading.sh` (~5s) → clean IB Gateway kill. So app "Stop" cleans up identically to the 22:15 job.
- **⚠️ NEVER use "Reset" in the Cloud app** — it's a hard power cycle that skips systemd shutdown, so `ExecStop` never runs and IB Gateway is NOT cleaned up. Use "Stop" only.
- **`/opt/scripts/start-trading.sh`**: reads `/home/aldohlys/config.yml` (`production.ibkr.api_port`→4001, `trading_mode`→'live'); verifies Xvfb on `:1`; launches IB Gateway via IBC — `TWS_MAJOR_VRSN=1037`, `IBC_PATH=/opt/ibc`, `IBC_INI=/opt/ibc/config.ini`, `TWS_PATH=/opt/ibgw`, `TWS_SETTINGS_PATH=/home/aldohlys/Jts`, `APP=GATEWAY`, via `/opt/ibc/scripts/ibcstart.sh 1037 -g --mode=$TRADING_MODE`; polls API port up to 240s; `wait`s on IBC PID. Log: `/var/log/trading-stack.log`.
- **`/opt/scripts/stop-trading.sh`**: `pkill -f ibcalpha.ibc`; `pkill -f "java.*ibgateway"`; sleep 5.

## SCP Gotchas
- `pscp` can't open files locked by other processes (e.g., SQLite DB open in R) — copy to temp first via `cmd //c copy`
- Target `~` gets interpreted as literal filename — always use explicit paths like `/home/aldoh/mydb.db`
- `aldoh` user can't write to `aldohlys` directories — upload to `/home/aldoh/` then `sudo mv`

## Users
- `aldoh` — created by gcloud SSH, mostly empty
- `aldohlys` — main user with RProjects, IBKR (Jts), Claude Code, Python venv

## Filesystem Layout
```
/home/aldohlys/
├── RProjects/
│   ├── RApplication/data/  — mydb.db, mydb.sql
│   ├── RJournal/app/       — Shiny app with local config.yml
│   ├── Tbasics/
│   ├── Tdata/
│   ├── Tlogger/
│   └── Tuser/
├── Jts/                    — IBKR TWS
├── ibc/                    — IBC (IBKR auto-login)
├── trading-venv/           — Python venv
├── config.yml              — main config (has Windows paths in default profile!)
└── .claude/                — Claude Code config
```

## Shiny Server
- Config: `/etc/shiny-server/shiny-server.conf`
- Apps dir: `/srv/shiny-server/` (contains `rjournal`, `sample-apps`, `index.html`)
- Runs as user `shiny` (`run_as shiny;`) — **permission issue**: can't read aldohlys files
- Port: 3838
- Restart: `sudo systemctl restart shiny-server`

## R Environment (VM)
- R version: 4.5.2
- R_HOME: `/usr/lib/R`
- Renviron.site: `/usr/lib/R/etc/Renviron.site` — configured with:
  - `R_BOX_PATH=/home/aldohlys/RProjects/Tuser`
  - `R_CONFIG_FILE=/home/aldohlys/config.yml`
  - `R_CONFIG_ACTIVE=default`
  - `R_DB_PATH=/home/aldohlys/RProjects/RApplication/data/mydb.db`
  - `R_LOG_DIR=/home/aldohlys/RProjects/RApplication/logs`
- **Note**: `R --vanilla` skips Renviron.site — use `R --no-restore` to test env vars
- SQLite version: 3.37.2 (does NOT support `unistr()` — needs `replace()/char(10)` instead)

## RJournal Config
- `/home/aldohlys/RProjects/RJournal/app/config.yml` has correct Linux paths:
  - default DB: `/home/aldohlys/RProjects/RApplication/data/mydb.db`
- App uses `config::get("DB")` (reads local config.yml, not Renviron)

## SQL Dump Compatibility
- SQLite 3.41+ uses `unistr()` for newlines in text — incompatible with VM's 3.37.2
- SQLite 3.50+ uses `replace('...','\n',char(10))` — compatible with all versions
- `backup_database.R` updated with `unistr()` post-processing safety net
- Dump verified: all 17 tables, all row counts match after restore

## RStudies / scanner / analyze on VM (added 2026-06-02)
- RStudies is seeded (subset) at `/home/aldohlys/RProjects/RStudies/` — `reports/{analyze,shared,swing_scanner,macro_context}/`. `/analyze` runs via `/opt/scripts/analyze.sh <TICKER> <DIR>` (needs IB Gateway up).
- **Blessed deploy:** `RApplication/scripts/push-rstudies-to-vm.ps1` — but it's INTERACTIVE (a `Read-Host` y/N confirm), so it can't be run non-interactively. For a targeted push, stage LF-normalized files (`tr -d '\r'`) → `gcloud compute scp --recurse` to `/tmp/...` → ssh `rsync -a /tmp/.../reports/ $R/reports/`. (gcloud is authed as aldohlys@gmail.com; runs non-interactively fine.)
- **VM DB has NO `scanner_results` table** — the swing scanner runs LOCALLY (writes scanner_results to the local mydb.db); the VM only runs `/analyze`, which doesn't touch that table. So scanner schema/column changes need no VM DB migration.
- Inline `Rscript -e "..."` over `gcloud ssh --command='...'` is quote-hell — for parse checks etc., scp a small `.sh` to /tmp and `bash` it instead.

## TODO (next session)
1. Check `/srv/shiny-server/rjournal/` — how it links to the app
2. Fix permissions — app runs as `shiny` user but DB owned by `aldohlys`
3. Test Shiny app in browser
