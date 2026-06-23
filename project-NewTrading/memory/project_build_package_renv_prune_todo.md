---
name: project_build_package_renv_prune_todo
description: "build_package.R quick_build prunes the devtools toolchain via renv snapshot, breaking every Tdata build — needs fixing"
metadata: 
  node_type: memory
  type: project
  originSessionId: c63ff519-1cac-4027-a4b5-d2730bebb486
---

`RApplication/build_package.R` `quick_build()` is broken: its `renv::snapshot(force=TRUE)` step rewrites `Tdata/renv.lock` and **prunes the dev toolchain** (`sessioninfo`, `rcmdcheck`, `gert`, `gh`, `gitcreds`, `downlit`, `urlchecker`, `httr2`, `whisker`, `zip`, … all shown as `[ver -> *]` = removed, because they aren't Tdata *runtime* deps) — then the very next step, `devtools::build()`, fails with `aucun package nommé 'sessioninfo' n'est trouvé`. This breaks **every** Tdata build, not just one change; re-running re-prunes each time. The build dies before the deploy step, so nothing gets half-installed (apps keep running the prior version).

**Why:** discovered 2026-06-05 deploying the CAD interest-rate fetcher ([[reference_tdata_interest_rate_utils]] work). Had to bypass `quick_build` entirely and deploy via direct `R CMD INSTALL --library=<lib> --no-multiarch --no-docs <Tdata source>` into `RLibrary` + the 6 app renv libs (RJournal/ROrder/RPreTrade/RReporting/RStudies/Tuser, R-4.4) — see [[reference_tdata_install_topology]]. That path skips version-bump + auto-commit + roxygen `document()`, so the man pages can drift (man/ is gitignored in Tdata anyway, regenerated on build).

**How to apply:** fix options — (a) run `devtools::build()` BEFORE the `renv::snapshot`, (b) `renv::restore` the dev deps right before build, (c) declare the dev toolchain so the snapshot stops pruning it, or (d) drop the in-build snapshot. Until fixed, deploy Tdata via the direct `R CMD INSTALL` loop. Opened 2026-06-05.
