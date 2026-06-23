---
name: Tdata force_refresh on every /analyze call
description: Pass force_refresh=True on every getOptValue / compute_spread_risk_reward in /analyze; cache TTL 30 min causes stale-quote false-fails
type: feedback
originSessionId: 791d7ea0-3b98-46b6-b585-5209bd9337c8
---
Every Tdata `getOptValue` and `compute_spread_risk_reward` call in `/analyze` MUST pass `force_refresh=True`. Same for any direct surface pulls.

**Why:** The Tdata quote cache TTL is 30 minutes ("Quote cache full hit: N strikes for SYM EXP RIGHT (TTL 30 min)" appears in the log). When /analyze runs back-to-back or shortly after another tool used the same chain, cached mids feed into `max_risk` and produce verdicts that are inconsistent with the live market — the user observed this directly, where stale quotes pushed structures above the $300/lot cap when fresh mids put them well below it.

**How to apply:** Hard-code `force_refresh=True` in every spread/option pull inside `/analyze`. Do NOT skip it on retry runs or on widths beyond the first — the cache repopulates fast and a 5-minute-old quote is still stale relative to a $300-cap decision.
