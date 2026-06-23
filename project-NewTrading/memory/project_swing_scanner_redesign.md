---
name: Swing Scanner redesign — front-run option flow lens
description: Locked spec for swing_scanner rewrite around "buy cheap options, sell expensive but still buyable" — three axes Pull/Cheap/Setup-Chain-RR, replacing 2-gate composite
type: project
originSessionId: 53dab338-46f2-4aba-b466-d9cda0aa1007
---
Redesign locked 2026-04-27 after JPM rotation miss diagnosis. The scanner's overarching purpose is reframed: **identify names where option demand is about to arrive, so the trader can buy options cheap and sell them when expensive-but-still-buyable.** This is a liquidity-provider/front-running edge, validated by the 39 BOT winners in the Trades table (winners explicitly priced exits "attractive for buyer" — see JNJ 19DEC25 200C, DOW 17APR26 35C, URA 30JAN26 55C remarques).

**Why:** The previous scanner scored long/short swing setups, treating sector gate as binary and ignoring option-market state. Result: JPM (Financial RS −2.9) surfaced as TOP PICK while Tech led the tape, and the call spread structurally limited headroom regardless of direction. The reframe forces every row to answer three coupled questions: directional pull, option cheapness, and room to sell into incoming flow.

**How to apply:** When asked about scanner enhancements, indicators, gates, or filter logic, work from this three-axis frame. The pipeline order is fixed (universe → Axis 2 → Axis 1 → Axis 3) and is cost-driven: cheap technicals run wide, expensive option queries run narrow.

## Anti-noise conventions (cross-cutting)

Per-strike, single-snapshot option data is noisy: retail panic-bids strikes when momentum is visible, dumps them when momentum dies. Confirmed by inspecting `strikes/JNJ/JNJ/historical_*.parquet` 30-min midpoints (single strike swung 0.13 → 0.75 → 0.26 within hours). All numeric IV/OI/skew inputs are smoothed:

- **N.1 IV inputs:** wherever IV30 / IV90 / IV180 / ATM IV is consumed (Steps C.2, D.4), use the **5-trading-day median of daily-close values**, not the live close. Applies to IVP_2y numerator, VRP, term-structure ratio, BS-forward IV input.
- **N.2 OI / chain reads:** `OI_Cap_Call`, `OI_Cap_Put`, `Total_Chain_OI` (Step D.3) use the 5-day median of per-strike daily OI, persisted via `option_chain_oi_history`. Live walk fills today's row but cap calculation reads median window. Trade volume per strike ignored — only OI growth matters, not volume spikes (churn vs new exposure). First-day-in-universe fallback: live snapshot, `Chain_Walk_Status = PARTIAL`, demoted to WATCH until 5 days of history accumulate.
- **N.3 skew read:** Step C.2 skew bonus uses 5-day median of (25Δ call IV − 25Δ put IV) vs the 6–12m median of those daily 5-day-medians (compounded smoothing).

Calibration consistency: R:R_min calibration from BOT winners' entry-time R:R (Step E.2 input) MUST use the same 5-day-median smoothing on the historical IV reads, else threshold is biased by entry-day noise.

5-day window is defensible default, tunable during validation.

## Pipeline — five phases, hierarchical step numbering

```
Phase A — Universe (Step A.1)
   ↓ ScannerUniverse_Rich
Phase B — Pull (Steps B.1–B.4)
   ↓ Pull_Score ≥ 6 AND Stage ∈ {early, continuation}
Phase C — Cheap (Steps C.1–C.3)
   ↓ Cheap_Score ≥ 6 AND Cheap_Side aligns with Pull_Direction
Phase D — Setup, Chain, R:R (Steps D.1–D.4)
   ↓ Targets_Agreeing ≥ 2 AND R:R ≥ R:R_min
Phase E — Classification & display (Steps E.1–E.2)
   ↓ TOP PICK / WATCH / SKIP + Entry interval
```

## Phase A — Universe (weekly, end-of-day)

**Step A.1. Rich-options gate.** Stock enters universe only if both hold:
- Weekly options listed within next 14 days
- ATM bid/ask width ≤ 12% on nearest monthly cycle, end-of-day only (live quotes unreliable in thin sessions)

No total-OI percentile threshold — other filters suffice. Names entering/leaving logged for review.

## Phase B — Pull (daily, full rich universe, no option fetch)

All indicators reused from `indicators.R`. No new technical computation.

**Step B.1. Stage classification (4 pts, mutually exclusive).**
- early breakout (4): `score_breakout` returns S ≥ 5 AND BK ≥ 3
- continuation pullback (3): `Tdata::isTrendContinuation()` passes
- extended trend (1): price > MA50 × 1.15
- no trend (0): drops out
Output: `Stage`, `Pull_Direction` ∈ {up, down, neutral}.

**Step B.2. Sector flow context (3 pts).** Among LONG-passing sectors (binary gate from `evaluate_sector_gates`), rank by RS vs SPY using `macro_context_results`:
- top-3 RS sectors → +3
- rank 4–6 → +2
- bottom-half → 0
- SHORT-only sector → −2

**Step B.3. Footprint confirmation (3 pts).** Each +1 if aligned with `Pull_Direction`:
- `obv_slope`
- `updn_ratio` (>1.1 long / <0.9 short)
- `RS_3m` from `isTrendContinuation`

**Step B.4. Pull score and cutoff.** `Pull_Score = B.1_pts + B.2_pts + B.3_pts` (0–10). Survival to Phase C requires `Pull_Score ≥ 6` AND `Stage ∈ {early, continuation}`. Extended trends survive only if `Pull_Score ≥ 8`.

## Phase C — Cheap (option fetch fires here, ~15–30 names)

**Step C.1. Option-state fetch.** IV30 / IV90 / IV180, RV30, ATM mid, 25Δ skew per survivor. Reuse `vol_profile.R`, extended to write `option_skew_history` row.

**Step C.2. Cheap score components (0–10).**
- IVP_2y (4 pts) — IV30 vs own 2-year history
- VRP (2 pts) — must be ≤ small positive
- Term structure IV30 vs IV90/IV180 (2 pts)
- Skew vs own history (1 pt)
- Cross-sectional rank within sector (1 pt)
Output: `Cheap_Score`, `Cheap_Side` ∈ {call-cheap, put-cheap, neutral}.

**Step C.3. Cheap cutoff.** Survival to Phase D: `Cheap_Score ≥ 6` AND `Cheap_Side` aligns with `Pull_Direction`.

## Phase D — Setup, Chain, R:R (~5–15 names)

**Step D.1. Vehicle and expiry selection** (per `bot_strategy_checklist.md`):
- Stock < $150 AND `Cheap_Score ≥ 7` → long call OTM
- Stock > $150 OR `Cheap_Score ∈ [6, 7)` → debit call spread
- Stock < $10 OR option width > 8% → stock direct
- Target expiry: 30–45 DTE early, 21–30 DTE continuation. Monthly anchor preferred for OI; weekly only if liquid + short-dated thesis.

**Step D.2. Structural spot target consensus.** Three sources, take the cluster:
1. closest unbroken prior monthly swing high above current
2. 52-week high if within 15% of current
3. nearest round-number above on tiered grid:
   - stock < $50 → nearest $5 above
   - stock $50–$200 → nearest $10 above
   - stock > $200 → nearest $25 above
   - any $100 multiple supersedes grid value if closer (strongest psychological anchor)
- Fib 1.272/1.618 used **only as confirmation overlay** if within ±2% of one of the above. Lone Fib levels ignored.
- Output: `Spot_Target_Low`, `Spot_Target_High`, `Targets_Agreeing` (1/2/3), `Fib_Confirms`.
- If `Targets_Agreeing < 2` → fails Phase D.

**Step D.3. Chain positioning (per-strike OI walk).** Walk ~20 strikes around target band on chosen expiry monthly anchor. Source: `ib.reqMktData(option, genericTickList='101, 105')` returning `callOpenInterest` / `putOpenInterest` / `avOptionVolume`. Pattern in `Tdata/inst/python/scripts/open_interest_code.py`.

Outputs:
| Field | Type | Meaning |
|---|---|---|
| `OI_Cap_Call` | numeric (strike $) | Strike with max call OI above current |
| `OI_Cap_Call_Magnitude` | integer | OI at that strike |
| `OI_Cap_Put` | numeric (strike $) | Strike with max put OI below current |
| `OI_Cap_Put_Magnitude` | integer | OI at that strike |
| `OI_Concentration_pct` | numeric (0–100) | % of walked-strike OI between current and `Spot_Target_Low` |
| `Total_Chain_OI` | integer | Sum across walked strikes |
| `Chain_State` | enum | chain-capped / structurally bounded / mixed |
| `Crowded_Flag` | boolean | TRUE if `Total_Chain_OI` in top quartile vs sector peers (cross-sectional, after all D.3 rows computed) |
| `Effective_Target` | numeric (spot $) | min(`Spot_Target_Low`, `OI_Cap_Call`) — binding target for D.4 |
| `Chain_Walk_Status` | enum | OK / PARTIAL / FAILED |

Side-effect: walked-strike data persisted to `option_chain_oi_history` (one row per strike-symbol-date).

**Step D.4. Risk:Reward and entry interval (merged).**
- `Entry_Floor` = current vehicle mid (live or last-close, flagged)
- Risk = full vehicle premium at `Entry_Floor` (debit for long call/spread; stop-distance × size for stock)
- Reward = BS-forward vehicle price at `Effective_Target` with IV_now + 2%, T = DTE − 5 trading days, minus `Entry_Floor`. Spreads capped at max payoff.
- `R_R` = Reward / Risk
- `Entry_Ceiling` = vehicle price where R:R = R:R_min (BS-inversion using same Reward formula)
- Display: `Headroom_Band` = "+X% to +Y%" at `Effective_Target` (lower) / `Spot_Target_High` (upper, if `Chain_State = structurally bounded`) — diagnostic only

Output: `R_R`, `Entry_Floor`, `Entry_Ceiling`, `Headroom_Band`.

## Phase E — Classification & display

**Step E.1. Entry interval state.**
- `Entry_State`:
  - `IN BAND` if `Entry_Floor ≤ Entry_Ceiling`
  - `PRICED OUT` if `Entry_Floor > Entry_Ceiling`
  - `NO CHAIN` if `Chain_Walk_Status = FAILED`
- Entry_Floor and Entry_Ceiling surfaced in HTML row for direct reference.

**Step E.2. TOP PICK / WATCH / SKIP gate.**

TOP PICK — all hold:
1. Phase A passed (Step A.1)
2. Phase B passed (Step B.4)
3. Phase C passed (Step C.3)
4. Step D.2: `Targets_Agreeing ≥ 2`
5. Step D.4: `R_R ≥ R:R_min`
6. Step E.1: `Entry_State = IN BAND`
7. Step D.3: `Chain_State ≠ chain-capped` OR (chain-capped AND R:R still ≥ R:R_min to `Effective_Target`)

WATCH — Phases A+B+C passed, fails one of D.2 / D.3 / D.4 / E.1.
SKIP — fails Phase A, B, or C.
SECTOR CLUSTER badge if ≥2 TOP PICK from same sector.

`R:R_min` calibrated from 39 BOT winners' entry-time R:R distribution (target 25th percentile). Default seed 2.0 if calibration inconclusive.

## Rendering — interactive single-page HTML, per-day file

Decision locked: **Option 3 — single interactive HTML with DataTables, per-day file** (e.g., `swing_scanner_20260427.html`). Self-contained: data embedded as JSON via `jsonlite::toJSON`, DataTables loaded from CDN, no client-side fetches, no framework, no build step.

**Sane defaults visible on first paint (no interaction required):**
- **Funnel bar** at top — 5 boxes showing count drop per phase: Universe → Pull → Cheap → Setup → TOP PICK. Immediately surfaces degenerate cuts (too loose / too tight).
- **Filter chips** below funnel, all unselected: All phases / Sector / Vehicle / Show SKIP.
- **Master table** — DataTables, default sort Rank then R:R desc.
  - Default visible rows: TOP PICK + WATCH only (SKIP hidden behind chip).
  - **Tier 1 visible columns (~12, decision-driving):** Ticker, Sector, Stage, Pull_Score, Cheap_Score, Vehicle, Spot_Target, R:R, Entry_Floor, Entry_Ceiling, Entry_State, Chain_State.
  - **Tier 2 hidden-by-default columns (~15, diagnostic):** Pull_Direction, Cheap_Side, Targets_Agreeing, Fib_Confirms, OI_Cap_Call, OI_Concentration, Crowded_Flag, Headroom_Band, Phase_Of_Drop, per-component scores (B.1/B.2/B.3, individual Cheap components). Toggled via DataTables column-menu button. Same JSON payload, no reload.
- Interactions add visibility, never remove it. Open file → scan top of table → decide is the unchanged normal-day flow.

**Constraints reaffirmed:**
- No client-side data fetching — all data embedded.
- DataTables only — no React/Vue/build step.
- Per-day filename — preserves run-archive workflow, no broken-link risk in old reports.
- CSV output unchanged from current scanner.

## Implementation status (2026-04-27)

**LANDED:**
1. ✅ R:R_min calibration via `scripts/calibrate_rr_min.R` — DB-only, no external data. R:R_min = 0.5 (25th pctile of R:R_max from 37 winners).
2. ✅ `Tdata::get_chain_oi(sym, expiration, strike_min, strike_max)` added to `chains_manager.py` and exported via `__init__.py`. Reads per-strike OI via `reqMktData genericTickList='101, 105'`, batched at 40 strikes/round, ~2s wait per batch.
3. ✅ DB tables created (`scripts/init_scanner_tables.R`): `option_skew_history`, `option_chain_oi_history`, `scanner_rich_universe`, `scanner_results_v5`.
5. ✅ `swing_scanner/universe_filter.R` — Step A.1 (permissive default until daily fetch wired).
6. ✅ `swing_scanner/pull_score.R` — Steps B.1–B.4.
7. ✅ `swing_scanner/cheap_score.R` — Steps C.1–C.3.
8. ✅ `swing_scanner/setup_chain_rr.R` — Steps D.1–D.4.
9. ✅ `swing_scanner/final_classify_v5.R` — Steps E.1–E.2.
10. ✅ `swing_scanner/render_html_v5.R` — DataTables + funnel + Tier1/Tier2 columns + filter chips. Per-day file (`swing_scanner_v5_YYYYMMDD.html`).
   ✅ `swing_scanner/main_v5.R` — orchestrator wiring all phases.

4. ✅ `scripts/daily_option_fetch.R` — daily post-close fetch (Steps 4). Populates `option_skew_history` (proxy 25Δ skew via lognormal-strike approximation) and `option_chain_oi_history` (per-strike OI walk via `get_chain_oi`). Schedule via Windows Task Scheduler ~22:30 CET, after US close.

**Unit tests:** `scripts/test_v5_modules.R` — 27 tests passing covering D.2 structural targets (round-grid + clusters), B stage classification (priority: extended > early > continuation > none), C cheap scoring (IVP/VRP/term/skew/cross-sectional points), D.4 vehicle dispatch (call/spread/stock), E.1 entry state (IN BAND / PRICED OUT / NO CHAIN), E.2 classification (TOP PICK / WATCH / SKIP routing per phase).

**PENDING:**
- Schedule `scripts/daily_option_fetch.R` in Windows Task Scheduler (manual setup).
- Step 11: Validation — replay 30 BOT winners + JPM Apr-24 day. Run after daily fetch has populated 5+ days of skew/OI history (anti-noise N.1/N.2 require it).

**KNOWN limitations on first runs (until daily fetch lands):**
- Phase A: passes everything by default (no expiry/bid-ask check).
- Phase C: skew_25d column unpopulated → skew bonus point always 0; Cheap_Score caps at 9 (was 10).
- Phase D.3: chain_state="NO DATA", chain_walk_status="FAILED" → all rows demoted to WATCH or SKIP.
- Net effect: TOP PICK count = 0 until daily fetch populates skew + OI history.

**Run command:** `Rscript RApplication/RStudies/reports/swing_scanner/main_v5.R`. Outputs: `reports/swing_scanner_v5_YYYYMMDD.csv` and `.html`. Legacy `main.R` preserved for transition.

## Infra prerequisites
- New DB tables: `option_skew_history` (daily 25Δ skew), `option_chain_oi_history` (daily per-strike OI for rich-universe front month). 1-year retention.
- Daily fetch task post-close (~22:30 CET) — separate from existing `RunScanner` 09:00 CET task because chains need to be stable.

## What this replaces / supersedes
- The work-item-1 sector RS overlay (`scanner_methodology_TODO.md`) is now subsumed inside Axis 2 step "Sector flow context (3 pts)" — same logic, integrated.
- The work-item-2 pattern catalog is deferred — patterns are Axis 2 inputs, but the three-axis frame must land first.
- The current 2-gate composite in `final_filter.R` is fully replaced.

## Validation evidence (winners that motivated lens)
JNJ 19DEC25 200C (+$1,040) — exits priced via BS with IV+2%, "attractive price for buyer" stated verbatim.
DOW 17APR26 35C (+$1,058) — Fib extensions + IV bump → option target price.
URA 30JAN26 55C (+$490) — limit at 90% of max profit, "room left for buyer" reasoning.
NTR 20JUN25 60C (+$464) — exited because IV at 6-month low, "no move expected" → headroom collapsed.
GLD 17OCT25 350C (+$515) — closed at 85% of max value when option saturated.

These winners share: (a) entered when option was cheap (low IVP, normal skew), (b) directional pull was emerging not extended, (c) exit priced where the next buyer would still pay.

## What is NOT in scope
- Pattern catalog (work item 2 of methodology TODO) — deferred until three-axis frame is observed for 2-3 weeks.
- Probability-based EV machinery — explicitly rejected by user in favor of R:R.
- Sole-Fib targets — used only as confirmation overlay.
- Live-quote universe filter — end-of-day only.
