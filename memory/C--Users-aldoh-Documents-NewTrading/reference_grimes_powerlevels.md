---
name: reference_grimes_powerlevels
description: "Adam Grimes PowerLevels = proprietary pre-computed S/R levels (daily from prior bar, weekly for the week), black box, ~±3 ATR cluster; 6 named patterns (Clear Air, Yo-yo, Pong, PowerPush/Pull, Polarity); evidence weak; MarketLife membership"
metadata:
  node_type: memory
  type: reference
  originSessionId: f580ecee-39df-4dfe-ac2b-699cec815aeb
  modified: 2026-10-16T06:26:48.657Z
---

Researched 2026-10-16 from adamhgrimes.com posts (Oct 2020: "Introducing PowerLevels", "How to Trade Support and Resistance Levels") + marketlifetrading.com/subscriptions.

- What: support/resistance levels generated before the bar opens; a rule-based filter of his older volatility / prior-bar / trend levels, meant to mimic his discretionary level choice. Formula not disclosed → cannot be backtested or added to the BOT scanner; only testable by logging published levels forward.
- Sets: daily (from prior completed bar, intraday use) and weekly (computed before the week, valid all week; Grimes says it matters for daily-chart swing structure → the one relevant to the user's 2-4 wk BOT horizon). Levels scale with volatility, cluster within ~±3 ATR of prior bar.
- Uses claimed: stops, target refinement, daily bias. No exact entry/stop/target rules or hit rates published.
- Patterns: Polarity (broken level flips on retest); PowerPull (momentum accelerates into a level); PowerPush (open near level, fast move away → session extreme often set early); Yo-yo (chop around one level, breakout runs ≥ next level); Pong (range between two close levels, false breakouts); Clear Air (trend day beyond most extreme level → extension into close; weekly-on-daily variant).
- Only evidence: closest level within ±0.23% of session high/low vs ±0.63% for random levels over same range — weak (many levels → nearest is close by construction; no N, counts or dispersion).
- Related Grimes survey (~1,500 traders): S&P prior-session and overnight high/low look non-random; EURUSD round numbers do not. He says moving averages and round numbers show no demonstrated edge.
- Access: 2020 = PreMarket Edge report \$19/mo; now listed in MarketLife Stocks & Options membership ("Mantis Scans & PowerLevels"). User has a MarketLife login; whether the plan includes them was unconfirmed on 2026-10-16.

Related: [[reference_marketlife_transcribe]], [[reference_atr_empirical_band_limits]], [[feedback_price_action_rr_gate]].
