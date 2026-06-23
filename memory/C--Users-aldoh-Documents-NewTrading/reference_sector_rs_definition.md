---
name: sector-rs-definitions-in-swing-scanner
description: sector_rs_rank = sector-vs-SPY (NOT stock-vs-sector). Stock-vs-sector RS is computed as rs_etf but never surfaced. LONG-only ranking is the asymmetry to fix.
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3b70ddd8-ed66-422b-bfce-e9ef777ec479
---

Three RS concepts coexist in `RStudies/reports/swing_scanner/` — they are NOT interchangeable.

**1. Sector-vs-SPY** (`evaluate_sector_gates` returns `$rs` per sector)
- Formula: sector ETF 20d return − SPY 20d return
- Used to build `sector_rank_map` in `main.R:88-90`
- Currently LONG-only: filter to `$long`-passing sectors, rank descending by RS, top-3 = 3 pts.
- For shorts: need symmetric path — filter to `$short`-passing sectors, rank ASCENDING (weakest first).

**2. Stock-vs-sector** (`rs_etf` in `pull_score.R:85`)
- Formula: `stock_ret20 - etf_ret20` (in pct points, rounded to 1 decimal)
- Computed and returned in `score_pull()` result, but **never surfaced** in scanner CSV or /analyze.
- Cheap to add — both inputs already in pipeline.
- 60d variant requires `ret60` on both stock and sector ETF — neither is currently computed; add to `calc_ind`.

**3. Stock-vs-SPY** (`rs_3m` from `Tdata::isTrendContinuation`)
- 3-month relative strength of the ticker against the benchmark.
- Passed into `score_pull` as `rs_3m`, used as input to stage classification, not as a standalone surfaced metric.

The user's framing (`/analyze` redesign 2026-05-12): "leader-vs-laggard within sector" is the missing signal. Stock-vs-sector at 20d AND 60d, displayed as separate rows, gives the four-cell decoder when combined with sector-vs-SPY.
