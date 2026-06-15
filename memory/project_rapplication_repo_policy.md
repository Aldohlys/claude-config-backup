---
name: project_rapplication_repo_policy
description: Canonical statement of what is tracked / ignored / locally-excluded across all 7 R apps under RApplication. Set 2026-04-30 via TODO #55.
type: project
---

Unified policy across all 7 R apps under RApplication (RApplication root + Tuser, RPreTrade, RReporting, RJournal, ROrder, RStudies). Set 2026-04-30 via TODO #55. Tdata/Tbasics/Tlogger/Tstudy follow the same conventions where applicable (Tdata is a package — no config.yml).

**Tracked in git, everywhere:**
- `config.yml` — verified secret-free 2026-04-30 (paths, account list, smtp host/user/from/to, IBKR timezone, /analyze parameters in RStudies)
- `renv.lock`
- `.Rprofile`
- `renv/activate.R`, `renv/settings.json` (if present), `renv/.gitignore`
- `data/mydb.sql` (RApplication only — DB dump for sync to VM)

**Ignored everywhere (gitignore):**
- `quotes/` — runtime parquet quote caches (parallel to `chains/`/`strikes/` cache infrastructure managed by tdata_py.parquet_storage and TODO #27 janitor)
- `renv/library/`, `renv/staging/`, `renv/cellar/`, `renv/local/`, `renv/lock/`, `renv/python/`, `renv/sandbox/` — auto-handled by renv-managed `renv/.gitignore` inside each repo (see `reference_renv_safe_to_track.md`)
- standard R/RStudio: `.Rhistory`, `.RData`, `.Ruserdata`, `.Rproj.user/`, `*.log`, `logs/`
- Generated outputs: `*.html`, `*.pdf` (in RApplication root + RStudies)

**Locally excluded — never enter git index, even at root:**
- `Renviron.site` — contains `ALERT_EMAIL_PASSWORD` and other secrets. Excluded via `RApplication/.git/info/exclude` (so `git add .` won't catch it). Encrypted off-repo backup is the user's responsibility, not git's.

**Why this design:**
- Secrets isolated to one file (Renviron.site), excluded by mechanism rather than convention.
- All other configuration in git so multiple machines (dev box + VM) can run from a clone without per-machine config drift.
- 6 of 7 config.yml are byte-identical → drift is a real ongoing risk → see TODO #58 for consolidation to a single canonical config.

**How to apply:**
1. When asked "should I commit X?" check whether X matches a tracked or ignored pattern above.
2. When introducing a new app or new repo under RApplication, mirror this policy: `git add .Rprofile renv.lock renv/{activate.R,settings.json,.gitignore} config.yml`; gitignore `quotes/`; do NOT track Renviron.site.
3. If a memory or doc says "NEVER commit X" and X is in the tracked list above, the rule is stale — verify and surface (see `feedback_verify_policy_assertions.md`).
4. When unifying a new rule across repos, audit `git ls-files` per repo (see `feedback_unify_rule_audit_full_surface.md`); rule may be silently violated in repos with no modified files.

**Audit trail:** TODO #55 closure commits in `project_todo_55_unified_repo_policy.md`.
