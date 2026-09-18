---
name: reference_getdte_date_midnight
description: "Tbasics::getDTE() reads a plain Date start as MIDNIGHT in the expiry's time zone while the expiry sits at the exchange close — every date-only DTE is ~0.67 day too long (~10% a week out); Tuser's analysis screens use dte_at_close()"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7b2126a4-19e3-44e8-8acf-3feca1d38bea
  modified: 2026-09-15T10:26:03.717Z
---

**How `getDTE(s_datetime, expdate, exchange = "SMART", sec_type = "OPT", ...)` places the two ends:**
- Expiry = `expdate` at the exchange close, from config `expiration_times` (none in config.yml as of 2026-09-15) or the built-in defaults: SMART/CBOE equity option 16:00 New York, index option 09:30 NY, FOP 16:00 NY, EUREX equity option 17:30 Zurich, EUREX index option 12:00 Zurich. `get_expiration_params()` is **not exported**.
- Start given as a `Date` → `as_datetime(date, tz = exp_tz)` = **00:00**. A `POSIXct` start is used as is.
- So `getDTE(as.Date("2026-09-11"), as.Date("2026-09-18"))` = **7.6667**, and projecting onto expiry day gives 0.67 instead of 0. TODO #18 claimed the screens "default to 16:00" — false.

**Fix shipped 2026-09-15 (Tuser `5cdb8ed`):** `dte_at_close(start, expdate)` in `analysis/logic/an_portfun.R` — for a Date start it subtracts `getDTE(start, start)` (the midnight-to-close offset getDTE applies to that same day), so the close time stays wherever getDTE takes it from; datetimes pass through. Used by all seven date-only calls in `an_portfun.R`, `an_portfUI.R`, `an_symUI.R`. `getDTE()` itself is unchanged for its other callers. A week across the US DST change gives 7.04 (real elapsed time). Test: `Tuser/tests/test_dte_at_close.R`.

**Still open (TODO #18):** those calls pass no `exchange`/`sec_type`, so index and EUREX options get the SMART equity-option expiry hour; no hour/minute input.

**How to apply:** for any new DTE computed from a date (slider, dateInput, YYYYMMDD), use `dte_at_close()` or build a datetime at the close — never pass a bare Date to `getDTE()` expecting market-close semantics. Pass `exchange`/`sec_type` when the position is an index or EUREX option.

Related: [[feedback_verify_before_acting]].
