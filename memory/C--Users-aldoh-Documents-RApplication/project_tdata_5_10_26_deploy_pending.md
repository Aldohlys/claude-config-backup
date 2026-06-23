---
name: project_tdata_5_10_26_deploy_pending
description: Tdata 5.10.26 (cache-warning surfacing) committed but NOT yet deployed to app libraries
metadata: 
  node_type: memory
  type: project
  originSessionId: 92762a60-9765-437c-baa5-843b8fe004b9
---

As of 2026-06-14, **Tdata 5.10.26** is committed + pushed (`stable/prod` commit `3a7d207`) but **not deployed** — the build/deploy was deferred because the Tdata tree had concurrent #67 trades/cash work uncommitted ([[feedback_build_package_git_add_all.md]]).

**What's pending:** the new `surface_cache_warnings()` (TODO #27 Part 1 R-side surfacing — see [[project_vm_no_option_cache_consumer]]) is in git but the running apps still load the OLD Tdata, so stale-cache deletions are NOT yet surfaced to the user. The DESCRIPTION version bump to 5.10.26 is already committed.

**How to apply:** once the Tdata tree is clean (other session's cash/trades work landed), deploy with `build_package('Tdata', auto_version = FALSE, force = TRUE)` — `force` because the committed bump leaves no diff for the default "nothing to build" check. Verify it reached all 8 install locations ([[project_tdata_install_locations]]). Delete this memory once deployed.
