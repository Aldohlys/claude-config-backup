---
name: project_intermarket_macro_scenarios
description: "macro_context intermarket layer (global movie, 11 scenarios, panels, sector map) + 2026-10-02 tenets block, commodity/credit rows, COT section 11; open: stress ladder, crowding×price, roll flag, correlation gauge"
metadata:
  node_type: memory
  type: project
  originSessionId: d0bd0d7e-3332-405c-afbe-bdb65214a63b
  modified: 2026-10-01T20:09:44.946Z
---

Built 2026-10-01 inside RStudies/reports/macro_context, committed + pushed as RStudies 466c213 (2026-10-02): intermarket_config.R, intermarket.R, archetypes.R, intermarket_reads.R, intermarket_render.R; template sections 00 (global movie + scenario cards), 09 (panels), 10 (BOT sector map). DB: intermarket_cache, macro_intermarket_sectors, macro_intermarket_scenarios; CSV Reports/intermarket_sectors_YYYYMMDD.csv.

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
