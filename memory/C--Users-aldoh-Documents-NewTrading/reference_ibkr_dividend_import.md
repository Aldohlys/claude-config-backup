---
name: reference_ibkr_dividend_import
description: IBKR dividends reach the Trades table only via Tdata::importIBKRDividends(statement CSV) — TWS API has no cash ledger; one net row per PAID dividend, EventType "Dividend", Pos 0, on the trade holding the stock
metadata:
  type: reference
---

TWS/ib_async exposes no cash ledger. Tdata's `getNTMDividend` (tick 456) returns the announced per-share dividend (past/next 12 months), not what was credited. Paid dividends come from an IBKR Activity Statement CSV ("Dividends" + "Withholding Tax" sections). A MULTI statement covers U1804173 and U25343478 in one file; each account block opens with `Account Information,Data,Account,<id>`.

`Tdata::importIBKRDividends(file, apply = FALSE)` (5.20.7):
- Writes one Trades row per payment: EventType `Dividend`, Pos 0, Total = NET (gross + tax) in the dividend currency, TradeDate = pay date. The user chose net-only.
- Books on the trade holding the symbol on the pay date. A stock moved between accounts is booked on its trade even when the cash landed in the other account (CRST → trade 702 in U25, paid into U1804173).
- Idempotent. The default is a dry run.
- Books on PAYMENT only: unpaid accruals ("Change in Dividend Accruals") are skipped. The user's choice. Japanese dividends are paid 2–3 months after the ex-date.
- Realized P&L: RReporting and the Tuser All view pick it up from the cash. The Tuser Position table needed `stats_one_position` to add dividend rows outside the has_closing gate.

First run 2026-09-23: CRST GBP 7.20, CA EUR 72.75 + 31.50, MRD CAD 25.50. Pending, to import once paid: 9273.T JPY 6,103 net (pay 2026-09-28), 3440.T JPY 14,227, 5982.T JPY 5,081, MRD CAD 25.50 (pay 2026-09-29). Download a fresh statement after those dates and re-upload it.

UI (the user's choice of place, Tdata 5.20.8, Tuser 24791db, RReporting 69133e4):
- **Tuser**, routine Trade tab: "Import dividends" file button under the History header. It previews with `planIBKRDividends(file, getAllTrades())` and books with `importIBKRDividends(apply = TRUE)`, straight to the DB.
- **RReporting**, New Trades tab: the same button, but it plans against the LOADED `all_trades()` and appends in memory; the user then clicks Save.

RReporting keeps Trades in memory and `saveTrades` OVERWRITES the table. Tdata 5.20.8 therefore makes `saveTrades` abort when DB dividend rows are missing from the input (the TradeNr check alone could not see them), unless `force = TRUE`. Never write Trades behind an open RReporting session without that guard.

Related: [[reference_internal_transfer_cashflows]], [[feedback_import_ui_placement]].
