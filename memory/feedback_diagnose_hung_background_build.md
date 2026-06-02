---
name: Diagnose a hung background build without stdout visibility
description: Side-channel signals (tasklist, find -mmin, project logs, git, renv/staging) to figure out what a silent background build is actually doing
type: feedback
originSessionId: 29cdc5d5-050d-4ed2-a819-c20c72b37017
---
When a background `Bash(run_in_background)` task is silent (either by design,
because of pipe buffering, or because the build itself isn't logging), don't
guess — read these side channels in parallel:

1. **Process state** — `tasklist | grep -iE "Rscript|python"` shows live
   PIDs and RSS. Two key signals:
   - RSS *growing* over time = active work
   - RSS *shrinking* and stable = phase finished or hung idle
   - Worker process disappearing = subprocess phase complete
2. **File-system activity** — `find <project_root> -type f -mmin -5` reveals
   what's actually being written. If nothing's been touched in N minutes, the
   build is stalled or doing pure CPU/network work.
3. **Application logs** — for Tdata, `logs/t-<YYYYMMDD>.log` (Tlogger
   namespace log) shows what the test/build is *trying* to do. Last log line
   timestamp + gap to "now" pinpoints the hang.
4. **Build artifacts** — `grep ^Version: <pkg>/DESCRIPTION` and `git log -3`
   tell you whether the build is pre- or post-version-bump and whether a
   commit happened.
5. **Deploy targets** — `find <app>/renv/library -path "*<pkg>/DESCRIPTION"`
   per app shows which renv apps already have the new version. Combined with
   `<app>/renv/staging/` contents, you can pinpoint the active install.
   Beware: stale `staging/<n>/` from prior builds can look "active"; check
   the directory mtime, not just existence.

**Why:** invented this recipe ad-hoc on 2026-04-30 while the Tdata 5.10.11
build was hung for 30+ minutes with `.output` at 0 bytes (because of the
`| tail -120` pipe). Combined the five signals above to determine: build
was in test phase, test was `test-har-volatility.R`, hung on Python
`getHistoricalBars(SPY, 200 D, 15 mins)`, no version bump or commit had
happened, no renv app had been touched. That diagnosis took 5 minutes; the
killing-and-retrying was then unambiguous. Without the recipe I'd have
either waited indefinitely or killed prematurely.

**How to apply:**
- Run all five checks in one parallel Bash call when investigating a silent
  hung background task — don't iterate.
- For Tdata specifically, the application log path is
  `C:/Users/aldoh/Documents/RApplication/logs/t-<YYYYMMDD>.log`.
- If you see `getHistoricalBars` or `getOptValue` as the last log line,
  it's the asyncio-uninterruptible pattern — kill and retry with
  `quick_tests=TRUE` (see `feedback_tdata_quick_tests_for_python_only.md`).
