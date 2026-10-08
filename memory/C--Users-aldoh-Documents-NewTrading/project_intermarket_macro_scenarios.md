---
name: project_intermarket_macro_scenarios
description: "macro_context intermarket layer (global movie, 11 scenarios, panels, sector map) + 2026-10-02 tenets block, commodity/credit rows, COT section 11; open: stress ladder, crowding×price, roll flag, correlation gauge"
metadata:
  node_type: memory
  type: project
  originSessionId: d0bd0d7e-3332-405c-afbe-bdb65214a63b
  modified: 2026-10-08T08:44:08.168Z
---

Built 2026-10-01 inside RStudies/reports/macro_context, committed + pushed as RStudies 466c213 (2026-10-02): intermarket_config.R, intermarket.R, archetypes.R, intermarket_reads.R, intermarket_render.R; template sections 00 (global movie + scenario cards), 09 (panels), 10 (BOT sector map). DB: intermarket_cache, macro_intermarket_sectors, macro_intermarket_scenarios; sector map file Reports/intermarket_sectors_YYYYMMDD.xlsx (Data + Legend sheets, intermarket_xlsx.R, openxlsx) since 2026-10-08 — the CSV (10-01..10-08) is no longer written; nothing read it.

User intent: qualitative "global movie" (e.g. USD up, other FX down, US/EU/UK yields up, oil up, gold down -> liquidity stress / dollar funding), read in order stocks -> FX -> precious metals -> oil -> other commodities -> rates -> Europe/LatAm/Asia (Nikkei, KOSPI). Present compared with past analogs, never assumed to repeat. Model = BPT "Macro Outlook" videos (Vimeo 1232001265 -> bpt_macro_outlook_20261001, 1232064228 -> bpt_intermarket_20261001; transcription killed for low memory, audio in Transcripts/).

Data date: Tdata::getYahooData default include_today=FALSE, so an evening run still shows the PREVIOUS close (2026-10-01 22:39 run = 30 Sep data); say so when reporting, offer a rerun with today's bar.

2026-10-02 (RStudies e8f686b): first BPT transcript compared (Transcripts/macro_outlook_20261001_summary.md). Changes: uranium metal row SRUUF + URA relabelled miners (see [[reference_uranium_tickers]]); conditional "chain" field on archetypes (stagflation: oil -> USD -> US10Y, the user's petrodollar mechanism = importers sell Treasuries to raise dollars for oil), rendered in its own block under the cards because only the top-2 scenario cards are shown; events.R SCHEDULED_EVENTS manual list (Brazil Oct-04/Oct-25, US midterms Nov-03, 30-day lead, boost=FALSE keeps non-US events out of the regime catalyst boost) - needs new dates added by hand. Gaps the comparison exposed, not yet fixed: no India, no measured-move/"room left" notion, breadth 20% labelled "oversold - reversal watch" while BPT reads it as correction confirmation, NG front-month vs BPT's January contract.

Open: scenario weights untested; more BPT transcripts to fold into the scenarios; sector map not yet backtested vs breakout_5y_results.csv; oil uses =F continuous (roll gaps); Bund/gilt via IS0L.DE/GLTL.L price proxies; optional daily claude -p commentary not built. See [[feedback_scenario_persistence]].

2026-10-02, second transcript (commodities session, `Transcripts/wherestrade_20261091_summary.md`, with a 12-item proposal table organised by [[reference_ceresna_macro_tenets]]).

Done, pushed to RStudies:
- 9f187a1:
  - #1 reading-principles block
  - #3 LQD/IEF + adjusted closes for HYG/LQD/IEF/TIP/TLT
  - #4 stock-vs-commodity ratios
  - #8 COT staleness offset 3→4
  - #10 instruments: VIX3M, PA, HO, CC, KC, SB, LBR, gold in EUR/CHF
  - #11 XLE + REMX groups
  - FX as-of join ([[reference_yahoo_fx_calendar_shift]])
- 3693ff3:
  - #6 COT dashboard, section 11 ([[reference_cotsignal_api_rejected]])

Not added to the sector map: CRAK, XEG.TO, tankers, coal, WOOD. No universe name has them as `bench`, and the map only covers universe benches.

Still open:
- #2 stress ladder: 30Y→DXY→MOVE→IG→HY→KRE→RUT/EW→SPX→VIX, z≥1 per rung
- #5 sector-correlation gauge
- #7 crowding × price follow-through labels
- #9 actor table beyond the "other side" collapsible
- #12 futures roll flag; the sugar Oct→Mar roll showed as +7.7% 1W on 2026-10-02

Known quirk: re-running `refresh_cot.R` in the same COT week drops the "Since <prev date>" clause from the `positioning.R` notes and rewrites the file. Restore it with `git checkout` after manual test runs. Not fixed.

2026-10-02 addendum in the same file (`wherestrade_20261091_summary.md`, section "Addendum 2026-10-02"): proposals #13–#25 from the 09-24..09-30 sessions — yield velocity, bond-proxy rung, dollar vs 2Y differential, liquidity headline rule, COT long/short split, UST crowding caveat, vol tenors (VIX6M/1Y, OVX, GVZ), CTA proxy, capital-supply row, index-vs-internals flag, pivot + laggard-relief archetypes with flip triggers, intervention tripwires + calendar, India/real crude/Dec-27 crude. All open.

2026-10-05: Macro Outlook 10-02 compared with macro_context (section at end of `Transcripts/macro_outlook_20261002_summary.md`, commit ec0986d). Findings to carry: (1) his structure (bond-driven, narrow, dollar-led) was constant all week while his timing changed daily; the scanner's ranking was equally stable — scanner = structure, transcripts = timing. (2) Open check: regime input `credit_stress` = 0.80 while HY/Treasuries ratio is flat/up (+0.7% 1M) — the two disagree, look at `scenarios.R`. (3) Brent–WTI spread ~\$11 (Brent +6.9% vs WTI +0.1% 1M) has no row. (4) Bear steepening on dovish repricing (3M bill −8 bp, 10Y +9 bp 1W) = bond-rout signature, scanner doesn't label it. (5) The file has a relief-confirmation checklist of scanner rows (10Y→5.0%, DXY down / EUR 1.14-1.15, EW & Russell vs S&P up, XLP/KRE/VNQ off SHORT, gold 4,400-4,500, bond rout <40%) = what proposal #23(b) should score. Next: compare the September Macro Outlooks (being transcribed 2026-10-05) to test the age rules against a month of his calls.

2026-10-08 (RStudies e164741, 2a1030c; NewTrading 48fc07f): Methodology tab (methodology.R, generated from live config). 12 scenarios: + leadership_unwind, + policy_pivot; carry_unwind removed and turned into a 3-session alert (CARRY_ALERT: yen z>=2 + 2 of AUDJPY/Nikkei/BTC z<=-2, VIX>=VIX3M). New assets SMALL (IWM/^GSPC), SHORT (^IRX, yield), ABS_BREADTH ((s5fi-50)/15 from macro_context_results, morning reading = previous session). Calibration (calibrate_scenarios.py, Reports/scenario_weight_calibration_20261008.md): 40% kept (lowest with mean FP <=10%); six fingerprints changed (pivot, stagflation, goldilocks, wrecking ball, bond rout, debasement). Debasement FP 19% at 40%; goldilocks weakest (LOEO AUC 0.62). Script stops if it can't parse every fingerprint (comment lines above fp once dropped 4 silently). Later 10-08: + credit_event scenario (13 scenarios, held-out AUC 0.86, kept as written); regime credit_stress now HYG/IEF adjusted (center -0.52, scale 2.34), IEF added to macro tickers, fetch.R appends tickers missing from the day cache. ABS_BREADTH weights untested ([[project_abs_breadth_weights_untested]]).

2026-10-08, two-score display (RStudies 978e2be, NewTrading 347f80c): state score (position in 252-session range, same weights) decides "in place" at STATE_ACTIVE 0.50; 1-month move gives direction (SCEN_ACTIVE 0.40 = strong move). Statuses BUILDING/ESTABLISHED/MATURE/FADING/EMERGING/FADED/INACTIVE, ranked by state. Evidence: compare_state_move.py (state held-out AUC 0.81 vs move 0.78, mix 0.85, but state 2-4x later, lingers up to 36 sessions, misses pivot/unwind turns). HISTORY_DAYS 520. DB column state_score.
State scale is signed, −100%..+100% like the move (asset positions rescaled −1..+1 × signed weights; negative = assets at the opposite ends of their ranges, e.g. leadership unwind −83% mirrors narrow leadership +86%). The user asked about a 0–100% scale on 2026-10-08 and accepted the signed one. 0–100% stays only for per-instrument 52-week position in the panels.

2026-10-08 evening (RStudies 90ec0c0): WTI curve in the oil panel. Fixed contracts +3/+6 delivery months after the front (Yahoo CLxYY.NYM, ~360 bars each, no roll gaps), front minus +3/+6 spreads in $/bbl (panel kind "usd"), curve sentence in the movie. `crude_curve()` in intermarket_config.R applies the CME last-trade rule (holidays ignored). Gap 2 of macro_context_vs_ceresna_20261007.md closed. Running main.R from reports/macro_context (not the RStudies root) skips renv, so openxlsx is missing and the sector workbook step fails after the HTML is written.

2026-10-08 comparison (NewTrading Reports/macro_context_vs_ceresna_20261008.md, commit 4e0c03f), two new gaps, not fixed: (a) at 09:00 Yahoo returned the previous session's bar with NULL Close for 35 symbols (all Europe/Asia indices, IS0L.DE, GLTL.L, European group members); get_close drops NA, so the report silently used Tuesday. Check `SELECT ticker FROM intermarket_cache WHERE date=<last> AND Close IS NULL`. (b) OIL fingerprint / petrodollar chain / panel 1M read CL=F, which loses the roll in backwardation: 1M to 10-08 CL=F −5.1% vs Nov contract −1.8%, Feb-27 +5.8%. Fix = fixed contract (extends #12).
Both fixed same evening (RStudies 18c12c8, pushed): roll_adjust() for CL/BZ/NG/HO/RB (FUT_LTD rules in intermarket_config.R; expired contracts not on Yahoo, so old roll days use the current listed front as proxy); refetch_empty_closes() + amber banner. IS0L.DE / GLTL.L / NUCL.L 10-07 still empty on Yahoo at 23:30. FXC.SW re-append fixed in RStudies 4dd1438 (pushed): fetch.R requests Tickers.YahooName and renames rows back to the universe symbol; EUR.CHF now reads EURCHF=X. Tdata::getYahooData itself still maps only 3-letter names.
