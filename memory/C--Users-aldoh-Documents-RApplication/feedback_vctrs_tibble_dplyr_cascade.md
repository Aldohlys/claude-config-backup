---
name: feedback_vctrs_tibble_dplyr_cascade
description: When tibble or dplyr is upgraded in RLibrary but vctrs isn't, getIBKR / other Tdata-using scripts crash mid-run with "vctrs 0.6.5 already loaded but >= 0.7.1 required". Upgrade vctrs to clear it.
type: feedback
originSessionId: d15d36be-5525-4cb1-836d-35a4a21221dd
---
`tibble` ≥ 3.3.0 and `dplyr` ≥ 1.2.0 require `vctrs` ≥ 0.7.1. If only some of these get upgraded in `C:/Users/aldoh/Documents/RLibrary` (the main user library used by `IBKR.bat`, `daily_portfolio_update.R`, etc.), the dependency mismatch lies dormant until a workflow lazy-loads the affected namespace mid-execution. Then it crashes with:

```
le package 'vctrs' 0.6.5 est déjà chargé, mais >= 0.7.1 est requis
Appels : getIBKR -> loadNamespace -> namespaceImport -> loadNamespace
```

**Why:** Renv-managed apps are insulated (each app has its own renv lib pinned in `renv.lock`), but RLibrary is a shared bag with no lockfile. `install.packages()` upgrades latest versions opportunistically, and there's nothing that re-checks that vctrs floor stays ≥ what tibble/dplyr need.

**How to apply:**
- When a script run from `RApplication/scripts/*.R` (uses RLibrary, not renv) crashes with a "vctrs X already loaded but >= Y required" error, the fix is: `install.packages('vctrs', repos = 'https://cloud.r-project.org', lib = 'C:/Users/aldoh/Documents/RLibrary')`. Same pattern for any of {rlang, lifecycle, cli} — they're foundational deps for tibble/dplyr.
- Diagnostic: from `RApplication/scripts/`, `Rscript -e "ip <- installed.packages(); cat('vctrs:', ip['vctrs','Version'], 'tibble:', ip['tibble','Version'], 'dplyr:', ip['dplyr','Version'])"` confirms what's mismatched.
- Don't propagate the upgrade into the renv apps — they're locked at their own versions and don't have the same problem.
- Tdata `/build` deploys to RLibrary now (per change.log 2026-04-27) but doesn't audit or refresh transitive deps. So this kind of drift can recur whenever a top-level CRAN install happens against RLibrary.
