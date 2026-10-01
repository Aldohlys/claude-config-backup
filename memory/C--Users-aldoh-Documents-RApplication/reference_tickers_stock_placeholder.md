---
name: reference_tickers_stock_placeholder
description: "Tickers row Name='STOCK' (Type STK, no YahooName) is a deliberate default used by RPreTrade and other apps — never delete or 'fix' it"
metadata:
  node_type: memory
  type: reference
  originSessionId: 1c26deb6-0a13-4dd0-99ab-91d81259daed
  modified: 2026-10-01T01:28:09.261Z
---

`Tickers` holds a row `Name = 'STOCK'`, `Type = 'STK'`, empty `YahooName`. It is the
default symbol of RPreTrade and several RTrading applications: keeping it in
Tickers lets those apps set the other parameters (STK, ...) for a not-yet-chosen
symbol. The user decided 2026-10-01: leave it in Tickers, do not touch it.

Side effect: BOT_monthly (and any Yahoo loop over Tickers) logs
"getYahooData: Skipping invalid ticker: STOCK" — harmless, not an anomaly to
report. If the warning is to be silenced, filter `Name <> 'STOCK'` in the
consumer's query, never by changing the row.
