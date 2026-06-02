---
name: Tbasics::display_error_message calls stop() — return(NA) after it is dead code
description: name suggests "display a message" but implementation stops execution; affects defensive code patterns in twr, getOptIBKRPrice, calculate_target_vol, etc.
type: feedback
originSessionId: 110f4600-4558-489d-88b2-cdf4eb853b4a
---
`Tbasics::display_error_message("...")` doesn't just display — it `stop()`s. Several functions in Tdata have an unreachable `return(NA_real_)` or `return(NA)` immediately after a call to it, e.g.:

- `account.R::twr` — duplicate-dates branch: `display_error_message(...); return(NA_real_)` — the return never fires.
- `ibkr.R::getOptIBKRPrice` — `tradingClass == "Stock"`: same pattern.
- `volatility.R::calculate_target_vol` — older version had this in invalid input arms.

**Why:** the function name conflicts with its behaviour. Callers expecting graceful NA propagation get a hard crash instead, and the dead `return(NA)` line looks like it's the API contract when it isn't.

**How to apply:**
- When writing tests for Tbasics-using functions, default to `expect_error(...)` not `expect_equal(..., NA_real_)` when the bad-input branch is exercised.
- When writing new Tdata code that wants graceful failure, use `display_message()` (non-stopping) + `return(NA_real_)`, OR `logger::log_error(...)` + `return(NA_real_)`. Reserve `display_error_message` for genuinely fatal conditions.
- When fixing existing functions: either remove the dead `return(NA)` line, or downgrade the call to `display_message` if NA propagation was the original intent.
