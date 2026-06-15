---
name: feedback_apply_df_row_coercion
description: R gotcha — apply over data.frame rows stringifies logical/numeric values; isTRUE() on a coerced "TRUE" string returns FALSE
type: feedback
originSessionId: 57dce0f1-1905-4f9f-a79f-6683750fbd0a
---
`apply(df, 1, fn)` on a data.frame coerces each row to a character vector before calling `fn`, because `apply()` requires a uniform-type matrix. Logical columns become `"TRUE"` / `"FALSE"`, numerics become strings. Downstream `isTRUE(rr$pass)` then always returns FALSE because `isTRUE("TRUE")` is FALSE.

**Why:** silently broke the /analyze Phase B per-criterion breakdown (2026-05-05) — every row was rendering "FAIL" even when the underlying logical was TRUE. Hard to spot because the values displayed correctly but only the badge column was wrong.

**How to apply:** when iterating over data.frame rows in R, never use `apply(df, 1, fn)` if the row contains anything other than character. Use one of:

```r
# Preferred — preserves types, single-row data.frame per call:
vapply(seq_len(nrow(df)), function(i) fn(df[i, , drop = FALSE]), character(1))

# Or for free-form return type:
lapply(seq_len(nrow(df)), function(i) fn(df[i, , drop = FALSE]))

# If fn takes a list, convert explicitly:
lapply(seq_len(nrow(df)), function(i) fn(as.list(df[i, , drop = FALSE])))
```

**Symptom to watch for:** binary state columns (PASS/FAIL, TRUE/FALSE) all rendering as one value despite the source data being mixed.
