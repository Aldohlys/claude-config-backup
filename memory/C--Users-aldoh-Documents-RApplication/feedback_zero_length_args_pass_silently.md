---
name: feedback_zero_length_args_pass_silently
description: "R: sprintf() with any zero-length arg returns character(0), cat() then prints NOTHING, and stopifnot() on logical(0) PASSES - a test can go green having verified nothing"
metadata:
  node_type: memory
  type: feedback
---

Hit 2026-09-01 writing a smoke test for the BOT cluster counts. The test printed
only its final "OK" line and exited 0, having verified **nothing**.

The chain:
1. A helper was called with the wrong argument type, so it returned `NULL`.
2. The function under test hit an early return that sets **no attributes**, so
   `attr(x, "trend_count")` was `NULL`.
3. `sprintf("...%d...", NULL)` returns **`character(0)`** - not an error.
4. `cat(character(0))` prints **nothing** - so the per-case output silently
   vanished rather than showing wrong values.
5. `stopifnot(0 == NULL)` evaluates `logical(0)`, and `all(logical(0))` is
   `TRUE`, so **the assertion passes**.

Green test, no output, zero coverage. Same family as
[[feedback_any_na_crashes_scalar_if]] and
[[feedback_max_cumsum_na_rm_minus_inf]] - zero-length and NA propagate through
R's vectorised primitives instead of failing.

**How to apply:** in any hand-rolled check, assert the *shape* before asserting
the value:
```r
a <- function(n) { v <- attr(b, n); stopifnot(!is.null(v), length(v) == 1); v }
stopifnot(!is.null(last), nrow(last) == 1)
```
And treat **missing expected output as a failure signal**, not a formatting
quirk - if a loop was meant to print one line per case and printed none, the
loop did not do what you think. Do not trust "exit 0 + final OK line".

## The same trap in list assembly: a NULL field kills the whole data frame

Hit again 2026-09-18 in `bot_monthly`. A function returned a short list on its failure path:

```r
if (no_data) return(c(base, list(status = "FAILED", notes = "no price history")))
# ... the success path returns 20 more fields
```

The row builder then read `r$atr_band` on that short row, got **NULL**, and
`as.data.frame()` rejected the **entire** list of 335 rows:

```
les arguments impliquent des nombres de lignes differentes : 1, 0
```

Three real tickers triggered it (`STOCK` is not a symbol, `MCL=F` fails at Yahoo, `NG` returns non-leading NAs), and it fired **after** a 23-minute fetch of all 347. Unlike the sprintf case this one is loud — but it is the same root: a zero-length value flows instead of failing, and it surfaces far from where it was produced.

**How to apply:** give every early return the full field set, or coerce on read with NULL-safe accessors (`.s()` for character, `.i()` for integer, alongside the usual numeric one). And cache an expensive fetch before assembling it — assembly is where this class of bug lands, and it should not cost the fetch.
