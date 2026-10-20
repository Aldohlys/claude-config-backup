---
name: reference_ibkr_data_entitlements
description: "IBKR entitlement gaps and how they fail - no fundamentals (Error 10358, use yfinance); an unsubscribed instrument returns price 0, NOT NaN (Error 354)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 6175419f-570b-4d69-aee4-49e4b7c9ced2
  modified: 2026-10-20T14:58:01.874Z
---

Tested 2026-09-29 (TODO #92) on AAPL, TWS up: `ib.reqFundamentalData(contract, rep)` for `ReportsFinSummary`, `ReportSnapshot` and `RESC` all return empty with **Error 10358, "Fundamentals data is not allowed"**. The account has no fundamentals subscription; do not build on IBKR for P/E, EPS or estimates.

What does work (yfinance 1.2.0, miniconda python): `Ticker(s).info` gives `trailingPE` / `forwardPE` / `trailingEps` for US and European stocks and `trailingPE` for ETFs (SPY, IWM, QQQ, XL*, SMH; no forward for ETFs, convention for loss-makers undisclosed). `quarterly_income_stmt` only reaches back 5-6 quarters. `get_earnings_dates()` needs the `lxml` package, not installed in miniconda base.

**An unsubscribed instrument returns price 0, not NaN** (Error 354, "Requested market data is not subscribed"). Probed live 2026-09-22: `getValue(reqType=4)` on NUCL, DTLA and CNYA (all LSEETF) returned a plain `0`; TRE7, same exchange, returned 36.51 — so it shows up as a partial failure, not an outage. `getGonet` tested `is.nan(price)` alone, so the 0 was taken for a real quote: `mktValue` became 0, the whole cost read as a loss, and `NetLiquidation` lost those positions. Fixed in Tdata 5.20.1 — `gonet_price_missing()` treats any **non-finite OR non-positive** price as missing.

**How to apply:** never test a fetched IBKR price with `is.nan()` / `is.na()` alone; `<= 0` is the one that fires. When falling back to a last known price, search back for a snapshot with `price > 0` and **skip rows already zeroed by this bug**, or the position freezes at zero for good. The `Prices` table is a poor fallback (only appended on hand-typed prices — it held NUCL from April and nothing for CNYA); the Gonet snapshot is written several times a day.

Other known entitlement gap: WSH event data needs the News Feed entitlement (Error 10276) — see [[project_wsh_news_feed_gotcha]]. Related quote traps: [[reference_option_quote_liquidity]], [[reference_stock_quote_frozen_vs_live_and_cost]], [[reference_ibkr_tsej_bad_mark_price]].
