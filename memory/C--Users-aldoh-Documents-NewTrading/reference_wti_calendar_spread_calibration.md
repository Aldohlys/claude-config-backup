---
name: WTI calendar spread calibration bands
description: Reference levels for WTI 1-month calendar spread to read what backwardation premium implies — normal vs tight vs crisis
type: reference
originSessionId: 0544fcb3-a0ac-4de6-93f8-9e759ecc9335
---
**Reference bands for WTI 1-month calendar spread** (front - next, USD per barrel):

| Regime | Spread (1-month) | What it means |
|---|---|---|
| **Contango** (oversupply) | −$0.50 to +$0.10 | Storage cost dominates; market expects price to rise. Bearish current physical demand. |
| **Normal backwardation** | $0.10 - $0.30 | Healthy market, modest near-term tightness |
| **Tight** | $0.50 - $1.00 | Real near-term demand or modest supply tension |
| **Stress** | $1.50 - $2.50 | Elevated geopolitical / inventory anxiety |
| **Crisis-level** | $3+ | Physical supply panic (2022 Russia invasion, Hormuz threats, refinery outages) |

### Origin

Observed 2026-05-11: CL Jun = $97.58, CL Jul = $93.97 → Jun-Jul = $3.61. Iran-war headline; 7 days from Jun expiry. Crisis-band reading.

### Practical use

- A move from $0.30 → $1.00 within a week is a **regime shift** signal even if spot price didn't move much. Often leads spot direction by days.
- Crisis-level $3+ tends to mean-revert within 2-4 weeks once the catalyst is priced. Trade fades aggressively if news flow stagnates.
- Spread tightness > spot level for assessing trade urgency. Spot can hold while spread collapses — that's the early warning to scale out.

### Caveats

- **Expiry week distortion** (see `reference_futures_expiry_week_dynamics.md`): final 1-2 weeks before front expiry, convergence + roll buying contaminate front-vs-next. Use the *first non-expiring* spread (e.g., Jul-Aug, not Jun-Jul) when within 2 weeks of front expiry.
- Spread bands apply to WTI specifically. Brent runs tighter in normal markets but spikes harder on Middle East news (more sensitive to Hormuz/Suez).
- Storage capacity matters: at peak Cushing storage utilization, contango compresses (no room to store the spread arbitrage).
