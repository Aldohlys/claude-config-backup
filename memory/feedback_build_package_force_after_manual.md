---
name: feedback_build_package_force_after_manual
description: /build refuses with "No changes since last commit" if you've committed the CHANGELOG + DESCRIPTION bump manually; force=TRUE bypasses the check and re-deploys
type: feedback
originSessionId: 110f4600-4558-489d-88b2-cdf4eb853b4a
---
`build_package('Tdata', auto_version = FALSE)` does a pre-flight `git_status` check and aborts with `✅ No changes since last commit - build not needed` if the working tree is clean. This is the right default — but it traps you if you've already done the version bump + CHANGELOG entry + commit + push manually and only want the build/deploy half (rebuild tarball + install into renv apps + system library + snapshot lockfiles).

**Why:** the script can't tell the difference between "nothing happened" and "version+changelog already committed manually; just deploy". Both look like clean repos.

**How to apply:** call with `force = TRUE`:

```r
build_package('Tdata', auto_version = FALSE, force = TRUE)
```

This bypasses the clean-repo guard and proceeds through tests + docs + tarball + 6-app renv deploy + system library install. `force = TRUE` also propagates to `renv::snapshot(..., force = TRUE)` per `build_package.R:263, 366`, which bypasses the "package from unknown source" preflight (relevant because Tdata is local-installed).

build_package.R will still create its own follow-up commit (`Build Tdata vX.Y.Z: Package updates`) — so you'll end up with two commits for the same version: your manual one and the auto one. Acceptable; not a bug.
