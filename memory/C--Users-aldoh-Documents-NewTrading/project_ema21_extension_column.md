---
name: project-ema21-extension-column
description: "bot_daily ema21_disp_atr column (2026-10-20): display only, no gate; 5y backtest + 92 real BOT trades show below-EMA21 entries were the best winners"
metadata:
  type: project
---

2026-10-20: added `ema21`, `ema21_disp_pct` (Detail) and `ema21_disp_atr` (Data sheet, 14th column) to bot_daily via indicators.R (`d$ema21`), gates.R, bot_read.R, legend in bot_daily_xlsx.R. Committed db3ce3c on RStudies branch claude/macro-context-main-detail (not main).

Evidence (scripts `NewTrading/Strategies/Breakouts/ema_distance_test.py`, `ema21_at_entry.py`):
- 5y, 78 BOT names: percent distance not comparable across names (RICH names ~2.5x the % of LOW for the same ATR distance) -> ATR unit. Spearman vs fwd 5/10/20 ~0 for EMA21 and EMA50, % and ATR.
- Breakout proxy (close > rising EMA50, new 20d high, 4,473 signals): <1 ATR above EMA21 weakest (fwd20 -0.21 ATR, n=125); 1-5 ATR better the further above; >5 ATR (40 signals) gives back over 10 sessions, MAE -2.1 ATR. No "too expensive" effect below 5 ATR.
- Real BOT trades (90 longs 2022-07..2026-03): 14 of 37 winners entered below EMA21, incl. the three largest (GLD 569, AAPL 376/393). >1 ATR below EMA21: 11 trades, 73% WR, +5,448 — these are support/pullback entries.

**Why:** user wanted a "below EMA = not bullish / far above = wait for pullback" warning. Data contradicts a gate in both directions except the rare >5 ATR tail.

**How to apply:** keep it a display column; don't propose a gate or tier rule on it without new evidence (bot_fwd review ~11-02 can add it). Related: [[project-bot-universe-scanner]], [[project_ma50_ema_switch]]
