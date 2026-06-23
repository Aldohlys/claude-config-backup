---
name: Futures front-month convergence + roll buying in expiry week
description: In the final 1-2 weeks before front-month expiry, the front-vs-back move differential is NOT a clean signal about catalyst duration — convergence + speculative roll buying contaminate it
type: reference
originSessionId: 0544fcb3-a0ac-4de6-93f8-9e759ecc9335
---
**Pattern:** In the final 1-2 weeks before a futures front-month expires, the front contract often *underperforms* the next month even on a bullish catalyst. The standard "front rallies more than back on supply shock" intuition breaks down.

### Three mechanisms (additive)

1. **Physical convergence pinning the front.** As expiry nears (days, not weeks), the front contract converges to spot/cash. Market-makers run flat, speculative longs have already rolled out. The front loses its ability to rally freely — it's tethered to physical settlement.

2. **Catalyst pricing migrates to the new front.** Any speculative long that wants exposure beyond expiry must buy the *next* month. Bullish news → flows hit the back, not the front.

3. **Mechanical roll buying.** Holders of expiring longs who don't want to take delivery roll → buy back → sell front. This creates structural buy pressure on the back during the roll window, independent of fundamentals.

### Why it matters

Front-vs-back outperformance in expiry week is **NOT** a clean read on whether the catalyst has duration vs. is a 1-week event. You can't conclude "market is pricing multi-week supply tension" from a 7bp differential the week before expiry — convergence + rolls would produce the same picture.

### How to extract a cleaner duration signal

Use the **first non-expiring spread** as the read. Example: in WTI, with June expiring May 18:
- Don't read Jun-Jul (contaminated by expiry mechanics).
- Read **Jul-Aug** (or Jul-Sep) — first spread where both legs are clean of immediate convergence.
- Jul-Aug widening (Jul firmer) post-expiry = real supply-tension persistence.
- Jul-Aug compressing = catalyst fading despite spot price.

### Origin

Observed 2026-05-11 on MCL Jun (+2.30%) vs MCL Jul (+2.37%) following Iran-war headline gap. CL/MCL Jun expires 2026-05-18 14:30 ET, only 7 days out. The 7bp Jul outperformance is dominated by mechanisms (1)-(3), not a clean macro statement.
