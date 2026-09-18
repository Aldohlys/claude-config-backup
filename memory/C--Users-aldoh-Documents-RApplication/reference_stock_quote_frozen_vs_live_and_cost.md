---
name: reference_stock_quote_frozen_vs_live_and_cost
description: "For stocks, reqType=2 (frozen) matches live to the cent — the frozen trap is option-specific; the real staleness is the Prices DB table, and each getStockPrice() call costs ~12 s"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 81e8468e-53c1-4b02-9a9c-9c1f7f2651b6
  modified: 2026-09-03T18:51:31.320Z
---

Measured 2026-09-03 13:07 CEST (US pre-market, TWS up):

| symbol | frozen (reqType 2) | live (reqType 1) | `getStoredMetrics()` (DB) |
|---|---|---|---|
| SPY  | 765.23 | 765.16 | — |
| AAPL | 324.34 | 324.34 | **314.44 @ 20260831 19:50** |

**Frozen is not a staleness source for underlyings.** `determine_req_type()`
(`Tdata/R/prices.R:13`) defaults to `2`, and that is fine for STK. The frozen
trap in [[reference_option_quote_liquidity]] is specific to **option bid/ask
width**, where frozen quotes look healthy and mean nothing. Do not generalise it
to spot prices — I nearly chased it and it was not the bug.

**The real staleness is the `Prices` table.** It lagged 3 days above.
`getStoredMetrics()` reads it; `getStockPrice()` with TWS reachable fetches live
*and appends the fresh row to `Prices`*, falling back to the last stored row
when TWS is down. So `getStockPrice()` is the honest call and
`getStoredMetrics()` is the cached one — the names do not say so.

**Cost: ~12 s per `getStockPrice()` call.** Own IB connect + `qualifyContracts`
+ `reqTickers` + `ib.sleep(0.5)`; `reqTickers` alone was ~11 s of it. Budget for
this — anything per-keystroke or on a short timer is not viable, and a
click-through test that changes symbol twice takes minutes.

**Provenance is derivable without a second probe:** `getValue()` stamps
`datetime` with the fetch time, a DB fallback carries the stored row's own
(older) timestamp. Comparing it to `Sys.time()` tells you which path ran,
cheaper than calling `isIBAvailable()` again.

RPreTrade now pays the 12 s on every debounced symbol change: its sidebar
Current Price is fetched live, and `symbol_manager.R`'s old "only refresh if the
stored row is more than a day old" tolerance — which accepted yesterday's close
as current across Tabs 1-3 — is gone.
