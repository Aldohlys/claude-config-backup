---
name: project_bot_trading_plan_review
description: BOT trading-plan doc revised against the 2026-06-14 punch-list and SPLIT into separate long + short plans
metadata: 
  node_type: memory
  type: project
  originSessionId: ede11afc-7fd8-4f96-9c93-0e22f399a27c
---

BOT docs in `Strategies/Breakouts/`: `bot_strategy_checklist.md` (full reference) + the prose trading plan(s) reviewed in `bot_strategy_trading_plan_review_20260614.md` (What/Why/Action punch-list).

**2026-06-15 revision DONE.** Revised `bot_strategy_trading_plan.md` against the punch-list AND split long/short into two docs (user's call — long vs short breakouts have different market psychology, not reciprocals; wanted clarity-of-mind per trade):

- `bot_strategy_trading_plan.md` — now scoped **LONG only**, retitled "Asymmetric LONG Breakout (BOT)".
- `bot_short_strategy_trading_plan.md` — **new** short/breakdown plan; mirrors the gate scaffolding but short-specific (upward drift = headwind, squeeze/borrow tail, inverted vol skew → put-spread default, finite targets, faster covers, regime as enabler). Short criteria are structural inversion, **NOT yet backtested** — has an "Empirical TODO" section.

**Punch-list items addressed in the long plan:** portfolio-heat + correlation + earnings entry gates added to Gate 4; pay-for-entry base-rate stated (Step 1); time-stop reworded as demand-for-evidence not auto-close; spread "60% covers 3 lots" arithmetic fixed (needs 3× debit, only if debit ≤ 1/3 width); EMA/SMA caveat added to Annex 1; small-N cells tagged directional; MID-ATR graveyard promoted to Gate 2 (ATR prior); MM/retail preamble replaced with execution ladder; new Entry Execution section; VRP-haircut caveat in expected-move annex; duplicate Annex 3 renumbered (now 1-5); sector-C table demoted to footnote.

**Also added (user request):** multi-timeframe structure at top of Gate 2 — weekly=SETUP (base, scan filter), daily=SIGNAL (trigger), intraday=execution; rationale = removes graphical bias (two scales must agree) + filters wide universe + builds conviction. Same structure mirrored in the short plan.

**2026-06-15 follow-on edits (same session, after user iteration):**
- Section 5 (Gate 3 Vehicle) restructured: decision tree (5.1) upgraded to a top-down waterfall folding in selection criteria; swapped so 5.2=selection criteria (next to tree), 5.3=characteristics/exit. Call/Put → Call (longs-only).
- ATR (4.3) integrated into the vehicle tree (5.1) as an **embedded fork on the outright-call leaf** (ATR≥3% rich → outright+size up; <1.7% quiet → outright only if catalyst-mobile megacap, else spread/stock), paired with VRP as the two convexity justifications. MID-ATR graveyard moved to a **Gate 4 entry veto** (not a vehicle branch) — user's explicit call.
- Style passes applied across both plans: stripped inline bold (kept only leading-label bold); de-abbreviated (win rate/reward-risk/drawdown/market maker/Black-Scholes; removed the `N` variable → "sample size" / spelled out); escaped all `\$`; fixed tautological sector-context rows (upstream signals, not the sector's own price; weak-dollar = shared commodity driver).

**Regime redesign — DONE (Gate 0 + Gate 1 rewritten 2026-06-15).** Gate 0 recast as a **risk/correlation/sizing overlay, NOT a per-trade kill switch**. Old drawdown-from-ATH-only def mislabeled early-bull (Oct-2022, >−10% until mid-2023) as BEAR. New model:
- **Three levels, decide bottom-up / size top-down:** stock (setup+trigger+liquidity, the trade) → sector (trend + where opportunity is, primary go/no-go) → market (risk/correlation/size overlay). Both stock & sector are weight-agnostic (liquidity+trend); the index is not in the trade path.
- **Measure market weight-agnostically:** breadth + equal-weight (RSP); cap-weight SPX kept only for risk-on/off + correlation/stress (SPX = cap-weighted ≈ mega-cap Tech proxy, hides dispersion).
- **Two axes:** direction (200-day slope+price; flat=sideways) primary, drawdown depth secondary.
- **Regimes:** BULL / BULL pullback / SIDEWAYS-range (accumulation or distribution = worst for breakouts, whipsaw, favors vol-selling) / BEAR. **RECOVERY = confirmed transition** (range resolves up), not an early breadth-thrust call.
- **Sideways/concentration override:** a flat/Tech-driven index does NOT veto a clean trending+liquid sector setup UNLESS cross-sector correlation is high (then collapse to market level).
- Gate 1 gained a **dispersion & correlation read** (high dispersion/low corr = stock-picker's market; high corr = one bet, respect index + Gate 4 heat).
- Thresholds set as tunable defaults: flat 200-day slope band over ~20 sessions, ADX<20 = no trend, healthy breadth = majority above 200-day.

**NEXT SESSION (resume here, 2026-06-16) — rewrite Section 2 / Gate 0 as a clean 3-step actionable checklist.** User's verdict 2026-06-15: still poorly written, not concise, doesn't translate into a checklist. Despite many edits this session (it iterated: weight-agnostic read, weekly review cadence with daily-data MAs, Trend×Depth regime matrix, breadth+dispersion as modifiers), the prose structure is wrong. Target shape the user dictated:
1. **Compute** market indicators — for SPX and RSP: 50-day EMA, 200-day SMA, 200-day SMA slope (~1-month), price vs 200-SMA, 50-EMA vs 200-SMA (golden/death cross); plus market breadth (% above 200-day SMA) and sector dispersion/correlation. (Indicators on daily data, reviewed weekly.)
2. **Deduce** (a) market regime (BULL / PULLBACK / RECOVERY / RANGE / BEAR, from Trend×Depth) AND (b) **regime quality = confidence high/medium/low** — NEW dimension, derived from how well the indicators AGREE (SPX & RSP aligned, breadth confirms, dispersion supports). Confidence is not yet in the doc.
3. **Act**: full size / reduced size / no trade — as a function of regime × confidence (bullish regime + high confidence = full; degraded agreement = reduced; low/bear = none).
The elegant target: a 2-D output — regime gives direction-permission, confidence (indicator agreement) gives size. Make it a literal checklist/table, minimal prose. Current content is all there (correct substance), just needs restructuring into compute→deduce→act.

**Section 2 / Gate 0 REWRITE DONE (2026-06-16)** as compute→deduce→act. Framing: regime = **US market "temperature," a risk/permission+sizing overlay, NOT per-trade alpha.** Step 1 compute (SPX & RSP: price vs 200-SMA, 200-SMA slope ~1mo; drawdown; breadth = % above 50-day). Step 2: **Trend = UP = price>200-SMA AND 200-SMA rising** (DOWN = mirror; else SIDEWAYS) × Depth → BULL (PULLBACK folded in as a 5-10% depth note — persistence basis), RECOVERY (UP+>10%), RANGE (SIDEWAYS), BEAR (DOWN); Confidence = participation (RSP confirms Trend + breadth broad → HIGH/MED/LOW) scales size. Step 3: regime=permission, confidence=size. Dispersion → Gate 1/Gate 4, not regime. Indicators daily, reviewed weekly.
- **Golden cross (50-EMA vs 200-SMA) REMOVED from Trend (2026-06-16)** — backtest showed it changes the UP verdict only 2.7% of days (96%+ redundant with price>200-SMA + rising slope); do NOT re-add it. The only remaining 50-day indicator is breadth (kept because confidence-based sizing was kept). At ATH, drawdown = 0% = shallowest = BULL (no gap).

**Regime backtest findings (2026-06-16, scripts in temp):**
- Frequency (SPX weekly 2005-26): BULL 50%, PULLBACK 6%, RECOVERY 11%, RANGE 18%, BEAR 15%. RANGE is a minority (~15-18%), not the default.
- **Durability VALIDATED (user's retained point):** GO (longs-eligible) spells avg ~32 weeks, flip only ~2×/yr; **P(still GO 4 weeks later | GO now) = 93%** → regime is stable enough for a 2-4wk hold, not whipsaw. (PULLBACK label whipsaws — median 1wk, 62% one-week — but shares BULL's action, so harmless → folded into BULL.)
- **Per-regime breakout PERFORMANCE is NOT statistically significant.** Split `breakout_5y_results.csv` (594 mechanical screen signals, NOT the DB trades) by regime: RECOVERY looked best / PULLBACK worst, BUT RECOVERY's 24 signals = **1 market episode**, PULLBACK's 44 = 8 months, and all return CIs span zero except BULL. 5 years has too few independent regime episodes to calibrate. Confound: raw signals not gate-filtered. **Conclusion: regime is a risk overlay justified by logic, not a calibrated per-regime edge — same weak base undermines Annex 1's by-regime table.**

**DOC ARCHITECTURE DECIDED (2026-06-17): TWO docs by purpose.**
- `BOT_Comprehensive_Checklist.md` = canonical **OPERATIONAL** doc (act) — lifecycle Entry/Monitor/Exit, checkbox+table style (user loves this style). To be UPDATED with newer decisions (its Jan-2026 content is partly stale: VIX-regime, 25/50 DMA, IVP thresholds).
- `bot_strategy_trading_plan.md` = **RATIONALE/design** reference (why) — backtests, regime derivation, calibration. Checklist links to it with one-liners. Stop forcing it to be a checklist (that caused the convolution).
- Old `bot_strategy_checklist.md` DELETED + committed (superseded). Backups: `Strategies/Breakouts/_archive/*_20260617.md` + all .md now committed to git (plan was untracked, now tracked). Also a user-kept `bot_strategy_trading_plan v0.md`.
- **Process:** agree skeleton first → draft whole sections in checklist style → user reacts at section level → NO rationale in the operational doc. (Micro-edits caused the mess.)

**DECISION-3 reconciliation = CASE-BY-CASE, not newer-always-wins; pause when both may be wrong. Principles confirmed 2026-06-17, grounded in real trades (NVDA #735 225C 1-lot \$4.20; AMZN #736 270C 1-lot \$2.71; NVDA #718 205/215 spread +\$676):**
- Risk cap = **CHF 600 per BOT** (checklist's number is right; plan's \$400 is wrong → fix).
- **Dollar risk < full premium**: exit before option→0 while still sellable (option liquidity falls with delta) → exit ~15-20Δ, don't wait for 0/10Δ. Size by risk-to-planned-exit, not premium. (= the parked premium-vs-risk fix.)
- **Pay-for-entry → reframed "make the trade riskless if possible"** — an OBJECTIVE not a gate; achievable even 1-lot by legging (718: bought naked call, sold a call once breakout started → riskless). Not always achievable.
- **Minimum asymmetry (reward:risk, ~≥1.5-2:1) replaces the 6x-bagger / OTM-triple / <\$2 bar.**
- **Technical score = DIAGNOSTIC, not pass/fail gate** — tells where the stock stands (setup in place? breakout started? 1st vs 8th inning?) → understanding → trade classification → matching stop/target/vehicle. (Matches plan Annex "screener = discovery, not exclusive filter".)
- **Vehicle = vol-driven (keep newer plan logic):** cheap vol (low IVP / VRP<0) → outright; rich vol → spread.
- **PARKED (user thinking, no rush):** are poor-technical Fib/S-R + vol-edge trades (AMZN #736 had price<EMA50) really BOT breakouts or a SEPARATE setup type? Decide before encoding entry rules.

**SESSION-END 2026-06-17 — checklist rebuild started; RESUME at Section A4.**
- Done today: all .md committed to git (plan was untracked→now tracked; old `bot_strategy_checklist.md` deletion committed — superseded); archive copies in `_archive/*_20260617.md`. Created `ref_delta_mission_vol_surface.md` (vol-surface "buy cheap region / sell rich region" rule + long-BOT distilled menu). Ported the regime model into `BOT_Comprehensive_Checklist.md` A1 (replaced VIX table with our 200-SMA Trend×Depth + confidence; kept its sector-tier table + historical-P&L comment; VIX → informational).
- **Vehicle widened: puts ARE bullish expressions** (put-call parity) — long-BOT menu now: calls, call spreads, **bull put spreads** (sell rich ATM/downside vol), short puts, ITM calls (cheap downside) — picked by where the surface is rich/cheap.
- **Tool sources confirmed:** `/analyze` already computes the vol read (IVP/VRP, skew RR25Δ, term IV30/IV90, OI + bid/ask + chain_state, with LIVE/CACHED/NO-DATA status = the live-data guard). RPretrade Breakout tab = the **R/R-at-target BS engine** (`breakout_options.R: calculate_reward`) = our asymmetry/BS-grid calc. (RPretrade does NOT do term/skew/liquidity — that was a user misattribution; it's in /analyze.) Term-structure could later be automated from the option-surface table (`migrate_option_surface.R`) — park TODO.
- **NEXT (tomorrow), targeted updates preserving existing checklist detail:** (1) **A4 Vehicle** — vol read via /analyze (require LIVE) → surface-aligned vehicle incl. puts → delta 30–50 (not tool's 15–30) → risk-to-exit sizing + asymmetry ≥~1.5–2:1 (RPretrade R/R) → make-riskless → DTE. (2) **A3** add multi-timeframe (weekly setup/daily signal) + score-as-diagnostic/classification. (3) **A2** MaxRisk as DB param (currently CHF 600). (4) **B/C exit** reconcile Day-21 vs mid-life dated check; add liquidity floor ~15–20Δ. Parked: is an AMZN-style poor-technical Fib/S-R+vol trade a BOT or a separate setup type?

**Still pending (plan-side):** re-run `bot_retroactive_test.py` + 5y backtest on EMA50 (Annex 1 only caveated); run the short backtest (Empirical TODO); premium-vs-dollar-risk fix (Gate 4 max-loss = premium − value-at-stop, not premium; split the $2 outright gate into far-OTM convexity vs near-money risk-to-stop) — discussed, not yet written. Next doc target after Section 2: condense Gate 2 (section 4) to checklist form like Gates 0/1. See [[project_ma50_ema_switch]].

**Why:** ongoing review cycle, not derivable from code/git. **How to apply:** plan is longs-only; shorts have their own doc; the regime rewrite is the next doc edit awaiting go-ahead + thresholds.

**Why:** ongoing review cycle, not derivable from code/git. **How to apply:** plan is longs-only now; shorts have their own doc; offer the EMA re-run + short backtest as the next concrete tasks.
