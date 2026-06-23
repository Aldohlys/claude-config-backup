---
name: /analyze off-universe fallback path
description: When ticker is not in v5 ScannerUniverse, use a documented fallback chain (yfinance history + chains, BS solve from quoted IV) and mark each gap explicitly.
type: feedback
originSessionId: a5c2009f-c1bb-4d78-aafa-fafb8bd015a9
---
When `/analyze` is invoked on a ticker outside the v5 ScannerUniverse (e.g. IBIT 2026-04-28), don't bail or skip — fall back through this chain:

1. **Price history**: yfinance `Ticker.history(period='1y')` for MA50/MA200, RS vs SPY, HV20/HV60/HV1y, BOT S1-S6 + BK1-BK4 indicators.
2. **Option chains**: yfinance `Ticker.option_chain(exp)` for bid/ask/IV/OI/volume by strike. Compute deltas via Black-Scholes from quoted IV when delta isn't returned.
3. **Vol surface (term + skew)**: pull 4-5 expiry buckets (~7d, 17d, 31d, 51d, 80d), extract ATM IV per bucket, compute 25Δ skew = 25dC_IV − 25dP_IV in vol points.
4. **IVP**: estimate from 1y absolute-IV range if no DB history; explicitly mark as "estimated" in the report.
5. **Sector RS rank**: mark N/A if ticker has no sector cohort (thematic ETFs like IBIT, ARKK).
6. **Live OI from TWS**: skip silently if TWS not running; yfinance OI is acceptable for chain-bound analysis on off-universe tickers.

**Why:** Spec says "Run scanner first or proceed with --force". User memory `feedback_analyze_force_all_phases.md` confirms force is implicit — always run all phases. The off-universe path needs a concrete data fallback, not a dead-end.

**How to apply:**
- In every phase output, mark which inputs are estimated/data-limited so the user can weight them.
- Don't fabricate IVP percentile from a 2-row Prices table — say "estimated from 1y absolute IV range" and give the range.
- Yahoo encoding gotcha: cp1252 console can't print Δ / unicode; use ASCII labels (`|d|`, `delta`) in print statements on Windows.
