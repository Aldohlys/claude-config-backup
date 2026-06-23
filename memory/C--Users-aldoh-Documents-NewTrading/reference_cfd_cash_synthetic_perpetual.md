---
name: CFD "cash" / "spot" instruments are synthetic perpetuals, not physical prices
description: Retail CFD providers (BLACKBULL, IG, etc.) build "cash" instruments by rolling weighted exposure between CME futures months; price tracks the roll algorithm, not any physical/cash market
type: reference
originSessionId: 0544fcb3-a0ac-4de6-93f8-9e759ecc9335
---
**CFD "cash" / "spot" instruments are synthetic perpetuals**, not quotes from any physical market. They are constructed by continuously rolling exposure between CME (or other exchange) futures contracts. The "cash" label is marketing — there is no underlying cash market trade.

### How the construction works

- Provider holds weighted exposure across the front contract and the next-month contract
- As expiry approaches, weight migrates from front → next-month over ~10-14 days
- ~7 days from front expiry: typically 80-85% rolled to next month
- Result: synthetic price = w·front + (1-w)·next-month + small financing/carry adjustment

### Empirical example (BLACKBULL WTI cash, 2026-05-11)

- BLACKBULL WTI cash CFD: **$94.515**
- CME Jun WTI: **$97.58** (front, 7 days to expiry, $3.61 backwardation premium)
- CME Jul WTI: **$93.97** (next month)
- Implied weighting: **~15% Jun / 85% Jul** → 0.15·97.58 + 0.85·93.97 = $94.51 ✓

The $3 gap to Jun is NOT lag, stale data, or physical-vs-futures divergence — it's the structural difference between a back-weighted synthetic and a front-month with crisis-level backwardation.

### Practical implications

**Good use:** real-time 24/5 directional proxy for the back-weighted exposure (i.e., the contract you're rolling INTO). For a position in July, the CFD tracks July closely with a small premium.

**Bad use:** as a reference for physical/spot price. CFDs will never show physical absorbing news because they're already mostly rolled out of the affected month. To track actual physical convergence, use Bloomberg/Argus/Platts assessments or CME settlement prints (fixed-time updates during US hours).

**Bad use:** as a reference for front-month panic premium. CFDs systematically under-represent expiring-contract spikes because they've rolled out of them.

### Heuristic for the weighting

- 14+ days to front expiry: ~100% front
- 7-10 days: ~50% / 50%
- 3-5 days: ~10% front / 90% next
- Post-expiry: 100% next-month

Vendor algorithms differ — verify empirically by solving w from observed prices when in doubt.
