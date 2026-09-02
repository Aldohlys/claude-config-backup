---
name: feedback_install_packages_python_stale
description: "Tdata's Python is only ever run from the INSTALLED tree, never the working copy — editing inst/python changes nothing until deployed, which install tree is used depends on CWD, and a plain install.packages() can silently fail to overwrite the .py files"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T04:13:36.338Z
---

Tdata ships Python under `inst/python/` for reticulate. Three separate ways that
tree gets out of step with what you think you are running.

## 1. Editing the working tree does nothing until you deploy

`reticulate` imports from `system.file("python", package = "Tdata")` — the
**installed** package, not `RApplication/Tdata/inst/python/`. Editing the source
and immediately testing runs the OLD code.

2026-08-27: after patching `spread.py` to accept `market_data_type`, the test
failed with `argument inutilisé (market_data_type = 1)` — the source had it, the
install did not. For a quick edit-test loop, copy the module into the resolved
install path before Python starts (a fresh Rscript process, so the copy must
happen before the first `tdata_py` access), then `/build` properly once it works.

## 2. Which install tree you get depends on CWD

There are 8 install locations ([[project_tdata_install_locations]]) and
`system.file()` resolves against `.libPaths()`, which the CWD's `.Rprofile`/renv
sets. The same script gave:

- from `RApplication/` → `.../renv/cache/.../Tdata-9fd37c52/windows/R-4.4/.../Tdata/python/tdata_py`
- from the scratchpad → `C:/Users/aldoh/Documents/RLibrary/Tdata/python/tdata_py`

Run identical tests from the **same** directory, or they silently exercise
different code. A scratchpad CWD also loses `config.yml`, producing
`WARNING: Config file not found` and `Database not found at data/mydb.db`, which
makes the run fail for an unrelated-looking reason.

## 3. Install trees can be stale in *part*

Copying two patched files into an install tree produced
`ImportError: cannot import name 'ParquetQuotesStorage' from
'tdata_py.parquet_storage'` — that tree's `parquet_storage.py` predated the
class. A partial copy yields a **mixed** tree that is broken in a new way. Copy
the whole module (`list.files(src, pattern = "[.]py$")`) rather than the files
you touched.

## 4. install.packages() can report DONE and not replace the .py files

The install prints `* DONE (Tdata)` and `packageVersion('Tdata')` returns the new
version, **but** `<lib>/Tdata/python/tdata_py/*.py` can be unchanged if another
process holds a lock (a hung Rscript, an open Shiny session, an IDE holding the
module via reticulate). R-side files (DESCRIPTION, R/, Meta/) do update; only the
unmanaged Python tree silently fails. 2026-04-21: installed 5.10.5 twice, both
DONE, both times `account.py` kept the old mtime and the new identifier was
absent.

**How to apply** when deploying a patch that touches `inst/python/`:

1. Prefer `remove.packages('Tdata')` then `install.packages(tarball)` over a
   plain reinstall.
2. Verify the installed file directly — don't trust `packageVersion()`:
   ```bash
   grep "<known new identifier>" "<lib>/Tdata/python/tdata_py/<file>.py"
   ```
   After a `/build`, check the tree that is actually active for the app
   ([[reference_renv_discipline]] §5).
3. If a hung Rscript holds the lock, kill it
   (`tasklist //FI "IMAGENAME eq Rscript.exe"`, then `taskkill /F /PID <pid>`).

Pure-R packages aren't affected.
