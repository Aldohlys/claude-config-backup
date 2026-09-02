---
name: reference-rstudies-indicators-source
description: indicators.R is the single source of truth for BOT S/BK criteria; scoring.R no longer exists
metadata:
  type: reference
---

`RApplication/RStudies/reports/shared/indicators.R` — `calc_ind()` + `compute_breakdown()` —
is the single source of truth for swing_scanner AND /analyze. **`swing_scanner/scoring.R` no
longer exists** (refactored into cheap_score.R, classify.R, flow_score.R, sector_gate.R).

Details that are easy to get wrong and that I did get wrong first time:
- `d$ma50 <- TTR::EMA(Close, 50)` — named ma50, IS an EMA
- `vol_surge = Volume / vol_ma20` — 20-session mean, NOT 50
- `updn_ratio` — 10-session up/down volume, up day = `Close >= lag(Close)`
- `ma50_slope`, `rsi_slope` — 5-session change, not a fitted slope
- `obv_slope = obv - lag(obv, 20)` — a raw difference
- RS: `rs_vs_sector_20d = stock_ret20 - etf_ret20` in `compute_sector_rs_context()`
  (`reports/shared/live_sources.R`), arithmetic difference of percent returns
- min history 130 bars

**How to apply:** port from the code, never from the checklist prose. Any reimplementation
that diverges means the two scanners disagree on the same name.

Related: [[project-bot-universe-scanner]]
