---
name: Check the "Welcome Tdata version X.Y.Z !" log line first
description: When a Tdata-based script behaves unexpectedly, the loaded Tdata version printed at startup is the fastest diagnostic — usually before re-reading the script
type: feedback
originSessionId: 967d9aec-f0cc-4ee7-a35d-cd2e96f70722
---
When a script that uses Tdata (`daily_portfolio_update.R`, `RGetIBKR.R`, scheduled tasks, ad-hoc Rscript runs) behaves unexpectedly — hangs, missing API, wrong return shape — read its log for the line:

```
Welcome Tdata version X.Y.Z !
```

(emitted by Tdata's `.onAttach` / namespace logging at library-load time)

If the printed version is older than the source tree's `DESCRIPTION` version, the script's R environment is loading a stale install — not running the code you're reading. Rebuild + reinstall into the relevant library (see `project_tdata_install_locations.md` for the 8 places it can live).

**Why:** 2026-04-21 session — user said `daily_portfolio_update.R` was "stuck and uses old Tdata library." I started reviewing the script's API usage; the `Welcome Tdata version 5.9.14 !` line in the pasted log was the actual diagnosis (current source was 5.10.4). The script code was already correct. Diagnosis would have been seconds instead of dozens of file reads if I'd looked at the version line first.

**How to apply:** Whenever the user reports unexpected Tdata behavior and pastes a log, scan for the version banner before opening source files. If they don't paste a log, ask them to run the script once and share the first 5 lines of output, or run `Rscript -e "cat(as.character(packageVersion('Tdata')))"` from the same shell/CWD the failing script uses.
