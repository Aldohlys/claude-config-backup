---
name: git add returns exit 1 for tracked files in ignored dirs (breaks && chains)
description: `data/` is gitignored but `data/mydb.sql` is tracked; `git add data/mydb.sql` stages the file BUT emits a warning and returns exit code 1, killing `git add && git commit` shell chains
type: feedback
originSessionId: 3e9e8622-37ca-48f5-8116-4573ce74aa2f
---
When a file is tracked but its parent directory matches a `.gitignore` rule (e.g. `data/mydb.sql` under a `data/` rule), `git add <file>`:
- **Does** update the index (the file gets staged)
- **Also** prints "The following paths are ignored by one of your .gitignore files: <dir>" and **returns exit code 1**

**Why:** Git stages tracked files unconditionally, but treats the path-vs-gitignore mismatch as a warning condition.

**How to apply:**
- Use `git add -f data/mydb.sql` to suppress the warning and get exit 0 — safe because the file is already tracked.
- Or don't chain with `&&`: run `git add` first, ignore the exit code, then `git commit` separately.
- Symptom this caused on 2026-05-11: `git add data/mydb.sql && git commit ... && git push` aborted at the add step even though the file was correctly staged; I had to re-run the commit+push separately.
- RApplication has this exact setup: `data/` in `.gitignore` (line 52), `data/mydb.sql` tracked. Same pattern applies to any future `data/*.sql` commits.
