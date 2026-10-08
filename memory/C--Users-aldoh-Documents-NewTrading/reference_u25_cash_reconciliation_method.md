---
name: reference-u25-cash-reconciliation-method
description: "Reconcile U25343478 cash per currency as the CHANGE between two portfolio snapshots vs Trades in the window - a full-history ledger fails because Account.CashFlow also holds position transfers; leftovers = interest, dividends"
metadata:
  node_type: memory
  type: reference
  originSessionId: e87e0ecc-b1ff-410a-a8f3-4449c3418f84
  modified: 2026-10-08T08:31:15.711Z
---

Method validated 2026-10-08 (the 02/09 FX conversions):
1. Cash per currency: the portfolio snapshot table `U25343478`, rows with symbol in (EUR, GBP, JPY, CAD, KRW); `pos` = balance (native).
2. CHF cash = Account.TotalCashBalance − sum(mktValue of the foreign cash rows). Use snapshots that are a day apart or more: the 09-03 20:44 row had "positions fetch failed" and was off by 172 CHF.
3. Pick a start snapshot before the events. The change to now should equal sum(Trades.Total where Currency = X) + sum(Pos where Instrument = X, for cash legs) + CashFlow in the window.
4. The leftover should be interest only (debited early in the month, about balance × borrow rate / 12) plus dividends. A positive jump mid-month is an unbooked dividend (09-28 JPY 6,103 = 9273; 09-29 CAD 25.50 = MRD).

Don't build a full-history ledger. The 2026-04-16 Account.CashFlow rows include position transfers (JPY 386,800 etc.), which aren't cash, so the totals never close.

Flex rows still in RReporting's "new trades" list have an empty Status and aren't in Trades yet. Check `Trades` before assuming a row was saved.

Related: [[reference-internal-transfer-cashflows]], [[project-u25-fx-hedge-policy]], [[reference_ibkr_dividend_import]].
