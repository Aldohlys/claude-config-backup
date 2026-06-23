---
name: Regime Backtest Results
description: Backtest completed 2026-03-19 — macro signals don't predict BOT outcomes; regime system replaced by simple VIX>25 warning
type: project
---

## Regime Backtest — Completed 2026-03-19

### Result: Macro signals have near-zero predictive power for BOT trades

Backtest of 98 closed BOT trades (2022-07 to 2026-03) against 10 macro signals:
- Best correlation: copper_gold at r=+0.161 (not statistically significant, p≈0.11)
- 96 of 98 trades classified as "Neutral" regime — no differentiation
- Sigmoid calibration produced no meaningful improvement over domain-knowledge defaults
- Signal quintile analysis showed mostly flat win rates across all signals

### Key insight
BOT success is driven by stock-level factors (TA quality, entry discipline, position sizing), not macro conditions. The loss streaks (Mar-May 2024: 8 losses at VIX 12-18) were caused by poor trade selection, not macro stress.

### One exception: genuine liquidity stress
The Apr 2025 streak (VIX 30.6, HYG -2.3%, copper/gold -18.8%) is the single case where a macro gate would have helped. But this is a rare event.

### Decision: replace regime system with simple VIX warning
- Removed sigmoid calibration effort
- Added `STRESS_BANNER` to macro_context report: flashing red banner when VIX >= 25
- The full regime machinery (scenarios.R) remains for sector flow analysis in the swing scanner
- Sigmoid parameters left as domain-knowledge defaults — no empirical basis to change them

### Remaining work
- [ ] Sector redefinition: oil services + shipping → Energy group
- [ ] Saxo COT Monday fetch workflow

**Why:** 98 trades across 10 signals showed no macro → BOT outcome relationship. Calibrating against noise would be overfitting.

**How to apply:** Don't invest further in regime parameter tuning for BOT. The regime system's value is in sector flow scoring for the swing scanner, not as a BOT trade filter.
