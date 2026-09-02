---
name: reference_account_cashflow_conventions
description: "Account.CashFlow conventions - native currency storage, FX via AccountWithConversionRate view, NLV=0 snapshot hazard, Cash Flow button entry point"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 13d07ea0-0f5c-4fb7-8397-97b2e1c8249a
  modified: 2026-08-31T14:37:09.171Z
---

### Storage convention
`Account.CashFlow` is stored in **native currency** (matching the row's `Currency` column). One row per currency per leg. Sign = from this account's perspective (positive = IN, negative = OUT). `Account.Currency` has a foreign key onto `Currencies.Name`, so only active currencies are valid.

### FX conversion happens on read
`Tdata::readAccount()` queries the `AccountWithConversionRate` SQL view, which left-joins `ConvertToCHF` / `ConvertToUSD` and multiplies `CashFlow * chf_conversion_rate` (or USD, depending on `BaseCurrency`). So:
- **Writers**: store native amount. Do not pre-convert.
- **Readers**: receive base-currency values. Do not re-convert.
- Daily FX rates come from `ConvertToCHF` table (date, currency, chf_value). The view falls back to the most recent prior date if the exact date isn't present.

### Entry point: the Cash Flow button (2026-08-31)
Deposits, withdrawals and inter-account transfers are entered from the **Cash Flow** button in `Tuser/account/view/accountUI.R`'s side panel — so it appears in RReporting, `Tuser/routine` and the standalone `Tuser/account` app at once. Logic lives in `Tuser/account/logic/accountf.R`: `record_cash_flow()`, `cashflow_rows()` (shared with the modal preview), `cashflow_accounts()` (excludes the derived `Live`), `cashflow_currencies()`, `cashflow_dates_without_snapshot()`. Transfer mode writes both mirrored legs in one confirm. Hand-written SQL is no longer the normal path.

### The NLV = 0 hazard (fixed, but know why)
Cash-flow rows carry `NetLiquidation = 0`. `accountf$account_extend_data()` picks each day's NLV with `slice_max(heure)`, and `Tdata::twr()` computes `rn[i] <- e_nlv[i] / denom` as a **running product** — so one day resolving to NLV 0 gives `rn = 0` and zeroes **every subsequent day**, pinning TWR to -1. The NA guard in `twr()` does not catch it (0 is not NA).

Two defences, both in place:
- Rows are written at `heure = 00:00:0N` (one slot per leg, allocated by `cashflow_heure()`) so they sort ahead of every real snapshot.
- `account_extend_data()` selects the day's NLV only from rows with a real `NetLiquidation`, and buckets flows onto the next observed snapshot date via `findInterval(..., left.open = TRUE)`. A flow on a date with **no** snapshot therefore counts from the next recorded day instead of destroying the series.

### Why this matters
The TWR caller sums CashFlow across all rows for the day. Since `readAccount` has already FX-converted each row, the sum is in base currency — meaningful even when legs span multiple currencies. Missing or wrong-signed legs produce phantom market moves in the TWR series.

### Known gap
`Tdata::getAccountLive()` inner-joins sub-accounts with `multiple = "any"` over `date >= today` only, and its rows carry no `Currency`. A flow booked on `U1804173` is **not** reliably reflected in the `Live` row. Worth a TODO.

### Related
- [[reference_ibkr_activity_statement]] — where to source the per-symbol/per-currency ground truth
- [[project_account_transfer_cashflow_signs]] — the 2026-04-16 case study and the Tdata 5.10.18 `twr` interpolation fix
- [[reference_live_virtual_account_no_table]]
