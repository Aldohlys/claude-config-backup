---
name: feedback_max_cumsum_na_rm_minus_inf
description: "max(cumsum(x), na.rm=TRUE) silently yields -Inf, and pmin(NA, cap, na.rm=TRUE) silently yields the cap — na.rm on a running-total aggregate fabricates values instead of dropping them"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 41459763-2fbf-4113-b37e-7182e31a285f
  modified: 2026-08-27T03:51:24.780Z
---

`max(cumsum(x), na.rm = TRUE)` is a trap wherever `x` can contain `NA`. Two
separate base-R behaviours compound:

- `cumsum()` **propagates** `NA` forward: `cumsum(c(NA, 0, 0))` is all `NA`, and
  `cumsum(c(1, NA, 2))` is `1, NA, NA`. A single missing element poisons every
  later running total, not just its own.
- `max(numeric(0))` returns **`-Inf`** with a warning, not `NA`. So once `na.rm =
  TRUE` has dropped the poisoned tail, an all-`NA` run yields `-Inf` — a
  plausible-looking negative number that flows downstream.

In French locales the only warning is `aucun argument pour max ; -Inf est
renvoyé`, easy to skim past inside a `dplyr::summarize()` warning summary.

The sibling trap: **`pmin(x, cap, na.rm = TRUE)` returns `cap` when `x` is `NA`.**
`na.rm` on a two-argument `pmin` doesn't mean "skip"; it means "use the other
one". A ceiling written to protect against a missing cap will silently
substitute the cap for an unknown value. Drop `na.rm` if the cap side can never
be `NA`.

**Why:** in both cases `na.rm = TRUE` reads as defensive, but on a running-total
aggregate it fabricates a value rather than admitting the total is unknown. The
honest result for "one delta is missing" is `NA` (the peak is unknowable), never
`-Inf` and never the budget.

**How to apply:** for a peak-of-running-total, guard explicitly —
`if (length(x) == 0) return(0); if (anyNA(x)) return(NA_real_); max(cumsum(x))`
— and let the `NA` reach the display. This is what
`RReporting/app/R/compute_functions.R::compute_max_outlay_vec()` does; it was
added 2026-08-27 after exactly this chain crashed the Summary tab (see
[[project_trades_signed_delta_risk]]). Related NA-scalar traps:
[[feedback_any_na_crashes_scalar_if]].
