---
name: daily-ohlc-source-for-studies
description: Where RStudies scripts get current daily OHLC, and where regenerable outputs go
metadata:
  type: reference
---

Current daily OHLC for studies: `Tdata::getYahooData(tickers, from_date, to_date)` (columns date, Open, High, Low, Close, Adjusted, Volume; same loader as `reports/shared/indicators.R::fetch_single_ohlcv`). Closes matched IBKR daily bars for AAPL in Oct 2026. `ivol_probe_out/{TICKER}/stock_prices.parquet` stops at 2026-04-27 (IVolatility trial expired) — see [[ivolatility_data_pipeline]].

ATR convention in the repo: `TTR::ATR(n = 14)` (Wilder). Regenerable study outputs go under root `output/` (git-ignored), e.g. `output/atr_gap/{TICKER}/`. Used by [[atr-gap-volatility-study]].
