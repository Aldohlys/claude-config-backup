---
name: feedback_commit_with_pathspec
description: "Commit only the intended files with `git commit -- <paths>`; `git add X && git commit` also sweeps anything already staged (mydb.sql incident)"
metadata:
  node_type: memory
  type: feedback
  originSessionId: d0bd0d7e-3332-405c-afbe-bdb65214a63b
  modified: 2026-10-01T22:39:35.804Z
---

When committing on the user's behalf, check `git diff --cached --name-only` first and commit with an explicit pathspec (`git commit -m ... -- <paths>`). `git add <file> && git commit` commits everything already in the index too.

**Why:** 2026-10-02, committing RApplication docs/TODO.md also committed a pre-staged 116 MB data/mydb.sql; GitHub rejected the push (100 MB limit). Fixed by `git reset --soft HEAD~1` + pathspec commit (safe only because unpushed).

**How to apply:** every commit in RApplication / RStudies / NewTrading / Tdata / Tuser. In RApplication the dump is now LFS + weekly-only with a pre-commit guard ([[project_rapplication_repo_policy]]), but other repos have no guard.
