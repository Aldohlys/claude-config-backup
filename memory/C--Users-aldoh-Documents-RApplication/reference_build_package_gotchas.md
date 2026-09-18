---
name: reference_build_package_gotchas
description: "build_package.R / /build traps — the 'All tests passed' banner that prints on a RED suite and still deploys, CHANGELOG-before-build (and the generic-message fallback that persists anyway), --quick-tests silently skipping mis-named test files, quick-tests for Python-only Tdata changes, force=TRUE after a manual bump, and the blanket `git add .`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T02:22:45.265Z
---

Merged 2026-08-27 from five separate memories. All concern
`scripts/build_package.R` and the `/build` slash command.

## 1. Never /build with a dirty tree of someone else's work

`build_package.R` stages the **entire** package repo with `system("git add .")`
(~line 595) before committing, bumping the version, deploying to all renv apps +
RLibrary, and **pushing**.

**Why it matters:** unrelated concurrent uncommitted changes (e.g. another
session mid-task) get swept into the build commit and deployed + pushed — an
outward-facing publish of someone else's unfinished code.

**How to apply:** run `git status` in the package dir before `/build`. If the tree
has changes that aren't yours, do NOT build. Commit only your own files by
explicit pathspec, bump version + CHANGELOG yourself, and defer the deploy until
the tree is clean — then deploy the already-committed version with §4's
`force = TRUE`. Re-check `git status` immediately before any commit to catch
pre-staged ride-alongs like `data/mydb.sql` ([[feedback_git_status_before_commit]]).

## 2. Update CHANGELOG.md BEFORE the build — but expect the generic message anyway

Add a `## [x.y.z] - YYYY-MM-DD` section to `<Package>/CHANGELOG.md` for the
*target* version before running `/build <package> auto`. `build_package.R` reads
it to generate the commit title/body; with no entry it falls back to summarizing
changed file names ("Build Tdata v5.10.9: Update contract.py, parquet_storage.py")
instead of explaining *why*. Observed on Tdata 5.10.9 and 5.10.10.

For `auto patch`, increment in your head and add the matching section. If the user
runs `/build` without one, flag it and offer to draft it — don't let the generic
message ship silently. CLAUDE.md's "Changelog Workflow Summary" already says this;
the point here is that **I** should enforce it rather than assume it was done.

**UPDATE 2026-06-13 (Tdata 5.10.24):** a correct entry was present before the
build and the commit STILL came out generic ("Build Tdata v5.10.24: Package
updates"). So a CHANGELOG entry is **necessary but not sufficient** — the
message-generation step appears to need an interactive LLM call unavailable in the
headless Rscript `/build`, and falls back regardless. Inspect `git log -1` after
every `/build` and fix with `git commit --amend -F tempfile`. (Each package is its
own repo — amend in the package dir; `--amend` ignores unstaged unrelated files.)

**UPDATE 2026-06-15 (Tdata 5.10.29):** the amend works, but
`git push --force-with-lease` is **not** silent/proactive — force-pushing an
already-pushed commit rewrites remote history and requires the user's explicit
go-ahead. Surface the generic message, propose the amend, get an explicit
"amend + push". The local `--amend` alone is harmless but leaves local/remote
diverged until the approved push; verify with `git log -1 origin/<branch>` if a
push was denied mid-sequence.

## 3. --quick-tests skips a mis-named test file silently and still builds green

`build_package.R:120` builds the test path as
`file.path("tests","testthat", paste0("test-", base_name, ".R"))` where
`base_name` is the changed R file's stem. The match is **exact**, so
`R/vol_metrics.R` requires `test-vol_metrics.R` — underscore, not hyphen.

On a miss the build prints `No matching test files found, skipping tests` and
**continues to a green build**: version bumped, tarball made, commit pushed,
deployed to all six renv trees. Nothing fails. The path is intentional (it is what
makes §4 cheap) but cannot distinguish "no R tests exist" from "the author
mis-named the file". Observed 2026-08-26: a new `test-vol-metrics.R` never ran and
Tdata 5.14.2 shipped without it, despite the tests existing and passing.

**How to apply:** name a test file from the R file's stem verbatim
(`R/foo_bar.R` → `test-foo_bar.R`) — copy the stem, don't retype it. After any
`--quick-tests` build, read the `Running quick tests` block and confirm it names
the files you expected. **A green exit code is not evidence that tests ran**, so
never report "tests passed" without checking that line
([[feedback_check_welcome_log]] is the same read-the-log discipline).

## 4. Python-only Tdata changes → always --quick-tests

When `/build Tdata` only touches `Tdata/inst/python/...` (the `tdata_py` module),
always pass `--quick-tests` (or `quick_tests = TRUE`). Python-only changes have no
matching R test files, so quick-tests runs zero tests — fast and deterministic.

**Why:** Tdata's full `devtools::test()` includes `test-har-volatility.R`, which
calls `Tdata::fitHAR("SPY", source="ibkr")` → Python
`getHistoricalBars(SPY, 200 D, 15 mins)`. Against a slow or busy IBKR session that
can hang 15+ minutes, and R **cannot** bound it with `tryCatch`/`withTimeout`
([[feedback_reticulate_asyncio_uninterruptible]]). Build #1 of v5.10.11 on
2026-04-30 hung 30+ min on exactly that call; the retry with `quick_tests=TRUE`
finished in ~12 min (deploy time only).

- Bash: `build_package('Tdata', auto_version=TRUE, version_type='patch', quick_tests=TRUE)`
- Slash: `/build Tdata auto --quick-tests`
- The change should include at least one corresponding R file to *force* coverage;
  otherwise rely on the manual Python test workflow before building.
- Specific to **Tdata** — Tbasics/Tlogger have no live-IBKR tests.

## 5. force=TRUE when the version bump was committed manually

`build_package('Tdata', auto_version = FALSE)` pre-flights `git_status` and aborts
with `✅ No changes since last commit - build not needed` on a clean tree. Right
default, but it traps you when you've already done the bump + CHANGELOG + commit +
push by hand and only want the build/deploy half. The script can't distinguish
"nothing happened" from "already committed manually; just deploy".

```r
build_package('Tdata', auto_version = FALSE, force = TRUE)
```

This bypasses the clean-repo guard and proceeds through tests + docs + tarball +
6-app renv deploy + system library install. `force = TRUE` also propagates to
`renv::snapshot(..., force = TRUE)` (`build_package.R:263, 366`), bypassing the
"package from unknown source" preflight — relevant since Tdata is local-installed.
build_package.R still creates its own follow-up commit
(`Build Tdata vX.Y.Z: Package updates`), so you end up with two commits for the
same version. Acceptable, not a bug.

**Note:** the deploy snapshots lockfiles in every app — revert that churn
afterwards, see [[reference_renv_discipline]] §4. Tdata also lives in 8 install
locations while /build covers 6 ([[project_tdata_install_locations]]).

## 6. "All tests passed successfully" is printed even when tests FAILED

Observed Tdata 5.14.4 (2026-08-31): testthat reported
`[ FAIL 2 | WARN 16 | SKIP 9 | PASS 868 ]` and `build_package.R` printed
**"✅ All tests passed successfully"** on the very next line, then bumped the
version, committed, pushed, and deployed to all 6 renv apps + RLibrary.

**Why it matters:** the success banner is not a gate. A red suite deploys and
pushes exactly like a green one, so trusting the banner means shipping on
failing tests without ever knowing.

**How to apply:** never read the banner. Grep the build log for the testthat
tally line and check `FAIL` yourself:
`grep -E "\[ FAIL" build.log`. If FAIL > 0, inspect each failure and decide
whether it is pre-existing before reporting the build as clean. Report the
counts to the user verbatim rather than echoing the banner.

Cross-check whether a failure can possibly be yours before blaming/absolving it:
`git show HEAD -- <file> | grep -E "^[+-]" | grep -vE "^[+-]{3}|^[+-]\s*#|^[+-]\s*$"`
returning nothing proves the change was comments-only. The two Tdata 5.14.4
failures were `test-ticker.R:24/25` (`addTicker("XOM")` returns NULL because
"Ticker XOM already exists in DB" — leftover state from an earlier aborted run,
self-healing since the suite's own `removeTicker` cleans up at the end). Classic
test-isolation flake, unrelated to the build.

## UPDATE 2026-09-15

- §5's remark about `renv::snapshot(..., force = TRUE)` and reverting lockfile churn is obsolete: the deploy records only the built package ([[reference_renv_discipline]], UPDATE 2026-09-15).
- A cascade build (Tbasics → Tdata) runs only QUICK tests on the downstream package (51 tests for Tdata 5.19.1). If the downstream package has its own changes, run its full suite yourself first.
- A full Tdata `devtools::test()` from a plain Rscript skipped 27 tests as "Python not available", because `tdata_py` initialises lazily and nothing had touched it yet. Calling `get_tdata_py()` right after `devtools::load_all()` gave 922 pass / 8 skip. Compare the SKIP count with the last build, not just FAIL ([[project_tdata_py_lazy_init_startup]]).

## UPDATE 2026-09-15 (b) — failure handling and testing the script itself

- **A failed `devtools::build()` no longer burns a version** (RApplication `1a5a4fd`, TODO #40 item 2): `build_package()` keeps the exact bytes of DESCRIPTION taken before the bump and writes them back on build failure. Tests already ran before the bump, so a red suite never bumped.
- **`base_dir` argument** (default the RApplication root) lets the script be exercised on a throwaway git package in a temp folder, with `testthat::with_mocked_bindings(build = ..., .package = "devtools")` making the build fail or succeed — no real repo, remote or deploy touched. Use `deploy = FALSE, skip_tests = TRUE, cascade = FALSE`.
- **Still unhandled (TODO #84):** a failing `devtools::document()` does not stop the build — its `return(FALSE)` sits inside the tryCatch error handler and only leaves the handler; the version-update block has the same pattern.
