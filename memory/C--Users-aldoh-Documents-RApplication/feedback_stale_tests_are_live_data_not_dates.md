---
name: ""
metadata: 
  node_type: memory
  originSessionId: fa3b67a6-0b78-49eb-8163-8feaf658fb58
---

When auditing this codebase's tests for "date staleness," do NOT assume a 2024/2025 date means the test breaks. Most flagged dates are **self-consistent fixtures** — the date is both the input and the expected output (e.g. `test_structure_analyzer.R` echoes `expdate` into `expected_filename`; `test-compute-return.R` asserts `time_ratio == DIT/(Exp.Date-Open.Date)`; `test-earnings.R` mocks the return value to equal the expectation; `getDTE(...) < 0` only gets more negative). These pass regardless of today's date — bumping the year is pure churn with zero behavioral effect.

**Why:** A 2026-06-14 audit by 4 sub-agents flagged ~30 files as "date-broken"; on inspection almost none actually fail with time. The genuine failure mode is **live-data tests hanging/failing** (live Yahoo, live IBKR/TWS with expired contracts) plus **REPL scratch scripts rotting in test dirs** (one literally sent a live IBKR order). That is the "stuck tests / worsened coverage" symptom — not date math.

**How to apply:** Triage in this order — (1) delete scratch non-tests (no `test_that`, live broker calls); (2) add skip guards to live-data tests: `skip_if_offline()` for Yahoo, and a `skip_if_not(py_available())` + `skip_if_not(isIBAvailable())` helper for IBKR-reaching blocks (place it AFTER any pure-validation `expect_null` assertions so those still run offline); (3) skip cosmetic year-bumps. Verify a date is genuinely load-bearing (recomputed against `Sys.Date()` and flips an assertion, or hits a now-expired live contract) before "fixing" it. See [[reference_isIBAvailable]], [[feedback_test_tws_first]], [[reference_app_subdirs_are_separate_repos]]. Tdata's working branch is `stable/prod`, not master.
