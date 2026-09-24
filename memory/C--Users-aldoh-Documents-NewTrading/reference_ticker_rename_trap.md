---
name: reference_ticker_rename_trap
description: When an exchange renames a ticker (Roche ROG -> RO), the old symbol on IBKR SMART resolves to a different company and the old Yahoo symbol dies; update every symbol field together
metadata:
  type: reference
---

Roche's SIX ticker changed from ROG to RO in 2026. Snapshots still written with `ROG` (2026-04-01 and 04-03, 04-23) got **Rogers Corp (NYSE: ROG, ~105–130 USD)** from IBKR SMART, not Roche (~330 CHF). That made Gonet ~12k CHF low, with no error raised. Yahoo `ROG.SW` returns no data; the live symbol is `RO.SW`.

A second case of the same mechanism: the Tickers row for AMRZ describes the NYSE line (USD, kept for the scanner) while Gonet holds the SIX line in CHF. Tdata 5.20.6 `gonet_quote_to_position_ccy` now converts such quotes.

**How to apply:** after a rename, change every symbol field in the same pass: GonetPos.csv `sym_ibkr` and `sym_yahoo`, GonetTrades.csv `sym_yahoo` (lots and legs match on it, so the two files must agree), and Tickers `Name`/`YahooName`. Then check one fresh snapshot price against the exchange close. A price that is plausible but 60–70% off usually means the symbol resolved to another company.

Related: [[reference_gonet_account_history]], [[reference_gonet_csv_bookkeeping]].
