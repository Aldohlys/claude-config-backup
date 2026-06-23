---
name: TODO — split swing_scanner into two timescales (candidate pool vs daily trigger)
description: Refined edge thesis: stocks in trend where options don't yet reflect it. Phase B = slow-moving candidate identification (reviewed weekly). Phases C/D = daily action signal among candidates. v5 currently runs everything same-day, conflating thesis durability with entry timing.
type: project
originSessionId: 9cbc474c-d67a-496b-92d1-fb42a1346715
---
## Status: Future redesign (opened 2026-04-27)

## The edge, articulated

> Identify stocks **in trend or starting to trend** where **option prices do not yet reflect the trend**, then **buy cheap / sell expensive**.

The asymmetry comes from a timing gap: real-money flow into the stock is starting (or already underway), but the options surface hasn't caught up — IV hasn't expanded, skew hasn't tilted, premium hasn't been bid up. That's the "front-running flow that hasn't been paid for" window. By definition it's a **transient** condition: as more participants notice, IV expands, skew steepens, the edge closes.

## Two timescales, two questions

| Question | Cadence | Phase | What it answers |
|---|---|---|---|
| Is this stock a *candidate* — i.e. trending or starting? | Slow (weekly review of the pool, stable membership) | **B** (Pull score) | "Should I be watching this name at all?" |
| Is *today* the day to act on a candidate? | Daily (re-evaluate every session) | **C / D** (cheap_score + setup/chain/R:R) | "Are options still cheap AND is the entry premium IN BAND?" |

v5 currently re-runs all phases A→B→C→D→E every day and re-ranks the whole universe, conflating "thesis is intact" (which doesn't change much day-to-day) with "today is a tradeable entry" (which DOES change with vol mean reversion, gap-up moves, etc.).

## Implications of the split

**Phase B → Candidate Pool**
- Membership turnover slow — measured in weeks, not days
- A name leaves only when the trend definitively breaks (stage drops to `none` for >N days, sector RS rank collapses out of the actionable band, or footprint reverses)
- Could be persisted across runs in a `Candidates` table with `entered_at`, `last_confirmed`, `exit_reason`
- Reduces noise: a name that was a candidate yesterday and is still trending today shouldn't get re-litigated; only the *exit* condition matters
- Candidate pool size stays ~stable (maybe 20-40 names at any time across the 130-symbol universe)

**Phases C / D → Daily Action Trigger**
- Run daily on the candidate pool only — much smaller universe, much faster
- Fires "actionable today" only when:
  - cheap_score still ≥ 6 AND cheap_side still aligns with pull_direction (vol still cheap),
  - AND entry_state = IN BAND (premium hasn't run up),
  - AND chain isn't structurally capped below structural target
- Most days for most candidates: nothing fires. That's expected — the entry window is rare on purpose
- An action firing on a candidate signals a real "today is the day" — it's not "this stock is interesting", it's "the gap is open right now, take it"

## Why this matters operationally

1. **Reduces false sense of urgency.** Today's CSV listing 30 names doesn't mean 30 trades — it means 30 candidates of which 2-3 might have an action signal.
2. **Captures the asymmetry decay.** The edge closes as IV catches up. C/D firing tells you the window is still open; absence of C/D firing on a candidate tells you it's already been priced in (move on, even if technically still "in trend").
3. **Aligns with how the trader actually operates.** You don't re-evaluate every name every day. You watch a pool, and react when the trigger fires for one of them.
4. **Cleaner backtest design.** Backtesting the candidate-pool stability is a separate exercise from backtesting trigger timing — and the two have very different statistics (turnover, hit rate, avg holding period). Mixing them in v5's monolithic scoring obscures both.

## Relation to existing redesign memo

`project_swing_scanner_redesign.md` (2026-04-27, three-axis Pull/Cheap/Setup-Chain-RR) describes the *axis* structure but keeps the same-day evaluation model. This TODO refines THAT redesign by adding a **time-axis** decomposition on top of the score-axis decomposition.

Likely a v6 design, not a v5.x patch.

## Open questions before implementing

- Persistence layer: SQLite table `Candidates` vs append-only parquet vs in-memory rebuild from history? (Probably SQLite for queryability.)
- Entry/exit criteria for the pool itself: what *promotes* a name into the pool, what *evicts* it? Thresholds need calibration on historical data
- How to treat the C/D trigger: a separate report focused only on candidates with action today, vs a "candidate dashboard" that shows pool + status?
- Action-trigger persistence: log when a trigger fires to enable hit-rate tracking ("of N triggers, M led to entries, K of those won")

## The central trade-off — trend clarity vs option pricing

The edge is **inversely correlated with trend obviousness**:

- **Obvious trend** (SMH 2026-04, DBA 2026-04, NVDA, etc.) → options expensive (IVP top decile, positive call skew, rich VRP). The trend IS priced in. No buy-cheap edge.
- **Ambiguous trend** (early stage-2, first higher-low, sector RS just turning, light-volume breakouts) → options still cheap (IVP middle, neutral skew, VRP near zero). The trend MIGHT be real but most people haven't noticed. **This is the only sweet spot.**
- **No trend** (chop, base-building, range-bound) → options cheapest, but no underlying signal — paying theta against random walk. Negative edge.

So the candidate pool can't just be "stocks in clear uptrend" — that's the SMH trap. It has to be **"stocks where trend is plausible but not yet consensus"**, which is harder to define precisely.

### Design implications

1. **Candidate criteria must include a "not-yet-obvious" filter** — explicitly demote names where the surface has already paid for the move:
   - Reject candidates with IVP > ~70 + positive call skew + VRP > ~30 (log-ratio) — even if trend signals are perfect
   - These names "graduate out" of the candidate pool when their options get bid up
   - That's not failure, it's the pool doing its job: the asymmetry has decayed and we move on

2. **The pool naturally has high false-positive rate.** Many ambiguous-trend names will never resolve into a real trend. Accept this:
   - Position sizing must reflect it (small premium, capped loss)
   - Hit rate < 50% is expected; the edge is in payoff asymmetry, not win rate
   - This is consistent with the empirical R:R calibration (R:R_min = 0.5 from `project_rr_calibration_result.md`) — wins are not enormous, and losses on un-resolved trends must stay small

3. **Vol cheapness is the gate, not the sweetener.** Phase C is what discriminates "ambiguous trend with edge" from "ambiguous trend without edge". Without cheap options, an ambiguous trend setup is just a low-conviction directional bet — no asymmetry. With cheap options, the same setup becomes "tail-event protection at fair price" — which IS asymmetric. So the C/D daily trigger isn't optional polish on a B-driven pool; it's THE thing that makes the candidate worth trading.

4. **Pool composition will skew toward names "almost nobody is watching".** Beautiful charts get expensive options. Ugly-but-improving charts (post-correction names retesting MA50 from below, sector laggards starting to outperform, fallen-angel names with stabilizing flows) are where the cheapness persists. This may feel uncomfortable — picking against the grain of the obvious leaders — but it's where the math works.

5. **Calibration target.** The right test for the pool isn't "did the names trend?" — many won't. It's: **for the subset where the daily trigger DID fire (C/D in band), what was the realized R:R distribution?** That's the only honest measure of edge.

### Worked example of the trade-off

Two hypothetical Phase B candidates on the same day:

| | Name A (SMH-class) | Name B (ambiguous) |
|---|---|---|
| Trend signal | Clear stage-2, RS top-3, footprint 3/3 | Stage-2 ambiguous, RS rank 6, footprint 2/3 |
| IVP | 88 | 35 |
| VRP (log-ratio) | +45 | +5 |
| Call skew | positive (rich) | neutral |
| Pull score | 9/10 | 6/10 |
| Cheap score | 1/10 | 7/10 |
| **Edge** | None (priced in) | **Yes (front-running)** |

v5's monolithic scoring would rank Name A higher because Pull dominates the composite. The two-timescale redesign correctly admits BOTH to the candidate pool but only Name B will ever fire a C/D action signal — because Name A's options are already paid up.

## Trigger to revisit

Next time the swing_scanner produces a long Trade/Watch list and the question "which of these should I actually look at *today*" becomes hard to answer. That's the symptom this redesign treats.
