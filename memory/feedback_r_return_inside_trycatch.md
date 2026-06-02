---
name: r-return-inside-trycatch
description: "return() inside tryCatch({...}) body exits the ENCLOSING function, not the tryCatch — silently aborts caller, mimics fatal error"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2a7027ba-133f-4da5-9fcb-8762d589585d
---

`return()` inside the body of `tryCatch({ ... })` returns from the enclosing function, not from the tryCatch expression. So this pattern silently aborts the caller:

```r
f <- function() {
  x <- tryCatch({
    if (cond) return(NA_real_)   # ← exits f() entirely, NOT just tryCatch
    expensive_thing()
  }, error = function(e) NA_real_)
  do_more(x)  # never reached when cond is TRUE
}
```

Notably, the `error =` handler *does* return from the tryCatch correctly (its last expression becomes the result), so the two branches of a tryCatch can disagree — easy to overlook in code review.

**Why:** R's `return()` is a function that walks the call stack to the enclosing `function` definition and signals it. `tryCatch({...})`'s brace block is just an expression, not a function boundary.

**How to apply:** Inside `tryCatch({})` bodies, use the **last expression** as the value (e.g. bare `NA_real_` on its own line in an `else` branch) OR call `stop("msg")` to route through the existing `error =` handler. Never `return()`. Spotted live 2026-05-26 in `Tdata/R/account.R:905` (HTTP-non-200 fallback for ZKB precious-metals scrape) — when triggered by a ZKB HTTP 500, `getGonet()` aborted with NA → no manual-price prompt, no Gonet DB write, no `getAccountGonet()` call. Only visible symptom was `[1] NA` printed by Rscript. See [[getgonet-pm-price-flow-bugs]].
