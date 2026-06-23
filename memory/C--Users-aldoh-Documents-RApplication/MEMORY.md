# Claude Code Memory - RApplication

## User Profile & Preferences
- [Colorblind — use blue/vermillion/amber, not green/red/orange](user_colorblind.md)
- [Present UI design before coding — text plan + ASCII previews, wait for approval](feedback_design_before_code.md) — 2026-05-19 BSM session surfaced 4 hidden reqs only after layout shown
- [Comments explain shipped code, not the implementation plan — no "Phase N"/plan refs in code or identifiers](feedback_comment_describe_shipped_code.md) — 2026-06-10; spawned TODO #74
- [Don't embed transient schema-migration logic in committed code — target final schema, defer until migration done](feedback_no_transient_migration_logic_in_code.md) — 2026-06-14; sync_stop_risk.R pick() resolver already collapsed (commit 6e5e4a2)
- [User edits SQLite config tables directly in DB Browser — don't build in-app editors for DB-resident config unless asked](feedback_user_edits_db_directly.md) — 2026-06-15; closed #75 MaxRisk residual to ~0 work; flag the memoise-cache restart caveat

## /analyze Command
- [Always generate HTML report regardless of verdict](feedback_analyze_html.md)
- [Neutral stance — bare facts, no GO/NO-GO, no editorial adjectives](feedback_analyze_neutral_stance.md)
- [DOW-style template: decomposed Gate 2 (Trend/Momentum/RS/Volume), clean time header](feedback_analyze_report_style.md)
- [Use BOT columns (BOT_Score, BOT_Flags S1-S6/BK1-BK4), not swing Long_Score](feedback_analyze_bot_vs_swing.md)
- [Every indicator live-fetched; no n/a cells; no phase short-circuit for data fetch](feedback_analyze_live_data_fallback.md)
- [De-gated 2026-06-02 (TODO #60): provenance vocab LIVE/CACHED/NO DATA/FETCH FAILED](project_analyze_degate.md) — NO DATA≠FETCH FAILED; shared/live_sources.R is /analyze-only
- [TWS-gated at startup via isIBAvailable(); short-circuits to "FETCH FAILED" if probe fails](project_analyze_tws_gate.md) — residual hang TODO #52
- [/analyze command lives in NewTrading/.claude/commands/ — thin wrapper to RStudies reports/analyze/main.R](reference_analyze_command_location.md) — NOT RApplication; gitignored→only in config-backup; Windows Rscript (cwd RStudies) / VM /opt/scripts/analyze.sh; current = neutral data report

## Risk, Trades & FX
- [Trades table: signed-delta Risk + EventType + stored Return](project_trades_signed_delta_risk.md) — BOT shipped 2026-05-18; generalized to all strategies 2026-06-10 (Phase 5)
- [Strategies.MaxRisk per-strategy Risk budget + signed-delta generalized to OFI/WHEEL/CS/BPT/Sharpe2/Perso](project_strategies_maxrisk_cap.md) — #75 CLOSED 2026-06-16; values set to 1%-of-IBKR-sleeve (~800 ceiling) policy budgets; MaxRisk caps DISPLAY for all strategies
- [Account↔strategy topology + per-trade risk sizing base](reference_account_strategy_topology.md) — strategy trades live in IBKR sleeve (U1804173 main, VALUE in U25343478); Gonet 402k stocks-only CSV untagged; size vs ~80.7k sleeve not 483k total; aggregate overlay → TODO #76
- [Tdata::saveTrades drops+recreates Trades table (overwrite=TRUE)](reference_savetrades_overwrite.md) — smoke-test scripts/smoke_test_savetrades.R before Save
- [Verifying per-trade invariants: don't filter-then-aggregate event rows](feedback_sql_aggregation_event_filter.md) — false-alarm rollback panic 2026-05-18
- [Trades↔portfolio join key depends on instrument type](reference_trades_portfolio_join_key_by_type.md) — stocks via Ssjacent==symbol, options/futures via Instrument; bit stats_one_position (trade 697)
- [FX/cash trade Risk = 30% of notional](project_fx_risk.md) — CHF/JPY max drawdown ~27%
- [scripts/sync_stop_risk.R: stock+futures Risk from live TWS stops + Trades.Stop column](project_stop_based_risk_sync.md) — TODO #67; match on Symbol==order symbol (STK mult=1, FUT ×multiplier); daily Step 3/4; 10 U25343478 trades 2026-06-14
- [TODO #35 COMPLETE 2026-06-15 (Tdata 5.10.29): Ssjacent→Symbol (normalized root) + Underlying col; all 4 phases shipped](project_trades_symbol_underlying_model.md) — Phase 0 (Underlying)+Phase 1 (5-col rename Ssjacent→Symbol/Prix→Price/Comm.→Commission/Statut→Status/Remarques→Notes) 2026-06-14; columns NOW renamed in DB+code (old "Ssjacent"/"Prix" refs in other memories are stale); docs/TRADES_REFACTORING_PLAN.md
- [CASH/FX trades: identify by Instrument-in-currencies, NOT Symbol=="CASH"; Symbol=foreign (non-base) currency](project_trades_cash_identification.md) — TODO #35 Phase 2+3; Symbol-as-cash-detector unsafe (CHF future aliases); 687 join fix = symf gate type%in%c("Stock","CASH") + pass Currency to displaytradeUI
- [Trades column-rename playbook (flag-day)](reference_trades_column_rename_playbook.md) — sed must cover test dirs; no field.types edit for TEXT; summarize() self-ref trap; smoke-test symf via load_all not library; gate symbol-join filter(type=="Stock")
- [Adding a Trades column: update RReporting trade_fields + all 4 row builders](reference_rreporting_trade_fields_schema_sync.md) — else New-Trade rbind breaks + Save silently drops the column; is_options_block crashes on staged data (use Strike/Right gate)
- [Summary-tab open-trade realizedPnL from the trade's OWN legs, not broker avgCost](project_rreporting_summary_realized_pnl.md) — accumulate-only=0; avgCost carries fees Total omits (702 = UK stamp duty); Risk capped at MaxRisk in this view
- [RReporting Summary is trade-driven — untagged CASH (TradeNr=NA, no CASH trade) is invisible](reference_untagged_cash_hidden_in_summary.md) — 2026-06-23 U1804173 GBP 31.59 hidden (no GBP trade); EUR/JPY/USD show via 659/687/704
- [Activating a currency auto-enrolls FX conversion but NOT interest rates (needs a fetcher)](reference_adding_currency_two_pipelines.md) — Tdata 5.11.0 added GBP/KRW fetchers; "No ConvertToCHF rate for X; defaulting to 1.0" = missing data not a bug

## Account / TWR / CashFlow
- [readPortfolio() date is Date class, NOT YYYYMMDD int like Trades.TradeDate](feedback_readportfolio_date_type.md)
- [Account.CashFlow conventions](reference_account_cashflow_conventions.md) — native ccy, FX via AccountWithConversionRate view, synthetic rows for position transfers
- [2026-04-16 transfer TWR fix](project_account_transfer_cashflow_signs.md) — record in-kind + cash legs; Tdata 5.10.18 chain-links observed dates
- [Strategy/slice value pitfalls](reference_strategy_value_pitfalls.md) — use unPnL not mktValue (futures notional); TWR breaks on cumulative series; r_factor UI-only; TODO #69
- [IBKR Activity Statement layout](reference_ibkr_activity_statement.md) — manual export; official TWR, per-ccy cashflows, in-kind; pull both source+dest
- [Portfolio TradeNr=NULL when leg entered after snapshot](project_tradenr_backfill_stale.md) — Rscript data/fix_tradenr.R; no Account filter, unsafe for DU5221795 cash

## Scanner & Swing
- [ScannerUniverse = all Tickers IV=YES & price ≤$500; Tickers sector names canonical (singular)](project_scanner_universe.md)
- [Flow_Score = flow/context composite, NOT raw BOT score](project_scanner_flow_score.md) — renamed from Pull_Score; schema v7; sectorization→TODO #71
- [Asymmetric trade thesis](project_asymmetric_trade_thesis.md) — "2nd/3rd inning" over breakout; 30-60 DTE; 10-day earnings buffer; VRP=log-ratio; IVR+IVP_2y
- [Earnings flag: NextEarnings + EarningsInDays in swing scanner (yfinance, not WSH)](project_earnings_flag_feature.md)

## IBKR / TWS
- [isIBAvailable() is the canonical TWS reachability probe — don't re-implement](reference_isIBAvailable.md)
- [Test IBKR/TWS code against live TWS BEFORE editing; halt if TWS down](feedback_test_tws_first.md)
- [IB.RequestTimeout caps stuck ib_async requests — Tdata 5.10.12](project_ib_request_timeout.md) — 60s; partially closes TODO #52
- [TWS subscription state populates even when end-event await times out](feedback_tws_subscription_outlives_end_event.md) — read portfolio()/accountValues() after timeout
- [reticulate/asyncio uninterruptible — timeout must live in Python](feedback_reticulate_asyncio_uninterruptible.md)
- [TWS drops connection on malformed requests — corrupts rest of batch (Error 320/200)](feedback_tws_drops_connection_on_malformed.md)
- [Never send NaN to TWS API — filter at call boundary](feedback_no_nan_to_tws.md)
- [IBKR sub-account API: portfolio subscription required; accountSummary tags split per-sub vs All](project_ibkr_subaccounts.md)
- [FUT/FOP contract fields](reference_ibkr_fut_fop_contract_fields.md) — symbol vs localSymbol; FOP needs root+tradingClass; reqSecDefOptParams unreliable for FUT
- [lastTradeDateOrContractMonth='YYYYMM' picks by last-trade-date, not label](feedback_ibkr_lasttradedate_yyyymm.md) — use full date or conId
- [Tickers FUT row: Name=local symbol, TradingClass=options class](reference_tickers_fut_row_convention.md) — MCO for MCL, LO for CL
- [chains_manager.py FUT bug fixed Tdata 5.10.16/17](project_chains_manager_fut_bug.md)
- [Flex Trade section excludes transfers/corporate actions — use Monthly Activity Statement](project_flex_query_coverage.md)
- [Master-level Flex returns identical CSVs per sub-account](project_flex_query_master_level.md) — add ClientAccountID or per-sub queries
- [WSH event-data API needs News Feed entitlement (Error 10276)](project_wsh_news_feed_gotcha.md)

## Tdata, Python & Build
- tdata_py CONFIG loads once at import & caches — restart R for config.yml changes; Shiny apps need config.yml in CWD
- Each Shiny app dir needs a hardlink to C:\Users\aldoh\config.yml (fsutil hardlink create; mklink /H unreliable in Git Bash)
- [Tdata::tdata_py is lazy — touch it before py_run_file() importing tdata_py](project_tdata_py_lazy_init_startup.md) — RPreTrade is reference
- [tdata_py API return field names](reference_tdata_py_api_returns.md) — active_contract_details (not contracts); errors plural
- [Check Welcome Tdata version log line first when "old library"/hang](feedback_check_welcome_log.md)
- [Tdata lives in 8 install locations; /build covers only 6](project_tdata_install_locations.md) — RLibrary + renv sandbox need explicit install
- [install.packages() can leave Python files stale on Windows](feedback_install_packages_python_stale.md) — verify by grep or remove.packages() first
- [Update CHANGELOG.md before /build](feedback_changelog_before_build.md) — else generic commit message
- [quick_tests=TRUE for Python-only Tdata changes](feedback_tdata_quick_tests_for_python_only.md) — full suite hits live IBKR, can hang 15+min
- [build_package.R force=TRUE when version bump committed manually](feedback_build_package_force_after_manual.md)
- [build_package.R does `git add .` — never /build with a dirty tree of others' work](feedback_build_package_git_add_all.md)
- [Tdata 5.10.26 cache-warning surfacing committed (3a7d207) but deploy PENDING](project_tdata_5_10_26_deploy_pending.md) — force-deploy on clean tree
- [Tests hitting the live tdata_py active binding fail in build CWD — inject the imported module](feedback_tdata_py_binding_test_injection.md)
- [Leave renv lockfiles alone — never snapshot to silence out-of-sync](feedback_renv_leave_lockfiles_alone.md)
- [Don't fix renv warning with renv::restore(packages='renv') — corrupts activate.R](feedback_renv_restore_corrupts_activate.md)
- [renv state safe to track: .Rprofile + renv.lock + renv/{activate.R,settings.json,.gitignore}](reference_renv_safe_to_track.md)
- [vctrs load-time bomb in RLibrary from tibble/dplyr drift — upgrade vctrs](feedback_vctrs_tibble_dplyr_cascade.md)

## R Gotchas & Idioms
- [any()/all() over NA-bearing vector returns NA → crashes scalar if()](feedback_any_na_crashes_scalar_if.md) — regexpr() returns NA for NA input; make predicates NA-total at source; bit RReporting Summary tab 2026-06-17
- [dplyr summarize() args evaluated in order — a new col shadows an input col of the same name for later args](feedback_dplyr_summarize_self_reference.md) — `Total_native=sum(Total)` after `Total=sum(base_Total)` reads the new scalar; bit RReporting realizedPnL 2026-06-15
- [Rscript -e multiline segfaults on Windows — use temp script files](feedback_rscript_segfault.md)
- [display_error_message calls stop() — code after is unreachable](feedback_display_error_message_stops.md)
- [return() inside tryCatch({}) body exits the ENCLOSING function, not the tryCatch — use last expr or stop()](feedback_r_return_inside_trycatch.md)
- [with_mocked_bindings can't mock active bindings](feedback_mock_active_bindings.md) — swap .tdata_state$value via local_mock_tdata_py()
- [$ partial matching: result$error matches result$errors — use result[["error"]]; empty list() ≠ NULL](r-partial-matching.md)
- [apply(df,1,fn) stringifies rows → isTRUE() always FALSE](feedback_apply_df_row_coercion.md) — use vapply over seq_len(nrow)
- [Filter(Negate(is.null), list(...)) — c(NULL,x) vs parallel sapply mask length-mismatch](feedback_filter_negate_isnull.md)
- [No on.exit() at top-level of sourced scripts — use explicit dbDisconnect()](feedback_sourced_script_on_exit.md)
- [Use .libPaths()[1], don't hardcode RLibrary path](feedback_libpaths_not_hardcoded.md)
- [Sys.getenv("HOME") on Windows = USERPROFILE, not Documents](feedback_sys_getenv_home_windows.md)
- [read.csv/type.convert makes all-numeric columns <integer> → bind_rows clash with character](feedback_readcsv_typeconvert_integer.md) — numeric tickers (JP 1976/5658) broke RReporting flex upload
- ["Stale" tests are mostly self-consistent date fixtures (pass forever); real fragility is live-data hangs + scratch scripts](feedback_stale_tests_are_live_data_not_dates.md) — 2026-06-14 audit; skip_if_offline/isIBAvailable guards over year-bumps

## Shiny
- [No priority= in observers — buggy ordering; one observer per trigger](feedback_no_shiny_priority.md)
- [RReporting standalone Rscript tests: filter()→stats::filter (mod:stats masks dplyr); qualify dplyr::filter](feedback_dplyr_filter_masked_standalone.md) — "column not found in filter()" is a harness artifact, not a regression
- [selectInput crashes on missing selectize-plugin-a11y — default selectize=FALSE](feedback_selectize_plugin_a11y.md)
- [shinytest2 AppDriver + app$get_logs() for silent client-side bugs](feedback_shinytest2_for_silent_bugs.md)
- [Snapshot DT selection at button press, not modal OK](feedback_snapshot_dt_selection_for_modals.md) — input$_rows_selected drifts; caused trade 721 leak
- [ggplotly() needs layout(autosize=TRUE)+config(responsive=TRUE) to fill container](feedback_ggplotly_autosize.md)
- [DT: sort currency-formatted cols by hidden base-ccy col + pin TOTAL via orderFixed.pre](reference_dt_basecurrency_sort_pin_row.md) — portf_datatable(), both account branches
- [fileInput's green "Upload complete" is a built-in label (transfer only), not custom text — add showNotification for processing](feedback_shiny_fileinput_upload_complete.md)
- [plotly mode="lines" connects in ROW order — sort by x before add_trace](feedback_plotly_lines_sort_by_x.md) — bind_rows(include_archived=TRUE) isn't chronological; jagged line is a plotting bug not a data bug
- [Historical-option / IV-history charts live in RPreTrade, NOT RReporting](reference_historical_option_module_in_rpretrade.md) — Tuser/studies/view/historicalOptionUI.R, wired via RPreTrade/R/historical_options.R

## Git Discipline
- [Re-run git status immediately before commit — pre-staged files ride along](feedback_git_status_before_commit.md)
- [Each app subdir is its OWN git repo with own remote](reference_app_subdirs_are_separate_repos.md) — root status won't show app edits; stage only task files
- [Unified RApplication repo policy — canonical "should I commit X?"](project_rapplication_repo_policy.md) — audit trail of closing commits in [project_todo_55_unified_repo_policy](project_todo_55_unified_repo_policy.md)
- [.gitignore edits resurface state — re-run git status, inspect new ?? files](feedback_gitignore_edit_resurfaces_state.md)
- [git add of tracked file in ignored dir exits 1 (data/mydb.sql) — use -f or separate](feedback_git_add_ignored_dir.md)
- [Re-survey state before executing stale TODO (snapshot ≠ ground truth)](feedback_resurvey_stale_todos.md)
- [Verify "NEVER X" policy assertions — check if reason still applies, surface & ask](feedback_verify_policy_assertions.md)
- [Unify-the-rule needs full per-repo audit (git ls-files + .gitignore)](feedback_unify_rule_audit_full_surface.md)
- [Verify sub-agent "orphaned/unused" claims with fresh Grep before destructive ops](feedback_verify_audit_claims.md)
- [Move DONE TODO items to trailing "✅ COMPLETED ITEMS" section](feedback_todo_done_to_completed_section.md)

## Tooling, Infra & Automation
- [Google Cloud trading-vm](gcloud-vm.md) — rtrading-basic/us-east1-b, auto-terminate 22:15 UTC; Shiny as `shiny` user; SQLite 3.37.2 no unistr()
- VM runs /analyze (RStudies subset); scanner runs LOCALLY (no scanner_results on VM); targeted deploy = scp LF + rsync
- [VM has NO option chain/strike cache consumer — /analyze never reads them; don't sync caches to VM](project_vm_no_option_cache_consumer.md)
- [data/mydb.sql dump: backup_database.R; CRLF full-file diff is NORMAL; pathspec commit](reference_mydb_sql_dump.md)
- [DB sync: Account/Trades have non-unique keys — never merge_db.ps1 -Update, use sync_db.ps1 Push](project_nonunique_keys.md)
- [Delete cached snapshot before re-running diff_db.ps1](feedback_verify_db_snapshot_fresh.md)
- [Run project slash command headlessly: claude -p "/cmd" --permission-mode acceptEdits](reference_claude_headless_slash_command.md)
- [Transcribe pipeline auto-summarizes; NewTrading is LOCAL-ONLY git (no remote, don't push)](project_transcribe_autosummary.md)
- [Claude config backup + /save & /sync commands](reference_claude_config_backup.md) — Aldohlys/claude-config-backup; user-level commands work in any project; memory auto-discovered per project key; rsync absent→cp-mirror; sync on "sync claude config backup" or /sync
- Windows Task Scheduler: .bat + cmd //c (schtasks flags intercepted by Git Bash); fsutil hardlink > mklink /H; StartWhenAvailable in XML for missed-task wake
- [Register schtasks from XML: use PowerShell tool (Bash mangles /create) + XML must be UTF-16](reference_schtasks_xml_registration.md) — RestartOnFailure + exit /b %ERRORLEVEL% for TWS-down retry
- [daily_portfolio_update subprocesses contend with parent DB writes — busy_timeout + honor exit codes](reference_daily_update_subprocess_db_contention.md) — exit-5 = SQLITE_BUSY crash, not real staleness; ROOT fix = busy_timeout in Tdata safe_db_connect (5.10.30); 15049ff was incomplete (patched wrong connection)
- [Don't pipe background Bash through tail — stream raw, tail file later](feedback_no_tail_on_background_bash.md)
- [Diagnose hung background build via side channels (tasklist / find -mmin / logs / git / renv)](feedback_diagnose_hung_background_build.md)
- [python -u when piping through tee — else log empty until exit](feedback_python_unbuffered_with_tee.md)
- [PowerShell: $bt=[char]96 for literal backticks in strings](feedback_powershell_backtick_quoting.md)
- [PowerShell here-string @'...'@ leaks @ into git messages when used in the Bash tool — use -F tempfile](feedback_powershell_herestring_in_bash_tool.md)
- [Don't write scratch scripts to C:\Users\aldoh\ — use project subdir](feedback_no_scratch_in_home.md)
- [Memory slug convention: filename ≡ name: ≡ [[link]], all snake_case; summary goes in description:](feedback_memory_slug_convention.md)
- [Economic events: Equals Money calendar works with WebFetch (ForexFactory 403)](reference_events_calendar.md)
- [iOS Mail opens HTML in QuickLook — JS doesn't fire; test via real Safari over HTTP](feedback_ios_mail_quicklook_no_js.md)
- [Offline PWA pattern for iPhone](reference_pwa_offline_pattern.md) — manifest+sw.js; bump CACHE key; tools/bs_calculator (TODO #66)
- [BSM Calculator design conventions](project_bs_calculator_design.md) — shared inputs above tabs; segmented selectors; Taylor PnL Risks tab
- [RPreTrade Position Analysis IV/skew projection model](project_position_analysis_iv_skew_conventions.md) — projected IV = entry + vol offset (all legs) + skew (OTM |delta|≤0.30, puts & calls); per-IV chart anchors to the ATM leg; 3 P/L charts = date/price/IV
