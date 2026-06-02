---
name: project_tdata_py_lazy_init_startup
description: "Shiny apps that mix library(Tdata) with py_run_file() on scripts importing tdata_py must access Tdata::tdata_py first — the active binding lazy-inits sys.path; without it, py_run_file crashes with ModuleNotFoundError"
metadata: 
  node_type: memory
  type: project
  originSessionId: 5b642673-88ce-4e88-b740-7050efd0fd99
---

`Tdata::tdata_py` is an active binding (`zzz.R::makeActiveBinding` — see [[feedback_mock_active_bindings]]). On first access it runs `.init_python_environment()` which inserts `system.file("python", package="Tdata")` into Python `sys.path`, making `tdata_py` importable from `py_run_file()` scripts.

`library(Tdata)` only *creates* the active binding — it does NOT trigger init. The Python path is empty until something touches the binding.

**Why:** ROrder crashed on startup 2026-05-12 with `ModuleNotFoundError: No module named 'tdata_py'`. `global.R` called `py_run_file("IBSimpleOrderOptions.py")` (which does `from tdata_py.IB_connection import safe_ib_connect`) immediately after `library(Tdata)` — before the active binding had ever been read, so `sys.path` was still empty. Fixed by `116bf3e` on `stable/prod`: added `invisible(Tdata::tdata_py)` between `library(Tbasics)` and the `py_run_file()` block.

**How to apply:** Any Shiny app whose `global.R` mixes `library(Tdata)` with `py_run_file(...)` on a script that imports `tdata_py` must first access `Tdata::tdata_py` (or call any Tdata R function that uses Python — `Tdata::getAllTickers()` works too). Matching pattern: `RPreTrade/global.R:42` already does `tdata_py <- Tdata::tdata_py`.

**Audit candidates if more crashes surface:** any app whose `global.R` calls `library(Tdata)` then `py_run_file()`. Today (2026-05-12) only ROrder used this pattern and it's fixed. RReporting, RJournal, RStudies, RPreTrade either don't `py_run_file` external Python files or access `Tdata::tdata_py` first.

**Anti-pattern note:** `library(Tdata)` is intentionally non-eager — see Tdata `.onLoad`. Don't "fix" this by making init eager; many test workflows depend on being able to load Tdata without spinning up Python.
