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

## `%||%` written scalar-only silently eats every list

A null-coalesce defined as `function(a, b) if (is.null(a) || length(a) != 1 || is.na(a)) b else a`
looks harmless until it meets a list. `bot_monthly/main.R` did
`v <- q$value %||% NULL` where `q$value` is an 8-element list: `length(a) != 1`
fires, `v` is NULL **every time**, and the downstream `is.finite()` guard fails
silently. `Tickers.AtmBidAskPct` and `BOT_VehicleHint` were NULL on all 352 rows
for weeks while the row recorded `bid-ask: LIVE`, because the *fetch* had
succeeded and only the unwrapping threw it away.

**Tell:** a column that is NULL for 100% of rows while the log says the source
returned data. Check the unwrapping before blaming the source. The `length() != 1`
and `is.na()` clauses make these operators safe for scalars and wrong for
everything else — grep for `%||%` before passing it anything structured.
