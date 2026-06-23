---
name: /analyze is a measurement instrument, not a recommender
description: /analyze tabulates data; do NOT produce GO/NO-GO, conviction, Best/Alternative/Avoid rankings, edge narratives, or pre-committed plans
type: feedback
originSessionId: 791d7ea0-3b98-46b6-b585-5209bd9337c8
---
`/analyze` is a **measurement instrument**, not a recommender. The user reworked the spec on 2026-04-30 to make this explicit. Compute, tabulate, surface; do not editorialize.

**What to remove from /analyze output:**
- GO / NO-GO / CONDITIONAL verdicts
- Conviction levels (HIGH / MEDIUM / LOW) and conviction-point arithmetic
- "★ Best / ★ Alternative / ★ Aggressive / ✗ Avoid" rankings of structures
- Edge-source narratives ("this is a vol-mispricing edge", "no clear edge")
- Pre-committed plans (scale-outs, stops, time stops, exit conditions) — these are a user decision
- "Surface supports/fights the thesis" prose translations of the funnel grid (report the count, not the verdict)
- Words like "consider", "favor", "lean", "I'd suggest", "preferred"

**What stays:**
- Mechanical phase outcomes: PASS / SKIP / NO SIGNAL / STALE — these are scanner-level labels, not directional advice
- Direction-alignment fact: ALIGNED / MISMATCH (do NOT call it a "warning")
- Regime band labels (cheap / mid / rich) for IVP — these are descriptive
- Term-shape labels (contango / backwardation / U / hump / flat) — descriptive
- All the numbers: IVs, deltas, OI, R:R, probabilities, EV, debits/credits

**How to apply:** when re-running `/analyze`, do not import the old terminal-verdict template (the one with PREFERRED STRUCTURES section). The new spec puts the data table at the bottom of Phase D and stops there. The user makes every decision; this command supplies inputs.
