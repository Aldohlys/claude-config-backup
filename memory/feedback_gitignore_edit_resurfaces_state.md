---
name: feedback_gitignore_edit_resurfaces_state
description: After removing a pattern from .gitignore, re-run `git status` and inspect any newly visible files before staging.
type: feedback
originSessionId: 66254968-8c04-4ad8-bfc3-2b1d56f655a0
---
After editing `.gitignore` (especially removing an exclusion), re-run `git status` and visually scan any new `??` entries. Don't assume the file set you planned to commit is the same set git now sees.

**Why:** During TODO #55, removing `config.yml` from `RApplication/.gitignore` exposed `scripts/tracking_manager/config.yml` (~990 bytes, present since Feb but silently ignored). Without the re-check, that file would have been left untracked indefinitely while every other config got committed, perpetuating the inconsistency the user was trying to fix. The same risk applies in reverse: adding a pattern can hide files that were about to be committed.

**How to apply:** Sequence is — edit `.gitignore` → `git status --short` → inspect every `??` not in your original plan → decide its disposition (commit / explicit ignore / delete) → THEN stage. Treat newly-surfaced files like any other unknown file: read them, check for secrets, confirm with user if the disposition is non-obvious.
