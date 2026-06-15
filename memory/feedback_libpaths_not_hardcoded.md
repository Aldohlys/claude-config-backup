---
name: feedback_libpaths_not_hardcoded
description: In build_package.R and any other R script that needs to install into the user's main (non-renv) R library, prefer `.libPaths()[1]` over hardcoding `C:/Users/aldoh/Documents/RLibrary`. The user's library order is configured via `R_LIBS_USER` in Renviron.site and could be reordered or moved.
type: feedback
originSessionId: 38ba25ab-2b85-4e02-b278-742aad088c20
---
When extending deployment / install logic to target the user's main R library (the non-renv "system" library that Desktop launchers and scheduled tasks load from), use `.libPaths()[1]` rather than the literal path `C:/Users/aldoh/Documents/RLibrary`.

**Why:** The library search order is driven by `R_LIBS_USER` set in `<R_HOME>/etc/Renviron.site` and the user's environment — currently it resolves to `C:/Users/aldoh/Documents/RLibrary` first, but that ordering is configuration, not a constant. Hardcoding the path breaks silently if the user reorders `R_LIBS_USER`, moves RLibrary, or runs the script under a different R version with a different layout. `.libPaths()[1]` always tracks the active R session's primary library, which is what `install.packages()` defaults to anyway.

**How to apply:**
- In `build_package.R::deploy_package()` (and any analogous deploy/install code): use `sys_lib <- .libPaths()[1]; install.packages(tarball, lib = sys_lib, repos = NULL, type = "source")`.
- Log the resolved path so the build output is self-documenting (e.g. `cat("📚 Deploying to system R library:", sys_lib, "...\n")`).
- Same principle for any future cross-machine or VM scripts — `.libPaths()` is portable; absolute paths are not.
- Exception: when you specifically need a *non-default* lib (e.g. an renv project library), use `renv::paths$library(project = app_dir)` — also portable, also no hardcoded paths.
