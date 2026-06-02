# Google Cloud VM - trading-vm

## Connection
- Project: `rtrading-basic`
- VM: `trading-vm`, zone `us-east1-b`, machine type `e2-small`
- SSH: `gcloud compute ssh trading-vm --zone=us-east1-b --project=rtrading-basic`
- SCP: `gcloud compute scp <local> trading-vm:/home/aldoh/<file> --zone=us-east1-b --project=rtrading-basic`
- **VM auto-terminates at 22:15 UTC** (cost-saving schedule)
- gcloud CLI path (Windows): `"/c/Users/aldoh/AppData/Local/Google/Cloud SDK/google-cloud-sdk/bin/gcloud.cmd"`
- Auth: `aldohlys@gmail.com`

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
