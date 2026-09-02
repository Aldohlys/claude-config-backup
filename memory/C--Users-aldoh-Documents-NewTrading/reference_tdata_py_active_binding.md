---
name: reference-tdata-py-active-binding
description: Never call reticulate::import("tdata_py") — use Tdata::tdata_py, an active binding that initialises sys.path
metadata:
  type: reference
---

`tdata_py` is exported by Tdata as an **active binding**, not a plain object. `Tdata/R/zzz.R` `.onLoad` does `makeActiveBinding("tdata_py", ...)`; the first *access* runs `.init_python_environment()`, which is what calls `add_python_path_if_needed(system.file("python", package = "Tdata"))` and puts the Python package on `sys.path`.

Calling `reticulate::import("tdata_py")` directly skips the initialiser, so the module is never importable and fails with `ModuleNotFoundError: No module named 'tdata_py'`. This bit `Tuser/spread/view/spreadUI.R` (fixed 2026-08-26) — every Compute failed. `ligne/app.R` has it right via `box::use(Tdata[tdata_py])`.

Correct forms: `box::use(Tdata[tdata_py])` at module top, or `Tdata::tdata_py` inside the handler when you want Python init to stay lazy so app startup doesn't pay for it.

Related: [[reference_tdata_install_topology]], [[feedback_tdata_rebuild_restart_r]], [[reference_tdata_option_fetch_internals]]
