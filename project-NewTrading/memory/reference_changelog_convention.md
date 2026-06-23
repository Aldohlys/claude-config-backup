---
name: change.log convention
description: User keeps rolling change logs at repo root as `change.log` (lowercase, .log extension), markdown-formatted, newest entry prepended
type: reference
originSessionId: dbd241e8-773c-4681-bb7b-e2c19d1a98e3
---
Both `RApplication\change.log` and `NewTrading\change.log` follow the same convention:

- **File:** `change.log` at repo root (lowercase, `.log` extension — not `CHANGELOG.md`).
- **Order:** newest entry at the top (prepend, don't append).
- **Format per entry:**
  ```
  ## YYYY-MM-DD - Short title

  **Project:** RepoName (subprojects affected)

  **Summary:** 1-2 sentence narrative of what shipped and why.

  **Changes:**

  1. **`path/to/file`** — what changed and why.
  2. ...

  **Notes:** (optional) follow-up items, conventions established, cross-repo cross-references.
  ```
- Date suffixes like `(later)` or `(part 2)` are used when multiple entries land on the same day.

**Gitignore status:**
- `RApplication\change.log` is gitignored (matches `*.log`). Maintained as a local rolling log; the user updates it but does not commit it. It's the user's own reference, not a public artifact.
- `NewTrading\change.log` is **not** gitignored (created 2026-05-07). It IS committed.

**How to apply:** When updating either repo at the user's request, prepend a new entry following the format above. Match the prose style of recent entries — paragraphs of context, not just bullet lists. Cross-reference between repos when a single piece of work touches both (the 2026-05-07 launcher work spans both files).
