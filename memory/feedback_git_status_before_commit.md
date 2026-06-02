---
name: Re-run git status immediately before git commit
description: When chaining git add + git commit + git push, always re-check git status first to avoid sweeping pre-staged unrelated files into a topical commit
type: feedback
originSessionId: 6bd2f12e-5604-4dc1-8c3d-43f23ae6d408
---
Before running `git commit` (especially as part of a chained `git add ... && git commit ... && git push`), re-run `git status` to see exactly what is staged. Or use `git commit -- <paths>` to scope the commit explicitly to specific files.

**Why:** On 2026-04-27, a launcher-consolidation commit (12 .bat moves + launcher.py + .gitignore) silently bundled a pre-staged 234K-line `data/mydb.sql` diff because that file had been staged before the session started (`M  data/mydb.sql` showed M in the *index* column of the initial git status, easy to misread as "modified working tree"). The commit subject became misleading; it was already pushed to master, so no clean fix without a force-push (forbidden per Git Safety Protocol). User asked to leave it as-is.

The trap: `git add <paths>` adds; `git commit` commits *everything in the index*, not just what the most recent `git add` touched. They're independent operations. A `git status` snapshot from earlier in the session is stale by the time you commit.

**How to apply:**
- Before any `git commit` after a session that had pre-existing staged files, run `git status` again as the immediate previous step.
- Or, when the intent is "commit only these specific files", use `git commit -- <paths>` (the `--` form scopes the commit to those paths, ignoring other staged files).
- Reading git status: 1st column = index/staged, 2nd column = working tree. `M  file` (M in col 1, space in col 2) means already staged. `?? file` = untracked. `M file` (space col 1, M col 2) = modified-not-staged.
- This is more important on the user's repos because the user often has multi-session work-in-progress staged across many files.
