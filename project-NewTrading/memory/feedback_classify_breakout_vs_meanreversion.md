---
name: feedback-classify-breakout-vs-meanreversion
description: "Before advising stop/target on any directional trade (esp. long-off-a-low), classify it breakout vs mean-reversion — they need opposite logic. The BOT score is the discriminator."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a5f1ac1e-c6b9-425f-8eb8-21c585ad13d4
---

Every directional trade is one of two types that need **opposite** stop/target/horizon logic. Classify first; misclassifying imports the wrong exit plan — the #1 way these get mismanaged. "Long off a low" does NOT classify it: both types can be.

- **Breakout (BOT)** — a bet about *structure*: a level broke, new trend started. Target **open-ended away from the base**, stop **back inside the base**, needs **volume expansion** (low vol = kill signal), can take time. With-trend; price above rising MAs.
- **Mean reversion** — a bet about *price*: overstretched, snaps back to the mean. Target **finite and overhead** (e.g. 50 EMA), stop **below the extreme low** (the origin), low-vol bounce tolerable for entry but weak follow-through, must work **fast**. Counter-trend; price below falling MAs.

**Two-question test:** (1) target finite-at-the-mean vs open-ended? (2) stop below-the-extreme-low vs back-inside-the-base?

**The BOT score is the discriminator:** S1 price>MA50, S2 MA50 rising, S4 OBV rising, BK4 vol-surge — a reversion bounce fails all of these. High score = breakout; if long with failing S1/S2/BK4, you're in a mean-reversion trade — use reversion exits.

**Lifecycle:** a reversion bounce can mature into a breakout, but that's a NEW trade (new stop/target) — the runner doesn't silently switch frameworks. See [[feedback-dont-switch-frameworks-midtrade]].

**Why:** co-developed 2026-05-28 during the MCL Aug'26 trade — a textbook reversion (counter-trend, fresh unbased low, finite \$92 target, no volume) that stopped −\$444 on a flash sell breaking the washout — contrasted against NVO (breakout: old based low, reclaiming MA, open-ended target). Same month, the Jul'26 MCL *breakout* (with-trend, volume, open-ended) made +\$1,604. The two outcomes are the cleanest illustration of the distinction.

**How to apply:** when the user asks about a directional trade's stop/target, state the classification first, then apply the matching exit logic. Full discriminator table + NVO/MCL worked example + BOT-score mapping live in `Strategies/Breakouts/bot_strategy_checklist.md` → "Trade-Type Classifier" section. Related: [[feedback-hard-to-exit-winners]], [[feedback-pyramid-per-lot-rr-audit]].
