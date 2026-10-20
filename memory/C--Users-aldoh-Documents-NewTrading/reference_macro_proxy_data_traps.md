---
name: reference_macro_proxy_data_traps
description: "Proxy and data traps found in the 2026-10-20 macro-forces rerun — TIP CPI accrual, HYG/IEF unstable beta, FRED HY spread 3 years only, IBKR weekly closes unadjusted"
metadata:
  node_type: memory
  type: reference
  originSessionId: cfd01905-af04-48c7-8713-2272d5ea18b3
  modified: 2026-10-20T20:58:15.877Z
---

Found while rerunning the macro-forces work on Yahoo/FRED daily data, 2004–2026 (`NewTrading/scripts/macro_forces/daily_check.py`):

- **TIP as a real-yield proxy is biased.**
  - TIP's price also accrues CPI, about +0.25% per 4 weeks, which reads as a small fall in real yields.
  - The weekly correlation of −TIP with FRED DFII10 is still +0.94, but near zero the sign flips. On 2026-10-19, TIP said real yields were down (z −0.6) while DFII10 said up (z +0.76).
  - Use DFII10, not TIP.
- **The HYG − β·IEF hedge ratio is not stable.**
  - Weekly β by period: −0.38 (2009-13), −0.15 (2014-19), +0.23 (2020-22), +0.37 (2023-26).
  - HYG/IEF tracks the 10Y yield (r +0.52 over 2004-26), so it is not a clean credit measure. With β = 0.45 the link to the 10Y still remains (r +0.28).
  - Use the FRED high-yield spread for the live reading, and a β fitted on the trailing year for history.
- **FRED BAMLH0A0HYM2 (ICE BofA HY OAS) keeps only 3 years** (from 2023-10). Longer backtests need a duration-neutral HYG proxy.
- **IBKR weekly closes are not dividend-adjusted.**
  - They gave a false "Debasement → GLD up" +10 pt edge, which disappeared on adjusted Yahoo data.
  - Use yfinance `auto_adjust=True` for any return backtest.
- **Yahoo and FRED both work** from this machine via yfinance and `fredgraph.csv?id=…`, no key needed. A raw curl to the Yahoo chart API returns 429; yfinance does not.

Related: [[project_macro_context_redesign]], [[reference_dxy_yahoo]].
