---
name: Tdata is installed in multiple renv libraries; /build deploys to all
description: Tdata source is at RApplication/Tdata; installs go to per-project renv libraries (RJournal, ROrder, RPreTrade, RReporting, RStudies, Tuser) plus RLibrary. /build Tdata auto = patch bump only.
type: reference
originSessionId: 9cbc474c-d67a-496b-92d1-fb42a1346715
---
**Source of truth:** `C:\Users\aldoh\Documents\RApplication\Tdata\` (git repo Aldohlys/Tdata, branch stable/prod).

**Install targets** (verified 2026-04-27, all on 5.10.9 after `/build Tdata auto`):
- `RApplication/RJournal/renv/library/windows/R-4.4/x86_64-w64-mingw32/Tdata/`
- `RApplication/ROrder/renv/library/windows/R-4.4/x86_64-w64-mingw32/Tdata/`
- `RApplication/RPreTrade/renv/library/windows/R-4.4/x86_64-w64-mingw32/Tdata/`
- `RApplication/RReporting/renv/library/windows/R-4.4/x86_64-w64-mingw32/Tdata/`
- `RApplication/RStudies/renv/library/windows/R-4.4/x86_64-w64-mingw32/Tdata/`
- `RApplication/Tuser/renv/library/windows/R-4.4/x86_64-w64-mingw32/Tdata/`
- `RLibrary/Tdata/` (the user-default R library, used by Rscript when no renv is active — e.g. `VolMetrics.bat`)

Stale copies that DO NOT get refreshed (manual cleanup if it ever matters):
- `RApplication/Tuser/renv/library/R-4.3/x86_64-w64-mingw32/Tdata/` (R-4.3, version 2.4.5)
- `RApplication/Tuser/renv/library/windows/R-4.5/x86_64-w64-mingw32/Tdata/` (R-4.5, version 5.5.1)

**To verify a deploy went everywhere:**
```bash
find /c/Users/aldoh/Documents -path "*/Tdata/DESCRIPTION" -not -path "*/.Rproj.user/*" 2>/dev/null \
  | xargs -I {} sh -c 'echo "=== {} ==="; grep "^Version:" "{}"'
```

**Build conventions:**
- `/build Tdata auto` → bumps PATCH only (e.g. 5.10.8 → 5.10.9). For MINOR or
  MAJOR bumps, use explicit subcommand.
- Document changes in `Tdata/CHANGELOG.md` BEFORE running build, and tag the
  entry with the version the build will produce — not the current DESCRIPTION
  version.
