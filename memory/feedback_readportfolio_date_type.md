---
name: feedback_readportfolio_date_type
description: "Tdata::readPortfolio returns date as Date class, NOT YYYYMMDD integer like Trades.TradeDate — easy to write a silent zero-match filter"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: dad29eee-3519-49b0-8d78-c2257ddb1dfb
---

`Tdata::readPortfolio(account)` (and `readPortfolioDate`, `readLastPortfolio`) return the `date` column as **R `Date` class**.

In contrast, the **Trades** table — both via `getAllTrades()` and direct DB reads — returns `TradeDate` (and other date columns) as **INTEGER YYYYMMDD** (e.g. `20260518L`).

**Why it matters:** mixing the two in a single filter silently matches zero rows without error.

```r
# WRONG — port$date is Date, initial_ymd is 20260101
filter(portfolio, date >= initial_ymd, date <= end_ymd)
# Returns 0 rows because Date(20260101) is treated as days since 1970 (far future)
```

**Fix:** convert window bounds to Date for portfolio filters, keep YYYYMMDD ints for Trades filters:

```r
initial_d   <- as.Date(initial_date)
initial_ymd <- as.numeric(format(initial_date, "%Y%m%d"))

port    <- filter(portfolio, date >= initial_d)        # Date vs Date
trades  <- filter(all_trades, TradeDate >= initial_ymd) # int vs int
```

**How to apply:** any time you join/intersect Trades data with position-table data, check both column classes. Same trap exists with `Trades.Exp.Date` which is stored as string "DD.MM.YYYY".

Hit while building [[project_strategy_evolution_plot]] (Tuser/account/view/strategyAccountUI.R) on 2026-05-18 — initial filter returned 0 portfolio rows, the entire series ended up as close-date-only entries before I caught it.
