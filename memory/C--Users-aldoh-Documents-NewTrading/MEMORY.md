# NewTrading Project Memory

## Purpose / User
- Trading advisory (options/stocks/futures) + SQLite DB analysis, not code dev. Swing trader, 10-15 trades, breakout + vol selling, IBKR.
- [Irish UCITS](user_prefers_irish_ucits.md) not US funds; [Framework confidence](user_framework_confidence_map.md) pricing high, sector/technical qualitative; [EM/China](user_em_china_view.md) tradeable-not-investible, FXC sold → CNYA

## IBKR Accounts (base CHF, U1804173)
- [FX exposure](ibkr_fx_exposure.md) Nt Lqdtn USD 0–+6k band, hedge market value; [KRW route](reference_krw_conversion_route.md) KRW→USD→CHF
- [U25 FX hedge](project_u25_fx_hedge_policy.md) own-ccy funding 90-110%; [U25 margin cushion](reference_u25_margin_cushion.md) size from EL/NLV not BP
- [Transfers need CashFlow in BOTH accounts](reference_internal_transfer_cashflows.md); [Data-integrity check](reference_data_integrity_check.md) Sat 09:00, add check per bug class
- [FX table outliers](reference_fx_table_outliers.md) GBP/JPY double inversion, 5% guard; [Dividend import](reference_ibkr_dividend_import.md) statement CSV, net PAID; [Import UI placement](feedback_import_ui_placement.md) Tuser Trade tab + RReporting

## BOT Strategy
- Ref: `Strategies/Breakouts/BOT_Comprehensive_Checklist.md` + `BOT_Quick_Reference.md`; Greeks/LEAPs `Greeks_ratios_and_LEAPS.md`
- [P&L greeks 116 trades](project_bot_pnl_greeks_decomposition.md) spot R² .89, theta = losers; [Book design 93 trades](project_bot_book_design.md) USD 300 cap, ATR barbell, 2-4wk; [Universe + scanner](project_bot_universe_scanner.md) bot_daily CSV → xlsx
- [indicators.R = source of truth](reference_rstudies_indicators_source.md); momentum monitor `bot_momentum_monitor.py` (+ .md doc, 17:00 task); [Alert e-mail window](reference_alerts_email_date_window.md) AlertDate next weekday or never sent; [EMA50 trend gate](project_ma50_ema_switch.md)
- [Timeframe missions](reference_timeframe_missions_bot.md); [Gate 4 static-median flaw](reference_gate4_static_median_flaw.md); [Trading-plan review](project_bot_trading_plan_review.md) LONG/SHORT split
- Reports: `bot_analysis.md`, `bot_breakout_analysis_2026.md`, `breakout_5y_results.csv`, `backtest_results.md`, `strategy_performance_report.md`

## RStudies Reports (repo Aldohlys/RStudies main → NewTrading/reports/, task RunScanner 09:00)
- macro_context (VIX/rates/DXY/breadth/regime); shared cache.R, universe.R
- [Intermarket layer](project_intermarket_macro_scenarios.md) 11 scenarios, panels, sector map, COT section 11; open #2 stress ladder/#5/#7/#9/#12; [Scenario age rule](feedback_scenario_persistence.md); [China onshore/offshore](reference_china_index_onshore_offshore.md) ^HSCE + 000001.SS, no CSI 300 on Yahoo
- [Layout](feedback_macro_context_layout.md) last-session strip top of 00, sections collapsible (00/09 open), panels collapsed, window label first; [Yahoo STOXX50E close mismatch](reference_yahoo_stoxx50e_close_mismatch.md) 10-02 +1.02% vs 1.21%
- [Ceresna 5 tenets](reference_ceresna_macro_tenets.md) present/value(warning)/liquidity 2-way/consensus/actors; advisory excluded; proposals #1-25 in wherestrade_20261091_summary.md (addendum #13-25); MO 10-02 vs scanner in macro_outlook_20261002_summary.md
- [BOT group benchmark + grp_rank](project_bot_group_benchmark.md) S3 vs anchor/peer median; rotation +0.30 ATR
- [Swing scanner retired 10-05](project_swing_scanner_retired.md) group rank → BOT grp_rank; [old redesign](project_swing_scanner_redesign.md) history only
- Regime: 3 regimes, 12 signals, no BOT predictive power ([detail](project_regime_backtest.md)); COT auto via refresh_cot.R, commit positioning.R by hand ([closed](project_positioning_r_automation_todo.md)); [COT categories](reference_cot_trader_categories.md); [COTSignal API](reference_cotsignal_api_rejected.md) view rebuilt from CFTC (section 11, cot_positioning_latest.csv); API = cross-check only

## Git Repos
- RApplication (master) DB/scripts; RPreTrade on `refactor/server-modularization-hybrid`; Tdata (stable/prod); [Tuser own repo](reference_tuser_repo.md) CHANGELOG.md
- NewTrading (master) [remote](project_newtrading_remote_todo.md), [tracking policy](project_newtrading_repo_policy.md) docx/xlsx/pdf/csv yes, html/json no

## Macro / Portfolio
- [AI equity-supply wave](project_ai_equity_supply_wave_watch.md) SpaceX trigger \$135; [Portfolio review 03-20](project_portfolio_review_20260320.md); [CRST distress](project_crst_distress.md)
- [Sandoz](project_sandoz_coverage.md) ~15x EV/EBITDA priced; [Galderma](project_galderma_coverage.md) Nemluvio, Q1-27 reset; [DSPF](project_dspf_coverage.md) agio, enter at raise
- [NB EMD vs VEMT](project_nb_emd_hard_currency_coverage.md); [Carrefour](project_carrefour_position.md) target €21; [CA.PA thin chain](project_carrefour_thin_options.md)
- [ESTX50 hedge](project_estx50_hedge_roll_20260601.md) Dec-26 5700/4600, 11-13 size decision; [Gonet hedge 09-30](project_gonet_hedge_report_20260930.md) GLD Mar-27 350/310; [Daubasses](project_daubasses_portfolio.md)

## Open Work Programs
- [Claude API](project_claude_api_integration_todo.md); [Vol ATR guardrails](project_vol_atr_band_guardrails_todo.md); [Vol refactor](project_vol_module_refactor.md) P2-5 pending
- /analyze: [prefer-monthly](project_analyze_prefer_monthly_expiry.md), [VM](project_analyze_on_vm_deployment.md), [improvements](project_analyze_improvements_20260527.md), [spread fetch](project_analyze_spread_module_fetch_todo.md), [OI cap](project_oi_cap_otm_filter.md), [R port](project_analyze_r_port.md)
- Screens: [XLF](project_xlf_breakout_screen_20260608.md), [XLV](project_xlv_breakout_screen_20260608.md) closed, [Refiners](project_refiner_bot_screen_20260606.md) DK/DINO
- [methodology](project_scanner_methodology_todo.md); [R:R calib](project_rr_calibration_result.md) 0.5
- [renv-prune bug](project_build_package_renv_prune_todo.md); [config.yml #58](project_shared_config_yml_todo.md); [track commands](project_track_claude_commands_todo.md); [Condition alerts](project_alerts_condition_based_todo.md) + [e-mail window](reference_alerts_email_date_window.md)

## Refs — Execution & Risk
- [Execution ladder](feedback_trade_execution.md); [STP LMT ≥ STP](reference_buy_stop_limit_direction.md); [Vert exit 80%](reference_vertical_spread_exit_80pct.md); [Exit at pre-set limit](feedback_hard_to_exit_winners.md)
- [No framework switch mid-trade](feedback_dont_switch_frameworks_midtrade.md); [Don't add to losers](feedback_losing_trade_decisions.md); [Wing-sale into vol spike](feedback_hedge_wing_sale_timing.md); [Re-check strike at spot](feedback_recheck_strike_at_execution_spot.md); [Pyramid per-lot R/R](feedback_pyramid_per_lot_rr_audit.md)
- [Hedge: size before strike](feedback_hedge_size_before_strike.md); [Size % of book first](feedback_size_risk_before_flagging_materiality.md); [Portfolio DD ≠ index](feedback_portfolio_drawdown_vs_index_move.md); [Net Greeks first](feedback_spread_greeks_monitoring.md); [Verticals discard vol edge](feedback_structure_selection_vs_vol_thesis.md)
- [Directional dashboard](feedback_directional_dashboard_minimum.md); [Correlation regime](feedback_correlation_regime_and_outliers.md); [Positioning lens](feedback_counterparty_positioning_lens.md)

## Refs — Strategy Concepts
- [Breakout IV direction](feedback_breakout_iv_direction.md); [Price-action R:R gate](feedback_price_action_rr_gate.md); [First-touch barrier](reference_first_touch_barrier_test.md); [Continuation character](reference_bot_name_continuation_character.md) SLB/EQT revert
- [JHEQX collar](reference_jheqx_collar_structure.md); [Classify BO vs MR](feedback_classify_breakout_vs_meanreversion.md); [No MR framing](feedback_no_mean_reversion_framing.md); [ATR multiples](reference_atr_move_multiples.md); [ATR band limits](reference_atr_empirical_band_limits.md); [Box spread](reference_box_spread_cash_parking.md)

## Refs — Instruments
- [CL mult 1000](reference_cl_options_multiplier.md); [OESX](reference_oesx_eurex_option_mechanics.md); [OSMI](reference_smi_osmi_option_mechanics.md); [WTI cal bands](reference_wti_calendar_spread_calibration.md); [Expiry week](reference_futures_expiry_week_dynamics.md); [Verify futures month](feedback_verify_futures_contract_month.md); [Rebalance volume](reference_index_rebalance_volume_signature.md)
- [CFD perpetual](reference_cfd_cash_synthetic_perpetual.md); [Futures MTM](reference_futures_daily_mtm.md); [TWS term structure](reference_tws_futures_term_structure.md); [No candle-close trigger](reference_tws_candle_close_trigger.md); [Closed-market marks](reference_closed_market_option_marks_stale.md); [Ex-div drop](reference_exdiv_mechanical_drop_diagnostic.md); [HV30 artifact](reference_hv30_window_artifact.md)
- [AR structure](reference_ar_antero_name_structure.md); [DXY](reference_dxy_yahoo.md); [SMH](reference_smh_composition_2026.md); [REMX](reference_remx_composition.md); [Uranium tickers](reference_uranium_tickers.md); [XOP thin](reference_xop_thin_chain.md); [OI by expiry](feedback_check_oi_by_expiry_for_outright.md)

## Refs — /analyze
- [Data-only](feedback_analyze_data_only.md) ([nuance](feedback_data_only_doesnt_strip_synthesis.md)); [Live fallback](feedback_analyze_live_data_fallback.md); [Vehicle rule](feedback_analyze_vehicle_rule.md); [Off-universe](feedback_analyze_off_universe_fallback.md); [Lot \$300](feedback_analyze_lot_definition.md)
- [Two expiries](feedback_analyze_two_expiries.md); [SKIP=fade](feedback_analyze_skip_as_fade_signal.md); [TWS-down](reference_analyze_tws_down_degraded.md); [Redesign](project_analyze_redesign_2026_05.md); [Sector RS](reference_sector_rs_definition.md); [Short direction audit](feedback_audit_direction_blindness_when_adding_shorts.md)

## Refs — Tdata / IBKR Data
- [Tdata vs MCP](reference_tdata_vs_ibkr_mcp.md); [Entitlements](reference_ibkr_data_entitlements.md); [Option fetch](reference_tdata_option_fetch_internals.md); [getValue batch -1](reference_tdata_getvalue_batch_all_or_nothing.md) one bad contract unprices all; [Spread module](reference_tdata_spread_module.md); [force_refresh](feedback_tdata_force_refresh.md); [Rebuild→restart R](feedback_tdata_rebuild_restart_r.md); [Install](reference_tdata_install_topology.md)
- [Yahoo FX dated 1 day early in BST](reference_yahoo_fx_calendar_shift.md) as-of join for FX ratios; [getStoredMetrics no age check](reference_getstoredmetrics_no_freshness.md); [tdata_py binding](reference_tdata_py_active_binding.md); [getLastRate](reference_tdata_interest_rate_utils.md); [IR refresh](reference_tdata_ir_refresh_system.md); [VRP](reference_tdata_vrp_formula.md); [Vol pctile](reference_tdata_vol_percentile_helpers.md); [Solve IV](feedback_iv_solve_when_tws_returns_null.md); [IV column](reference_u1804173_iv_column.md); [Trades schema](reference_trades_right_column_sparse.md)

## Refs — Environment
- [box::use cache](reference_box_module_session_cache.md); [R env](reference_r_environment.md); [conda](reference_python_conda_launch.md); [PS scope](reference_powershell_variable_colon.md); [gcloud scp](reference_pscp_recursive_destdir.md); [MD→PDF](reference_markdown_pdf_pipeline.md); [transcribe atomic, ~2.5-3.5× audio, AC sleep was 5 min](reference_transcribe_pipeline_atomic.md); [BPT archive→Vimeo IDs, yt-dlp titles, NAME arg, FIFO queue, sequential chain](reference_bigpicture_archive_vimeo.md); [change.log](reference_changelog_convention.md); [renv track](reference_renv_safe_to_track.md); [COT URLs](reference_cftc_cot_urls.md); [SQLite auto-approved](feedback_no_sqlite_prompts.md)

## Refs — Working Style
- [display_error_message = stop()](reference_display_error_message_is_stop.md); [Legend = definitions](feedback_legend_definitions_not_rationale.md); [No static option judgements](feedback_no_static_option_judgements_in_screens.md); [Escape every \$](feedback_no_latex_math_blocks.md); [Concise](feedback_concise_no_repeated_phrases.md); [Minimal bold](feedback_minimal_bold_formatting.md); [Plain language](feedback_plain_language_no_jargon.md); [No tautological rows](feedback_no_tautological_signal_rows.md); [~40d horizon](feedback_size_analytics_to_trade_horizon.md)
- [Anchor on reconciliation](feedback_anchor_net_claims_on_reconciliation.md); [Intraday moves: pull OHLC](feedback_verify_price_move_vs_close.md); [Grep all repos](feedback_verify_before_claiming_codebase_facts.md); [Install time vs run time](feedback_check_install_time_vs_run_time.md) read run log first; [Check before 'not set'](feedback_check_memory_before_absence_claims.md); [Verify NEVER X](feedback_verify_policy_assertions.md); [Re-survey TODOs](feedback_resurvey_stale_todos.md); [.gitignore edit](feedback_gitignore_edit_resurfaces_state.md); [Unify-rule audit](feedback_unify_rule_audit_full_surface.md)
- [Payload before open(w)](feedback_build_payload_before_open_w.md); [Map fetch sites](feedback_map_all_fetch_sites_before_optimizing.md); [V5 lessons](feedback_v5_implementation_lessons.md); [RApplication repo policy](project_rapplication_repo_policy.md) mydb.sql LFS weekly; [Commit with pathspec](feedback_commit_with_pathspec.md) takes working tree, check ls-files after rm

## Gonet Account (CHF base)
- [CSV bookkeeping](reference_gonet_csv_bookkeeping.md) FX/fee rows, DSPF SMART+delayed routing; [Movements + positions export](reference_gonet_movements_statement.md) value date, net WHT, Liquidités = cash check; [History USD→CHF](reference_gonet_account_history.md) 2025-10-27 switch; [NLV attribution recipe](reference_gonet_nlv_attribution_method.md) native-ccy snapshots, FX from cash rows
- [Ticker rename trap](reference_ticker_rename_trap.md) ROG→RO; [Tuser WeeklyunPnL](reference_tuser_weekly_unpnl.md) mean gap vs last 5 sessions; [Dividend reconciliation](project_gonet_dividend_reconciliation.md) FR WHT email, W-8BEN open; [FR loyalty ISIN](reference_loreal_loyalty_bonus.md); [Theoretical WHT](feedback_theoretical_wht_is_entitlement.md)

## Database
- `RApplication\data\mydb.db`; ScannerUniverse ~130; [saveTrades guard](project_savetrades_overwrite_bug.md); [Portfolio hardening](project_daily_portfolio_hardening.md)
