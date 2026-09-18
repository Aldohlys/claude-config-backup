---
name: positioning-r-automation-todo
description: "CLOSED 2026-09-03 — positioning.R is now generated weekly from the CFTC archives by refresh_cot.R; staleness banner backs it up"
metadata: 
  node_type: memory
  type: project
  originSessionId: 8c65e4a9-c01b-4a86-95fe-f6292e3db0e8
---

# positioning.R hand-curation drift — automation TODO (CLOSED 2026-09-03)

Opened 2026-05-27 after the file went 10+ weeks without refresh and was discovered stale during a XOP/crude follow-up question.

## Current state

`RStudies/reports/macro_context/positioning.R` is a hand-edited list of 5 COT positioning entries (Crude, USD, Gold, Grains, Copper). The file header says "EDIT THIS FILE EVERY MONDAY after reading Ole Hansen's COT report" — but there's no reminder, alert, or staleness check.

It feeds `compute_positioning_stress()` in `scenarios.R`, which counts `extreme = TRUE` flags to compute `crowding_score` (up to 0.5 contribution). A stale file with old `extreme` flags can either over-weight crowding (regime stays "fragile" past the actual extreme) or under-weight it (missed new extreme).

## Why

- 2026-03-16 entry sat unchanged through 2026-05-27 (10 weeks). Crude extreme=TRUE was no longer true (specs unwound 80% from peak). Grains extreme=TRUE was also stale. Copper net=short was inverted (now net long).
- No CI/scheduled task ever touched it. No staleness banner in the macro_context HTML report.

## How to apply

When the user wants regime-scoring output and `COT_DATE` is more than ~10 days old:
- **Flag it before deriving anything** — don't silently use stale crowding contributions.
- Offer to refresh from raw CFTC ([[cftc-cot-urls]]) if Saxo digest unavailable.

## Possible fixes (not shipped)

1. **Staleness warning at read time** — `scenarios.R` could emit a `message()` when `COT_DATE` is >10d old; render that as a banner in the HTML.
2. **Auto-refresh from CFTC structured data** — CFTC publishes CSV/XLS feeds; could pull weekly via cron and update the 5 entries. Loses Ole Hansen's narrative judgment on `extreme` flag.
3. **Hybrid**: auto-pull raw numbers each Friday after CFTC release, leave `extreme` flag manually editable Monday after reading Saxo.

Option 3 is best — preserves the narrative judgment that makes the file useful while removing the silent-drift failure mode.

## Sources used in 2026-05-27 refresh

See [[cftc-cot-urls]] for the URL map.

## Refresh log

- **2026-09-03** — refreshed to COT week ending 2026-08-25 after 14 weeks stale (previous refresh 2026-05-27 / data 2026-05-19). Source: raw CFTC weekly pages + the annual historical archive. **Option 3 (hybrid) is now half-shipped in substance if not in automation:** the `extreme` flags are no longer hand-judged from Ole Hansen's narrative — they are set by a documented rule (5-year percentile >=90 crowded long / <=10 crowded short) computed from `fut_disagg_txt_<YYYY>.zip` + `fut_fin_txt_<YYYY>.zip`, 2021-2026, 295 weekly obs. The rule is written into the file header, so the next refresh is reproducible and does not need the Saxo digest.
- What the 14-week drift actually cost: **nothing downstream.** All five flags were FALSE, so `crowding_score` was 0; the refresh moves it to 0.2 (Grains + Copper extreme, 2/5 x 0.5), still under the `> 0.3` gate in `scenarios.R:381`, so no regime score changed. The visible damage was in the report narrative and the direction labels - USD was still labelled `net = "short"` when Lev+AM had flipped to +23.2k net long at a one-year high.
- **Design note worth acting on:** with 5 assets at 0.5 weight, `crowding_score` can only take the values 0/0.1/0.2/0.3/0.4/0.5, and the `> 0.3` threshold means **4 of 5 assets must be extreme** before positioning influences the regime at all. That is a very insensitive gate - it may be why the stale file never showed symptoms. Worth revisiting alongside the staleness banner.
- **Staleness warning SHIPPED 2026-09-03 (RStudies `e147f61`)** — option 1 is done. `compute_positioning_stress()` now takes `cot_as_of` and returns `cot_age_days` / `cot_missed_releases` / `cot_stale`; `run_scenarios()` emits a `message()`; the report shows an amber `{{DATA_BANNER}}` plus an always-on inline "COT as of <date> (Nd)" marker on the positioning bar. Staleness is counted in **missed weekly releases**, not raw days: COT covers Tuesday and publishes the following Friday, so a current file is 3-9 days old and `missed = floor((age - 3) / 7)`, stale at >= 1 (age >= 10). Passing no date keeps the old behaviour, so a positioning.R without `COT_AS_OF` still works.
- **AUTO-REFRESH SHIPPED 2026-09-03 — this TODO is now CLOSED.** `RStudies/reports/macro_context/refresh_cot.R` (RStudies `0651668`) generates positioning.R outright: positions, WoW, 5y/1y percentiles, extreme flags and notes, all from the CFTC annual archives (`fut_disagg_txt` / `fut_fin_txt`), which are rebuilt weekly and carry both the newest release and the history — so one source serves current values AND percentiles, and no HTML page is scraped. Scheduled `\RApplication\RefreshCOT`, Saturday 08:00 (RApplication `f7941b4`; bat + XML mirror the RefreshInterestRates pattern). positioning.R is now a GENERATED file — hand edits are overwritten; change `refresh_cot.R` instead.
- Script behaviours worth remembering: past years cached, only the current year re-fetched; extracted text deleted after parsing (cache stays ~16 MB of zips, under the already-gitignored `output/`); atomic write that parses the new file before replacing the live one; no write at all when nothing changed, so a quiet week makes no git churn; `--dry-run` prints the table only; notes carry "DIRECTION FLIPPED" / "NEWLY EXTREME" and the delta since the previous file's as-of date.
- **The two mechanisms back each other up:** if a Saturday run fails, the file stops advancing and the staleness banner surfaces it in the report within days. Neither silent drift nor a silent job failure survives both.
- **Actor-detail CSV added 2026-09-03** (RStudies `37ba939`, path fix `5347bc1`): `NewTrading/Reports/cot_actors_latest.csv`, rewritten every run. Latest week, 7 contracts (grains split into corn / soybeans / SRW wheat) x every trader category — producer, swap dealer, managed money, other reportables, retail; TFF categories for the dollar index — with long, short, net, week-on-week change in each, and percent of open interest, plus derived legacy commercial / large-spec / small-spec rollups. positioning.R keeps only the one speculative net the regime score consumes; this is the detail behind it. Category mapping and the spreading trap: [[reference_cot_trader_categories]].
- `RefreshCOT.bat` sits in `RApplication/scripts/`, so it appears automatically in the RApplication launcher (`scripts/launcher.py` auto-discovers every `*.bat` in its own directory — nothing to register). The separate `NewTrading/scripts/launcher.py` ("Trading launcher") scans only NewTrading and does not show it.
- Open question left deliberately: the weekly run dirties two tracked files (positioning.R and the CSV), so someone must commit them (no auto-commit from a scheduled task).
- Alternative sources considered and rejected: [[reference_cotsignal_api_rejected]].
