---
name: feedback-no-static-option-judgements-in-screens
description: Never freeze an IV-dependent judgement into a screening universe; it belongs to the live-chain step
metadata:
  type: feedback
---

User (2026-08-27): "Options is a different matter and need a live chain to be checked. So the
vehicle choice is a second step... Unless a 300 USD is out of reach - but that should depend
upon current IV."

I twice hardcoded option-level judgements into `book_BOT` and both were wrong:
- `NOT_SIZEABLE`/`STRIKE_UNREACHABLE` (cocoa, coffee) — a static Black-76 estimate at one
  moment's realised vol, which froze that vol regime into the universe permanently.
- A >80% realised-vol hold-back on 6 semis, justified as "premium punitive".

**Why:** affordability is a function of *current* IV. If cocoa's IV halves, the vertical
becomes tradable and a hardcoded set still excludes it. Also VRP does not select entries at
all (validated, see [[project-bot-book-design]]), so nothing option-derived belongs in an
entry screen.

**How to apply:** screens gate on price/technical/ATR facts only. Also — a rule-based
recompute silently wipes hand-asserted exclusions; that bug bit twice. Prefer no assertions
to remembering to re-apply them.

Related: [[project-bot-universe-scanner]]
