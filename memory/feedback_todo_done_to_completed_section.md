---
name: feedback_todo_done_to_completed_section
description: "When marking a TODO item DONE in docs/TODO.md, move it to the trailing \"## ✅ COMPLETED ITEMS\" section, don't leave it in place."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3ed4255d-f6de-4459-bd06-43a46d51dc9a
---

`docs/TODO.md` has a trailing `## ✅ COMPLETED ITEMS` section that holds all fully-DONE items, reverse-chronological by completion date. New DONE items go there — don't leave them in the active section.

**Why:** the user (2026-05-18) flagged that DONE items were being marked in-place rather than relocated, which leaves the active section cluttered with completed work and makes it hard to scan for what's still open. The re-org commit `2e7883c` established the COMPLETED section convention and moved 8 stragglers (#66, #10, #62, #61, #55, #33, #32, #31).

**How to apply:**
- When closing a TODO item, write the resolution body, then **move** the whole entry to the bottom of the COMPLETED section (newest first).
- Keep PARTIAL / deferred-residual entries in the active section — they still have open work. Current examples: #52, #60.
- Pre-2026-03 fully-done items live in the external archive `docs/archive/TODO-ARCHIVE-2025-11-24.md`; the COMPLETED section holds 2026-03 onward.
- Order of operations: insert at top of COMPLETED, delete from in-place location, single commit.
