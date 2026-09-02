---
name: reference-krw-conversion-route
description: KRW cannot be converted directly to CHF at IBKR; the working route is KRW -> USD -> CHF in two legs
metadata:
  type: reference
---

KRW is a restricted currency at IBKR — there is no direct KRW/CHF conversion. The route that works is **KRW -> USD -> CHF**, executed as two separate conversions (and CHF -> USD -> KRW in the other direction).

Practical consequences:
- Two orders, two commissions (~USD 2 minimum each via FXCONV, since these sizes are far below the IDEALPRO ~25k threshold — see [[ibkr-fx-exposure]]).
- A transient USD exposure between the two legs. Do them back to back; don't leave the USD leg parked overnight if the point of the trade was to be CHF-neutral.
- This is why the Spigen Korea position (192440, KOSDAQ, account U25343478) was funded by **borrowing KRW on margin** rather than by converting CHF — the ledger has explicit CHF->JPY and CHF->GBP FOREX trades (TradeNr 720, 719) but none for KRW.

Cost context: the KRW margin loan runs ~7.5%/yr (confirmed three ways — MTD interest column, accrued interest vs cost basis, and the 71-day carry on the position). That is by far the most expensive borrow in the book: JPY ~2.2%, CAD ~2.8%, EUR ~3.3%, GBP ~5.2%. So the KRW hedge is the one bucket where the hedge cost is a large fraction of the currency risk it removes, and it is also the hardest to adjust.

Related: [[ibkr-fx-exposure]], [[project-daubasses-portfolio]]
