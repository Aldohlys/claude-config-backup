---
name: reference_gonet_nlv_attribution_method
description: "How to attribute a Gonet NLV change between two dates from mydb (Gonet snapshots in native ccy, FX from cash rows, price vs FX split, one-offs from GonetTrades.csv); first done 2026-10-05"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4ebe8014-d954-450d-8334-8d5556b87f33
  modified: 2026-10-05T06:21:01.011Z
---

Recipe used 2026-10-05 for "explain the Gonet NLV move" (408,981 → 397,558 CHF, 09-22 → 10-05).
Report: `Reports/gonet_nlv_drawdown_20260922_20261005.md`.

- NLV series: `Account WHERE account='Gonet'`, `date` is an INTEGER yyyymmdd. Pick start/end by NLV
  row, then match the `Gonet` snapshot with the nearest `heure`. Gonet snapshot times run about 30 s
  earlier than the Account row (22:45:18 vs 22:45:53).
- In `Gonet`, `mktValue` is in the position's NATIVE currency, not CHF. The cash rows CHF/EUR/USD hold
  the FX rate to CHF in `mktPrice` (EUR 0.94104, USD 0.82169 on 09-22). Use those rates, not a
  separate FX table.
- Split: price = pos × Δprice × start FX; FX = pos × end price × ΔFX. Lines bought inside the window:
  end value vs `init_cost` in GonetTrades.csv (the cost includes commission). Take fees, dividends
  and FX conversions from the ledger rows dated inside the window. The residual should be under
  ~100 CHF.
- CashFlow = 0 in the window means the whole change is P&L. Check this before attributing.
- A Monday 06:16 snapshot carries the previous Friday close.

The user's own estimate ("−2%") excluded one-offs; with them it was −2.8%. Quote both.
Related: [[reference_gonet_account_history]], [[reference_gonet_csv_bookkeeping]].
