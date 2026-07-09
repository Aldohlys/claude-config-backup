---
name: reference_ibkr_data_entitlements
description: "What the user's IBKR account can/can't pull — European historical blocked, options delayed-only; use Yahoo for history but not thin SIX/LSE lines"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 9086880f-22ad-4f6d-a57f-6eda3257a22b
---

Discovered by repeatedly hitting the limits (2026-06/07). Applies to both the claude.ai IBKR MCP and Tdata/ib_async on the user's account.

**No historical-data entitlement for European venues** — `EBS` (SIX Swiss), `IBIS` (Xetra), `SBF` (Euronext Paris). Historical bars/ticks return Error 162 "No market data permissions" or simply **time out**. Confirmed for ABBN, SIE, SU, and FXC.SW. Don't waste calls trying `reqHistoricalData`/`reqHistoricalTicks` on these — go to Yahoo.

**Options: no real-time entitlement, but DELAYED works.** MONEP (Euronext options) and Eurex quotes have no real-time sub → `getOptValue` (frozen, reqType 2) returns NaN / Error 354. Fix: set **`reqMarketDataType(4)` (delayed-frozen)** via ib_async, then `reqTickers` returns delayed bid/ask + modelGreeks (delta/IV). Used for SU (MONEP) and SMI OSMI (Eurex) puts. US options: fine in RTH, empty pre-open (no frozen support in the MCP).

**IBKR MCP quirks:** `get_price_snapshot` returns invalid IV + empty bid/ask on options when the market is closed; **combo/BAG orders do NOT appear** in `get_account_orders` (the IAU collar was invisible there but present in TWS). Live STOCK snapshots via MCP work fine (ABBN 87.22, SU 284.15, etc.).

**Price HISTORY workaround = Yahoo via the Tdata Python path — but not thin SIX/LSE lines.** Thinly-traded Swiss/London ETF lines (e.g. `CNYA.SW`, `FXC.SW`) have unreliable Yahoo *history* (compressed 52-wk range, garbage MAs). Their *current price* is fine; the *series* is not. For structure/ATR/MAs use the **liquid US-listed cousin** (US `CNYA` ~36, `FXI`) or `ASHR`, then convert to the UCITS scale by price ratio. Authoritative current price for a UCITS: IBKR MCP snapshot of its London/SIX line.

Related: [[reference_tdata_vs_ibkr_mcp]], [[reference_tdata_option_fetch_internals]], [[feedback_iv_solve_when_tws_returns_null]].
