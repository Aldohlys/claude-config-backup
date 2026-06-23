---
name: reference_tickers_fut_row_convention
description: When inserting a future into Tickers, Name uses the IBKR local symbol (e.g. MCLN6) and TradingClass stores the *options* trading class (e.g. MCO for MCL futures, LO for CL futures), not the future's own trading class shown in TWS.
type: reference
originSessionId: 435ffc45-0ae7-46ab-8d17-978675432004
---
For Tickers rows of `Type='FUT'`:

- **Name**: IBKR local symbol — e.g. `MCLN6` (Micro WTI Jul-26, last-trade 2026-06-18), `CLM6` (CL Jun-26), `SR3Z6`. Never the display form `MCL'Jul 26`.
- **TradingClass**: the **options** trading class on this future — `MCO` for MCL options, `LO` for CL options, `SR3` for SR3 options. The future's own trading class shown in TWS Financial Instrument Description ("MCL" for MCLN6) is NOT what goes here. Precedent: CLM6 row has TradingClass='LO'.
- **ConId**: the IBKR contract id for the specific future. Required to avoid the `lastTradeDateOrContractMonth='YYYYMM'` gotcha (see feedback memory).
- **Expiration**: full `'YYYYMMDD'` of the actual last trading date (not the contract-month label).
- **Multiplier, Currency, Exchange, OptExchange**: from the TWS Financial Instrument Description.
- **IV**: `'NO'` unless you want it in the IV scanner universe.

**How to apply:** When user asks "add this future to Tickers", get the TWS option description (right-click an option chain entry in TWS) — its Trading Class is what goes in the Tickers row. Don't infer TradingClass from the future's own description.
