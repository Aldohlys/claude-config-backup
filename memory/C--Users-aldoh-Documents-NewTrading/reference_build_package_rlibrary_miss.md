---
name: reference-build-package-rlibrary-miss
description: build_package.R deploys Tdata to the 6 renv apps but RLibrary stayed on the old version (5.20.12 after the 5.20.13/14 builds); it also commits and deploys despite failing tests - check versions and test output after every build
metadata:
  node_type: memory
  type: reference
  originSessionId: e87e0ecc-b1ff-410a-a8f3-4449c3418f84
  modified: 2026-10-08T08:31:21.555Z
---

Seen 2026-10-08 with `build_package("Tdata", auto_version = TRUE, cascade = FALSE)` run through Rscript from `RApplication/`:
- After both builds, all six renv apps had 5.20.14, but `RLibrary/Tdata` was still 5.20.12, even though the 5.20.13 log said "Deployed to: ... system library". Fix: `install.packages("<RApplication>/Tdata_X.tar.gz", lib = "C:/Users/aldoh/Documents/RLibrary", repos = NULL, type = "source")` from another directory. RLibrary is what plain Rscript tasks load.
- The script **does not stop on test failures.** 5.20.14 was committed, pushed and deployed with 10 failures in test-trades.R (stale mocks after a refactor). Grep the log for `FAILURE` and `[ FAIL` before reporting success.
- The script's commit title is always "Build Tdata vX: Package updates"; the details live in CHANGELOG.md.
- A run can take close to 10 minutes. Keep the Bash timeout at least 590 s and check every target's DESCRIPTION afterwards (the loop is in [[reference_tdata_install_topology]]).

Related: [[project_build_package_renv_prune_todo]], [[feedback_tdata_rebuild_restart_r]].

2026-10-16 (5.21.4): build ran >10 min → use run_in_background; RLibrary WAS updated this time; 0 FAIL / 1145 PASS. The script does not commit the apps' `renv.lock` pins — commit them per app with pathspec ("Pin Tdata X (...)") and push.
