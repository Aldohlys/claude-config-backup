---
name: feedback_renv_restore_corrupts_activate
description: "How to align a project's installed renv version with its lockfile WITHOUT breaking the bootstrap. The naive renv::restore rewrites activate.R leaving an unsubstituted ..md5.. placeholder, which makes the app fail to launch at all."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6f5786db-5489-4ea1-851d-9477ce02eb6c
---

All 6 app renv libraries (Tuser, RPreTrade, RReporting, RJournal, ROrder, RStudies) can drift in the **renv version itself**: the lockfile + working-tree `renv/activate.R` request one version (e.g. 1.2.2) while the project library has an older one installed (e.g. 1.1.5). Symptom on every launch: `renv X was loaded from project library, but this project is configured to use Y`. This is **non-fatal** — old renv still resolves the correct triplet libpath (`renv/library/windows/R-4.4/x86_64-w64-mingw32/`), so packages still load. It is NOT the cause of a `library(Tdata)` "package not found" crash (that's the install-location gap — see [[project_tdata_install_locations]]; renv app libs also need the package physically present).

**The trap (2026-06-02):** fixing the version warning by running, inside the app, `renv::restore(packages="renv", prompt=FALSE)` **under the old renv** installs the new renv BUT rewrites `renv/activate.R` leaving the literal placeholder `attr(version, "md5") <- ..md5..` (writer template not substituted). Next launch then dies hard with `Erreur dans eval(...): objet '..md5..' introuvable` — the app won't start at all. Worse than the original cosmetic warning.

**Why:** the old renv's activate.R writer used the new version's template without filling the `..md5..` / `..version..` placeholders.

**How to apply — align installed renv to the lockfile safely:**
- **If `activate.R` already requests the target version WITH a valid md5** (`attr(version,"md5") <- "bb69b6403b1bad0442657e9e8e57cc83"` for 1.2.2): do NOT run restore. Just drop the installed renv pkg dir in. Network-free copy from a known-good app lib:
  ```bash
  SRC=<good-app>/renv/library/windows/R-4.4/x86_64-w64-mingw32/renv
  DST=<app>/renv/library/windows/R-4.4/x86_64-w64-mingw32/renv
  rm -rf "$DST" && cp -r "$SRC" "$DST"
  ```
- **If `activate.R` got corrupted with `..md5..`** (or you must install fresh): install the target renv, then regenerate activate.R by loading the NEW renv directly from the lib (bypassing the broken bootstrap) and calling activate:
  ```r
  lib <- "<app>/renv/library/windows/R-4.4/x86_64-w64-mingw32"
  library(renv, lib.loc = lib); renv::activate(project = "<app>")
  ```
  This writes a properly-substituted activate.R for the installed version.
- **Verify** per app: `cd <app> && Rscript -e 'cat(as.character(packageVersion("renv")))'` should print the target version with NO mismatch/`..md5..` warning.

**Separately:** all 6 app lockfiles are also **out of sync for normal packages** — installed libs are NEWER than recorded (e.g. ggplot2 4.0.3 lib vs 3.5.1 lock, shiny 1.13 vs 1.10; ~70-90 pkgs each). Apps run fine on the newer installed versions; renv just warns "out-of-sync". To clear it, run `renv::snapshot()` per app to record current library state into the lockfile (deliberate — rewrites all 6 lockfiles). Not done as of 2026-06-02.
