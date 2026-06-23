---
name: reference_atr_move_multiples
description: Validated ATR%→N-day move multiples + pay-for-entry frequency by beta; gauge if any ticker can produce a monetizable breakout move
metadata: 
  node_type: memory
  type: reference
  originSessionId: 2bd4384e-3b59-46f3-a4cf-4b6e7212fea0
---

Empirically validated 5y daily across 12 tickers spanning 1.2–4.6% ATR (report: `Reports/atr_move_heuristic_validation_20260608.md`; script `Strategies/Breakouts/atr_move_validation.py`). Use **ATR14/price** as the beta proxy to estimate a name's directional move and judge vehicle fit.

**Scaling (random walk, confirmed):** N-day move ≈ daily move × √N. Measured σ₁₀/σ₁ = 3.11 vs √10 = 3.16. ATR ≈ **1.3×** close-to-close daily σ.

**10-day move multiples (× ATR%):**
- **Typical (median): 1.5×** ATR%  — general form 0.48 × ATR% × √N
- **Strong (p90): 4×** ATR% (~3.6× low-beta, ~4.4× high-beta; fat tails) — general form 1.25 × ATR% × √N
- The "√10 ≈ 3×" shortcut = an ~85th-pctile move, **NOT the base case**. Typical move is half that. (This corrected my own loose framing.)

**Sector calibration of C** (5y, 8–10 names/sector). The big cross-sector *vol level* gap (tech ~2× staples' daily range) lives in ATR% (the input), NOT in C — C is the normalized shape, far more universal. Driver of C = trend/gap/event-density, not beta (Financials cluster with Tech, not defensives):
```
                  C_med   C_p90(strong)
Tech / Financials  0.52      1.35   (fatter tails — bump strong ~10%)
Healthcare/Energy  0.47      1.23
Staples            0.46      1.18
```
Default 0.48 / 1.25 is accurate to ~10% in any sector; C_med spread is only ±6%, C_p90 (tail) is the more sector-dependent one. Nudge up for tech/financials, especially the strong-move coefficient.

**Pay-for-entry frequency** — P(≥6% move in 10d), proxy for "an OTM call can ~triple":
- ~1.5% ATR (PG/KO/JNJ/PEP): **6–8%** → long calls structurally lose (theta ~93% of windows); use shares or SELL premium
- ~2% ATR (ABBV/MRK/TMO): 18–22%; UNH 2.2%: 25% → spreads / pay-for-entry discipline
- ~4% ATR (NVDA/TSLA): **53–58%** → long OTM calls in sweet spot

**Realized-trade backtest (104 BOT trades, 2026-06-08, `bot_atr_validation.py`) — ATR is a PRIOR + sizing input, NOT a hard gate:** Winners richer-ATR (median 2.94 vs losers 2.33); high-ATR bucket carried +\$8,056 / 10 of 20 biggest wins (✓ aggregate). BUT low-ATR options won **47%** (+\$4,098) incl. AAPL OTM call +\$1,346 & put +\$1,277 at ~1.6% ATR — the 6–8% payable stat is *unconditional*; post-signal + skilled near-expiry strikes beat it, so low-ATR ≠ untradeable. ATR% conflates *quiet* names (PG/KO/PEP, never traded) vs *catalyst-mobile megacaps* (AAPL — episodic movers); don't reject the latter. **Real graveyard = MID 1.7–3% bucket (~32% WR, net negative), not low.** Use: rich ATR → size up; low ATR → need real signal + good strikes (megacaps OK); mid → danger zone.

**Application:** ATR14/price → typical (1.5×) & strong (4×) move at any horizon (√N) → read payable bucket for vehicle choice. This is the quantified version of the [[bot_strategy_checklist]] "Sufficient beta" filter; pairs with [[feedback_classify_breakout_vs_meanreversion]] (low ATR + compressing vol = SELL vol, not buy convexity — e.g. ABBV vs JNJ, 2026-06-08).

**Coefficients are quantiles of ONE distribution, not a per-ticker choice.** `0.48·ATR%·√N` = median (50th pctile), `1.25·ATR%·√N` = p90 (strong, top-decile). Both apply to every name at once; the ~2.6× gap IS the move-distribution spread (and the positive skew you buy with long options — don't collapse it). Per-ticker differentiation already lives in ATR% (the input); C is the near-universal shape. Which quantile to use is decision-driven: median → primary target/R:R; p90 → convexity justification / "can an OTM call print." Self-consistency check: 4% ATR → median 10d ≈ 6% → P(≥6%)≈50% (table 53–58% ✓); 1.5% ATR → p90 barely 6% → P(≈6–8% ✓).

**Cross-asset validity envelope (22 instruments, ~8y daily, N=3→60; report `Reports/atr_move_envelope_20260609.md`, script `Strategies/Breakouts/atr_move_envelope.py`):** coefficients TRAVEL well across equities/ETFs but have a 3-edge envelope. Grand mean C_med20=0.48 (on target), C_p90≈1.30 (so 1.25 was mildly conservative — use **1.30** as broad p90 center).
- **HOLDS:** US/EU/JP single-stocks + equity & commodity ETFs, ATR 1–5%, horizon **3–25 sessions**. C_med 0.45–0.55, C_p90 1.25–1.55. Shape survives EUR/JPY/GBP translation.
- **FX FAILS — recalibrate ~0.36 / ~0.95** (EURUSD): thin-tailed + mean-reverting, heuristic OVER-predicts strong moves ~25–30%; never size FX convexity off 1.25×.
- **Futures ~0.41 / 1.13** (CL=F, +continuous-roll caveat); term-structure work dominates anyway.
- **Horizon >~25 sessions: √N breaks both ways** — trending assets exceed it (SPY/QQQ/NVDA scale 1.3–1.5, NVDA C_med 0.54→0.71 at 60d → add drift uplift); post-parabolic high-ATR names mean-revert below it (MP scale→0.76, C_med→0.33 at 60d).
- **ATR/σ ratio = 1.13 pooled, ranges 0.86–1.57 by asset** (gappy equities ~1.1–1.3, continuous FX/cmdty ~1.0). NOT the assumed ~1.3 → another reason to measure ATR directly vs routing through σ√T (σ route needs asset-specific ATR/σ conversion anyway).
