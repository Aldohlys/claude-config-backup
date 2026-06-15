---
name: feedback_mock_active_bindings
description: testthat's with_mocked_bindings fires the active binding's getter when trying to assign the mock; required for Tdata's tdata_py and any other active-binding-backed package globals
type: feedback
originSessionId: 110f4600-4558-489d-88b2-cdf4eb853b4a
---
`tdata_py` in Tdata is an active binding (`zzz.R::makeActiveBinding`) — accessing it lazily triggers Python init and returns the imported module from `.tdata_state$value`. Standard `with_mocked_bindings(tdata_py = fake_module, { ... })` fails with an error like:

```
Error in `(function () { if (!.tdata_state$initialized) ... })(base::quote(...))`:
  unused argument (base::quote(...))
```

— because rlang's `env_bind` invokes the active getter while assigning the mock.

**Why:** active bindings are functions internally; testthat tries to replace the binding but the assignment path goes through the getter as if it were a regular function call. Same trap applies to any package using `makeActiveBinding` for lazy init.

**How to apply:** instead, write a helper that mutates the underlying state directly:

```r
local_mock_tdata_py <- function(fake_module, envir = parent.frame()) {
  state <- get(".tdata_state", envir = asNamespace("Tdata"))
  old_value <- state$value
  old_initialized <- state$initialized
  state$value <- fake_module
  state$initialized <- TRUE
  withr::defer({
    state$value <- old_value
    state$initialized <- old_initialized
  }, envir = envir)
}
```

Used in `test-earnings.R`, `test-position_sizer.R`, `test-account.R`, `test-ibkr.R`, `test-historical-options.R` (all in Tdata as of 5.10.13).
