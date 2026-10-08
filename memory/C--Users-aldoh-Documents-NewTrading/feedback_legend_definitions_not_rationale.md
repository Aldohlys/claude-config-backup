---
name: feedback-legend-definitions-not-rationale
description: Column legends carry definitions and formulas only, self-contained (no 'see doc X'); rationale and evidence live elsewhere
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

2026-10-08 refinement (intermarket_sectors legend): every definition must be self-contained,
"without any external references" — no pointer rows, no "see BOT_TOOLS_DESIGN", no function
names as explanation. Spell out windows, formulas, value meanings and symbol codes in the cell.

**How to apply:** legend = what it is, what threshold, how it is computed. Self-contained. Nothing else.

Related: [[feedback-plain-language-no-jargon]], [[project-bot-universe-scanner]], [[feedback-report-xlsx-with-legend]]
