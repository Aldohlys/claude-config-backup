---
name: reference_bot_three_tool_architecture
description: BOT tooling = bot_monthly / bot_daily / analyze, spec in docs/BOT_TOOLS_DESIGN.md; per-name read shared in reports/shared/bot_read.R; membership = ADV only; zones descriptive (no edge, #94); sort on asym_em; 12-column default sheet
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5f03a6df-1df0-4f76-b348-15024cf64b52
  modified: 2026-09-18T20:21:55.991Z
---

The BOT tools replaced the scattered scanners in September 2026. The spec is a
data dictionary — **`docs/BOT_TOOLS_DESIGN.md`**, every field with type, unit and
computation, each tagged `key` / `decision` / `detail`. Read it before changing
any output; it is the contract, not prose.

| tool | cadence | writes |
|---|---|---|
| `reports/bot_monthly/main.R` | monthly, **market hours** | `Tickers` membership columns + `bot_monthly_<date>.csv` |
| `reports/bot_daily/main.R` | daily, **no TWS** | `bot_daily_<date>.csv` (74 fields, 44 by default, all with `--detail`) |
| `reports/bot_name/main.R` | on demand, one name | `bot_name_<SYM>_<date>.csv` + a verdict that can be **no acceptable execution** |

Shared engines in `reports/shared/`: `zones.R` (S/R zones + Fibonacci levels),
**`gates.R` (the single implementation of the nine gates)**, `weekly.R`
(resample), `name_attributes.R` (gap share, ATR profile, ADV, breakeven).
`bot_zones_trial.R` at the RStudies root reproduces the published zone trial.

**The split is not arbitrary:** criteria that separate names are slow (ticker
attributes), criteria that describe a setup are fast. So option *cost* is
simulated monthly and chained only on demand, never daily. No vol *view* (IV
rank, VRP, skew) appears anywhere — see [[project_bot_three_class_framework]].

**Traps when working on these:**

- `calc_ind()` needs 130 rows, so the weekly resample needs **5 years** of daily
  bars (2 years gives ~104 weeks and the weekly leg silently never computes).
- `gates.R` exists because the nine gates had drifted across three copies —
  `indicators.R::compute_breakdown` computes five setup gates, `flow_score.R::score_breakout`
  six, because they disagree on whether S3 exists. Do not add a fourth.
- `BOT_monthly` owns membership and replaces the hand-curated `book_BOT` column
  in [[reference_bot_tradable_universe_csv]], so filters that lived in that
  curation (the ADV floor) had to become explicit code.
- Terciles (gap share, vol-of-vol) are **cross-sectional**, so membership cannot
  be decided one ticker at a time — hence the three-phase structure. Running with
  a symbol list or `--limit` produces degenerate terciles.
- `--db PATH` runs against a copy; use it before letting the `ALTER TABLE` touch
  production `Tickers`.

Open questions that make two criteria inert and the daily sort backwards:
**TODO #88**. See [[feedback_ratio_metric_anticorrelated_with_its_own_thesis]].
Evidence base is TODO #82 (measurement record, scripts lost).

## Running them (measured 2026-09-18)

| | per ticker | 347 tickers |
|---|---|---|
| `bot_monthly` with the live option probe | ~43 s | **~4.2 h** |
| `bot_monthly --no-tws` | 4 s | **~23 min** |

The TWS probe is ~**90% of the runtime**, and outside 09:30–16:00 New York it returns nothing usable — so a post-close run pays 10x for NULL columns. **`--no-tws` gives identical `BOT_Eligible`**: membership is `atr_band` + `gap_share` tercile + `adv_pass` + the opportunity count, none of which touch bid-ask. The probe only fills `AtmBidAskPct` and `BOT_VehicleHint`. So: run `--no-tws` for membership any time, and a TWS pass in market hours for the vehicle hint.

**Killing it mid-run loses everything** — phase 3 writes to `Tickers` only after all tickers are computed, because the terciles are cross-sectional. Phase 1 now `saveRDS`es to `tempdir()/bot_monthly_phase1.rds` so a fetch is not lost to an assembly bug (which happened twice). `bot_daily` is ~5–8 min for 69 names and needs no TWS at all.

First production run: **335 rows computed, 169 eligible**; excluded 93 `gap_share`, 42 `atr_band`, 22 `adv`, 9 `no_data`, and **0** `no_opportunity`. `bot_daily` now reads those 169 from `Tickers.BOT_Eligible` instead of falling back to the 69 `book_BOT` rows.

## UPDATE 2026-09-21 — three corrections, one of them to this file's own explanation

- **`bot_name` no longer exists.** It was retired into **`/analyze`**, which was
  already pricing all three vehicles and only lacked the accept test. The three
  cadences are now `run_bot_monthly.bat` / `run_bot_daily.bat` / `run_analyze.bat`
  (all in `NewTrading/scripts`). `run_flow_scanner.bat` is a *different strategy*
  and sits outside them. `bot_scan_universe.py` retired too; its F1/F2 entry
  factors were ported into `bot_daily` and cross-validated on 59 names first.
- **`gap_share` is no longer a membership criterion** (TODO 88.4). Membership is
  now `atr_band` + `adv_pass` + opportunity count. It gated the risk of gapping
  *through* a stop, but over 169 daily rows `GapShare` does not predict that:
  Spearman **0.073** against `gap_vs_stop`, while the stop distance itself is
  **−0.889**. `bot_daily` tests it per trade and vetoes with `gap_through_stop`.
- **The "outside market hours the probe returns nothing usable" claim above was
  wrong.** `AtmBidAskPct` and `BOT_VehicleHint` were NULL on all 352 rows in
  *every* run, TWS up or down, because `bot_monthly` unwrapped the result with
  `q$value %||% NULL` and that file's `%||%` is scalar-only (`length(a) != 1`
  falls through to `b`) applied to an 8-element list. The row logged
  `bid-ask: LIVE` because the fetch had genuinely succeeded. Fixed 2026-09-21;
  AAPL then read 2.7%, NVDA 1.0, GLD 2.3, SMH 4.1, JPM 4.9.
- **All four gate copies are now one.** `compute_breakdown` (/analyze) and
  `score_breakout` (swing_scanner) keep their formatting and scoring but take
  pass/fail from `eval_gates()`. That also fixed a latent NA path: `S3 <- rs > 0`
  was NA when `rs` was NA, making `setup_score` itself NA.

## UPDATE 2026-09-24 — TODO #88, #93, #94 closed or advanced; the sheet changed shape

- **Membership is `ADV_Pass` alone** (plus data checks). `atr_band` (88.2) and the
  opportunity count (88.1: no gate-firing definition separates names; nine-gate
  counts are Poisson, dispersion 1.17) left the ladder. `scripts/rederive_bot_eligible.sql`
  was run: 274 eligible / 52 adv / 9 no_data.
- **The per-name read is `reports/shared/bot_read.R::bot_read_row()`**, used by
  `bot_daily` AND `/analyze` (section after Phase D). Prefixed names (`bot_`,
  `BOT_`, `.br_`) because /analyze defines its own top-level `.pct`.
  `bot_read_ticker_rows()` gives named symbols their Tickers YahooName / bands.
- **Short rows are mirrored on the trade's axis** (`res_*` = target side, `sup_*`
  = stop side, distances >= 0). Until 2026-09-24 short rows silently carried a
  long's target/stop/asym.
- **Zones are descriptive, not validated** (#94, `RStudies/bot_zones_reaction_trial.R`):
  price reacts at the nearest zone no more than at a placebo band (lift ~0, SE
  0.008), and a long entered inside a zone does as well as outside (-0.003,
  SE 0.009). The in-zone veto is gone -> `zone_state` column; `tradable` =
  `gap_through_stop` only. Finer ZigZag thresholds (88.6) add zones, no edge.
- **Sort key = `asym_em`** = min(target dist, EM10 move) / max(stop dist, 1 ATR).
  Raw `asym` stays a column. Default CSV = 12 columns (`BOT_READ_DEFAULT`),
  `--detail` = all 88. See [[feedback_bot_edge_is_asymmetry_not_win_rate]].
- `rs_state` is `n/a` everywhere: no benchmark is wired (#93.4 open). S3 is NA
  (not FALSE) when rs20 is missing.
- The Excel view is `NewTrading/scripts/bot_daily_to_xlsx.py` (called by
  `run_bot_daily.bat`): tiers on `asym_em`, legend read from the spec, blue /
  amber / vermillion palette, `--out-dir` to write away from `Trades/`.
- Full-universe `bot_daily --direction both` (274 names) takes ~15-20 min.

## UPDATE 2026-09-29 — one level engine, calendars, benchmark
- **/analyze Phase D uses `level_read()`** (`analyze/structures.R::.level_targets()`), capped at the expected move over the sessions to the primary expiry (≈21 for 30 DTE) instead of BOT_daily's 10; `compute_structural_target()` is no longer used by /analyze (still in `setup_chain_rr.R` for the swing scanner). TODO 93 closed.
- **Options-session gating**: `reports/shared/market_calendar.R` (qlcal) — /analyze skips Phase A / funnel / option part of D when the name's options market is closed; BOT_monthly keeps last month's AtmBidAskPct. `bar_lag` counts the listing's business days.
- **S3 benchmark** = `Tickers.BOT_Bench` (hand-maintained Yahoo symbol). NOT in BOT_monthly's SCHEMA on purpose: its UPDATE writes every SCHEMA column and would null it.
- Zones (same-type and flipped) have no measured reaction edge (#94, #95); kept as chart reading.

