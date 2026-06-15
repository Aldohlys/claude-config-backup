---
name: project_asymmetric_trade_thesis
description: User's trading philosophy driving the swing scanner design — confirmed-trend-plus-pullback over breakout detection, with option structure chosen from vol regime
type: project
originSessionId: 85e16fdb-97b2-4ddc-a82d-4a001be0eb4e
---
Durable thesis (established 2026-04-21, ships in Tdata 5.10.3 + RStudies scanner update):

**Core rule — "2nd/3rd inning" over "1st inning":** Enter confirmed trends with active pullback, not first-inning breakouts. Breakout detection has a high false-positive rate; trend continuation with a dip gives better entry + lower option premium.

**Why:** Option premium catches up to the thesis fast. By the 4th-5th inning, calls are too expensive and convexity is gone. Sweet spot = trend confirmed (stage 2, HH/HL, 50-150-200 MA stack) BUT pullback to 20 SMA / RSI retraced — gamma buyers haven't piled in yet.

**How to apply (scanner + analysis):**
- Filter via `Tdata::isTrendContinuation()` — the `Trend_Passes` gate in swing scanner
- Pullback must be detectable (at SMA20 OR RSI retraced from >65 to 40-55) — not just extended stage-2
- Option structure chosen from IV regime, not pasted on: IVR low + VRP near-zero → long calls; IVR high + skew steep → short puts / put-credit spreads; middling → call-debit spreads
- VRP = `log(iv30/rv30)*100` (log-ratio, not additive difference)
- IVR (1y range) AND IVP_2y (2y density) both matter — they diverge when IV history has outliers

**DTE preference (empirical, validated against 108 BOT trades Apr 2023 - Apr 2026):**
- 30-60 DTE = best per-trade economics (+$128 mean, +$3718 total on 29 trades in 40-60d)
- <20 DTE = highest win rate (50%) but tiny samples (n=8, n=12)
- 60+ DTE = all losers (5 trades), thesis horizon doesn't match option expiry
- Default target: 30-60 DTE for directional plays

**Earnings buffer (set 2026-04-21 for scanner TREND CONT rescue):**
- Minimum 10 days between today and next earnings to enter a new directional trade
- Rationale: near-term events load option IV with event premium → unfair entry price + post-event IV crush risk
- Implemented in `final_filter.R` trend-continuation rescue: promoted only if `EarningsInDays >= 10` (or NA)

**Scanner philosophy — discretionary, not automated:**
- Trend_Passes is info-only for existing TOP PICK / TRADE / WATCH tiers — user eyeballs
- Only the rescue path (SKIP → LONG WATCH) applies the trend filter as a gate
- "Wide net that is robust in different environments" — the user reviews and narrows to 1-2 trades/week; scanner surfaces candidates, doesn't pick

**Trade sizing discipline:** 1-2 breakout trades per week maximum — monitor cost + mental overhead is the binding constraint, not signal availability.

**Reference commits:**
- Tdata 5.10.3 (isTrendContinuation, VRP log-ratio, iv15/90, IVR, IVP_2y)
- RApplication@bf3ca95 + RStudies@6254dca (scanner integration + BOT reclassification)
