---
name: hv30-window-artifact
description: Trailing HV30 has a mechanical rollout — VRP signals during vol regime transitions can be fake
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8c65e4a9-c01b-4a86-95fe-f6292e3db0e8
---

# HV30 rolling-window artifact — VRP signals can be mechanical

IBKR's `HISTORICAL_VOLATILITY` is trailing-30-day realized. As big-vol days enter or leave the window, HV30 jumps by multiple vol-points in a single session **without any change in current realized vol**.

## XOP example (2026-05-27 trade analysis)

```
Date        IV30%   HV30%    VRP(IV-HV)
2026-05-13   32.8    44.5    -11.7 vp
2026-05-18   33.5    42.0     -8.5 vp
2026-05-19   34.1    42.1     -8.0 vp
2026-05-20   34.9    35.0     -0.1 vp   ← -7vp in one session
2026-05-26   35.1    35.4     -0.3 vp
```

The -7vp HV step on 2026-05-20 is a big daily move from ~30 sessions earlier dropping out of the trailing window — not a regime change. IV stayed sticky at 33-35% throughout. VRP went from "deeply cheap" to "flat" in a week with no real vol regime shift.

## Diagnostic

When you see VRP swing several vol-points in days without commensurate IV move:
1. **Suspect a rollout artifact.** Pull IBKR HV30 daily history and look for sudden step-changes.
2. **Cross-check with rolling RV5/RV10/RV20** on close-to-close log returns. If RV5 << RV20, recent vol is decelerating; if RV5 >> RV20, vol is rising and HV30 will catch up.
3. **The forward path matters more than the trailing snapshot.** A HV30 reading at a local low *during* a sell-off means recent daily moves haven't been absorbed yet — HV30 will rise from here, not fall.

## Implication for VRP-based signals

- Phase C.1 VRP component (max 2 pts) is window-fragile when regime is in transition.
- "VRP cheap" during a sell-off where HV just dropped from rollout is **not** an edge — IV will likely catch up downward when HV stabilizes.
- "VRP rich" during a quiet stretch right after a vol shock can also be artifact — HV is still elevated from the prior shock window.

## Why:
Discovered when [[xop-long-analysis-20260527]] showed VRP_log = -3vp ("cheap") while spot was in a -7.6%/5d sell-off. Forward-RV showed realized was actually accelerating, not decelerating — VRP-cheap was an artifact of a May-1 high-vol day rolling out, not an opportunity.

## How to apply:
When /analyze reports VRP_log <= 0 (cheap), check if HV30 had a step-change in the past 5-10 sessions. If yes, treat the cheap-vol signal as fragile and don't size up based on it. The fix is to surface RV5/RV10/RV20 in Phase C.2 alongside HV30 — currently a noted follow-up, not shipped.
