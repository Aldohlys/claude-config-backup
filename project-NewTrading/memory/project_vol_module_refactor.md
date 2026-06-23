---
name: project_vol_module_refactor
description: "Tuser/vol module reorg — add empirical ATR expected-move lens, remove HAR, move skew/metrics to /analyze; multi-phase, Phase 1 done 2026-06-09"
metadata: 
  node_type: memory
  type: project
  originSessionId: 65083a85-2e71-48fe-97b7-db8b02bd59a8
---

Reorganizing the volatility module (`RApplication/Tuser/vol/`, a Shiny app embedded LIVE in **RPreTrade** via `volUI`, plus standalone `vol/app.R`). Goal: make it a coherent **expected-move** tool with three lenses — IV (parametric, forward), HV (parametric, realized), and a new **empirical** signed-quantile lens. Drop HAR (no edge); move VIX-skew + advanced vol metrics to `/analyze`.

**Key conceptual correction (user, 2026-06-09):** ATR is a sign-less dispersion measure — no better at skew than HV/GK/YZ. The asymmetry in `atr_expected_move` comes ENTIRELY from taking empirical quantiles of *signed* forward returns (the numerator); `ATR%·√N` is just the regime-scale denominator, interchangeable with HV. What kills skew in IV/HV panels is the symmetric *derivation* (`z·σ·√T`, `exp(±)`), not the measure. So the lens is really "parametric vs empirical," and ATR's honest value = model-free + gaps/intraday + path-oriented + native stop unit (NOT skew). Forward-looking skew would need the IV smile (the promoted `vix_skew` put/call decomposition).

**Logic home decision:** centralize in Tdata (reusable by vol UI, /analyze, scan, BOT screens).

**Phase 1 — Tdata — DONE (v5.10.21, committed+pushed, deployed to 6 apps + RLibrary):**
- New `R/atr_move.R`: `atr_pct`, `atr_move_distribution` (heavy fetch→standardized signed-move vector), `atr_expected_move_from_dist` (cheap quantile read → asymmetric band, any conf), `atr_expected_move` (wrapper). Envelope flags (horizon>25, ATR<1/>5, thin history). `+ test-atr-move.R` (781 tests pass).
- Promoted `R/vix_skew.R` (get_vix_skew/calculate_vix_optimized/format_vix_results) + `R/vol_metrics.R` (compute_spot_vol_correlation/compute_vol_of_vol/format_vol_metrics) from Tuser into Tdata.
- Removed 7 HAR-model fns from `volatility.R`; KEPT `get_har_price_data` (name unchanged — used by trend.R + test-trend.R mocks; doc de-HAR'd). Deleted test-har-volatility.R, trimmed test-volatility.R.
- Forced coupling pulled forward: removed harvol UI from `vol/app.R` + `vol/view/volUI.R` and deleted `harvolUI.R`, because its `box::use(Tdata[fitHAR,...])` resolves at load time and would break RPreTrade once Tdata lost those exports. **These Tuser edits are uncommitted.**

**Phase 2 — Tuser/vol — CODE DONE 2026-06-09 (uncommitted; pending live smoke test):**
- New `vol/view/atrvolUI.R`: empirical lens. Inputs horizon(sessions)+conf(%); heavy `dist` reactive keyed on sym+horizon (atr_move_distribution), cheap reactive on conf (atr_expected_move_from_dist) → asymmetric Lower/Median/Upper band table + asymmetry ratio + flags. (Initially had a "Symmetric (parametric-equiv.)" row = mean abs reach; REMOVED 2026-06-09 — it was a re-symmetrized version of the same empirical data, redundant w/ the explicit tails + ratio and mislabeled as an IV/HV band. A genuine triangulation would overlay a real z·σ·√T band from IV/HV — deferred.) Live-rendered OK in RPreTrade (USO 5d showed downside skew 0.87 — per-name beats class coeff).
- `vol/app.R`: dropped vixskew panel, added atrvol (histvol + impvol + atrvol).
- `vol/view/volUI.R`: dropped volmetrics + vixskew panels, added atrvol panel; server returns impvol/histvol/atrvol. Signature unchanged `server(id,sym,currency,i_rate)`.
- Triangulation: delivered as the in-panel empirical-vs-symmetric contrast (not a 3-way IV/HV/empirical single table — that needs unified horizon inputs + IV sourcing; deferred, confirm with user if wanted).
- Verified: deployed Tdata 5.10.21 in Tuser active lib (windows/R-4.4) exports the atr fns; RPreTrade `volUI$ui/server` call sites match unchanged signature (P3 compat OK). Live render test pending (needs running app; Rscript segfaults on Tdata load in agent sandbox).
- NOT deleted (still needed): vixskewUI.R, vixf.R(Tuser), vol_metrics.R(Tuser), scan/volmetricsUI.R — used by scan/wheel (P4) + source for /analyze (P5). Stale R-4.3 Tdata 2.4.5 leftover in Tuser/renv is inactive.

**Phase 2.5 — unified triangulation — CODE DONE 2026-06-09 (uncommitted; pending live verify):**
- New `vol/view/xmoveUI.R`: ONE panel, shared inputs horizon(sessions, default 5) + conf(%, default 80), three move rows in a table:
  - **IV** (auto): `getIV_DTE(sym,ccy,spot, DTE=max(round(horizon*7/5),14))` → nearest-usable-expiry ATM IV (getIV_DTE refuses DTE≤10 — it time-interpolates variance between bracketing expiries and only considers expiries ≥today+7, so sub-10d targets extrapolate/are noisy). Shows the DTE used. **Market-data-only, blank if NA** (manual IV would just duplicate HV — user's call). Heavy, keyed on sym+horizon.
  - **HV** (manual %): user reads it off the HistVol estimator table. Parametric.
  - **Empirical** (ATR asymmetric): atr_move_distribution/from_dist. Heavy, keyed on sym+horizon.
  - IV & HV use `z·σ·√(N/252)` lognormal exp(±) bands; all three anchored on spot (carry over a few sessions immaterial). Confidence/HV changes recompute only the cheap table.
- Replaced impvol+atrvol wiring in `app.R` (histvol + xmove) and `volUI.R` (HistVol estimator table as HV reference + xmove panel). Deleted `atrvolUI.R`. **Kept `impvolUI.R` file** (unwired) — RPreTrade tests do `test_box_import("vol/view/impvolUI")` + grep RPreTrade's own server.R for `impvolUI$server` (those are about RPreTrade files, unaffected). volUI server signature unchanged (i_rate now unused but kept for RPreTrade compat).
- Parses clean; Tdata imports resolve in deployed lib. **Live-verified + committed (Tuser 0ab8641).**
- Fix during verify: IV (getIV_DTE) was in the main output reactive → blocked the whole render in the standalone app (no TWS → hang). Moved IV to an **on-demand button** (reactiveVal + observeEvent(input$fetch_iv); cleared on sym/horizon change). HV+Empirical now render instantly (Yahoo-only); IV pulled only on click.
- Added empirical sample size (n_obs ~8y) to the panel header after user couldn't tell 8y from 10 sessions.
- Validated: QQQ skew FLIPS with CI — 80% asym 1.14 (upside body, +drift, median coef +0.27), 95% asym 0.87 (downside extreme tail), full skewness −0.71. Textbook index profile (drifts up, crash-tail left-skewed); confirms per-name empirical captures real tail skew the symmetric IV/HV bands erase. Earlier commits this phase: Tuser ec83708 (atrvol lens, since superseded), 2ac4706 (harvol removal).

**getIV_DTE efficiency fix — Tdata 5.10.22 (committed+pushed+deployed 2026-06-09):**
- Root cause: getIV_DTE used `getStrikesfromExpDate` (range [None,None]) → qualified the WHOLE chain per expiry (QQQ 552 strikes!) just to keep 4.
- Tried `getAllStrikes(sym)` (union grid, no qualify) — BROKEN: returns strikes invalid for the specific expiry (QQQ 20260623 is a $5-strike expiry; union grid's $1/$0.5 strikes → IBKR Error 200, 0 pairs). Per-expiry validity REQUIRES qualification.
- Fix: new internal `.atm_strikes_in_range(sym, expiry, forward)` ladders range 1%→3%→6%→12%, stops at first yielding ≥4 strikes, via `getStrikesInRange` (qualifies only near-money, per-expiry, cached). QQQ $5-expiry ~9 / $1-expiry ~14 / MT ~5 qualified vs 552. getStrikesInRange verified live (no bug; reuses proven getAllStrikes plumbing; footgun: pass expiration=/center_strike= as KEYWORDS).
- Validated strike-sourcing live; IV *value* couldn't be checked (tested pre-US-open: getOptValue returns null marks when market closed; MT only "worked" via 30-min quote cache from a prior run — not live). Verify IV populates after 15:30 CET.

**Tuser committed + PUSHED to Aldohlys/Tuser master (6ef9857, 2026-06-09)** — whole vol arc: harvol removal, unified xmoveUI (IV-auto/HV-manual/Empirical), on-demand IV + withProgress, horizon-date readout, n_obs, histogram of realized move distribution w/ bounds. CHANGELOG.md entry added (Keep-a-Changelog). Tdata at 5.10.22.

**P4 — DONE (Tuser 8b36fbd, 2026-06-09):** "repoint scan/wheel" was a NO-OP (scan/wheel use getStoredMetrics/getVolMetrics DB table, never the promoted fns). Actual P4 = retired the orphaned Tuser copies: deleted vixskewUI.R, vixf.R, vol_metrics.R (logic lives in Tdata, surfaced in /analyze).

**P5 — DONE (RStudies 36bfa1b → main, 2026-06-09):** /analyze "Volatility character" section (phases.R .compute_vol_character, report.R .render_vol_character, after Phase C): spot/vol-corr + vol-of-vol (Yahoo, always) + opt-in VIX put/call skew via `--skew` flag. Calls Tdata-promoted helpers.

**BONUS — /analyze option-fetch leaning (RStudies 36bfa1b, big perf win):** user pushed hard on "too much option data retrieved". Fixed all live option-fetch sites in shared/live_sources.R + analyze/structures.R:
- resolve_rv30: rvp from 252d hist-vol bars only (was Tdata::getVolMetrics → 8 iv-term-structure option fetches, pure waste).
- resolve_option_spread (Phase A) + .live_25d_skew (funnel): analytic ATM/30Δ/25Δ strike location (.rough_iv30 + BS), price only ~6 strikes (was ±35%/±25% wholesale, ~474 quotes).
- .live_atm_iv band 10→4%; resolve_chain_oi OI band 25→12%.
- structures.R: cap to **top-10 DEBIT by EV**; spread band **width/price-aware** `eff_moneyness=max(base, width/spot+0.03)` — FIX for low-priced names (fixed 0.05 gave ZERO width-10 spreads on MT $67/F $15). moneyness 0.20→0.05 base.
- Validated QQQ/MT/J/NVDA/F ($15–717, $1&$5 grids, long+short): quotes hundreds→1–6/leg, structures still produced.
- **Deferred:** residual cost is tdata_py.spread (Python) force-refreshing its whole band PER WIDTH; true fix = fetch once per expiry across widths + quieter DEBUG — a scoped Tdata/Python rebuild, not yet done.

**ALL PHASES COMPLETE & PUSHED.** Tdata 5.10.22, Tuser master @8b36fbd, RStudies main @36bfa1b. config.yml (RStudies, tracked) committed with moneyness; renv.lock churn left uncommitted.

Built on the ATR envelope work — see [[reference_atr_move_multiples]].
