---
name: WSH event-data needs News Feed entitlement (Error 10276)
description: IBKR's "WSH fee waived" subscription alone does NOT grant reqWshEventData access — need separate News Feed entitlement
type: project
originSessionId: 1347083d-d9f2-4281-99ce-26cad155e9e3
---
**Finding (2026-04-20):** User's IBKR Client Portal showed "Wall Street Horizon — Separate Data Services — North America — Fee Waived" as active. Despite that, `ib.getWshEventData(wsh_filter)` consistently returned empty string with IBKR error:

> Error 10276, reqId X: News feed is not allowed.

**Why:** WSH event data flows through IBKR's news-feed infrastructure, which requires a *separate* entitlement (e.g. "Reuters News Basic" or similar). Holding a WSH calendar subscription alone doesn't grant event-data API access.

**How to apply:**
- Before suggesting WSH-based earnings fetch for this account, check whether a News entitlement is active in Client Portal → Settings → Market Data Subscriptions
- If not, fall back to yfinance or another free earnings source — don't spend time debugging the WSH Python code
- The implementation in `Tdata/inst/python/tdata_py/earnings_utils.py` is correct (uses blocking `ib.getWshEventData` / `getWshMetaData`, not the fire-and-forget `req*` variants). The blocker is pure subscription, not code.
- ib_async 2.1.0 WSH API surface: `ib.getWshMetaData()`, `ib.getWshEventData(data)`, `ib.wshEvent`, `ib.wshMetaEvent` (NOT `wshEventDataEvent`)
