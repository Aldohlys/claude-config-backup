---
name: feedback_no_latex_math_blocks
description: "Don't use LaTeX math ($$...$$ or inline $...$) in output — the renderer garbles it; use plain text / code blocks"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2bd4384e-3b59-46f3-a4cf-4b6e7212fea0
---

The FleetView / Claude Code markdown renderer mangles LaTeX math — both `$$...$$` display blocks and inline `$...$` / `\(...\)`. Same renderer quirk behind the CLAUDE.md `\$`-escaping rule for prices.

**Why:** user reported a `$$|move|_N = C \times ATR\% \times \sqrt{N}$$` block rendered unreadable (2026-06-08).

**Dollar-amount garble (recurring — re-confirmed 2026-06-15):** TWO unescaped `$` on the SAME line make the renderer treat everything between them as a math block. Example the user flagged: `RIG (+$344) and LAC (−$172)` → the text between the dollars renders as garbled math. EVERY `$` in prose/tables must be written `\$` (`+\$344`, `−\$172`, `\$10-150`). This bites repeatedly because source files (e.g. `bot_strategy_trading_plan.md`) accumulate bare `$` over edits. Fix: escape on write; sweep before saving. (Exception: `$` inside fenced code blocks renders literally — leave unescaped there.)

**How to apply:** write formulas in plain text or fenced code blocks — e.g. `move(N) = C × ATR% × √N`. Use `×` not `\times`, `√` not `\sqrt`, spell out subscripts (`sigma_daily`, `move(N)`). No `\underbrace`, `\frac`, etc. Always escape dollar amounts as `\$`. When fixing a whole file, a regex pass works: escape `$` not preceded by `\`, skipping fenced code blocks.
