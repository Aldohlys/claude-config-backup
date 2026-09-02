---
name: reference-tuser-repo
description: RApplication/Tuser is its own git repo (Aldohlys/Tuser, master), not a submodule — its changes never show in RApplication's git status, and it uses CHANGELOG.md, not change.log
metadata:
  type: reference
---
`C:\Users\aldoh\Documents\RApplication\Tuser` is a **separate git repository**, remote `https://github.com/Aldohlys/Tuser.git`, branch `master`. It sits inside RApplication's working tree but is not a submodule and is not tracked by RApplication.

Consequence: editing a file under `Tuser/` and then running `git status` in `RApplication` shows nothing. Run git from inside `Tuser/`. This is easy to miss because [[project-rapplication-repo-policy]] frames Tuser as one of "7 R apps under RApplication", which reads as one repo.

**Changelog:** Tuser keeps `CHANGELOG.md` at its root in Keep a Changelog format — `## [YYYY-MM-DD] - Short title`, then `### Added` / `### Changed` / `### Fixed`, newest entry prepended. Entries are dense: bold **path/file** (`function`), then what changed and the mechanism behind it, with sub-bullets for detail. This is **not** the `change.log` convention that [[change-log-convention]] documents for RApplication and NewTrading — those two use a lowercase `change.log` with a different format. Tuser is the exception.

**Layout** (per `Tuser/CLAUDE.md`): box modules directly under `Tuser/`, each with `view/` for Shiny modules and `logic/` for its functions — `analysis`, `order`, `portfolio`, `symbol`, `studies`, `scan`, `ligne`, `vol`, `spread`, `journal`, `alert`, `account`, `ticker`, and others. `box` is never attached; always the `box::` prefix. RPreTrade, RJournal, ROrder etc. consume these modules by relative path, e.g. `box::use(../Tuser/scan/view/breakoutUI)`.

**How to apply:** when committing work in any Shiny module under `Tuser/`, run git from `Tuser/`, prepend a `CHANGELOG.md` entry in the house style before committing, and push to `origin master`. Leave `renv/activate.R` bootstrap bumps out of feature commits.
