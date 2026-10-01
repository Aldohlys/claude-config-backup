---
name: project_intermarket_macro_scenarios
description: "macro_context got a qualitative intermarket layer (2026-10-01) - global movie, 11 historical scenario fingerprints, panels, BOT sector map; BPT videos as structure reference"
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
