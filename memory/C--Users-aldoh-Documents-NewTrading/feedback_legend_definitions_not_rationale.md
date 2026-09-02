---
name: feedback-legend-definitions-not-rationale
description: Column legends carry definitions and formulas only; rationale and evidence live in the accompanying doc
metadata:
  type: feedback
---

User (2026-08-27): "a mere definition of gate4 is fine, not why we use it (this should be
present in the accompanying documentation)."

I had grown the XLSX legend into per-band R evidence, the move model and worked examples.
Trimmed to definition + threshold + source, with a pointer row to
`bot_book_design_20260827.md`.

User also rejects vague phrasing in definitions: "travelled at least 2 daily ranges" was
sent back — wants explicit algebra like `|Close_t - Close_t-20| / ATR14_t`, and units stated
(sessions vs calendar days, percent vs ratio vs ATR count).

**Why:** a column reference is read while looking at data; argument belongs where it can be
weighed. Writing the formula precisely also exposes bugs — doing so found that
`prior20_atr` was a ratio of percentages carrying a spurious price factor.

**How to apply:** legend = what it is, what threshold, where it comes from. Nothing else.

Related: [[feedback-plain-language-no-jargon]], [[project-bot-universe-scanner]]
