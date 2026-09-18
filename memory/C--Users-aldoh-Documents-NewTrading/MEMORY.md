# NewTrading Project Memory

## Project Purpose
- Trading strategy discussion & advisory, NOT code dev; Claude = options/stocks/futures advisor. Also SQLite trading-DB analysis.

## User Profile
- Swing trader, monthly income; 10-15 active trades; breakout + vol selling; broker IBKR. Past BOT/OFI/LTO analysis in DB.
- [Framework confidence map](user_framework_confidence_map.md) — pricing mechanical (high conf); sector/technical qualitative

## IBKR Account Structure
- Base CHF; multi-ccy USD/EUR/CHF/JPY; account U1804173
- [FX exposure](ibkr_fx_exposure.md) — net-exposure (Nt Lqdtn + FX Portfolio), FXCONV vs IDEALPRO, hedge policy (keep Nt Lqdtn USD 0–+6k CHF band, hedge market value not delta)
- [KRW conversion route](reference_krw_conversion_route.md) — no direct KRW/CHF at IBKR; go KRW → USD → CHF in two legs; per-ccy borrow rates (KRW ~7.5% vs JPY ~2.2%)
- [U25343478 FX hedge policy](project_u25_fx_hedge_policy.md) — fund each foreign holding in its own ccy at MARKET value; rebalance outside 90-110% coverage band; borrow rates; KRW exception
- [U25343478 margin cushion](reference_u25_margin_cushion.md) — illiquid foreign small caps get ~85% requirement; cushion = EL/NLV, IBKR alerts <10%, no margin calls; size from cushion NOT Buying Power

## Key Topics
- Greeks/LEAPs: vega/gamma ratios, put-sell vs buy-call (IVP), Delta vs P(ITM), fwd-price delta — `Greeks_ratios_and_LEAPS.md`
- BOT/planning: BS target grids, BPT validation — `Strategies/Breakouts/BOT_Comprehensive_Checklist.md`

## BOT Strategy (2026-03-26, revalidated 2026-08-27)
- Full ref (4 gates, S/BK scoring, regime, vehicle, exit, MR classifier): `Strategies/Breakouts/BOT_Comprehensive_Checklist.md` + `BOT_Quick_Reference.md` (bot_strategy_checklist.md was deleted)
- [Book design on 93 trades](project_bot_book_design.md) — USD 300 cap, ATR barbell confirmed, hold 2-4wk, F1/F2 entry factors, VRP rejected, MFE/MAE
- [Universe + scanner](project_bot_universe_scanner.md) — one merged CSV, `bot_scan_universe.py` -> dated XLSX in Trades/
- [indicators.R is the source of truth](reference_rstudies_indicators_source.md) — scoring.R no longer exists
- Momentum monitor `bot_momentum_monitor.py` (16h task, manual in Task Scheduler)
- [MA50→EMA switch](project_ma50_ema_switch.md) — trend gate now EMA50 (0223a0c); breadth/regime/backtests stay SMA
- [Timeframe missions](reference_timeframe_missions_bot.md) — macro=permission/size, weekly=room, daily=thesis+clock, intraday=trigger; capped by option life → time stop (Step 4, 6918d55)
- [Gate 4 static-median flaw](reference_gate4_static_median_flaw.md) — atr_med5y can't see a vol-regime change; GDX vs GDXJ + GIS worked cases; input to gate simplification
- [Trading-plan review](project_bot_trading_plan_review.md) — 2026-06-15 DONE: split LONG (`bot_strategy_trading_plan.md`)/SHORT (`bot_short_strategy_trading_plan.md`); pending EMA50 + short backtests

## Generated Reports
- `BOT_Comprehensive_Checklist.md` — BOT ref; `bot_analysis.md` — 2025 deep dive; `bot_breakout_analysis_2026.md` — DOW screener, TGT plan
- `breakout_5y_results.csv` (594 sig+fwd) / `breakout_backtest_5y.py`; `bear_market_check.py` — regime; `strategy_performance_report.md`; `backtest_results.md` — regime 2026-03-19

## RStudies Reports (RApplication/RStudies/reports/)
- macro_context: VIX/rates/DXY/sector/breadth/regime → HTML+DB; VIX≥25 banner. swing_scanner: 2-gate (technical+optionality), vol profile → HTML; Trade/Watch + LONG/SHORT badges. shared: cache.R, HTML helpers, ScannerUniverse (universe.R)
- Repo `Aldohlys/RStudies` (main); output `NewTrading/reports/`; `Desktop\run_scanner.bat`; scheduled `\RApplication\RunScanner` daily 09:00 CET

## Scanner Scoring (swing_scanner; scoring.R gone — see reference_rstudies_indicators_source.md)
- v5: 10 criteria→composite; TRADE≥7/WATCH 5-6/SKIP<5; optionality gate (IV30<40+IVP<60+VRP<0+Contango); room>1 ATR; \$500 long-only. Redesign: [project_swing_scanner_redesign](project_swing_scanner_redesign.md)

## Regime System (macro_context/scenarios.R)
- 3 regimes, 12 sigmoid signals + weekly COT + CPI/PPI. No BOT predictive power (2026-03-19) — kept for sector-flow scoring. [Detail](project_regime_backtest.md)
- COT positioning now AUTO-GENERATED weekly ([positioning.R automation](project_positioning_r_automation_todo.md) CLOSED) — `refresh_cot.R` + Saturday task + staleness banner; actor detail in `Reports/cot_actors_latest.csv`
- [COT trader categories](reference_cot_trader_categories.md) — legacy=disagg mapping, Commercial folds spreading, large spec ≠ managed money; [cotsignal API rejected](reference_cotsignal_api_rejected.md)

## Git Repos
- RApplication `Aldohlys/RApplication` (master) — DB, scripts, SQL dump. RStudies `Aldohlys/RStudies` (main) — reports. Tdata `Aldohlys/Tdata` (stable/prod) — R/Python IBKR TWS pkg
- Tuser [`Aldohlys/Tuser` is its OWN repo](reference_tuser_repo.md) (master) — nested in RApplication\Tuser, not a submodule; git status in RApplication misses it; uses CHANGELOG.md not change.log
- NewTrading `Aldohlys/NewTrading` (master) — this workspace; remote 2026-06-09 ([detail](project_newtrading_remote_todo.md)); [tracking policy](project_newtrading_repo_policy.md) — tracks docx/odt/xlsx/pdf/csv, ignores html/json/mp4/Discussions+textbooks

## Macro / Portfolio
- [AI equity-supply wave](project_ai_equity_supply_wave_watch.md) — 2026 mega-issuance = froth not funding crisis; trigger SpaceX day+1 vs \$135
- [Portfolio Review 2026-03-20](project_portfolio_review_20260320.md) — snapshot, CRST plan, QQQ; [CRST Distress](project_crst_distress.md) — profit warning, covenant risk, July interim
- [Carrefour long](project_carrefour_position.md) — 200sh, last add €16.17, target €21; [CA.PA deactivated](project_carrefour_thin_options.md) — thin chain removed; EU single-stocks need liquidity proof
- [ESTX50 hedge roll](project_estx50_hedge_roll_20260601.md) — Dec-26 5700/4600 live (US hedges closed); hold as crash insurance, 2027 roll decision 2026-11-13, 6.5%-OTM-at-live-spot rule
- [Daubasses Portfolio](project_daubasses_portfolio.md) — 6 positions, weekly review, `Strategies/Daubasses/`
- [EM/China](user_em_china_view.md) — tradeable-not-investible; onshore/offshore split (SOLD FXC @94.58 → building CNYA); captive domestic bid vs foreign flows

## Open Work Programs
- [Claude API integration](project_claude_api_integration_todo.md) — SDK: summarize Journal, classify trades, extract Daubasses PDFs; /claude-api, Batches
- [Vol ATR band guardrails](project_vol_atr_band_guardrails_todo.md) — clamp xmoveUI horizon→20 + conf→90; effective-n + tail-support guard in atr_move.R
- [Vol module refactor](project_vol_module_refactor.md) — ATR expected-move lens; Phase 1 DONE (v5.10.21), P2-5 pending
- [/analyze prefer-monthly](project_analyze_prefer_monthly_expiry.md) uncommitted; [on VM](project_analyze_on_vm_deployment.md) pending env/restore/smoke; [improvements](project_analyze_improvements_20260527.md) cheap_score/spread-EV/BS-grid; [spread-module fetch](project_analyze_spread_module_fetch_todo.md) once-per-expiry; [OI cap OTM](project_oi_cap_otm_filter.md); [R port](project_analyze_r_port.md) 4 gaps
- [XLF screen](project_xlf_breakout_screen_20260608.md) — C GREEN, financials=spread; [XLV screen](project_xlv_breakout_screen_20260608.md) CLOSED; [Refiner screen](project_refiner_bot_screen_20260606.md) DK/DINO pending
- [Swing scanner redesign](project_swing_scanner_redesign.md) three-axis; [two-timescales](project_swing_scanner_two_timescales_todo.md) v6; [methodology](project_scanner_methodology_todo.md) RS overlay+patterns; [BOT R:R calib](project_rr_calibration_result.md) R:R_min=0.5
- [build_package renv-prune bug](project_build_package_renv_prune_todo.md) deploy via R CMD INSTALL; [shared config.yml #58](project_shared_config_yml_todo.md); [track .claude/commands](project_track_claude_commands_todo.md); [NewTrading remote](project_newtrading_remote_todo.md)
- [Condition-based alerts](project_alerts_condition_based_todo.md) vs live IBKR

## Refs — Execution & Trade Management
- [Execution ladder](feedback_trade_execution.md) (target/acceptable/red-line/hard-no, leg-level mid); [Buy STP LMT: LMT≥STP](reference_buy_stop_limit_direction.md); [Vert exit 80%](reference_vertical_spread_exit_80pct.md) (last 20% needs 0 DTE); [Exit winners at pre-set limit](feedback_hard_to_exit_winners.md) (90%=success)
- [Don't switch frameworks mid-trade](feedback_dont_switch_frameworks_midtrade.md); [Don't add to losers](feedback_losing_trade_decisions.md) (path-vs-direction); [Hedge wing-sale into vol spike](feedback_hedge_wing_sale_timing.md); [Re-check strike at live spot](feedback_recheck_strike_at_execution_spot.md) if >~1%; [Pyramid per-lot R/R](feedback_pyramid_per_lot_rr_audit.md)

## Refs — Risk & Sizing
- [Size % of book first](feedback_size_risk_before_flagging_materiality.md) (-30% on 4%=~-1%); [Portfolio DD ≠ index move](feedback_portfolio_drawdown_vs_index_move.md); [Net Greeks first on spreads](feedback_spread_greeks_monitoring.md); [Verticals discard vol edge](feedback_structure_selection_vs_vol_thesis.md) (≥2 vol comps→flag)
- [Directional dashboard minimum](feedback_directional_dashboard_minimum.md) (drop IV/smile); [Correlation regime+outliers](feedback_correlation_regime_and_outliers.md) (hedges=crash insurance); [Counterparty/positioning lens](feedback_counterparty_positioning_lens.md) (execution→exit, positioning→entry)

## Refs — Strategy Concepts
- [Breakout IV direction](feedback_breakout_iv_direction.md) — no earnings-style crush; puts +, momentum calls +, index calls flat/-; vol_offset=0 = sticky-strike
- [Price-action R:R gate](feedback_price_action_rr_gate.md) — swing levels + OPTION payoff at target BEFORE trusting rr_meas (it has no clock); [First-touch barrier test](reference_first_touch_barrier_test.md) — method + AR/SLB/SCCO calib; [Name continuation character](reference_bot_name_continuation_character.md) — SLB/EQT mean-revert, SCCO continues
- [JHEQX/JPM collar rule](reference_jheqx_collar_structure.md) — puts fixed -5%/-20%, call solved zero-cost (~+3.5-5.5%), quarter-end expiry; SPY replication on 100sh
- [Classify breakout vs MR first](feedback_classify_breakout_vs_meanreversion.md); [No MR framing](feedback_no_mean_reversion_framing.md) (breakout/continuation+vol selling only); [ATR→move multiples](reference_atr_move_multiples.md) (1.5×/4×, √N); [ATR band limits](reference_atr_empirical_band_limits.md) (trust 70-80%, distrust ≥90%); [Box spread cash parking](reference_box_spread_cash_parking.md) (SPX/XSP not SPY; FX-hedge→home rate CIP)

## Refs — Instruments & Market Mechanics
- [CL opts mult=1000](reference_cl_options_multiplier.md) (MCL 100/ES 50/GC 100); [OESX/EUREX](reference_oesx_eurex_option_mechanics.md) (ESTX50 ×10 EUR/pt, Dec liquid, IV null→BS); [SMI/OSMI](reference_smi_osmi_option_mechanics.md) (CHF 10/pt, European cash-settled, thin far strikes; single-stock=American); [WTI cal-spread bands](reference_wti_calendar_spread_calibration.md); [Futures expiry-week](reference_futures_expiry_week_dynamics.md) (use first non-expiring); [Verify futures month](feedback_verify_futures_contract_month.md) (not =F front)
- [CFD cash=synth perpetual](reference_cfd_cash_synthetic_perpetual.md); [Futures daily MTM](reference_futures_daily_mtm.md); [TWS term structure](reference_tws_futures_term_structure.md); [TWS no candle-close trigger](reference_tws_candle_close_trigger.md); [Closed-market marks stale](reference_closed_market_option_marks_stale.md); [Ex-div mechanical drop](reference_exdiv_mechanical_drop_diagnostic.md); [HV30 window artifact](reference_hv30_window_artifact.md)
- [AR = NGL/LPG exporter + hedge book](reference_ar_antero_name_structure.md) — 1/3 revenue seaborne LPG not Henry Hub; hedges decouple realised from spot; single-basin Marcellus post-HG
- [DXY=DX-Y.NYB](reference_dxy_yahoo.md); [SMH comp](reference_smh_composition_2026.md) (NVDA 20%→8%); [REMX comp](reference_remx_composition.md) (~30-40% lithium; pure-RE MP/LYC/REXC); [XOP thin weeklies](reference_xop_thin_chain.md) (use monthly); [Check OI by expiry](feedback_check_oi_by_expiry_for_outright.md)

## Refs — /analyze
- [Data-only/neutral](feedback_analyze_data_only.md) ([nuance](feedback_data_only_doesnt_strip_synthesis.md)); [Run-all A→E + live fallback](feedback_analyze_live_data_fallback.md) (OI genericTickList=101); [Vehicle rule](feedback_analyze_vehicle_rule.md) (<\$150&IVP<60 outright / >\$150|IVP>60 spread / <\$10 stock); [Off-universe→yfinance](feedback_analyze_off_universe_fallback.md)
- [Lot=full structure \$300 cap](feedback_analyze_lot_definition.md); [Two expiries ~30 & ~50-60 DTE](feedback_analyze_two_expiries.md); [SKIP=fade signal](feedback_analyze_skip_as_fade_signal.md); [TWS-down degraded](reference_analyze_tws_down_degraded.md); [Redesign 2026-05-12](project_analyze_redesign_2026_05.md); [Sector RS defs](reference_sector_rs_definition.md); [Audit direction-blindness for shorts](feedback_audit_direction_blindness_when_adding_shorts.md)

## Refs — Tdata / IBKR Data
- [Tdata vs IBKR MCP](reference_tdata_vs_ibkr_mcp.md) (prefer Tdata via r-reticulate; MCP fallback); [Entitlements](reference_ibkr_data_entitlements.md) (no EBS/IBIS/SBF hist; delayed reqMarketDataType(4); thin→US cousin); [Option-fetch internals](reference_tdata_option_fetch_internals.md); [Spread module](reference_tdata_spread_module.md); [force_refresh=True](feedback_tdata_force_refresh.md); [Rebuild→restart R](feedback_tdata_rebuild_restart_r.md); [Install topology](reference_tdata_install_topology.md)
- [getStoredMetrics has NO age check](reference_getstoredmetrics_no_freshness.md) — newest Prices row however old (SMH 84d stale, 638 vs 557); verify datetime, else getStockPrice(close=TRUE)
- [tdata_py = active binding](reference_tdata_py_active_binding.md) — use `Tdata::tdata_py`, never `reticulate::import("tdata_py")` (skips sys.path init)
- [IR utils getLastRate(ccy,DTE)](reference_tdata_interest_rate_utils.md); [IR refresh system](reference_tdata_ir_refresh_system.md); [VRP two forms](reference_tdata_vrp_formula.md); [Vol pctile helpers](reference_tdata_vol_percentile_helpers.md); [Solve IV when null](feedback_iv_solve_when_tws_returns_null.md); [U1804173.IV=underlying iv30](reference_u1804173_iv_column.md); [Trades schema migrated](reference_trades_right_column_sparse.md) (parse Instrument; group by TradeNr)

## Refs — Environment & Tooling
- [box::use caches per R session](reference_box_module_session_cache.md) — module edits need `box::purge_cache()` or an R restart; runApp alone re-sources only app.R
- [R-4.4.3 env](reference_r_environment.md) (lib RLibrary); [Python/conda launch](reference_python_conda_launch.md); [PS `${Var}:` scope](reference_powershell_variable_colon.md); [gcloud scp needs dest dir](reference_pscp_recursive_destdir.md); [MD→PDF pipeline](reference_markdown_pdf_pipeline.md); [transcribe.py atomic](reference_transcribe_pipeline_atomic.md); [change.log convention](reference_changelog_convention.md); [renv safe to track](reference_renv_safe_to_track.md); [CFTC COT URLs](reference_cftc_cot_urls.md); [SQLite reads auto-approved](feedback_no_sqlite_prompts.md)

## Refs — Working Style & Process
- [display_error_message() = bare stop()](reference_display_error_message_is_stop.md) — shows nothing; never in a tryCatch handler (double-throw → raw traceback). Use showNotification / validate+need
- [Legend = definitions, rationale in the doc](feedback_legend_definitions_not_rationale.md); [No static option judgements in screens](feedback_no_static_option_judgements_in_screens.md); [No LaTeX math; escape EVERY \$](feedback_no_latex_math_blocks.md); [Concise, no repeated phrases](feedback_concise_no_repeated_phrases.md); [Minimal bold (leading labels only)](feedback_minimal_bold_formatting.md); [Plain language, expand jargon](feedback_plain_language_no_jargon.md); [No tautological signal rows](feedback_no_tautological_signal_rows.md); [Size analytics to ~40d horizon](feedback_size_analytics_to_trade_horizon.md)
- [Anchor net claims on reconciliation](feedback_anchor_net_claims_on_reconciliation.md); [Verify codebase facts wide (grep all repos)](feedback_verify_before_claiming_codebase_facts.md); [Verify "NEVER X" assertions](feedback_verify_policy_assertions.md); [Re-survey stale "execute" TODOs](feedback_resurvey_stale_todos.md); [git status after .gitignore edit](feedback_gitignore_edit_resurfaces_state.md); [Unify-rule needs full audit](feedback_unify_rule_audit_full_surface.md)
- [Map all fetch sites before optimizing](feedback_map_all_fetch_sites_before_optimizing.md); [V5 scanner lessons](feedback_v5_implementation_lessons.md); [RApplication repo policy](project_rapplication_repo_policy.md) (canonical "commit X?"); *Global memory:* No approval prompts, Simplicity over complexity, User Trading Framework

## Database
- `C:\Users\aldoh\Documents\RApplication\data\mydb.db`; 29+ tables, 2039 trades, 29 strategies. ScannerUniverse ~130 symbols (119 scanner + 10 ETFs + macro)
- [saveTrades overwrite bug](project_savetrades_overwrite_bug.md) — now freshness-guarded; [Daily portfolio hardening](project_daily_portfolio_hardening.md) — lift TWS managed-accounts filter into Tdata; PY timeout→R error
