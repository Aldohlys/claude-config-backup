---
name: project_tdata_install_locations
description: Tdata exists in user RLibrary, an renv sandbox, and 6 app renv libraries. As of 2026-04-27 the /build slash command + build_package.R deploy step covers RLibrary in addition to the 6 app renvs — only the RApplication renv sandbox (#2) still needs a separate install.
type: project
originSessionId: 967d9aec-f0cc-4ee7-a35d-cd2e96f70722
---
Tdata is installed in (at least) 8 distinct locations on this machine:

1. `C:/Users/aldoh/Documents/RLibrary/Tdata` — **user library**. Used by `Rscript` invocations that don't activate any renv (CWD without `.Rprofile`). Loaded by Desktop batch files: `IBKR.bat`, `Gonet.bat`, `IBKR - Currencies.bat`, `IBKRMetrics.bat`, and any scheduled task whose script lives outside an renv-activated directory.
2. `C:/Users/aldoh/AppData/Local/R/cache/R/renv/library/Tdata-9fd37c52/...` — **renv sandbox** for the `RApplication/` directory itself. Loaded when `Rscript` is run from inside `RApplication/` (because that directory has its own `.Rprofile` that activates renv).
3-8. `<RApplication>/<App>/renv/library/.../Tdata` for each of `Tuser`, `RPreTrade`, `RReporting`, `RJournal`, `ROrder`, `RStudies` — **app-specific renvs**, one per Shiny app. Loaded by their respective `.bat` launchers via the app's `.Rprofile`.

**Why it matters:** Before 2026-04-27, `build_package.R`'s deploy step (and the `/build` slash command) iterated over the 6 app renvs (#3-#8) only. It did **not** touch #1 (`RLibrary`) or #2 (`renv` sandbox), so after a `/build Tdata` the apps were up to date but Desktop batch files / scheduled tasks loading from `RLibrary` kept running the old version.

**Current state (2026-04-27):** `deploy_package()` now also calls `install.packages(package_file, lib = .libPaths()[1], repos = NULL, type = "source")` after the renv loop, which writes into RLibrary (#1). #2 (the `RApplication/` renv sandbox) is still NOT covered — install separately if a script run from inside `RApplication/` needs the new version.

**Why:** 2026-04-21 — built Tdata 5.10.5, deploy step succeeded for the 6 apps, but `IBKR.bat` (and previous day's `daily_portfolio_update.R`) still loaded 5.9.14 because `RLibrary` was unchanged. Required two extra explicit installs:
- `install.packages(tarball, lib='C:/Users/aldoh/Documents/RLibrary')` for #1
- `install.packages(tarball)` from inside `RApplication/` for #2

**How to apply:**
- `/build` now keeps RLibrary in sync automatically — no manual install needed for Desktop launchers / scheduled tasks.
- If something specifically needs to run from inside `RApplication/` itself (renv sandbox #2), install explicitly:
  ```bash
  cd C:/Users/aldoh/Documents/RApplication && Rscript -e "renv::install('<tarball>')"
  ```
- To verify all libraries: enumerate the 8 paths and check `packageVersion('Tdata', lib.loc=...)` for each, OR for renv ones, `cd <app> && Rscript -e "source('renv/activate.R'); cat(as.character(packageVersion('Tdata')))"`
