---
name: reference-orders-tab-matching
description: "Tuser routine Orders tab stores no order-trade link; it matches live TWS orders to the latest portfolio snapshot by (account, symbol) on each Reload, TradeNr from the snapshot; a held position with no Trades row breaks it"
metadata:
  node_type: memory
  type: reference
  originSessionId: e87e0ecc-b1ff-410a-a8f3-4449c3418f84
  modified: 2026-10-08T08:31:00.776Z
---

Tuser routine → Orders tab (`order/view/openordersUI.R` → `Tdata::getTradeOrderCoverage`, `Tdata/R/orders.R`). Orders are never persisted: on Reload it reads the live resting orders (`tdata_py.orders.get_open_orders`), the latest portfolio snapshot (table `U1804173` / `U25343478`, max date then max heure) and the active trades. It joins on (Account, Symbol); an exact leg match beats a root match. Flags: NO ORDER (held, nothing working), UNMATCHED (order on nothing held), AMBIGUOUS.

So "associate new orders with positions" needs no action: cancelled and re-entered orders attach automatically. What breaks the link is a **held position whose snapshot TradeNr is empty** because it has no Trades row (2026-10-07: 6382, 9845, the SPY 722/712 spread). The fix is to book the trade (RReporting Flex import); then the snapshot carries the TradeNr.

Checking from Claude:
- `Rscript` calling `getTradeOrderCoverage` hung for more than 3 minutes on 2026-10-07 with TWS up.
- Calling Python `tdata_py.orders.get_open_orders(None)` directly (conda, `timeout 90`) returned all 30 orders in about 10 s, across both accounts.
- The IBKR MCP connector shows only U1804173 orders, positions and fills.

Related: [[reference-tuser-repo]], [[reference-trades-risk-conventions]].
