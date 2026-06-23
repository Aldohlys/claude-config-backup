---
name: feedback_design_before_code
description: "On non-trivial UI/UX iterations, present the design (text plan + ASCII mockups via AskUserQuestion `preview`) and wait for approval before editing files."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3ed4255d-f6de-4459-bd06-43a46d51dc9a
---

When the user is iterating on a UI/UX feature with multiple ambiguous layout choices, do NOT jump to code. Present the design first — a short text plan covering structure + formulas + persistence, plus ASCII mockups of the contested visuals — and wait for explicit "go" before writing.

**Why:** in the 2026-05-19 BSM calculator session, the user said verbatim "Give me your design before implementing" and earlier "Show me prototypes (basic table outlook) so I can better decide." Designing in chat first surfaced four hidden requirements the user only verbalised after seeing options:
- combine Δ and Γ into one cell (not separate)
- include 0% in the IV-bump list so vega contribution to Total can be turned off
- days should be a SELECTOR, not a column axis (matrix was about to be too wide)
- IV tab needs its own Price+Greeks mirror at the bottom

All four would have been wrong-and-rewritten if I'd coded straight from the initial prompt.

**How to apply:**
- Trigger: more than ~3 visual/interaction decisions to make in one feature, OR the user mentions wanting to "review", "see", "prototype", or "decide" before changes land.
- Format: brief plan as markdown (field order, formulas, persistence, edge cases), then `AskUserQuestion` with 2–3 layout options each carrying an ASCII `preview`. Side-by-side previews land much better than prose descriptions for layout choices.
- The `preview` field renders in a monospace box — use it for tables, ASCII art, code; not for paragraphs.
- After answers, write one consolidated final-design summary and ask for a single yes/no approval before touching code.
- Once approved, no further questions — just implement.

**Related:** [[reference_pwa_offline_pattern]] (the BSM calculator that triggered this), [[feedback_todo_done_to_completed_section]] (similar "user clarifies after seeing the first attempt" pattern).
