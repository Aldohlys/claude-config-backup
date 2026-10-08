---
name: reference_position_snapshot_uprice_unadjusted
description: Position-snapshot tables (U1804173/U25343478/DU5221795) store UNADJUSTED uPrice - stock splits fabricate huge phantom moves in any historical analysis
metadata:
  node_type: memory
  type: reference
  originSessionId: 3f7efce0-fd27-406b-b771-59dc993e4d3e
  modified: 2026-10-08T10:04:13.114Z
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

**More traps, found 2026-10-08 running the attribution over 386 closed trades
(all handled in `Tdata/R/pnl_attribution.R`, TODO #106):**
- **Missing Greeks** come as `delta = NA` with `vega = theta = -2` (IBKR
  placeholder: a negative vega is the tell, theta can legitimately be -2), or as
  an all-zero row with `uPrice = 0`, `IV = 0`. Recompute from IV + uPrice with
  `Tbasics::getBSOptGreeks` (same units as IBKR) or skip the snapshot.
- **IBKR theta is sometimes 10-25x the model's** while delta and vega agree:
  ESTX50 snapshots after the Eurex close (trade 706: -25.3/day vs BS -0.98).
  1.35% of 2025-2026 option rows, 39 of 43 ESTX50. Times calendar days over
  a long weekend it fabricated -2,736 theta / +2,600 residual on one interval.
  Check against `getBSOptGreeks` from the same IV + uPrice; correct only
  overstatement (> 3x). IBKR far *below* BS (16% of rows) is SOFR3 futures
  options, where BS is the wrong model, not an IBKR error.
- **`uPrice` can be NA or frozen next to live marks**: pre-market snapshots
  (and some intraday ones) keep the previous `uPrice` while option marks move
  (trade 753: 08.10 10:01 marks +50%, 02.10 18:26 marks -32%, uPrice
  unchanged). Drop a snapshot whose legs have exactly the last usable uPrice
  while marks moved > 5% (Tdata 5.21.3). Restricting to post-close snapshots
  instead was tested and is worse (too few days left).
- **Stale TradeNr:** later positions in the same contract keep an old trade's
  TradeNr (trade 386: rows two weeks after its close; 587: an unrelated 2026
  spread). Keep only legs the trade's fills name, and for a closed trade drop
  snapshot days after its last fill.
- **Names differ between fills and snapshots** for some contracts (CL futures
  option `CL NOV25 60 P` vs `CL 16OCT25 60 P`; SPX `20DEC24` vs `19DEC24`), so
  an Instrument join misses them.

See [[project_bot_gate_redundancy]] for what the attribution concluded, and
[[reference_trades_datetime_utc]] for placing fills against snapshot times.
