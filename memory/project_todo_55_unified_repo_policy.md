---
name: project_todo_55_unified_repo_policy
description: Index of the 7 commits that closed TODO #55 (clean uncommitted state across all repos) and established the unified config.yml/renv tracking policy on 2026-04-30.
type: project
---

TODO #55 (clean uncommitted state across all 9 repos) closed 2026-04-30. Established the unified RApplication repo policy now in force across all 7 R apps.

**Commit refs (commit + push, all to upstream):**

| Repo | Branch | Commit |
|---|---|---|
| RApplication | master | `cf3c6f1` (+ `fab7ef5` for TODO update) |
| Tuser | master | `8ba8753` |
| RPreTrade | refactor/server-modularization-hybrid | `b8a4272` |
| RReporting | stable/prod | `8605b79` |
| RJournal | main | `dd358af` |
| ROrder | stable/prod | `54b1c4e` |
| RStudies | main | `466e632` |

**Why:** policy umbrella for "what's tracked, what's ignored, what's secret" across all 7 R apps. Use this entry to find the canonical commits when something asks "since when has config.yml been tracked?" or "where was the renv unify decision made?"

**How to apply:** if a future "should X be committed?" question maps to one of these patterns (config.yml, renv state, .Rprofile, quotes/, Renviron.site), the answer lives in the policy file `project_rapplication_repo_policy.md` (umbrella) — this entry is just the audit trail.

Full details and follow-ups in `RApplication/docs/TODO.md` items #55 (DONE) and #58 (single shared config.yml — open).
