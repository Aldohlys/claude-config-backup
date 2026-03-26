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
