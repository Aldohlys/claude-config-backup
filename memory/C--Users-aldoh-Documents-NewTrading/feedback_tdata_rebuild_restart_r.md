---
name: Restart R after Tdata rebuild for Python changes to take effect
description: When modifying tdata_py Python code, R CMD INSTALL alone is not enough. Reticulate caches the imported module per R session — must restart R.
type: feedback
originSessionId: 9cbc474c-d67a-496b-92d1-fb42a1346715
---
After `/build Tdata auto` (or any rebuild that touches `Tdata/inst/python/`), the
new Python code does NOT take effect in already-running R sessions until the R
session is restarted.

**Why:** reticulate imports `tdata_py` once and caches it as a Python module
object inside the R session. Re-installing the package writes new files to
`<lib>/Tdata/python/tdata_py/`, but the R session still holds the previously
imported module object. Behaviour visible to the caller is the OLD code.

**How to apply:**
- After installing a new Tdata, restart R / RStudio / batch sessions before
  judging whether a change took effect.
- For BAT-launched scripts (e.g. `VolMetrics.bat`), each invocation is a fresh
  Rscript session — those pick up the new code automatically. The trap is
  long-running interactive sessions or anything launched once and reused.
- A quick way to verify the right version is loaded:
  `tdata_py$__version__` (or grep for a known new symbol like `force_refresh`
  in the loaded module). If the symbol is missing, you're on the old code.

**Diagnostic shortcut when "the change isn't working":**
1. Check `Tdata/DESCRIPTION` Version field
2. Check at least one renv copy's DESCRIPTION matches
3. Restart R session
4. Only then conclude the patch is broken
