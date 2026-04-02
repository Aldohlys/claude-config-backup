# Claude Code Memory - RApplication

## Critical R Gotchas

### R `$` Partial Matching Bug
- `result$error` will match `result$errors` due to R's partial matching on `$`
- An empty list `list()` is NOT NULL, so `!is.null(list())` is TRUE
- Combined: checking `result$error` when Python returns `errors` (empty list) causes false error detection
- **Fix**: Always use `result[["error"]]` for exact key matching when checking dict/list keys from Python
- See: [r-partial-matching.md](r-partial-matching.md)

## Python/reticulate Integration

### CONFIG Loaded Once at Import Time
- `tdata_py._core.CONFIG = load_config()` runs at module import (line 129)
- `load_config()` calls `find_config_file()` which searches from CWD upward
- Once Python module is imported in an R session, CONFIG is cached - won't reload on re-import
- **Must restart R** to pick up config.yml changes or new hard links
- Shiny apps set CWD to their app directory - need config.yml there

### Shiny App config.yml Hard Links
- Every Shiny app directory needs a hard link to `C:\Users\aldoh\config.yml`
- Without it, Python backend falls back to defaults (missing `historical_config` path, etc.)
- Create with: `fsutil hardlink create <target> C:\Users\aldoh\config.yml`
- `mklink /H` may silently fail in Git Bash; `fsutil` is more reliable on Windows

## Python API Return Structures

### `list_historical_config(return_dict=TRUE)` returns:
- `active_contract_details` (NOT `contracts`) - list of contract dicts
- `collection_settings` (nested) - contains `historical_duration`, `historical_frequency`, etc.
- `contract_summary` - contains `total_contracts`, `active_contracts`, `expired_contracts`
- Contract dicts include: symbol, trading_class, expiration, strike, right, exchange, active, last_updated (as of v5.8.22)

### `update_historical_data()` / `collect_data_for_active_contracts()` returns:
- `contracts_processed`, `contracts_errors`, `files_updated` (ints)
- `errors` (plural, list of strings) - NOT `error` (singular)
- Only returns `error` (singular string) when top-level exception occurs

## Google Cloud VM (trading-vm)
- See: [gcloud-vm.md](gcloud-vm.md) for full details
- Project `rtrading-basic`, zone `us-east1-b`, auto-terminates at 22:15 UTC
- Two users: `aldoh` (gcloud SSH) and `aldohlys` (main user with RProjects)
- Shiny server runs as `shiny` user — permission fix still needed
- SQLite 3.37.2 on VM — no `unistr()` support, use `replace()/char(10)`

## Rscript Multiline Segfault
- Multiline `Rscript -e "..."` causes segfaults on Windows — use temp script files instead
- See: [feedback_rscript_segfault.md](feedback_rscript_segfault.md)

## FX Trade Risk & Currency Conversion
- FX/cash trade Risk = 30% of notional (not 100%) — based on CHF/JPY historical max drawdown ~27%
- See: [project_fx_risk.md](project_fx_risk.md)

## Scanner Universe & Sector Naming
- ScannerUniverse must include all Tickers with IV=YES and price ≤$500
- Tickers table sector names are the canonical namespace (singular form)
- See: [project_scanner_universe.md](project_scanner_universe.md)

## User Profile
- Colorblind — use high-contrast colorblind-safe palette (blue/vermillion/amber, not green/red/orange)
- See: [user_colorblind.md](user_colorblind.md)

## /analyze Command Preferences
- Always generate HTML report (Step 5) regardless of GO/NO-GO verdict — preserves decision history
- See: [feedback_analyze_html.md](feedback_analyze_html.md)
- Use DOW-style report template: decomposed Gate 2 (Trend/Momentum/RS/Volume), clean header with time
- See: [feedback_analyze_report_style.md](feedback_analyze_report_style.md)
- Use BOT columns (BOT_Score, BOT_Flags S1-S6/BK1-BK4), NOT swing Long_Score/Long_Signal
- See: [feedback_analyze_bot_vs_swing.md](feedback_analyze_bot_vs_swing.md)

## Claude Config Backup
- Private repo: `Aldohlys/claude-config-backup` — commands, skills, memory, settings
- Sync manually: ask "sync claude config backup"

## Economic Events Calendar
- Equals Money calendar works with WebFetch (ForexFactory returns 403)
- See: [reference_events_calendar.md](reference_events_calendar.md)

## Windows Task Scheduler
- `schtasks` flags like `/create` get intercepted by Git Bash as paths - use `.bat` files with `cmd //c`
- `fsutil hardlink create` works better than `mklink /H` for programmatic use
- `StartWhenAvailable` in XML ensures missed tasks run when computer wakes up
- XML approach is more reliable than command-line flags for complex settings
