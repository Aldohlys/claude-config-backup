---
name: feedback_relative_criterion_excludes_a_fixed_fraction
description: "A tercile/percentile criterion removes a fixed fraction by construction and its cut point moves with the population — if the reason for the rule is absolute, make the threshold absolute"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5f03a6df-1df0-4f76-b348-15024cf64b52
  modified: 2026-09-21T09:24:39.456Z
---

BOT_monthly excludes names whose gap share falls in the high tercile. On the
first production run (2026-09-18, 335 names) that was the **largest single
exclusion: 93 names, 28% of the universe** — more than `atr_band` (42) and `adv`
(22) together. That is not a finding about those 93 names. A tercile removes
~1/3 whatever the absolute distribution looks like; it cannot do otherwise.

**The cut point also moved.** The run placed the high boundary at
**gap_share > 0.429** (universe min 0.113, p33 0.323, median 0.366, p67 0.429,
max 0.897). [[project_bot_three_class_framework]] measured the high tercile at
**0.61–0.74** on the 205-name scanner universe. **71 of the 93 excluded names sit
below 0.61**, so they would not have been in the measured high tercile at all.
`Tickers` (347) is a wider population than the scanner universe (205), so the
same relative rule lands somewhere else.

**Why:** the reason the criterion exists is absolute — a high-gap name moves
between sessions and gaps *through* a stop rather than trading through it, so
ATR-based sizing understates the loss tail. That is a property of the name, not
of its rank among whichever peers happen to be in the table today. A relative
rule re-cut on a changed population is no longer the rule that was measured.

**How to apply:** when a criterion is a percentile or tercile, ask two things
before shipping it. Does the justification refer to the name's own behaviour
(→ use an absolute threshold anchored on the measurement) or to its standing
among peers (→ relative is right)? And is a fixed exclusion fraction actually
intended, since that is what you get? Also expect membership to churn whenever
the population changes — here, whenever `Tickers` gains or loses rows.

Same family as the mismatch in [[reference_bot_three_tool_architecture]]'s
`atr_band`, where an absolute number was carried from a daily veto into a
membership band without being re-derived — but distinct: that is one number used
out of context, this is a rule that silently recomputes its own number. Open as
TODO #88.4.
