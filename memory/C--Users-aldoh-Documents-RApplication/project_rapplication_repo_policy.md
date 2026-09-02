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
- `renv/library/`, `renv/staging/`, `renv/cellar/`, `renv/local/`, `renv/lock/`, `renv/python/`, `renv/sandbox/` — auto-handled by renv-managed `renv/.gitignore` inside each repo (see `reference_renv_discipline.md`)
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
3. If a memory or doc says "NEVER commit X" and X is in the tracked list above, the rule is stale — verify and surface (see `feedback_verify_before_acting.md`).
4. When unifying a new rule across repos, audit `git ls-files` per repo (see `feedback_unify_rule_audit_full_surface.md`); rule may be silently violated in repos with no modified files.

**Audit trail — TODO #55 closure commits** (surveyed 2026-04-21, executed
2026-04-30; commit + push, all to upstream). Use these when something asks "since
when has config.yml been tracked?" or "where was the renv unify decision made?":

| Repo | Branch | Commit |
|---|---|---|
| RApplication | master | `cf3c6f1` (+ `fab7ef5` for the TODO update) |
| Tuser | master | `8ba8753` |
| RPreTrade | refactor/server-modularization-hybrid | `b8a4272` |
| RReporting | stable/prod | `8605b79` |
| RJournal | main | `dd358af` |
| ROrder | stable/prod | `54b1c4e` |
| RStudies | main | `466e632` |

Full details and follow-ups in `RApplication/docs/TODO.md` items #55 (DONE) and
#58 (single shared config.yml — open).
