---
name: reference_untagged_cash_hidden_in_summary
description: RReporting Summary is trade-driven — a CASH balance with no CASH trade (TradeNr=NA) is invisible
metadata: 
  node_type: memory
  type: reference
  originSessionId: 672ba0ce-0bbd-4810-bd8c-9c6150cb270e
---

A portfolio CASH balance only shows in RReporting's **Summary tab** if there is a corresponding CASH/FX trade in the `Trades` table for that account. The Summary reactive (`RReporting/app/R/reactives.R` ~line 188) is **trade-driven**: it starts from `getActiveTrades()` and `inner_join`s portfolio positions by `TradeNr`; cash legs come from trades whose `Instrument` is a currency. RReporting has **no raw-portfolio view** — `readLastPortfolio()` is used only inside that join — so a portfolio CASH row with `TradeNr=NA` (no linked trade) never appears anywhere in the app.

How a CASH row gets `TradeNr=NA`: `create_cash_portfolio_row()` (`Tdata/R/cash.R`) calls `getCashTradeForCurrency(account, currency)`, matching `Trades.Symbol == currency` for that account; no match → `TradeNr=NA`. Residual cash (dividends, fees, interest, leftover from a closed position) has no originating FX trade → stays untagged → hidden.

Worked example (2026-06-23): U1804173 GBP cash 31.59 (≈33.82 CHF) was invisible — no GBP trade in U1804173 (the only `Symbol='GBP'` trade is #719, in U25343478). Same account's EUR/JPY/USD showed because they link to trades 659/687/704. Contrast: U25343478 GBP −414.51 **does** show (links to #719). So the discriminator is "does a CASH trade exist for that account×currency", not the currency itself. User chose to leave untagged residual cash hidden (by design). See [[reference_trades_portfolio_join_key_by_type]], [[project_trades_cash_identification]].
