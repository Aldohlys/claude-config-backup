---
name: audit-shared-rules-for-direction-blindness-when-extending-to-shorts
description: "Long-authored code often has implicit upward bias. When adding short support, audit every shared rule, helper, and formula end-to-end before declaring done — partial fixes leak wrong-direction results."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3b70ddd8-ed66-422b-bfce-e9ef777ec479
---

The /analyze redesign added `direction = "short"` to the entry pipeline. Multiple shared rules silently produced wrong-direction results because they were authored long-only:

| Shared rule (RStudies/reports/shared) | Long-only bug |
|---|---|
| `setup_chain_rr.R::compute_structural_target` | Emitted swing **highs** / 52w **high** / round **above** for shorts → targets at $109/$110 on a $100 short stock |
| `setup_chain_rr.R::compute_rr_entry` | BS forward pricing hardcoded `type="Call"` for the spread branch → bear-put debits priced as bull calls |
| `vehicle_rule.R::pick_vehicle_expiry` | Returned `"call"` even for shorts (label, not selection) |
| `setup_chain_rr.R::walk_chain_oi` | `effective_target = oi_cap_call` always — should be `oi_cap_put` for shorts |
| `swing_scanner/main.R::sector_rank_map` | Only ranked LONG-passing sectors; no short equivalent |
| `analyze/structures.R::.live_entry_framework` strike picker | Picked ATM call strike for shorts (round-up to $5 grid for calls) |

**Why:** each of these helpers was written for the swing_scanner's LONG-only universe; "direction" wasn't a concept. When /analyze added the `short` direction the bugs surfaced one by one across multiple smoke tests.

**How to apply:** when a code change adds `direction` (or any new dimension) to an existing system:

1. **Grep every helper** the new dimension transits through. Don't trust that "obvious" functions were direction-aware — they probably weren't.
2. **Run a smoke test in the new direction immediately** after each helper is wired, not at the end. Each layer can mask the bug at the layer below.
3. **Audit `type="Call"`, `> price`, `ceiling(...)`, `max(...)`, `swing_high`** — these formulas often encode upward bias.
4. **Default the new param to the old behavior** (e.g. `direction = "long"`) so existing callers (swing_scanner) keep working without touching them.
5. **Bash test**: would running this with `direction = "short"` make economic sense? If the formula says "the closer target above the spot" and you want a short, that's an upward-biased helper.

Related: [[project_analyze_redesign_2026_05]] documents the specific fixes by step.
