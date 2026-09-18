---
name: feedback_named_vector_to_json
description: "read.dcf()[1, \"Version\"] (and similar matrix/list extractions) keeps its name — jsonlite and renv::record() then write the scalar as a JSON object; unname() before serialising, and test with the real input rather than a literal"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7b2126a4-19e3-44e8-8acf-3feca1d38bea
  modified: 2026-09-15T08:00:53.827Z
---

`read.dcf("DESCRIPTION")[1, "Version"]` returns a **named** character (`c(Version = "1.6.4")`). Printed it looks like a plain string, but anything that serialises it to JSON keeps the name: `renv::record()` wrote `"Version": {"Version": "1.6.4"}` into six app lockfiles during the Tbasics 1.6.4 / Tdata 5.19.1 deploy (2026-09-15), a shape renv expects to be a string. Fixed in `build_package.R` with `unname()` (RApplication `d952521`) and the lockfiles re-recorded.

**Why it slipped through:** the pre-ship test of the new `renv::record()` code set `installed_ver <- "5.19.9"` by hand, so it never saw the name that the real `read.dcf()` line produces. The re-test ran the actual extraction line from the script and caught it.

**How to apply:**
- Before passing a value extracted from a matrix, `read.dcf()`, `sapply()` or a named list to `jsonlite`, `renv::record()`, a DB parameter or an API payload, wrap it in `unname()` (or `as.character()` for a scalar).
- When testing code that transforms real data, feed it the output of the real upstream call, not a typed literal of what you expect that output to be. A literal tests your assumption, not the code path.
- Symptom: a JSON field that should be a string is an object with a single key equal to the field name.

Related: [[reference_renv_discipline]], [[feedback_zero_length_args_pass_silently]].
