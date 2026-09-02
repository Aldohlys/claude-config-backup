---
name: project-u25-fx-hedge-policy
description: U25343478 FX hedge policy - fund each foreign holding with a loan in its own currency sized to market value, rebalance outside a 90-110% coverage band
metadata:
  type: project
---

Account **U25343478** (Daubasses / VALUE strategy, base CHF) holds foreign small caps in JPY, EUR, GBP, CAD and KRW. Policy set and executed 2026-09-02.

## The convention
Fund each foreign stock holding with an IBKR margin loan **in that stock's own currency**, sized to its **market value** — not its purchase cost. The loan is the hedge; no separate FX instrument is used.

**Why market value, not cost:** funding at cost hedges the entry price, not the position. Winners drift under-hedged (you become long the currency), losers drift over-hedged (you become short it). That is exactly how EUR ended up at 48% coverage and GBP at 168% — CRST fell 40% while its GBP loan stayed at the original 458.

## The band
Coverage = local-currency loan / stock market value. **Rebalance any bucket outside 90-110%.** Check at the Daubasses weekly review.

- Price moves are the fast driver — a 15% move breaches the band immediately.
- Interest drift is slow and predictable: interest accrues in the foreign currency, enlarging the short leg, so **coverage creeps upward at exactly the borrow rate**. From 100%, reaching 110% takes roughly 10/rate years — JPY 4.5, CAD 3.6, EUR 3.0, GBP 1.9, KRW ~1.1.

## Borrow rates (2026-09, IBKR Tier 1 = benchmark + ~1.5%)
JPY 2.2% | CAD 2.8% | EUR 3.3% | GBP 5.2% | KRW 7.5%. Total financing ~690 CHF/yr, about 2.8% of the stock book. That is the price of holding foreign equity fully hedged on margin borrowing, and it is worth paying — CHF has appreciated against all of these for years (JPYCHF roughly -9%/yr over six years).

## The KRW exception
KRW at 7.5% against roughly 11% annual KRWCHF volatility is a poor price — you pay 7.5% to remove 11% of one sigma, versus JPY where 2.2% buys the same protection. It is also the one bucket that cannot be tuned cheaply: no direct KRW/CHF conversion exists, see [[reference-krw-conversion-route]]. **If the carry becomes intolerable, exit Spigen (192440) rather than lift the hedge** — lifting it swaps a known 133 CHF/yr for unhedged KRW exposure the user has explicitly said he does not want.

## Do not unwind a hedge because one leg lost
Verified 2026-09-02. KRW rose 11.8% against CHF between 23 Jun and 2 Sep (KRWCHF 0.000527 -> 0.000589; idiosyncratic to the won — EUR +1.85%, GBP +2.43%, JPY +1.48% over the same window). Effect: **+183 CHF on the stock, -183 CHF on the loan, net zero.** The entire 24 CHF change in the bucket was accrued interest.

It *felt* like the hedge was failing because **IBKR's Unrealised P&L column computes the gain in local currency and only then translates it — so the +183 FX gain on the stock never appears anywhere**, while the loan leg is plainly visible growing from -1,559 to -1,766. A working hedge reads on screen as a pure loss. Related: [[feedback-dont-switch-frameworks-midtrade]].

## Executed 2026-09-02 (all via market orders, FXCONV-style conversions)
EUR loan 1,444 -> 3,004 | JPY 16,453 -> 17,403 | GBP 458 -> 272. Gross non-CHF exposure fell from 2,880 CHF (10.3% of NLV) to 219 CHF (0.8%). Final coverage: EUR 100.0%, GBP 100.0%, JPY 99.9%, CAD 97.0%, KRW 101.8%.

Execution note: small FX orders (170-1,657 units) are odd lots far below the IDEALPRO ~25k threshold. A limit at the displayed ask will not fill — the TWS ticket snaps the price when opened and does not track the market. **Send these as market orders**; on 200-1,600 CHF of notional the slippage is worth less than the attention.

The JPY +112 residual is the 129 of dividends receivable and self-corrects: when paid it lands as JPY cash and shrinks the loan, taking coverage to 99.2%. No action needed.

Related: [[ibkr-fx-exposure]], [[reference-u25-margin-cushion]], [[project-daubasses-portfolio]], [[project-crst-distress]]
