# NewTrading Project Memory

## Project Purpose
- Trading strategy discussion & advisory, NOT code development; Claude = options/stocks/futures advisor
- Also SQLite trading-DB data analysis

## User Profile
- Swing trader, monthly income; 10-15 active trades; breakout + vol selling; broker IBKR
- Past BOT/OFI/LTO strategy analysis in DB
- [Framework confidence map](user_framework_confidence_map.md) — options pricing mechanical (high conf); sector/technical qualitative; calibrate tone

## IBKR Account Structure
- Base CHF; multi-ccy USD/EUR/CHF/JPY; account U1804173
- `ibkr_fx_exposure.md` — FX Cash mechanics, net-exposure (Nt Lqdtn + FX Portfolio), FXCONV vs IDEALPRO, hedge policy (keep Nt Lqdtn USD 0–+6k CHF band, hedge market value not delta)

## Key Topics Discussed
- **Greeks/LEAPs**: vega/gamma ratios, LEAP put-sell vs buy-call (IVP-driven), Delta vs P(ITM) (d1/d2), fwd-price delta — `Greeks_ratios_and_LEAPS.md` (+PDF)
- **BOT/planning**: BS target grids, BPT validation, 2026-03-26 formalization — `Strategies/Breakouts/bot_strategy_checklist.md`

## BOT Strategy (2026-03-26)
- Full reference (4 gates, S/BK scoring, regime, vehicle, exit, 5y backtest 819sig/56%WR/RR1.25, MR classifier): `Strategies/Breakouts/bot_strategy_checklist.md`
- Impl: `score_breakout()` in swing_scanner/scoring.R + BOT section in HTML
- Momentum monitor: `bot_momentum_monitor.py`; 16h task, create manually in Task Scheduler
- [MA50→EMA switch](project_ma50_ema_switch.md) — BOT trend gate now EMA50 (commit 0223a0c); breadth/regime/backtests stay SMA
- [Timeframe missions (entry)](reference_timeframe_missions_bot.md) — macro=permission/size, weekly=room+not-fighting, daily=thesis+clock, intraday=trigger; capped by 2-4wk option life; justifies time stop. Checklist Step 4 = time stop as vehicle rule (commit 6918d55); stock uses capital-efficiency stop
- [Trading-plan review](project_bot_trading_plan_review.md) — 2026-06-15 DONE: punch-list addressed + split into LONG (`bot_strategy_trading_plan.md`) and SHORT (`bot_short_strategy_trading_plan.md`) plans; multi-timeframe added to Gate 2; pending: EMA50 backtest re-run + short backtest

## Generated Reports
- `bot_strategy_checklist.md` — BOT complete reference (gates, vehicle, exit, benchmarks)
- `breakout_5y_results.csv` — 594 signals + fwd returns (5y); `breakout_backtest_5y.py` — backtest script (Yahoo, ScannerUniverse)
- `bear_market_check.py` — regime analysis (SPX DD, VIX, sector)
- `strategy_performance_report.md` — strategy performance; `bot_analysis.md` — BOT deep dive (2025)
- `bot_breakout_analysis_2026.md` — comprehensive BOT analysis, DOW screener, TGT plan
- `Greeks_ratios_and_LEAPS.md`/`.pdf` — Greeks/LEAPs export; `backtest_results.md` — regime backtest (2026-03-19)

## RStudies Reports (RApplication/RStudies/reports/)
- macro_context: VIX, rates, DXY/commodities, sector mismatches, breadth, regime → HTML+DB; VIX≥25 banner
- swing_scanner: 2-gate (technical+optionality), vol profile, history → HTML; Trade/Watch + LONG/SHORT badges
- shared: cache.R, HTML helpers, ScannerUniverse (universe.R)
- Repo `Aldohlys/RStudies` (main); output `NewTrading/reports/`; run `Desktop\run_scanner.bat`; scheduled `\RApplication\RunScanner` daily 09:00 CET

## Scanner Scoring (swing_scanner/scoring.R)
- v5: 10 technical criteria→composite; TRADE≥7/WATCH 5-6/SKIP<5; optionality gate (IV30<40+IVP<60+VRP<0+Contango); room>1 ATR; $500 long-only. Redesign: project_swing_scanner_redesign.md

## Regime System (macro_context/scenarios.R)
- 3 regimes from 12 sigmoid signals + weekly COT + CPI/PPI. Backtested 2026-03-19: no BOT predictive power — kept for sector-flow scoring only. [Detail](project_regime_backtest.md)

## Git Repos
- RApplication `Aldohlys/RApplication` (master) — DB, scripts, SQL dump
- RStudies `Aldohlys/RStudies` (main) — reports/dashboards
- Tdata `Aldohlys/Tdata` (stable/prod) — R/Python IBKR TWS pkg
- NewTrading `Aldohlys/NewTrading` (master) — this workspace; remote wired 2026-06-09 ([detail](project_newtrading_remote_todo.md)); [tracking policy](project_newtrading_repo_policy.md) — tracks docx/odt/xlsx/pdf/csv, ignores html/json/mp4/Discussions + 5 big textbooks

## Macro / Market Watch
- [AI equity-supply wave](project_ai_equity_supply_wave_watch.md) — 2026 mega-issuance (SpaceX/Alphabet/Anthropic/OpenAI) = froth/supply-indigestion not funding crisis; live trigger SpaceX day+1 vs \$135

## Active Portfolio Review
- [Portfolio Review 20/03/2026](project_portfolio_review_20260320.md) — snapshot, CRST plan, QQQ, actions
- [CRST Distress 2026-04-20](project_crst_distress.md) — profit warning, covenant risk, triggers, July interim
- [Carrefour long 2026-05-26](project_carrefour_position.md) — 200sh, last add €16.17, target €21, small size
- [ESTX50 hedge roll](project_estx50_hedge_roll_20260601.md) — EXEC 2026-06-02: closed Sep 5200P, opened Dec 5700/4600 @125 (823EUR/lot); GOnet partial cover; SPY 660P parked (id=61)

## Open Work Programs
- [Claude API integration](project_claude_api_integration_todo.md) — add Anthropic SDK calls to scripts: summarize Journal entries, classify trades, extract from Daubasses PDFs + transcripts, generate report text; use /claude-api skill, Batches for bulk
- [Vol ATR band guardrails](project_vol_atr_band_guardrails_todo.md) — clamp xmoveUI horizon max→20 + conf max→90 (app, now); effective-n flag + tail-support guard in Tdata atr_move.R (next rebuild)
- [/analyze prefer-monthly expiry](project_analyze_prefer_monthly_expiry.md) — `.pick_expiry_for_dte` prefers 3rd-Fri monthly over weeklies; uncommitted in RStudies
- [XLF financials screen](project_xlf_breakout_screen_20260608.md) — C only GREEN; financials = spread-not-call (MID beta); BX only long-call; `xlf_screen.py`
- [XLV healthcare screen](project_xlv_breakout_screen_20260608.md) — CLOSED no trade: LLY/UNH extended, JNJ low-beta, ABBV mid IVP 57 + priced out; `xlv_screen.py`
- [Refiner BOT screen](project_refiner_bot_screen_20260606.md) — 7 refiners; PARR rejected; DK/DINO workup pending
- [build_package.R renv-prune bug](project_build_package_renv_prune_todo.md) — quick_build prunes devtools toolchain → builds fail; deploy via R CMD INSTALL
- [Swing scanner two-timescales](project_swing_scanner_two_timescales_todo.md) — split slow pool (weekly) from daily trigger; likely v6
- [Track .claude/commands in git](project_track_claude_commands_todo.md) — slash-command specs gitignored; un-ignore
- [Scanner methodology TODO](project_scanner_methodology_todo.md) — sector RS overlay + pattern catalog
- [Swing Scanner redesign](project_swing_scanner_redesign.md) — three-axis Pull/Cheap/Setup-RR rewrite; order locked
- [BOT R:R calibration](project_rr_calibration_result.md) — empirical R:R_min=0.5 (25th pctile, 37 winners)
- [CA.PA deactivated](project_carrefour_thin_options.md) — thin chain, removed from universe; EU single-stocks need liquidity proof
- [/analyze ported to R](project_analyze_r_port.md) — RStudies/reports/analyze/ pipeline; 4 gaps deferred
- [Single shared config.yml #58](project_shared_config_yml_todo.md) — 6/7 byte-identical; consolidate via R_CONFIG_FILE
- [NewTrading GitHub remote](project_newtrading_remote_todo.md) — no origin; create `Aldohlys/NewTrading`
- [Condition-based alerts](project_alerts_condition_based_todo.md) — check_alerts.R date-only; add getTriggeredAlerts() vs live IBKR
- [/analyze on VM](project_analyze_on_vm_deployment.md) — seeded on trading-vm; pending env vars, renv::restore, smoke test
- [/analyze improvements](project_analyze_improvements_20260527.md) — vehicle cheap_score override, spread EV ignores slippage, BS grid diverges 26%
- [OI cap OTM filter](project_oi_cap_otm_filter.md) — `.summarize_oi()` filters OTM + thin-chain bypass; chain_state="thin"
- [/analyze spread-module fetch TODO](project_analyze_spread_module_fetch_todo.md) — tdata_py.spread re-fetches whole band per width; fetch once/expiry + quiet DEBUG; last fetch lever (Python+rebuild)
- [positioning.R automation](project_positioning_r_automation_todo.md) — COT file drifted 10wk stale; add staleness warn + auto-pull CFTC
- [Vol module refactor](project_vol_module_refactor.md) — empirical ATR expected-move lens + drop HAR + move skew/metrics to /analyze; Phase 1 (Tdata v5.10.21) DONE, P2-5 pending; skew=derivation not ATR

## Daubasses / Deep Value
- [Daubasses Portfolio](project_daubasses_portfolio.md) — 6 positions, weekly review, `Strategies/Daubasses/`

## Investing Views
- [EM/China experience](user_em_china_view.md) — China ETF failures; tradeable-not-investible framework

## References
- [R environment](reference_r_environment.md) — R-4.4.3 `C:\Program Files\R\R-4.4.3\bin`, lib `Documents\RLibrary`; Rscript on PATH
- [Python/conda launch](reference_python_conda_launch.md) — Claude Code launched from plain cmd (not conda shell); `source miniconda3/etc/profile.d/conda.sh && conda activate` before Python; use miniconda3/python, not Store `python3`
- [change.log convention](reference_changelog_convention.md) — root `change.log`, newest-first, ## YYYY-MM-DD; RApp gitignored, NewTrading tracked
- [DXY Yahoo ticker](reference_dxy_yahoo.md) — DX-Y.NYB
- [Ex-div mechanical drop](reference_exdiv_mechanical_drop_diagnostic.md) — Boursorama prev-close re-bases; TradingView needs div-adjust toggle; Yahoo Adj Close cross-check
- [TWS futures term structure](reference_tws_futures_term_structure.md) — right-click → Charts → Term Structure (graphical curve)
- [TWS no candle-close trigger](reference_tws_candle_close_trigger.md) — triggers fire on tick; CME futures Trigger Method greyed; workaround STP LMT tight or API watcher
- [WTI calendar spread bands](reference_wti_calendar_spread_calibration.md) — contango(-0.50/+0.10), normal(0.10-0.30), tight(0.50-1.00), stress(1.50-2.50), crisis(3+); first non-expiring
- [Futures expiry-week dynamics](reference_futures_expiry_week_dynamics.md) — last 1-2wk convergence+roll contaminate front-back; use first non-expiring spread
- [CFD cash = synthetic perpetual](reference_cfd_cash_synthetic_perpetual.md) — retail CFD cash = rolled futures synthetic, not physical; track back-weighted contract
- [CL options multiplier=1000](reference_cl_options_multiplier.md) — CL 1000, MCL 100, ES 50, GC 100; look up before sizing
- [OESX/EUREX mechanics](reference_oesx_eurex_option_mechanics.md) — ESTX50 pts×10 EUR/pt; Dec quarterly-only, most liquid; IV null→BS-solve; flat term
- [Closed-market marks stale](reference_closed_market_option_marks_stale.md) — exchange-closed = null last/close/uPrice, frozen MM marks; only post-open tradeable
- [Futures daily MTM](reference_futures_daily_mtm.md) — cash arrives daily at settlement; UNRLZD=close, DLY=today; drawdowns real cash
- [Scanner UI](feedback_scanner_ui.md) — clean display, badges, no redundant columns
- [Trade execution](feedback_trade_execution.md) — price ladder (target/acceptable/red-line/hard-no); leg-level mid; no probing bids
- [Spread Greeks monitoring](feedback_spread_greeks_monitoring.md) — compute net Greeks first; don't import outright-call warnings onto spreads
- [Losing-trade decisions](feedback_losing_trade_decisions.md) — don't add to losers; path-vs-direction; balancer needs offsetting book paying
- [Size risk before materiality](feedback_size_risk_before_flagging_materiality.md) — quantify as % of book first; -30% on 4% holding = ~-1%
- [Re-check strike at exec spot](feedback_recheck_strike_at_execution_spot.md) — recompute protection band at live spot if moved >~1% (ESTX50 5600→5700)
- [Portfolio DD ≠ index move](feedback_portfolio_drawdown_vs_index_move.md) — translate hedge target through book equity%; -10% GOnet = equity -11 to -18%
- [Structure vs vol thesis](feedback_structure_selection_vs_vol_thesis.md) — verticals discard vol edge; ≥2 vol components → flag vertical can't carry them
- [Hedge wing-sale timing](feedback_hedge_wing_sale_timing.md) — convert long-put hedge to spread into vol spike not trough; don't crystallize at low IV
- [Directional dashboard minimum](feedback_directional_dashboard_minimum.md) — long futures: chart, alerts, 1 thesis check, 1 curve spread, news; drop IV/smile
- [SQLite reads auto-approved](feedback_no_sqlite_prompts.md) — run sqlite3 SELECT/schema on mydb.db without prompts post-approval
- [V5 scanner lessons](feedback_v5_implementation_lessons.md) — no fabricated keys, fail-open early, multi-target R lib, observed-outcome calibration
- [Anchor net claims on reconciliation](feedback_anchor_net_claims_on_reconciliation.md) — derive net-aggregate direction from total=sum-of-parts, not a line item
- [Verify codebase facts wide](feedback_verify_before_claiming_codebase_facts.md) — grep all repos BEFORE claiming "X defined as Y", not after pushback
- [Re-survey before stale TODO](feedback_resurvey_stale_todos.md) — re-run survey for any "execute" TODO older than a few days
- [.gitignore edits resurface state](feedback_gitignore_edit_resurfaces_state.md) — re-run git status after editing .gitignore
- [Verify "NEVER X" assertions](feedback_verify_policy_assertions.md) — check whether the reason still applies; surface evidence, ask
- [Unify-the-rule needs full audit](feedback_unify_rule_audit_full_surface.md) — audit git ls-files + .gitignore per repo for "do X everywhere"
- [renv state safe to track](reference_renv_safe_to_track.md) — track .Rprofile + renv.lock + renv/{activate.R,settings.json,.gitignore}
- [Unified RApplication repo policy](project_rapplication_repo_policy.md) — what's tracked/ignored across 7 R apps; canonical "should I commit X?"
- [Vertical spread exit 80%](reference_vertical_spread_exit_80pct.md) — GTC sell at entry_debit+0.8×(max−entry); last 20% needs 0 DTE
- [SMH composition](reference_smh_composition_2026.md) — NVDA 20%→8%; more diversified; verify ETF weights before single-name event flag
- [REMX composition](reference_remx_composition.md) — ~30-40% lithium not pure RE; pull weights; Chg% tables 1-month. Pure-RE=MP/LYC/REXC
- [Tdata VRP two forms](reference_tdata_vrp_formula.md) — persisted vrp=log(iv30/rv30)*100; narrative uses vol-pts (iv30-rv30); don't confuse
- [Tdata rebuild needs R restart](feedback_tdata_rebuild_restart_r.md) — reticulate caches tdata_py per session; install alone insufficient
- [Tdata install topology](reference_tdata_install_topology.md) — 6+ renv libs + RLibrary; /build auto=patch bump; verify with find+grep
- [/analyze lot definition](feedback_analyze_lot_definition.md) — $300 risk cap is per LOT (full structure), not per leg
- [/analyze two expiries](feedback_analyze_two_expiries.md) — enumerate ~30 DTE AND ~50-60 DTE for R:R-vs-EV across time
- [Tdata force_refresh](feedback_tdata_force_refresh.md) — force_refresh=True on every getOptValue/spread RR; 30-min TTL gives stale false-fails
- [Tdata spread module](reference_tdata_spread_module.md) — tdata_py.spread.compute_spread_risk_reward enumerates verticals; use for Phase D.4
- [Solve IV when TWS null](feedback_iv_solve_when_tws_returns_null.md) — getOptValue often delta+bid+ask but iv=null; BS-bisect from mid
- [Tdata interest rate utils](reference_tdata_interest_rate_utils.md) — getLastRate(ccy,DTE) returns tenor closest to option life; never macro 10Y
- [Tdata IR refresh system](reference_tdata_ir_refresh_system.md) — Currencies table (percent), getLastRate DTE buckets, getInterestRates registry (USD/EUR/CHF/JPY/CAD)
- [U1804173.IV is underlying iv30](reference_u1804173_iv_column.md) — IV col = underlying ATM iv30, not per-option; pull IBKR hist option data for OTM vega
- [transcribe.py atomic](reference_transcribe_pipeline_atomic.md) — no resume; crash=full re-run; -SkipDownload skips Vimeo; ~80min/50min audio
- [Markdown→PDF pipeline](reference_markdown_pdf_pipeline.md) — pandoc (gfm→standalone HTML) → Chrome headless --print-to-pdf; tool paths + CSS; html gitignored, md+pdf tracked
- [/analyze TWS-down degraded](reference_analyze_tws_down_degraded.md) — FETCH FAILED markers = TWS closed; BS grid + structural targets still work
- [/analyze vehicle rule](feedback_analyze_vehicle_rule.md) — outright if stock<$150 & IVP<60, spread if >$150 or IVP>60, stock if <$10; apply BEFORE grid
- [/analyze off-universe fallback](feedback_analyze_off_universe_fallback.md) — non-universe ticker → yfinance history+chains+BS-solve; never dead-end
- [/analyze neutrality / data-only](feedback_analyze_data_only.md) — measurement not recommender: no GO/NO-GO/conviction/verdict; code-defined sort/filter/score OK ([nuance](feedback_data_only_doesnt_strip_synthesis.md))
- [/analyze run-all + live fallback](feedback_analyze_live_data_fallback.md) — always A→E (no SKIP early-stop); fetch live on cache miss; OI via genericTickList=101
- [/analyze redesign 2026-05-12](project_analyze_redesign_2026_05.md) — UPS test fixes: CSV staleness, LONG-only RS, direction-blind targets; outrights grid + IV30/RV30
- [Sector RS definitions](reference_sector_rs_definition.md) — sector_rs_rank=sector-vs-SPY; rs_etf=stock-vs-sector (unsurfaced); rs_3m=stock-vs-SPY (input); don't conflate
- [Audit direction-blindness for shorts](feedback_audit_direction_blindness_when_adding_shorts.md) — long-authored shared rules bias upward for shorts; smoke-test each layer
- [Tdata vol percentile helpers](reference_tdata_vol_percentile_helpers.md) — getIVPercentileLevels (252d, reliable) vs getVolMetrics (one call, RVP sometimes NA)
- [/analyze SKIP as fade signal](feedback_analyze_skip_as_fade_signal.md) — SKIP on X = confirmation for fade −X (parabolic+high IVP+priced out); ask framework
- [Pyramid per-lot R/R](feedback_pyramid_per_lot_rr_audit.md) — compute Lot N standalone R/R; if <1:1 global stop forces negative EV; use per-lot stop
- [Verify futures contract month](feedback_verify_futures_contract_month.md) — steep curve: don't quote =F front; identify the month from DB or ask
- [Buy stop-limit direction](reference_buy_stop_limit_direction.md) — BUY STP LMT: LMT ≥ STP (STP+$0.10-0.20); reversed misses continuation
- [Don't switch frameworks mid-trade](feedback_dont_switch_frameworks_midtrade.md) — each trade its own contract; new thesis → new position, don't extend runner
- [Hard to exit winners](feedback_hard_to_exit_winners.md) — pre-commit limit at structural target; 90% of plan = success
- [Classify breakout vs MR](feedback_classify_breakout_vs_meanreversion.md) — classify first; opposite stop/target; BOT score is discriminator; table in checklist
- [ATR→move multiples](reference_atr_move_multiples.md) — 10d move: typical 1.5×, strong 4× ATR%; payable freq by beta; prior not gate; mid-bucket graveyard; √N scales
- [ATR empirical band limits](reference_atr_empirical_band_limits.md) — Tuser/vol Empirical row: effective n≈n_obs/N (overlapping windows), regime-pooled→optimistic in tails; trust 70-80%, distrust ≥90%/long-horizon
- [No LaTeX math](feedback_no_latex_math_blocks.md) — renderer garbles `$$`/`$`; use plain text/code for formulas; escape EVERY `\$` in prices (2 bare `$` on one line = garbled math, e.g. `+$344 … −$172`)
- [Trades schema migrated](reference_trades_right_column_sparse.md) — French→English cols (Ssjacent→Underlying, Statut→Status…); Underlying+Right EMPTY for BOT → parse ticker+vehicle from Instrument; trade=group by TradeNr; PnL on closing fill
- [Size analytics to horizon](feedback_size_analytics_to_trade_horizon.md) — window metrics to 2-4wk horizon (~40d), adaptive ZigZag over fixed-window
- [PowerShell `$Var:` scope-qualified](reference_powershell_variable_colon.md) — `"$VMInstance:$Path"` errors; use `"${VMInstance}:${Path}"`
- [gcloud scp --recurse needs dest dir](reference_pscp_recursive_destdir.md) — pscp.exe won't create remote dir; pre-create via ssh, scp contents
- [HV30 window artifact](reference_hv30_window_artifact.md) — IBKR HV30 mechanical rollout; -7vp jumps without regime change; cross-check RV5/10/20
- [XOP thin on weeklies only](reference_xop_thin_chain.md) — weeklies empty, monthlies deep (Jul17 8686); use monthly for outrights
- [Check OI by expiry](feedback_check_oi_by_expiry_for_outright.md) — rank expiries by OI first; cheapest-IV chain may be empty weekly
- [Concise, no repeated phrases](feedback_concise_no_repeated_phrases.md) — state the point, don't label it; no reused framing phrases; no recurring canned conceptual paragraphs; fewer words
- [No mean-reversion framing](feedback_no_mean_reversion_framing.md) — user trades breakout/continuation + vol selling only; don't offer MR as a contrast case
- [Counterparty/positioning lens](feedback_counterparty_positioning_lens.md) — "who's on the other side" matters (MM vs instit vs retail; COT/dealer-gamma/crowdedness); don't dismiss as inert; execution-counterparty→exit, positioning→entry
- [Minimal bold formatting](feedback_minimal_bold_formatting.md) — bold only for leading list/paragraph labels (FOMO-entries pattern); never inline emphasis; structure from sections
- [Plain language, no jargon](feedback_plain_language_no_jargon.md) — spell out; no invented shorthand (small-N→small sample size) or bare variables; expand WR/R-R/DD/MM/BS; ask before expanding standard indicators (ATR/IV/EMA…)
- [No tautological signal rows](feedback_no_tautological_signal_rows.md) — condition→outcome maps need an upstream signal observable independently of the outcome; not the outcome's price restated; weak dollar = shared commodity driver
- [Map all fetch sites before optimizing](feedback_map_all_fetch_sites_before_optimizing.md) — pipeline perf: inventory ALL call sites up front, don't point-fix one log at a time
- [Tdata option-fetch internals](reference_tdata_option_fetch_internals.md) — getStrikesInRange(per-expiry) vs getAllStrikes(union→Error200); getIV_DTE DTE>10; getVolMetrics heavy (8 fetches); grids vary by expiry; force_refresh/closed-market
- [CFTC COT URLs](reference_cftc_cot_urls.md) — WTI petroleum_sf, gold/copper other_lf, DXY deanybtsf, grains ag_lf; Friday for prior-Tue
- *Moved to global memory:* No approval prompts, Simplicity over complexity, User Trading Framework

## Database
- `C:\Users\aldoh\Documents\RApplication\data\mydb.db`; 29+ tables, 2039 trades, 29 strategies
- ScannerUniverse ~130 symbols (119 scanner + 10 ETFs + macro)
- [saveTrades overwrite bug](project_savetrades_overwrite_bug.md) — Save used to replace Trades from stale snapshots; now freshness-guarded
- [Daily portfolio hardening](project_daily_portfolio_hardening.md) — open: lift TWS managed-accounts filter into Tdata; promote PY timeout to R error
