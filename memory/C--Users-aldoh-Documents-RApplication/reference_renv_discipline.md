---
name: reference_renv_discipline
description: "Everything about renv across the 6 app renvs — leave lockfiles alone (never snapshot to clear a warning), what's safe to track in git, how to align the renv version without corrupting activate.R, reverting /build's snapshot churn, and Tuser's per-R-version library trees."
metadata: 
  node_type: memory
  type: reference
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T02:22:10.170Z
---

Merged 2026-08-27 from five separate memories. Applies to the 6 app renvs:
Tuser, RPreTrade, RReporting, RJournal, ROrder, RStudies.

## 1. Leave the lockfiles alone (user policy, stated 2026-06-02)

The whole point of renv is that app libraries do **not** auto-track system package
updates. The lockfile is the pinned source of truth. The user updates libraries
**manually, as an occasional deliberate global update** — not per package, and
never to clear a warning.

**Why:** a casual `renv::snapshot()` does the OPPOSITE of what renv is for — it
blesses whatever is currently installed (often newer, drifted versions) as the new
pin, baking in exactly the automatic-update behaviour he wants to avoid.

- The renv "out-of-sync [lockfile != library]" warning is **expected and
  non-actionable**. Never propose or run `renv::snapshot()` to silence it.
- Across the 6 app renvs the drift is **library AHEAD of lockfile** (ggplot2 4.0.3
  installed vs 3.5.1 pinned; shiny 1.13 vs 1.10; ~70-90 pkgs each). The libraries
  already took un-recorded updates and are NOT actually frozen at the pins.
- His deliberate periodic update sequence is: `renv::restore()` per app (pull the
  library back to pins) → bump what he wants → `renv::snapshot()` to re-pin.
  **Don't auto-run this; it's his call.**
- Genuine launch blockers (a package physically missing, or a broken `activate.R`
  — §3) are still worth fixing. "Leave it alone" is about version drift and the
  snapshot temptation, not real breakage.

## 2. What's safe to track in git

Every renv-managed project ships a `renv/.gitignore` excluding the heavy/derived
dirs — confirmed identical across all 6 apps:

```
library/  local/  cellar/  lock/  python/  sandbox/  staging/
```

So `git add renv/` is **safe**: it picks up `renv/activate.R`,
`renv/settings.json`, and `renv/.gitignore`, but NOT the multi-hundred-MB library.
No need to enumerate paths manually.

- **TRACK:** `.Rprofile`, `renv.lock`, `renv/activate.R`, `renv/settings.json`
  (if it exists), `renv/.gitignore`
- **IGNORE** (auto-handled): `renv/library/`, `staging/`, `local/`, `cellar/`,
  `lock/`, `python/`, `sandbox/`

**RApplication-wide policy (set 2026-04-30 via TODO #55):** all 6 apps follow
this. A repo that gitignores `renv/` or `renv.lock` at top level is in violation —
un-ignore and bring into git. See [[project_rapplication_repo_policy]].

## 3. Aligning the renv version WITHOUT corrupting activate.R

Apps can drift in the **renv version itself**: lockfile + `renv/activate.R`
request e.g. 1.2.2 while the project library has 1.1.5. Symptom each launch:
`renv X was loaded from project library, but this project is configured to use Y`.
**Non-fatal** — old renv still resolves the correct triplet libpath
(`renv/library/windows/R-4.4/x86_64-w64-mingw32/`), so packages still load. It is
NOT the cause of a `library(Tdata)` "package not found" crash (that's the
install-location gap — [[project_tdata_install_locations]]).

**The trap (2026-06-02):** fixing it by running, inside the app,
`renv::restore(packages="renv", prompt=FALSE)` **under the old renv** installs the
new renv BUT rewrites `renv/activate.R` leaving the literal placeholder
`attr(version, "md5") <- ..md5..`. Next launch dies hard with
`Erreur dans eval(...): objet '..md5..' introuvable` — the app won't start at all,
strictly worse than the cosmetic warning. Cause: the old renv's activate.R writer
used the new version's template without substituting `..md5..` / `..version..`.

- **If `activate.R` already requests the target version WITH a valid md5**
  (`attr(version,"md5") <- "bb69b6403b1bad0442657e9e8e57cc83"` for 1.2.2): do NOT
  restore. Drop the package dir in, network-free, from a known-good app lib:
  ```bash
  SRC=<good-app>/renv/library/windows/R-4.4/x86_64-w64-mingw32/renv
  DST=<app>/renv/library/windows/R-4.4/x86_64-w64-mingw32/renv
  rm -rf "$DST" && cp -r "$SRC" "$DST"
  ```
- **If `activate.R` is already corrupted with `..md5..`** (or you must install
  fresh): install the target renv, then regenerate activate.R by loading the NEW
  renv directly from the lib, bypassing the broken bootstrap:
  ```r
  lib <- "<app>/renv/library/windows/R-4.4/x86_64-w64-mingw32"
  library(renv, lib.loc = lib); renv::activate(project = "<app>")
  ```
- **Verify** per app: `cd <app> && Rscript -e 'cat(as.character(packageVersion("renv")))'`
  — target version, no mismatch and no `..md5..` warning.

## 4. Revert /build's renv churn

`build_package.R` runs `renv::snapshot()` inside **every** dependent app repo on
deploy. Observed 2026-06-26 building Tdata 5.12.0:

- `renv.lock`: dozens of UNRELATED version bumps (digest, MASS, R6, …) — snapshot
  blessing drift, i.e. exactly the §1 behaviour the user rejects.
- `renv/activate.R`: bumped renv 1.1.5 → 1.2.2 (+~112 lines, adds `md5` attr) —
  the §3 rewrite.
- The just-built package is recorded **one version behind** in the lockfile (lock
  said Tdata 5.11.0) even though the installed files are correct. A snapshot/cache
  quirk: the *install* is fine, only lockfile metadata is wrong.

**How to apply:** after a `/build`, revert the churn in every touched app repo —
`git checkout -- renv.lock renv/activate.R` (some repos only have `renv.lock`
modified). Then commit only real source changes, staging explicitly so the churn
never rides along ([[feedback_git_status_before_commit]]). Don't try to "fix" the
lockfile to show the new version by re-snapshotting — it won't, and it violates §1.

## 5. Tuser has per-R-version library trees

`Tuser/renv/library/` holds several subtrees, only one live:

- `R-4.3/x86_64-w64-mingw32/` — STALE (held Tdata 2.4.5)
- `windows/R-4.4/x86_64-w64-mingw32/` — **ACTIVE** for R 4.4.3
  (`C:\Program Files\R\R-4.4.3`)
- `windows/R-4.5/x86_64-w64-mingw32/` — STALE (held Tdata 5.5.1)

When verifying `/build` actually deployed, check the `windows/R-4.4` tree
(`grep ^Version: .../R-4.4/x86_64-w64-mingw32/<pkg>/DESCRIPTION`). A naive
`find -name DESCRIPTION -path '*<pkg>*' | head -1` can grab a stale tree and
mislead — it once returned Tdata 2.4.5 while the real install was 5.12.0. The
other apps and the system library `C:/Users/aldoh/Documents/RLibrary` are
single-tree. See [[reference_build_package_gotchas]], [[project_tdata_install_locations]].
