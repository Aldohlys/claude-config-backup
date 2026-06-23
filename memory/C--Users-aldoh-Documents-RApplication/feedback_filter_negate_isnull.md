---
name: feedback_filter_negate_isnull
description: Combining `c(NULL, x)` (drops NULLs) with a parallel `sapply(list(NULL, x), is.null)` mask creates a length mismatch that recycles into NA — use Filter() instead
originSessionId: 67df7b93-72b2-4e92-a5f8-7541ce5b225c
type: feedback
---
```r
# WRONG — produces NA via length-mismatch recycling
v <- c(reason_a, reason_b)                       # c() drops NULL elements
mask <- !sapply(list(reason_a, reason_b), is.null)  # mask keeps both positions
joined <- paste(v[mask], collapse = "; ")        # mismatch → NA in output

# RIGHT — Filter operates on a list and preserves alignment
parts <- Filter(function(x) !is.null(x) && !is.na(x) && nzchar(x),
                list(reason_a, reason_b))
joined <- if (length(parts) == 0) NULL else paste(unlist(parts), collapse = "; ")
```

**Why:** `c(NULL, "x")` is `c("x")` (length 1) but `sapply(list(NULL, "x"), is.null)` is `c(TRUE, FALSE)` (length 2). Indexing the length-1 vector with the length-2 mask recycles, and the unmatched position resolves to NA. `paste(NA)` then renders as the literal string "NA".

**How to apply:** when joining optional reason strings (or any list of optional values where some entries may be NULL/NA/""), reach for `Filter(predicate, list(...))` over `c(...)[mask]`. Hit during /analyze rework 2026-04-30 — produced `FETCH FAILED: NA` cells in the funnel grid Term IV30/IV90 row.
