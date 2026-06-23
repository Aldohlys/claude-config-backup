---
name: feedback_analyze_bot_vs_swing
description: The /analyze command must use BOT_Score, BOT_Flags (S1-S6, BK1-BK4), not Long_Score/Long_Signal which are swing scanner signals
type: feedback
---

/analyze is a BOT (Breakout Option Trade) command — use BOT_Score, BOT_Flags, not Long_Score/Long_Signal.

**Why:** A ticker can be SKIP for swing but a strong BOT candidate (e.g., AMD had Long_Signal=SKIP but BOT S:5/6 BK:4/4). Using swing columns caused false NO-GO verdicts.

**How to apply:** Gate 2 checks BOT_Score (≥7), BOT_Flags (S:X/6 BK:Y/4). Scanner CSV fields are double-quoted — use `grep "\"TICKER\""` not `grep "^TICKER;"`. Indicator naming is canonical S1-S6/BK1-BK4, not T1/M1/V1 aliases.
