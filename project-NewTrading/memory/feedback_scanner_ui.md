---
name: Scanner UI preferences
description: User preferences for scanner report display — clarity over detail, no redundant columns, visual badges
type: feedback
---

Keep scanner reports clean and unambiguous:
- No redundant columns (if Composite = Score, show only one)
- Direction should be immediately obvious via colored badges (LONG green, SHORT red), not text in separate Long_Signal/Short_Signal columns
- Split TRADE and WATCH into separate sections — don't mix them
- Sector gate: show a clear direction verdict (badge), not confusing pass/fail text for each side
- Group related columns visually (technical together, optionality together)
- Methodology explanations should be grouped by theme with full detail, not cryptic abbreviations
- Remove columns that don't add decision value (ATR_pct, Target computed from arbitrary R:R)

**Why:** The user reads these reports to make trade decisions quickly. Ambiguous display wastes time and causes confusion.

**How to apply:** When adding or modifying scanner output, ask: "does this help the user decide in 5 seconds whether to investigate a stock?"
