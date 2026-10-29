---
name: project_pnl_attribution
description: "Trade P&L path split into delta/gamma/vega/theta/residual/execution (TODO #106 done, Tdata 5.21.x); ONE shared box module in the routine Trade tab and RReporting Trades tab; how to diagnose a big residual"
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

**Display = one shared box module (2026-10-22):** `Tuser/symbol/view/pnlPathUI.R`, `server(id, tradeNr, account, asOf, currency)` (tradeNr: number shows it, NULL hides, a string is shown as the reason). Used by the routine Trade tab (displaytradeUI) and RReporting's Trades tab (dropdown trade wins; else selected rows with one TradeNr). Change the path display there, never in a host app. Chart: days <= 30 slots, else ISO weeks ("W14") while <= 30, else months; legend above the plot (below, it collided with the axis title). Table stays daily, 20/page, totals in the tfoot. Box caches modules: restart the app after editing.

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
