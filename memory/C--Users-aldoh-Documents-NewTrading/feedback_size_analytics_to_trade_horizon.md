---
name: feedback_size_analytics_to_trade_horizon
description: "Size any analytic's lookback/window to the user's actual trade horizon (2-4 week breakouts), not an arbitrary default; prefer adaptive swing detection over fixed windows"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: c63ff519-1cac-4027-a4b5-d2730bebb486
---

When building a metric for the user, **size its lookback to the trade timeframe**, not a convenient default. Building the `/analyze` move-maturity overlay (2026-06-05), the first cut anchored the swing base on a **120-day** low — the user pushed back with a chart ("I don't see a 311 base") and then "I am looking for 2-4 weeks breakout (see my trading plan) — so interval considered should be in line with breakout duration." Recalibrated to a **40-day** window (`analyze.move_lookback_days`, tunable), matching the BOT plan's hold (8-14d options / 20-40d stock) and the 20d/40d squeeze base.

**Why:** the user reads metrics against how they actually trade. A 120d anchor made every name read "fully extended" because it reached back to stale multi-month lows — useless for a 2-4 week breakout entry. The horizon IS in their trading plan ([[bot_strategy_checklist]] / Strategies/Breakouts); consult it rather than guess.

**How to apply:** (1) For window-based metrics, default the lookback to the breakout horizon (~20-40 trading days) and make it config-tunable. (2) Prefer **adaptive swing detection** (ZigZag, reversal-threshold) over a fixed-window min/max so a stair-step trend anchors on the most recent higher-low, not the window edge. (3) When the user questions a number, expect a concrete chart example — reconcile your value against their charting (e.g. Fib extensions scale the *leg*, not the absolute price). Related: [[feedback_classify_breakout_vs_meanreversion]], [[user_framework_confidence_map]].
