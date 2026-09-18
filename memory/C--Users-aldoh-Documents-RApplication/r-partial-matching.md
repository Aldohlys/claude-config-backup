---
name: r-partial-matching
description: "R $ does partial matching — result$error matches result$errors; use result[[\"error\"]] for exact keys (esp. reticulate Python dicts). Empty list() is not NULL."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b7c72d7e-945c-4cbf-8075-4bd8bf2373ec
---

# R `$` Partial Matching - Detailed Notes

## The Problem

R's `$` operator uses partial matching on list/data.frame names:

```r
x <- list(errors = list(), contracts_processed = 1)
x$error   # Returns list() — matches "errors" via partial match!
x$error   # is NOT NULL (empty list is not NULL)
```

## Why This Is Dangerous with Python Dicts

Python functions often return dicts with both:
- `error` (singular) — for fatal errors
- `errors` (plural) — for per-item error lists

After `reticulate::py_to_r()`, R's `$` operator matches `error` → `errors`:
```r
result <- py_function()  # Returns {errors: [], contracts_processed: 1}
!is.null(result$error)   # TRUE! (matches result$errors = list())
```

## The Fix

Always use `[[` for exact matching:
```r
# WRONG
if (!is.null(result$error)) { ... }

# CORRECT
if (!is.null(result[["error"]])) { ... }
```

## Where This Bit Us

- `Tdata::update_tracked_options()` — R wrapper checked `result_list$error` but Python returned `errors` (empty list)
- `scripts/tracking_manager/app.R` — same issue in the "Run Update Now" handler
- Result: false "Update failed:" message with empty error text, even though data collection succeeded

## Rule of Thumb

When interfacing with Python via reticulate, ALWAYS use `[["key"]]` instead of `$key` for checking specific dict keys. Reserve `$` for interactive use only.

## The same trap without any partial match: duplicate names

`$` returns the **first** entry with a matching name, so building a list by concatenation can shadow the value you meant:

```r
base <- list(name = nm, notes = character(0))   # notes seeded empty
out  <- c(base, list(status = "PARTIAL", notes = notes))  # TWO entries named notes
out$notes   # character(0) — the FIRST one, always
```

This shipped in `bot_monthly/main.R` and emptied every reason string in the output while `fetch_status` still said `PARTIAL` — a row that says it is incomplete but will not say why. `[["notes"]]` does not help here: it also takes the first match. **The fix is to not seed the field in the base list.** Check for duplicates with `anyDuplicated(names(x))` when assembling a list from parts.
