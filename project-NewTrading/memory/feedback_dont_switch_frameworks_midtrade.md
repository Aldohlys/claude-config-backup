---
name: feedback-dont-switch-frameworks-midtrade
description: "Each trade has its own framework (BOT structural, macro thesis, vol trade). Don't let a new thesis renegotiate the terms of an active trade — open a new position with appropriate parameters instead."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 243c14ae-fe86-4883-bf49-ab3946ae5c8e
---

When mid-trade, the temptation appears to switch from the framework you entered with (e.g., BOT 2-4 week structural breakout) to a competing framework (e.g., macro bullish thesis, "this could go much higher"). This is the classic "started as a swing, became an investment" error.

**The rule:** Each trade is its own contract. The entry, stop, and target were derived from one framework; mid-trade re-derivation under a different framework breaks the discipline that made the trade tractable in the first place.

**Concrete example (MCL Jul26 close, 2026-05-18):**
- Entry: BOT breakout trade, 2-4 week horizon, \$1,000-\$1,500 expected profit, structural target \$102 (prior resistance)
- Mid-trade temptation: "Xi-Trump deadlock + Hormuz + no peace = macro thesis says price goes higher, hold the runner"
- Correct read: macro is a SEPARATE trade idea with different horizon (3-6 months), different vehicle (deferred), different sizing, different stop logic (thesis invalidation, not chart structure)
- Outcome: stuck with framework 1, exited at +\$1,604 (98% of plan max)

**Why:** Framework switching produces hybrid stops that satisfy neither discipline. You end up with a BOT-sized position held on macro horizon, or a macro-sized stop on a structural setup. Both are degenerate.

**Why:** User explicitly named this trap after the trade closed — both frameworks were live in their head; the discipline came from recognizing they belonged to different positions, not the same one.

**How to apply:** When the "but it could go higher" thought hits mid-trade, ask: "Is this the trade I entered? If a new thesis is real, would I OPEN a new position for it right now — at this price, with this sizing, with this stop?" If yes, exit the current trade by its rules and place the new trade fresh. If no, the new thesis is rationalization to hold.

Related: [[feedback-hard-to-exit-winners]] — the framework-switch temptation is one expression of the broader winner-exit difficulty.
