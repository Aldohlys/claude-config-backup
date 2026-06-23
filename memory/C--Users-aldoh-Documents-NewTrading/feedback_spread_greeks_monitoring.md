---
name: Compute net Greeks before proposing spread monitoring triggers
description: Don't copy outright-call monitoring (IV-crush triggers, vega warnings) onto debit/credit spreads without recomputing net Greeks — the short leg offsets
type: feedback
originSessionId: 10fa004f-7f11-4a79-88ea-5b9c8dc47262
---
# Rule

Before proposing monitoring triggers or risk warnings for a multi-leg option structure, **compute net Greeks from all legs** rather than importing monitoring logic from a single-leg structure. Vega and theta on spreads are meaningfully offset by the short leg — warnings that apply to outright calls/puts often don't apply to spreads.

**Why:** on the JPM May 15 $330/$340 debit call spread (2026-04-24), I flagged "IV-crush trigger" as a monitoring risk. User correctly pushed back: for a debit spread, the short leg offsets most of the vega, so IV compression is a much smaller risk than it would be on an outright call. I had imported outright-call thinking onto a spread structure without recomputing net Greeks. The error was systematic, not arithmetic — the kind of thing that recurs unless explicitly guarded against.

## How to apply

### 1. Always decompose net Greeks before writing a monitoring plan

For any spread, estimate net vega and net theta from each leg before proposing risk triggers:

```
net_vega  = long_leg_vega  − short_leg_vega
net_theta = long_leg_theta − short_leg_theta   (signs already built in)
```

Long leg typically has larger |Greeks| than short leg, so net is in the long-leg direction but reduced. For a typical OTM debit call spread, net vega is often ~40–60% of the long-leg-alone vega.

### 2. Reference mapping — what matters for which structure

| Structure | Primary chop/time risk | IV-crush risk | Monitoring focus |
|---|---|---|---|
| **Outright long call/put** | Theta (high) + Vega (high) | **Real** — IV drop hits directly | Both theta and vega triggers |
| **Debit vertical spread** | Theta (reduced) | **Small** — legs offset | Theta/time stop only; vega usually not needed |
| **Credit vertical spread** | Early assignment, tail gap | Small | Pin risk, gap risk |
| **Calendar/diagonal** | Path + IV shift | **Large** — intentional vega exposure | Vega and term-structure triggers essential |
| **Straddle/strangle (long)** | Theta (very high) + Vega | **Real** | Both; especially earnings cases |
| **Iron condor** | Gamma near short strikes | Moderate | Delta/breakeven triggers |

### 3. Edge cases where spread ≈ outright (and monitoring should reflect that)

- **Very wide spreads** where the short leg is nearly worthless → behaves like outright long, vega/theta back in play
- **Earnings-containing expiry** → event premium collapses in one session, net vega ≠ small
- **Deep-ITM long + far-OTM short** → net Greeks dominated by long leg, nearly outright
- **Spreads with legs in different expiries** (calendars/diagonals) → rules invert entirely

### 4. User heuristic to confirm

When in doubt, show the user the net-Greek calculation (at least rough numbers) before proposing a monitoring trigger. Lets the user cross-check and flags any cases where the short-leg offset is weaker than expected.
