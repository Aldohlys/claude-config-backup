---
name: feedback_shortened_lookback_fits_regime
description: "Shortening a lookback to make a name clear a threshold fits the current regime, not the name — the tell is that every peer moves the same way; only a corporate event justifies a per-name window"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6df3dc0a-4099-43b9-b384-ce0952b42b01
  modified: 2026-09-03T18:57:16.862Z
---

2026-09-03. Asked to look at gold miners, AEM and Barrick failed the BOT Gate 4 cut
(`atr_med5y >= 3.0`) at 2.92 and 2.95. Having just built a per-name `atr_window_y` column
for EXE, the obvious move was to give them a shorter window too. It would have "worked":

| | 5y | 2y | 1y |
|---|---|---|---|
| AEM | 2.92 | 3.25 | 3.91 |
| Barrick (B) | 2.95 | 3.32 | 3.95 |
| WPM | 2.89 | 3.18 | 4.05 |
| NEM (already in) | 3.01 | 3.35 | 4.09 |
| GFI | 3.87 | 4.09 | 4.97 |

**Every one of the ten precious-metals names tested rose monotonically as the window
shortened.** That uniformity is the tell: the shorter window is not measuring these names
better, it is measuring *gold's current volatility regime*. Applying it would admit the whole
complex at once, on the regime rather than on any name's character — the exact error
`bot_scan_universe.py:68-74` already warns about ("that froze one vol regime into the
universe"). The large-cap operators and the streamers (WPM, RGLD, FNV) sit below the cut
because they are structurally less levered to the metal. Gate 4 was working.

**Why:** a threshold on a long-window statistic is a claim about a name's *character*. Any
knob that shifts the statistic toward the threshold — window length, sample period, universe
composition — converts it into a claim about the current regime, and the gate stops
discriminating.

**How to apply:**
- Before shortening any lookback to admit something, **compute the same shortening for the
  peer group**. If they all move the same direction, the window is fitting the regime; stop.
- A per-name window is justified only by a **corporate event that makes the earlier series a
  different company** — EXE's CHK+SWN merger (Oct-2024) leaves 4 of 5 years belonging to
  another entity. AEM's Kirkland Lake merger (Feb-2022) sits at the edge of the window with
  the acquirer surviving: not a justification.
- Say so and leave the gate alone, rather than delivering the flattering number.

Same family as [[feedback_validate_metric_noise_floor_and_persistence]] (a constant verdict
is the tell) and [[feedback_normalize_window_metrics_to_current_size]].
See [[reference_bot_tradable_universe_csv]] for `atr_window_y` mechanics.
