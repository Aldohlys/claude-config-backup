---
name: project_pnl_attribution
description: "Trade P&L path split into delta/gamma/vega/theta/residual/execution (TODO #106, Tdata 5.21.x, routine Trade tab); how to diagnose a big residual"
metadata:
  node_type: memory
  type: project
  originSessionId: 3f7efce0-fd27-406b-b771-59dc993e4d3e
  modified: 2026-10-08T10:49:08.244Z
---

Built 2026-10-08 (TODO #106). `Tdata::getTradePnLAttribution(trade_nr, account, as_of)`
→ pure `attributePnL()` in `Tdata/R/pnl_attribution.R`; display in the routine
app Trade tab ("P&L path": grouped daily bars, weekly above 30 slots, daily
table with Flag tooltips). Exit intervals need `Trades.UnderlyingPrice`
([[reference_trades_underlyingprice]]).

**Validation baseline:** closed BOT trades on U1804173 — median |residual| ≈
12% of the first-to-last-mark P&L (study S-1 in TODO #82 measured 11.7%).
2024-2026 closed trades reconcile to their `Trades` total except contract-name
mismatches (CL futures options, SPX 19/20DEC24).

**Why:** the user reads the residual as a quality signal and asks "why is it
so large" trade by trade (706: theta/residual −254%/+207%; 753: 58%).

**How to apply — diagnosing a large residual:** the signature of bad input is
a residual that is large AND opposite to one component, or that reverses the
next day. Check, in this order, the snapshot traps in
[[reference_position_snapshot_uprice_unadjusted]]:
1. a Greek term far larger than the price move allows → IBKR Greek outlier
   (706: theta 25x Black-Scholes; compare with `Tbasics::getBSOptGreeks`);
2. `uPrice` identical to the previous snapshot while marks moved → frozen
   underlying (753, pre-market snapshots);
3. |dS|/S > 25% → split.
Test any new guard on 706 / 753 / 752 AND on all closed BOT trades before
shipping: a rule that fixes one trade can double the median (the post-close-
only snapshot rule did: 12% → 32%). Fills placed against snapshot times use
UTC ([[reference_trades_datetime_utc]]).
