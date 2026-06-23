---
name: feedback_any_na_crashes_scalar_if
description: any()/all() over an NA-bearing vector returns NA → crashes scalar if(); regexpr() returns NA for NA input
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b41408a7-903e-409b-9db1-9e8ab1f2cbad
---

`any(x)`/`all(x)` return `NA` (not FALSE/TRUE) when the vector has an `NA` and no element forces the result — e.g. `any(c(FALSE, NA))` is `NA`, `all(c(TRUE, NA))` is `NA`. Feeding that to a scalar `if()` throws **"valeur manquante là où TRUE / FALSE est requis"** (missing value where TRUE/FALSE needed). Row subsetting with the same NA-bearing logical (`dt[!ind, ]`) silently injects phantom NA rows.

A common upstream NA source: `regexpr(pattern, text)` returns `NA` for an `NA` element, so any `regexpr(...) != -1` predicate (e.g. RReporting `include_month_year()`) returns `NA` for `NA` input.

**Why:** real data carries NA (placeholder/simulation trades had `Instrument = NA`); the predicate looked total but wasn't.

**How to apply:** Make boolean predicates NA-total at the source — `result[is.na(result)] <- FALSE` — rather than patching each call site, OR guard the reduction with `any(x, na.rm = TRUE)`. Fixing the predicate also protects logical-index subsetting downstream. See [[feedback_filter_negate_isnull]] for adjacent vector-NA traps; bit RReporting Summary tab 2026-06-17 (commit e81ae00).
