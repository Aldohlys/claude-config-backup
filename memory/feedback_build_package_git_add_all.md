---
name: feedback_build_package_git_add_all
description: "build_package.R does a blanket `git add .` — never /build with a dirty tree of others' work"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 92762a60-9765-437c-baa5-843b8fe004b9
---

`build_package.R` stages the **entire** package repo with `system("git add .")` (around line 595) before committing, bumping the version, deploying to all renv apps + RLibrary, and pushing.

**Why it matters:** if the package working tree contains unrelated/concurrent uncommitted changes (e.g. another session mid-task), `/build` sweeps them ALL into the build commit and **deploys + pushes** them — an outward-facing publish of someone else's unfinished code.

**How to apply:** before running `/build <pkg>`, run `git status` in that package dir. If the tree has changes that aren't yours, do NOT build. Instead: commit only your own files manually (explicit pathspec), bump the version + CHANGELOG yourself, and defer the deploy until the tree is clean — then deploy the already-committed version with `build_package('<pkg>', auto_version = FALSE, force = TRUE)` (force needed because the committed bump leaves a clean tree; see [[feedback_build_package_force_after_manual]]). Also re-check `git status` immediately before any commit to catch pre-staged ride-alongs like `data/mydb.sql` ([[feedback_git_status_before_commit]]).
