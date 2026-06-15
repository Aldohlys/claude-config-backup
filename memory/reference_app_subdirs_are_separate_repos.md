---
name: reference_app_subdirs_are_separate_repos
description: "Each app/package subdir (Tuser, Tdata, ...) is its OWN git repo — root git status won't show their edits"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1c542077-4f43-4c81-8b96-d63da2d47d58
---

The 8 sub-projects — Tuser, Tbasics, Tdata, Tlogger, RReporting, RJournal, ROrder, RStudies, RPreTrade — are each an INDEPENDENT git repo with its own GitHub remote (e.g. `Aldohlys/Tuser`). They are NOT submodules (no `.gitmodules`) and the outer `RApplication` repo tracks ZERO files inside them (`git ls-files Tuser/` → 0).

Consequence: running `git status` at `C:\Users\aldoh\Documents\RApplication` will NOT show edits made under `Tuser/...`, `Tdata/...`, etc. To see/commit/push app changes you must operate inside the subdir: `cd Tuser && git status / git add / git commit / git push origin master`.

These app repos often carry unrelated pre-existing modified/untracked files (work-in-progress in other modules). Stage only the files for the current task — don't `git add -A`. See [[feedback_git_status_before_commit]].

The "unified RApplication repo policy" ([[project_rapplication_repo_policy.md]], [[project_todo_55_unified_repo_policy]]) means consistent .gitignore RULES applied per-repo, NOT one combined repo. The Bash tool runs bash, not PowerShell — don't use PowerShell here-string `@'...'@` for `-m` (it leaks literal `@` into the message); use `-F <tmpfile>` or a plain `-m "..."`.
