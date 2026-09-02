---
name: reference_ibkr_open_orders_semantics
description: "Live open-order retrieval from TWS — PreSubmitted vs Submitted, whyHeld, permId, DBL_MAX sentinel, Adaptive algo, combo leg inversion"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1e2ceadf-edaa-4f9c-82d1-d1e2a412227d
  modified: 2026-09-01T08:56:41.809Z
---

`Tdata::getOpenOrders(account)` (Tdata 5.15.0+, `inst/python/tdata_py/orders.py`) is the canonical live resting-order fetch: `ib.reqAllOpenOrders()` then `ib.openTrades()`. Don't re-implement it — `scripts/sync_stop_risk.R` used to inline its own `py_run_string` block and now consumes this. Verified against live TWS 2026-09-01 (29 orders, 3 accounts).

**Status semantics — the one that looks like a bug and isn't:**
- `Submitted` = live **at the exchange**, in the order book.
- `PreSubmitted` = accepted by IBKR but **held in IB's system**, not routed to the exchange.
- A **stop is a simulated order type**: IBKR holds it and only transmits it when the trigger price is touched (that's what keeps it invisible to the market). So *every* resting STP/TRAIL reports `PreSubmitted` with `orderStatus.whyHeld == 'trigger'` — its normal, permanent state, fully protecting the position. Never present this as "pending" or a coverage gap.
- `whyHeld` is empty for other held orders (closed session, IB algo engine); observed but not explained — don't assert a cause. MRD's LMT was `Submitted` while its STP was held, so "market closed" is not a clean rule.
- Displayed as `State`: `Working` / `Held: trigger` / `Held`.

**Field traps (all cost real debugging):**
- `orderId` is **0** for any order placed from TWS itself or another API client. `permId` is the only stable identifier.
- Unset numerics arrive as **DBL_MAX** (`1.7976931348623157e+308`), notably `trailStopPrice` on a plain LMT. Filter at the boundary — see [[feedback_no_nan_to_tws]] for the mirror-image rule.
- **`orderType` never says "Adaptive".** TWS shows "Adaptive LMT (IBKR)" but `orderType == 'LMT'`; the algo is in `order.algoStrategy`. Read both or the display disagrees with the order window.
- Operative price: TRAIL → `trailStopPrice` (fallback `auxPrice`), STP → `auxPrice`, LMT → `lmtPrice`. Test STP before LMT so "STP LMT" reports its trigger.

**Combos (BAG):** `contract.symbol` is the root (`"BRK B"` — see [[reference_ibkr_symbol_with_space]]), `localSymbol` is a meaningless combo id, `tradingClass == 'COMB'`. Legs carry only `conId`; resolve each with `ib.qualifyContracts(Contract(conId=..., exchange=...))` (works, ~instant). **Leg actions are expressed relative to BUYing the combo, so a SELL of the combo inverts every leg** — a SELL of a `+1 520C / -1 525C` bull spread is SELL 520C + BUY 525C. `getOpenOrders` expands a BAG to one row per leg with the effective side in `legAction`.

Consumed by the routine app's **Orders** tab (`Tuser/order/view/openordersUI.R`). Related: [[feedback_open_positions_from_portfolio_not_trade_status]], [[reference_isIBAvailable]].
