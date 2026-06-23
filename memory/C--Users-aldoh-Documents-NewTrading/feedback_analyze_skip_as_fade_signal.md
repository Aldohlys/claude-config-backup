---
name: feedback-analyze-skip-as-fade-signal
description: /analyze SKIP on a direction can be USED as a fade-confirmation signal — the user trades both sides of the framework
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 83012921-786f-469a-b241-73e9f15b5317
---

When /analyze returns SKIP for a given direction on a parabolic/extended name, the user may treat the result as confirmation for a counter-direction lottery ticket — i.e. SMH short SKIP'd on 2026-05-27 (rank 19/19 sector leader, IVP 91, +27% over MA50) was used as confirmation to put on a 557.5/547.5 put debit spread as a mean-reversion fade.

**Why:** The user's framework is *symmetric*. BOT-style trades ride momentum; mean-reversion lottery tickets fade exhaustion. A /analyze SKIP showing "stock is sector leader in strongest sector at extreme range position" is exactly the picture of an extended move that may be ready to fail. The SKIP isn't "don't trade" — it's "this is not a momentum entry," which can ALSO mean "this is a fade entry."

**How to apply:**
- Don't read /analyze SKIP as "the user shouldn't take this trade." Read it as "the user shouldn't take the queried direction as a trend trade." A fade in the opposite-direction-of-trend is a different framework.
- When the user discusses an /analyze result, ask which framework they're applying (trend vs fade vs vol) before commenting on fit.
- Specifically: parabolic + IVP high + sector-leader + R:R 0.02 PRICED OUT is the *signature* of a mean-reversion lottery setup, not a hard pass.
- For fade trades, the relevant data slices are different: effective_target is irrelevant (no structural break thesis), OI gravity *below* short strike matters, net vega sign of the spread matters (long vega helps if a flush expands IV).
