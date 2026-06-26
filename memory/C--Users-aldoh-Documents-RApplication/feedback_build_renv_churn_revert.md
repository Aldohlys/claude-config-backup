---
name: feedback_build_renv_churn_revert
description: "/build (build_package.R) runs renv::snapshot per app, churning each app's renv.lock + rewriting renv/activate.R — revert that churn after the build; the install itself is correct."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e7c37a4a-46bb-4b8f-a2ed-8b13b66f3b38
---

When `/build <pkg>` deploys, `build_package.R` runs `renv::snapshot()` inside every dependent app repo (Tuser, RPreTrade, RReporting, RJournal, ROrder, RStudies). Observed 2026-06-26 building Tdata 5.12.0, this produced unwanted git churn in each app repo:
- `renv.lock`: dozens of UNRELATED package version bumps (digest, MASS, R6, …) — snapshot blesses whatever drifted into the library.
- `renv/activate.R`: bumped renv `1.1.5 → 1.2.2` (+~112 lines, adds an `md5` attr).
- The just-built package is recorded **one version behind** in the lockfile (e.g. lock said `Tdata 5.11.0`) even though the installed files are the correct new version — a snapshot/cache quirk. The *install* is fine; only lockfile metadata is wrong.

**Why:** this is exactly the snapshot-blessing-drift behavior the user rejects ([[feedback_renv_leave_lockfiles_alone]]), plus the activate.R rewrite warned about in [[feedback_renv_restore_corrupts_activate]].

**How to apply:**
- After a `/build`, the package install (correct version in the active-R tree — see [[reference_tuser_multi_r_renv_trees]]) is what matters and is unaffected by reverting lockfiles.
- Revert the churn in every touched app repo: `git checkout -- renv.lock renv/activate.R` (some repos only have `renv.lock` modified). Then commit only real source changes, staging files explicitly so the renv churn never rides along ([[feedback_git_status_before_commit]]).
- Don't try to "fix" the lockfile to show the new version by re-snapshotting — it won't, and it violates the leave-renv-alone policy.
