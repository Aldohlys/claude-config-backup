---
name: feedback_comment_describe_shipped_code
description: "Code comments must explain the shipped code itself, never the implementation-plan phase it came from"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4152f0ec-a745-46da-8951-1e46783a5f4b
---

Comments must describe the **code as shipped** — what it does and why it's
correct — so they stand on their own for a reader seeing the code cold. Do NOT
reference the implementation plan that produced them: "Phase 5", "Phase 2a",
"Risk-refactoring (TODO #38)", or bake a plan number into an identifier
(`.PHASE5_STRATEGIES`). The user flagged this on 2026-06-10 ("what is Phase 5?")
— plan-phase comments are transient scaffolding that mislead and rot once the
plan completes.

**Why:** traceability to a plan/issue belongs in git history and commit
messages, not the source. A comment saying "Phase 5 generalization" is wrong the
moment Phase 5 stops being a live concept.

**How to apply:** write the rationale inline ("BPT's worst case isn't clean-cut,
so cap it at the global budget"), name identifiers by what they ARE
(`.GENERIC_RISK_STRATEGIES`, not `.PHASE5_STRATEGIES`). Distinguish two kinds of
plan references when cleaning up: pointers to *completed* phases = debt (remove);
pointers to *open* tracked work ("until the Stop column lands", an open TODO #) =
keep. So it's triage, not a regex delete. Tracked as [[TODO #74]]; see also
docs/LESSONS_LEARNED.md "Code Comments Describe the Shipped Code".
