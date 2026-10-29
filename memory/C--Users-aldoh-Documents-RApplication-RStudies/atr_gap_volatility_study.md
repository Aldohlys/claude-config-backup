---
name: atr-gap-volatility-study
description: AAPL ATR / true-range distribution and overnight-gap vs session volatility study — location, branch, headline results
metadata:
  type: project
---

Study in `RStudies/atr-gap-volatility/` (`atr_gap_study.R [TICKER] [YEARS]`, `FINDINGS.md`), README entry "3. ATR & Gap Volatility". Committed c827ae6 on branch `study/atr-gap-volatility`, pushed to origin 2026-10-22; no PR opened yet (not merged into main).

AAPL 2y to 2026-10-21: TR > ATR(14, prior close) on ~39% of days; high-low 33%; |close-close| 13%. Overnight gap = ~35% of daily variance; top-10 gaps = 56% of gap variance; rolling 63d gap share 56% (driven by 2026-07-31 −9% gap).

**Why:** user built this interactively from IBKR data, then asked to port it to an R study in RStudies.
**How to apply:** extend this script (other tickers, earnings-gap split) rather than starting a new one; check whether the branch was merged before building on it. Data source: [[daily-ohlc-source-for-studies]].
