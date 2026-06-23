---
name: feedback_unify_rule_audit_full_surface
description: When the user says "do X everywhere", audit `git ls-files` and `.gitignore` per repo to find places where the rule has been silently violated, not just where it's currently in flux.
type: feedback
originSessionId: 66254968-8c04-4ad8-bfc3-2b1d56f655a0
---
When the user says "the rule needs to be the same everywhere" (e.g. "track renv.lock everywhere", "use convention X across all apps"), the scope is almost always larger than the files currently showing as modified or untracked. Audit `git ls-files` and `.gitignore` in EVERY relevant repo against the rule before quoting effort or executing.

**Why:** During TODO #55, the user said "renv.lock — track everywhere." Initial read: "commit the 3 modified `renv.lock` files (RPreTrade, ROrder, RStudies)." Actual scope after `git ls-files` audit: 3 other repos (Tuser, RReporting, RJournal) had `renv.lock` and/or `renv/` in `.gitignore` and had NEVER tracked them — so "everywhere" required un-ignoring + first-time-adding `.Rprofile`, `renv.lock`, `renv/{activate.R,settings.json,.gitignore}` in each, on top of the 3 modified files. ~10 minutes vs ~40 minutes of work, and the gap was invisible from `git status` alone.

**How to apply:** For any "unify the rule" directive:
1. State the rule precisely (which file/pattern, which repos in scope).
2. For each repo in scope, run `git ls-files | grep <pattern>` AND `cat .gitignore | grep <pattern>` to see actual current state.
3. Build a table: repo × current-state × target-state × delta.
4. Quote the real effort and surface the table to the user before executing.
5. Watch for repos where the rule has been silently violated (file gitignored AND never tracked) — those need un-ignore + add, not just commit.
