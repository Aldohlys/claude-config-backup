---
name: feedback-breakout-iv-direction
description: After a breakout, IV does not reliably fall — no earnings-style crush. Direction depends on side (put/call) and on the name. Do not default to a negative vol offset.
metadata:
  type: feedback
---
Do not assume implied vol drops after a breakout. There is no scheduled event, so there is no mechanical IV crush — that belongs to earnings, where a known vol premium collapses on a known date. Importing that framing into breakout analysis is a category error.

The user's correction (2026-08-26): "a breakout may bring more people to the trade, and therefore increase IV because the option will get more expensive." Correct, and it is the dominant case for the names the BOT strategy targets.

**Why:** two independent effects set the post-move IV, and they do not share a sign.

1. Surface level, i.e. spot-vol correlation:
   - Downside target (long PUT) — spot down, vol up, reliably and hard. Positive offset, often +5 pp or more on a real break.
   - Upside target (long CALL) on an index or mega-cap grinding higher — leverage effect dominates, ATM vol drifts down. Flat to slightly negative.
   - Upside target (long CALL) on a momentum name, squeeze, small cap, or commodity — spot-vol correlation flips positive. Call crowding plus dealers short gamma chasing bids the vol up. Positive offset. This is the user's case.

2. Moneyness migration along the skew. A contract 15% OTM today sits at roughly ATM if spot reaches the target. On normal equity skew an OTM call carries lower IV than ATM, so migrating to ATM picks up vol — another positive push for calls. An OTM put carries higher IV than ATM, so migration is negative for puts, but effect 1 outweighs it.

**How to apply:** in RPreTrade's Breakout Analysis tab (and any equivalent hand calculation), `vol_offset = 0` is a sticky-strike baseline — the strike keeps the IV it carries today. Deviate from it by side and by name, not by a blanket rule: puts positive; calls positive on breakout/momentum names, flat-to-slightly-negative only on an index or mega-cap in a low-vol grind. Present it as a band (e.g. -3 / 0 / +5) and check whether the R:R ranking is stable, rather than as a point estimate. The offset there is a flat parallel shift across all strikes and expiries, so it cannot represent skew rotation or a term-structure change — only the level. `days_offset` is the dial that genuinely points one way: time passes regardless of who joins the trade.

Related: [[feedback-classify-breakout-vs-meanreversion]], [[feedback-structure-selection-vs-vol-thesis]], [[reference-rpretrade-breakout-tab]]
