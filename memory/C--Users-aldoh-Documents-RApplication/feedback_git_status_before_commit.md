---
name: feedback_git_status_before_commit
description: "Committing in a tree that holds other people's work — pre-staged files ride along unless every commit is scoped with --only, and when your edits sit on top of someone else's uncommitted changes, split what is separable and disclose what is not"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T04:12:19.812Z
---

The user routinely has multi-session work-in-progress, staged and unstaged,
across many files. Every commit here happens in someone else's tree.

## 1. `git add X && git commit` commits the WHOLE index

`git add <paths>` adds; `git commit` commits **everything already in the index**,
not just what the last `git add` touched. They are independent operations, and a
`git status` from earlier in the session is stale by the time you commit.

**This has now happened twice with the same file, `data/mydb.sql`:**

- 2026-04-27: a launcher-consolidation commit (12 `.bat` moves + launcher.py +
  .gitignore) silently bundled a pre-staged 234K-line `mydb.sql` diff. Already
  pushed, so no clean fix without a force-push. Left as-is.
- 2026-08-27: `git add scripts/Tuser-spread.bat && git commit -m ...` produced
  "2 files changed, **19017 insertions**" for a 28-line `.bat` — `mydb.sql` had
  been staged since before the session. Caught because the insertion count was
  absurd for the file being committed. Not yet pushed, so it was fixed with
  `git reset --soft HEAD~1`, `git restore --staged data/mydb.sql`, then redoing
  every commit with `--only`.

Knowing this rule is not enough — it was already written here and still happened.

**How to apply:** whenever the tree has ANY pre-existing staged file, make
`git commit --only <paths> -m ...` the default form for every commit, not a
special case. It pins the commit to those paths and ignores the rest of the
index. Read the insertion count in the commit output as a sanity check: a line
count far larger than the file you meant to commit means something rode along.

Reading `git status`: **1st column = index/staged, 2nd = working tree.**
`M  file` (M in col 1) is already staged — easy to misread as "modified".
` M file` is modified-not-staged. `MM file` is both. `?? file` is untracked.

## 2. When your work sits ON TOP of someone else's uncommitted changes

Before committing a file you edited, check whether it was already dirty. Diff the
working tree against HEAD, not against what you remember writing.

2026-08-27, the Tuser spread module: **both** files I touched already carried
uncommitted work from another session.

- `spread/app.R` — **separable.** My change was one isolated hunk (a text blurb);
  `lookup_ticker()` and `resolve_price()` were someone else's, in other hunks.
  Split it: `git diff -- <file> > full.patch`, keep the header lines plus only
  your hunk, then `git apply --cached <that.patch>`. The file goes `MM`: your
  hunk staged, theirs still unstaged. Verify with
  `git diff --cached -- <file>` before committing.
- `spread/view/spreadUI.R` — **not separable.** HEAD was 266 lines; the working
  tree had two fixes inside the very functions my rewrite replaced. No hunk
  boundary existed between their work and mine.

When it cannot be split, commit it and **say so in the commit message** — name
the earlier fixes being carried. Then check whether a pending CHANGELOG entry
now describes work that has already shipped, and add a note there pointing at
the commit, so the record matches history.

**Why it matters:** sweeping another session's unfinished work into your commit
publishes it under a message that does not describe it. See
[[reference_build_package_gotchas]] §1 — `build_package.R` does a blanket
`git add .`, so `/build` in a dirty tree does this automatically and pushes.

**Corollary:** a file disappearing from `git status` mid-session is not
necessarily something you reverted — the user may have committed it underneath
you. Check `git log --format='%h | %ad | %s' --date=format:'%Y-%m-%d %H:%M'`
before concluding you destroyed work ([[feedback_verify_before_acting]]).
