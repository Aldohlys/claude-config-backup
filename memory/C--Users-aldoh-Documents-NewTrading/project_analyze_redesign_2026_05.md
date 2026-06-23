---
name: analyze-redesign-2026-05-12
description: "/analyze command revisit — UPS test exposed scanner-CSV staleness, LONG-only sector RS, wrong-direction structural targets, junk structures table. Per-phase fixes in scope."
metadata: 
  node_type: memory
  type: project
  originSessionId: 3b70ddd8-ed66-422b-bfce-e9ef777ec479
---

User declared `/analyze` "useless" on 2026-05-12 after the UPS short test (`analyze_UPS_20260511.html`): Phase A STALE from 6-day-old CSV, IVP n/a with no live fallback, structural targets at $109-110 for a SHORT on a $100 stock, 25-row structures table with phantom phantom-credit rows (RR=32x because OTM put pricing rounded to zero).

**Why:** the April 2026 data-only neutralization went too far — stripped synthesis but kept stale data paths and direction-blind logic. Result is a measurement instrument whose measurements are wrong or missing.

**How to apply:** treat /analyze as a redesign target. Don't patch incrementally; rework Phase A/B/C/D against the locked-in decisions below.

## Decisions locked in (2026-05-12)

- **Phase A**: keep as informational only, never SKIP downstream. Replace CSV gate with live IBKR probe (getExpirationDates + ATM strikes + bid/ask spread). DB cache → CSV are fallbacks.
- **Phase B sector_rs_rank**: currently LONG-only (ranks among LONG-passing sectors by RS-vs-SPY descending). Make direction-aware: for shorts, filter to SHORT-passing sectors and rank by RS-vs-SPY ASCENDING (weakest first). `evaluate_sector_gates` already emits both flags; only the rank map is asymmetric. See [[reference_sector_rs_definition]].
- **Phase B stock-vs-sector RS**: surface as new rows in indicator breakdown. Compute already exists (`rs_etf = ret20 - etf_ret` in `pull_score.R:85`). Add **two rows**: 20d and 60d. Direction-aware read: long=leader is positive, short=laggard is negative. Four-cell decoder (stock vs sector × sector vs SPY) is the user's mental model.
- **Phase C.1 cheap_score**: surface all 4 components (ivp_pts/4 + vrp_pts/2 + term_pts/2 + rr_pts/1, max=9 not 10 as tooltip claims). Add live IVP fallback (compute IV30 rank from 252d history when DB Prices.ivp is NA).
- **Phase C.2 funnel**: "FETCH FAILED" today means live was attempted EXCEPT for IVP (no live path exists — that's the bug to fix).
- **Phase D structural targets**: must be direction-aware. Today calls `compute_structural_target` direction-blind and emits upward targets for shorts. Fix: shorts get prior swing lows / 52w low / round number below. `effective_target` uses `oi_cap_put` for shorts, `oi_cap_call` for longs.
- **Phase D structures table**: filter to `within_cap=TRUE` AND direction-appropriate (DEBIT only for the trade direction — bear puts for short, bull calls for long). Drop `spread_type` column. Drop phantom rows (risk<$5 or both legs<$0.05). Round numbers (2 decimals). $/% suffixes. Two expiries side by side (~30 DTE and ~50-60 DTE).

## All scope decisions locked (2026-05-12)

- **Phase B pull_score**: DROP entirely along with stage_pts / sector_pts / footprint_pts subtotals. Phase B output = per-indicator breakdown (S1-S6, BK1-BK4, AUX) + stage label (early/continuation/extended/none) + sector_rs_rank (direction-aware) + stock-vs-sector RS (20d + 60d rows). No composite score.
- **Structures table sort key**: `expected_value` descending. Top row = highest EV within cap. No composite-points layer.
- **Implementation order**: invert CSV-vs-live priority FIRST (top-down data sourcing rewrite — IBKR live primary, DB cache second, scanner CSV last-resort). Phase-by-phase fixes follow.

## Implementation plan (locked)

**Step 1 — Data sourcing inversion** (foundation)
- Rewrite `.read_scanner_row()` semantics: keep helper but demote to last-resort fallback.
- New helpers for live-first reads of: sector membership, IVP, IV30/IV90/RV30, structural OHLC, sector ETF returns.
- Freshness gate now applies CSV-or-DB→live escalation per field, not per phase.

**Step 2 — Phase B rewrite**
- Drop pull_score / stage_pts / sector_pts / footprint_pts from output.
- Keep stage label only.
- Add direction-aware `sector_rs_rank`: SHORT path filters to `$short`-passing sectors, ranks ascending by sector-vs-SPY RS.
- Add stock-vs-sector RS rows: 20d (already computed as `rs_etf`) + 60d (new — needs `ret60` on stock and sector ETF in `calc_ind`).

**Step 3 — Phase C rewrite**
- Surface all 4 cheap_score components (ivp_pts/4, vrp_pts/2, term_pts/2, rr_pts/1, max=9).
- Add live IVP fallback (IV30 rank from trailing 252 trading days when DB Prices.ivp NA).
- Funnel: confirm "FETCH FAILED" semantics, ensure IVP row has a live computation path.

**Step 4 — Phase D rewrite (highest user-visible impact)**
- Make `compute_structural_target` direction-aware: shorts get prior swing lows + 52w low + round below + fib retracement below.
- `effective_target` uses `oi_cap_put` for shorts, `oi_cap_call` for longs.
- Structures table: filter to `within_cap=TRUE` AND DEBIT-only for the trade direction (bear put debit for short, bull call debit for long).
- Drop `spread_type` column. Drop phantom rows (max_risk < $5 or any leg BS price < $0.05).
- Round numbers (2 decimals). `$` suffix on currency amounts (strikes, widths, debit, max_risk, max_reward). `%` suffix on `prob_success_delta` and `edge`.
- Sort by `expected_value` desc.
- Two expiries side by side (~30 DTE and ~50-60 DTE).

**Step 5 — Phase A rewrite (lowest priority)**
- Demote from gate to informational. Live IBKR probe: getExpirationDates succeeds, ATM strikes available, ATM bid/ask spread < threshold. Never SKIP downstream phases.

## UPS test artifacts

- `C:\Users\aldoh\Documents\NewTrading\Reports\analyze_UPS_20260511.html` — the report that surfaced all of the above.
- Spot $100.78, sector Industrials, scanner CSV mtime 2026-05-05 (6d stale).

## Post-Step-5 follow-ups (same day, after first live runs)

After Step 5 the user invoked `/analyze` live for the first time and surfaced four UX/correctness gaps. All fixed and pushed.

| Commit | Issue surfaced by | Fix |
|---|---|---|
| `4529b72` | UPS short live run | (a) vehicle rule labelled `call` for shorts — added `direction` param to `pick_vehicle_expiry`. (b) structures table wrapped in collapsed `<details>` when vehicle ≠ spread — opened it. (c) `.live_entry_framework` picked ATM call strike for shorts — direction-aware strike picker + BS `type` threading via `bs_type`. (d) `compute_rr_entry` `type="Call"` hardcoded — direction-aware `bs_type` and `max_payoff`. |
| `e57397c` | "Outright vehicle has no pricing grid visible" | New `enumerate_outrights()` in structures.R + `.render_outright_table()`. Strike × expiry grid (4 strikes × 2 expiries). Always rendered when TWS reachable, regardless of vehicle rule preference. |
| `eb76e0b` | "C.1 missing RV30/RVP for comparison" | New `resolve_rvp()`. C.1 row renamed `IVP component → IV30 / IVP component` (shows both); new informational `RV30 / RVP (info)` row (no points). |
| `dd46f5d` | "Redundant columns" + "want collapsible" | Drop `Max loss` from outrights (= Entry premium), drop `Max risk` from spreads (= Debit). Wrap both tables in `<details open>` like Phase B breakdown. |

## Lessons captured as separate memories

- [[feedback_data_only_doesnt_strip_synthesis]] — the data-only rule bans prose verdicts, not code-defined synthesis.
- [[feedback_audit_direction_blindness_when_adding_shorts]] — long-only helpers need a full audit when adding short support.
- [[reference_tdata_vol_percentile_helpers]] — getIVPercentileLevels vs getVolMetrics characteristics.

## Steps 3-5 completed (2026-05-12, second session)

**Step 3 — Phase C.1 component breakdown**
- `run_phase_c` always recomputes live from funnel; scanner CSV no longer read.
- `cheap_score` max changed `/10 → /9` (4+2+2+1). Each component (IVP/VRP/Term/RR) surfaced with its points / max / value / band in the C.1 table.
- New helper: `.compute_cheap_components()` in `phases.R`.

**Step 4 — Phase D direction-aware**
- `compute_structural_target()` accepts `direction`. Long path unchanged. Short path: prior swing **lows**, 52w **low**, round number **below**, fib retracement **down**. New helper `.nearest_round_below`. New `.structural_target_short()`.
- `.live_targets()` now takes `direction`; `.live_entry_framework` switches `effective_target` to `oi_cap_put` for shorts.
- Scanner-CSV entry-framework path **removed** (CSV is long-only).
- `enumerate_structures()` now accepts `expiries` (vector). Both `~30 DTE` and `~55 DTE` enumerated side-by-side. Each row carries an `expiry` column.
- Filter: within_cap=TRUE **AND** DEBIT-only **AND** max_risk ≥ $5 (drops phantom rows). Sort by EV descending.
- `.render_structures_table()` rewritten: drop `spread_type` column, format `$` for currency, `%` for prob/edge, `2 decimals` for ratios. Long/Short strike, Width, Debit, Max risk/reward, R:R, P(success), Edge, EV columns.

**Step 5 — Phase A demoted**
- `run_phase_a()` returns `INFO` always (never SKIP). Live IBKR `getExpirationDates` probe → DB `scanner_rich_universe` → scanner CSV last-resort.
- Reports `n_expiries` + `tradeable_expiries` (in 14-90 DTE window).
- `run_phase_e()` classification no longer gates on Phase A.
- Phase A badge removed from header; Phase A row removed from Phase E result table; Data Summary no longer references it.

**Smoke test (TWS down) on UPS short**
- Phase A: INFO (graceful degradation through DB → CSV last-resort).
- Phase B: SKIP (sector rank 12/19), unchanged from Step 2.
- Phase D: structural targets now $94.06 / $90.00 (both **below** $100 spot) ✅ — was $109/$110 upside in the original bug report.
- Phase E: classification SKIP, phase_of_drop=B (Phase A no longer in the chain).
- Structures table: "FETCH FAILED: TWS not reachable" — graceful, formatted error.

**Carry-forward when TWS reachable**: the DEBIT-only, EV-sorted, $/%-formatted, two-expiry structures table will render — not validated this session because TWS dropped mid-refactor.

## Step 2 completed (2026-05-12)

- **Deliverable**: Phase B rewritten — `pull_score` / `stage_pts` / `sector_pts` / `footprint_pts` dropped (triple-counted the indicator breakdown).
- **New Phase B output**: stage label (mechanical, MA50-based) + direction alignment (ALIGNED/MISMATCH from price-vs-MA50) + sector / sector ETF / stock-vs-sector RS 20d / stock-vs-sector RS 60d / sector-vs-SPY RS 20d / direction-aware sector rank.
- **Direction-aware sector rank**: long ranks descending (rank 1 = strongest sector vs SPY); short ranks ascending (rank 1 = weakest). PASS cutoff = top half of sectors.
- **New resolver**: `compute_sector_rs_context()` in `live_sources.R` walks all sector ETFs (~19) via yfinance to build the cross-sectional rank.
- **`ret60`** added to `shared/indicators.R::calc_ind`.
- **Smoke test results (2026-05-12 TWS down)**:
  - UPS short: stage=continuation align=ALIGNED sector_rank=12/19 (Industrials mid-pack on weakness). Stock vs Sector ETF 20d=-3.32% (laggard); 60d=-16.63% (deep laggard).
  - JPM long: stage=none align=MISMATCH sector_rank=13/19 (Financials weak vs SPY).

## Step 1 completed (2026-05-12)

- **Deliverable**: `RStudies/reports/shared/live_sources.R` — per-field resolver module (resolve_spot/sector/sector_etf/expiry/iv30/iv90/rv30/ivp/skew_25d/chain_oi/earnings). Uniform return shape `list(value, source, retrieved_at, reason)`. DB-fresh → live IBKR/yfinance → FETCH FAILED with cause.
- **IVP gap closed**: `resolve_ivp` calls `Tdata::getIVPercentileLevels` (5.8.10) for 252d OPTION_IMPLIED_VOLATILITY history; linear-interp between p10/p25/p50/p75/p90 returns the percentile rank.
- **Migrated**: `funnel.R` (entirely rewritten around resolvers); `structures.R` (`.resolve_expiry`/`.resolve_chain`/`.summarize_oi` deleted, moved to live_sources).
- **Smoke test results (2026-05-12)**:
  - UPS short: cheap_score 5→7, IVP=`47.8% (live interp) mid`, funnel 4/1/1 (was 4/0/2).
  - JPM long:  IVP=`74.4% (live interp) rich`, funnel 2/3/1.
- **Phase A and B still STALE** — they still read scanner CSV. That's Step 5 / Step 2 territory.
- **Pre-existing carry-forwards**: structures still mixed CREDIT/DEBIT, still 30 DTE only, structural targets still direction-blind. Step 4 fixes these.
