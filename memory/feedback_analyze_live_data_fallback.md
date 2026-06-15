---
name: feedback_analyze_live_data_fallback
description: When /analyze runs, NO field should report n/a / unknown / unavailable due to missing cache or skipped upstream phase. If a value isn't already computed, fetch it live.
type: feedback
originSessionId: 66254968-8c04-4ad8-bfc3-2b1d56f655a0
---
When `/analyze` runs, every indicator in the report must be computed or pulled live from IBKR / yfinance. No field should display `n/a`, `unknown`, `unavailable`, or "skipped because upstream phase failed". The user wants a complete data snapshot every time, not a partial one gated by cache freshness or phase-result short-circuits.

**Why:** /analyze is the user's measurement instrument for a single ticker. Half-empty grids (Term IV30/IV90 unknown, Skew unavailable, OI n/a, structures placeholder) make the report unusable for the actual decision. If TWS or yfinance is reachable, every cell should be filled — even when an upstream phase was SKIP.

**How to apply:**
1. **Phase short-circuiting is forbidden for data fetch.** Even if Phase B SKIPs, Phase C must still compute cheap_score, IVP, VRP, term structure, skew. Even if Phase C SKIPs, Phase D must still pull chain, OI, targets, R:R, structures. Phase results gate the *interpretation* (badge color, classification), not the *fetching*.
2. **Live fallback chain for every field:**
   - DB cache → if missing/stale → live IBKR via Tdata getOptValue / chains_manager / spread.compute_spread_risk_reward
   - For underlyings off the v5 universe: yfinance history + chains + BS-solve (existing rule, see `feedback_analyze_off_universe_fallback.md`)
   - For OI: live `reqMktData genericTickList=101` (existing rule, see `feedback_analyze_live_oi_pull.md`) — generalize the same approach to ALL chain fields
   - For IV when TWS returns null: BS-bisect from mid (existing, see `feedback_iv_solve_when_tws_returns_null.md`)
   - For term-structure and skew: live IBKR getOptValue across strikes/expiries; if a strike fails, drop that leg with a logged warning, don't drop the whole signal
3. **Surface missing data explicitly, never silently fall back to placeholder.** If a field genuinely cannot be fetched (e.g. TWS down, ticker delisted), the cell shows `FETCH FAILED: <reason>` with the cause — not a generic `n/a`.
4. **Cache TTL should not gate /analyze.** Per `feedback_tdata_force_refresh.md`, pass `force_refresh=True` on every getOptValue / compute_spread_risk_reward call from the analyze pipeline.

**Reinforces and extends:** `feedback_analyze_force_all_phases.md` (run all phases, don't early-stop), `feedback_analyze_live_oi_pull.md` (live OI), `feedback_analyze_off_universe_fallback.md` (yfinance fallback), `feedback_iv_solve_when_tws_returns_null.md` (BS-bisect for IV), `feedback_tdata_force_refresh.md` (bypass cache TTL).

The PSX 2026-04-30 run failed this rule on: Phase C cheap_score (skipped on B fail), Term IV30/IV90 (not fetched), Skew RR 25Δ (not fetched), Phase D targets/chain/OI/R:R (all skipped on C fail), structures (placeholder rows because pricer not invoked). All should have been live-pulled.
