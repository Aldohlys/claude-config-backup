---
name: feedback_todo_done_to_completed_section
description: "When marking a TODO item DONE, move it out of docs/TODO.md into docs/TODO_COMPLETED.md (newest first), don't leave it in place."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3ed4255d-f6de-4459-bd06-43a46d51dc9a
---

Fully-DONE items live in **`docs/TODO_COMPLETED.md`**, reverse-chronological by completion date. New DONE items go there — don't leave them in the active `docs/TODO.md`.

**Location corrected 2026-09-01:** this used to be a trailing `## ✅ COMPLETED ITEMS` section *inside* `TODO.md`. That section no longer exists — the items were split into the separate `TODO_COMPLETED.md` file (CLAUDE.md documents it as "Completed & archived TODO items (moved out of TODO.md)"). Verify before quoting the old name. Note `TODO.md` still has stragglers marked `✅ DONE` in place (e.g. #77); that is untidiness, not the convention.

**Why:** the user (2026-05-18) flagged that DONE items were being marked in-place rather than relocated, which leaves the active section cluttered with completed work and makes it hard to scan for what's still open. The re-org commit `2e7883c` established the COMPLETED section convention and moved 8 stragglers (#66, #10, #62, #61, #55, #33, #32, #31).

**How to apply:**
- When closing a TODO item, write the resolution body, then **move** the whole entry to the **top** of `TODO_COMPLETED.md` (it is newest-first), and delete it from `TODO.md`.
- Keep PARTIAL / deferred-residual entries in the active section — they still have open work. Current examples: #52, #60.
- Pre-2026-03 fully-done items live in the external archive `docs/archive/TODO-ARCHIVE-2025-11-24.md`; `TODO_COMPLETED.md` holds 2026-03 onward.
- Order of operations: insert at top of `TODO_COMPLETED.md`, delete from `TODO.md`, single commit touching both. Most recent example: #78, commit `646e494`.
