---
name: reference_position_snapshot_uprice_unadjusted
description: "Position-snapshot tables (U1804173/U25343478/DU5221795) store UNADJUSTED uPrice - stock splits fabricate huge phantom moves in any historical analysis"
metadata:
  node_type: memory
  type: reference
---

The per-account position tables store a daily per-leg snapshot with `IV`, `delta`,
`gamma`, `vega`, `theta`, `uPrice`, `optPrice`, `multiplier` and **`TradeNr`**
(the join key to `Trades`). Coverage: U1804173 from 2022-10, DU5221795 from
2022-10, U25343478 from 2026-04.

**`uPrice` is NOT split-adjusted.** Across a split date the snapshot-to-snapshot
underlying move is the raw price gap, not the economic move. Found 2026-09-01
during BOT P&L attribution: one interval showed `du = -94.91` over 2 days
(AMZN/NVDA-class split), which through `0.5 * gamma * du^2` produced a **+69,094**
gamma term against **-484** actual - 88% of the whole gamma sum, and it pushed
the attribution residual to **-309% of actual**. With the guard, residual fell to
**-7%**.

**How to apply:** any historical computation over these tables that differences
`uPrice` (P&L attribution, realized-move stats, delta-hedge simulation) must drop
or adjust implausible intervals - `abs(du)/uPrice > 0.25` over <=2 snapshots is a
workable guard. A large near-cancelling pair of terms (huge gamma, huge opposite
residual) is the signature. Same class of bug as using raw `Close` instead of
`Adjusted` from Yahoo.

Other units settled at the same time: `IV` is **decimal** (0.32 = 32%); IBKR
`vega` is per **1 percentage-point** of IV, so `vega_c = vega * dIV * 100`
(calibrated empirically - residual 0.117 vs 0.190 for the per-1.0 convention);
`theta` is per calendar day. There can be **multiple snapshots per date** - dedupe
on `heure` before differencing.

See [[project_bot_gate_redundancy]] for what the attribution concluded.
