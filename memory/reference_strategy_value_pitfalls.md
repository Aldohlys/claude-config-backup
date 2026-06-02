---
name: reference-strategy-value-pitfalls
description: "Per-strategy / per-slice account-value calculations — futures mktValue is notional (not cash), TWR breaks on cumulative-PnL series, r_factor lives in RReporting UI"
metadata: 
  node_type: memory
  type: reference
  originSessionId: dad29eee-3519-49b0-8d78-c2257ddb1dfb
---

When computing aggregate "value" or "P&L" for any subset of an account (per strategy, per symbol, per currency), three pitfalls compound:

### 1. Futures `mktValue` = notional, not cash committed

In position tables (U1804173, etc.), `mktValue` is `pos × mktPrice × multiplier`. For **futures**, this is the **notional contract value** — for MCL (1 contract, mult=100) at $60/bbl this is $6,000, but the cash deployed is the margin (often ~$1k). For options, `mktValue` ≈ premium paid; for stocks, ≈ shares × price.

A naive `sum(mktValue)` for a strategy that includes a single futures position will dwarf everything else. Example caught 2026-05-18: TradeNr 721 MCL JUL26 peaked at `mktValue = $58,742` while its actual realized PnL was only `$1,614`.

**Use `unPnL` instead of `mktValue`** when computing strategy-level value/return. `unPnL` is real $ P&L for all instrument types (options, stocks, futures), already in base currency.

### 2. `Tdata::twr()` returns fractions and breaks on zero-crossing series

`Tdata::twr(dates, e_nlv, cashflows)` returns values where **`0.05` means 5%** — i.e. fractional returns. The whole-account UI (`accountUI.R`) then passes them through `scales::label_percent()` which multiplies by 100 for display.

Two consequences:
- For series that legitimately grow >100% (small base → large value), label_percent renders e.g. "8086%". Genuine math, just unreadable. If you want raw-magnitude display, divide-by-100 before plotting.
- **For cumulative-PnL series** (NLV = `unPnL_open + cum_realized_in_window`) that start at or near zero or cross zero, both `twr()` and the percent formula `metrics/first(metrics)-1` produce nonsense — denominator is small/negative. Volatility breaks too because daily-return is computed as `NLV/lag(NLV)-1`.

**Strategy-level TWR has no clean meaning** without a positive "starting capital" baseline. See [TODO #69](#TODO-69) (per-strategy capital allocation) for the planned fix. Until then, workarounds in `Tuser/account/view/strategyAccountUI.R`: divide-by-100 for TWR/Vol display + R-scaled percent mode for NetLiquidation.

### 3. R risk-factor (`r_factor`) is a UI-only numericInput

The "R value" per-trade risk unit is **not** in config.yml, the `Param` DB table, or any data file. It lives in `RReporting/app/ui.R:95` as a `numericInput("r_factor", "R value:", 200)`, defaulting to **200** (user mental model says 300). Each session sets its own; the same numericInput pattern is now also in `Tuser/account/view/strategyAccountUI.R` (default 300).

**How to apply:** any new "value-per-strategy" or "value-per-slice" feature should use `unPnL` not `mktValue`; expect TWR/Vol to be meaningless without an explicit baseline; for R-scaling the divisor comes from a per-app numericInput, not config. The clean architectural fix is the StrategyCapital table — see [TODO #69](#TODO-69) before re-deriving any of this from scratch.
