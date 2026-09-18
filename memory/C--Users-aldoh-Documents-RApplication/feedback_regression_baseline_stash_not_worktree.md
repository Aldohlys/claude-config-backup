---
name: feedback_regression_baseline_stash_not_worktree
description: "To prove test failures predate your change, stash your files in place — a git worktree lacks every gitignored file, so its test counts are not comparable"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 81e8468e-53c1-4b02-9a9c-9c1f7f2651b6
  modified: 2026-09-03T18:51:41.505Z
---

2026-09-03, proving that 22 RPreTrade test failures predated my edit:

| where | result |
|---|---|
| `git worktree add --detach <tmp> HEAD` | 60 tests, 38 failed |
| `git stash push -- <my files>`, in place | **71 tests, 22 failed** — identical to my run |

**Why:** a worktree checks out tracked files only. No `config.yml` hardlink, no
`renv` library, no other gitignored state — so `global.R` degrades and whole
test blocks are skipped. The totals differ for reasons that have nothing to do
with the code, and comparing them proves nothing.

**How to apply:**

- Baseline in place: `git stash push -- <only your files>`, run, `git stash
  pop`. Naming the files keeps unrelated pre-existing churn (`renv.lock`,
  `renv/activate.R`) out of the stash — cf. [[feedback_git_status_before_commit]].
- Run `git worktree add` **from inside the app directory**. From the
  RApplication root you get a worktree of the RApplication repo, which does not
  contain the app's files at all — each app subdir is its own repo
  ([[reference_app_subdirs_are_separate_repos]]). I made this mistake twice
  before noticing the test set had changed.
- `git worktree prune` does not remove a live worktree; use `git worktree remove
  --force <path>` and check `git worktree list` afterwards.
- Useful side effect: the stash round-trip normalises line endings through
  `core.autocrlf`, which *undoes* the LF churn that `sed`/heredoc file splicing
  introduces into CRLF sources. Check `git diff --stat` for a full-file diff
  before assuming your edit was small.
