---
name: feedback_tdata_py_binding_test_injection
description: Tests touching the live tdata_py active binding fail in build CWD — inject the imported module
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 92762a60-9765-437c-baa5-843b8fe004b9
---

A test that calls a Tdata function which reaches the package's `tdata_py` active binding (zzz.R `makeActiveBinding`, lazy `.init_python_environment`) can PASS under `devtools::load_all()` from the package dir but FAIL during `/build` — the isolated test working directory has no `config.yml`, so the binding's init yields NULL and `tdata_py$...` errors. Symptom seen 2026-06-14: `surface_cache_warnings()` returned `character(0)` (its tryCatch swallowed the binding error) so the drained-warning assertion failed only in the build.

**Why:** the active binding depends on init state/CWD; a bare `reticulate::import("tdata_py")` in the test is a different access path than the package binding, so injecting warnings via one and reading via the other isn't guaranteed consistent in every harness.

**How to apply:** give the function an optional module handle (e.g. `tp = NULL`, defaulting to the package binding) so the test can pass `tp = reticulate::import("tdata_py")` — the same singleton it wrote into. Production callers still use the no-arg form. Note `reticulate::import("tdata_py")` only works after the package binding has configured `sys.path`, so touch `tdata_py` once first to boot it. Related: [[feedback_mock_active_bindings]], [[project_tdata_py_lazy_init_startup]].
