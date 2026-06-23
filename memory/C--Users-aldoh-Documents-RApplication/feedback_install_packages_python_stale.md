---
name: feedback_install_packages_python_stale
description: On Windows, install.packages() of a Tdata source tarball reports DONE and bumps packageVersion() but may leave inst/python/*.py files unchanged if a stale R/Python process holds them. Always verify by grep, or remove.packages() first.
type: feedback
originSessionId: 967d9aec-f0cc-4ee7-a35d-cd2e96f70722
---
On Windows, when reinstalling a Tdata source tarball with `install.packages('Tdata_X.Y.Z.tar.gz', repos=NULL, type='source')`:

- The install can report `* DONE (Tdata)` and `packageVersion('Tdata')` returns the new version
- BUT files under `<lib>/Tdata/python/tdata_py/*.py` can be **unchanged** if another process holds a file lock (e.g. a hung Rscript, an open Shiny session, an IDE keeping the Python module imported via reticulate)
- The R-side code (DESCRIPTION, R/, Meta/) does get updated; only the unmanaged Python tree silently fails to overwrite

**Symptom:** `packageVersion()` lies. Patches you "installed" don't take effect. Grep finds the OLD content in the installed Python file.

**Why:** 2026-04-21 session — installed Tdata 5.10.5 twice, both reported DONE, `packageVersion()` returned 5.10.5, but `account.py` mtime stayed at the previous install's timestamp and `grep "reqAccountUpdatesAsync"` found nothing in the installed file. Only `remove.packages('Tdata')` followed by a fresh `install.packages()` actually replaced the Python files.

**How to apply:** When deploying a Tdata patch that touches `inst/python/`:

1. Prefer `remove.packages('Tdata')` then `install.packages(tarball)` over a plain reinstall
2. Verify the patched file directly:
   ```bash
   grep "<known new identifier>" "<lib>/Tdata/python/tdata_py/<file>.py"
   ```
   Don't trust `packageVersion()` alone for Python-side patches
3. If a hung Rscript holds the lock, kill it first (`tasklist //FI "IMAGENAME eq Rscript.exe"`, then `taskkill /F /PID <pid>`)

This is specifically for Tdata because it ships Python under `inst/python/` for reticulate; pure-R packages aren't affected.
