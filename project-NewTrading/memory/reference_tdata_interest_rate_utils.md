---
name: Tdata interest rate utilities
description: Tdata::getLastRate(currency, DTE) returns the right tenor for an option's life — use it instead of macro 10Y for any BS / IV solve
type: reference
originSessionId: 791d7ea0-3b98-46b6-b585-5209bd9337c8
---
`Tdata::getLastRate(currency, DTE)` is the canonical helper for picking the risk-free rate that matches an option's life.

**Path:** `RApplication/Tdata/R/interest_rate_utils.R` (line ~222). Python equivalents live in `RApplication/Tdata/inst/python/tdata_py/interest_rate_utils.py`.

**Source of rates:** the `Currencies` table in `mydb.db`. Columns store rates **in percent**; `getLastRate` divides by 100 and returns a decimal. The DB row carries `ir1week, ir1month, ir3months, ir6months, ir1year, ir2years` per currency, refreshed by `getInterestRates()` (FRED for USD, ECB for EUR, SNB for CHF, etc.).

**DTE → tenor mapping (mechanical, from the source):**

| DTE | Tenor returned |
|---|---|
| < 18 | ir1week |
| 18–59 | ir1month |
| 60–134 | ir3months |
| 135–269 | ir6months |
| 270–544 | ir1year |
| ≥ 545 | ir2years |

**Companion functions:**
- `getInterestRates(update_db=TRUE)` — refresh DB rows from upstream sources.
- `getAllTenors(currency="All")` — return all tenors at once.
- `getInterestRate(months, currency)` (Python) — futures-implied rate via TWS (slower, used when DB stale).
- `get_interest_rate_for_expiry(ib, expiry_date)` (Python) — Treasury-derived rate for a specific expiry.

**Usage rule:** any BS pricing or IV-solve in /analyze uses `getLastRate(option_currency, DTE)`. Pass the option's currency (USD/EUR/CHF/JPY), not always USD. Never use the 10Y from macro context — that lives there for regime/rates-narrative purposes, not for discount-factor work on options at 7d-180d.

**Staleness check:** `getLastRate` warns if `last_ir_update` in `Currencies` is more than 7 days old; if you see that warning during /analyze, run `Tdata::getInterestRates()` once before continuing.
