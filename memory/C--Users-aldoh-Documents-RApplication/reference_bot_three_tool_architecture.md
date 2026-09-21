---
name: reference_bot_three_tool_architecture
description: BOT tooling is three tools on three cadences (BOT_monthly / BOT_daily / BOT_name) specified field-by-field in docs/BOT_TOOLS_DESIGN.md; shared engines live in RStudies/reports/shared
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
