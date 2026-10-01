---
name: reference_ibkr_data_entitlements
description: IBKR account has NO fundamentals entitlement - reqFundamentalData returns Error 10358 "Fundamentals data is not allowed" for every report type; use yfinance for P/E and EPS
metadata:
  type: reference
---

Tested 2026-09-29 (TODO #92) on AAPL, TWS up: `ib.reqFundamentalData(contract, rep)` for `ReportsFinSummary`, `ReportSnapshot` and `RESC` all return empty with **Error 10358, "Fundamentals data is not allowed"**. The account has no fundamentals subscription; do not build on IBKR for P/E, EPS or estimates.

What does work (yfinance 1.2.0, miniconda python): `Ticker(s).info` gives `trailingPE` / `forwardPE` / `trailingEps` for US and European stocks and `trailingPE` for ETFs (SPY, IWM, QQQ, XL*, SMH; no forward for ETFs, convention for loss-makers undisclosed). `quarterly_income_stmt` only reaches back 5-6 quarters. `get_earnings_dates()` needs the `lxml` package, not installed in miniconda base.

Other known entitlement gap: WSH event data needs the News Feed entitlement (Error 10276) — see [[project_wsh_news_feed_gotcha]].
