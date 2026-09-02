---
name: reference-getstoredmetrics-no-freshness
description: getStoredMetrics() returns the newest Prices row with NO age check — always verify the datetime before using it as a current price
metadata:
  type: reference
---

`Tdata::getStoredMetrics(sym)` is literally `SELECT * FROM Prices WHERE sym = ? ORDER BY ROWID DESC LIMIT 1`. No freshness test whatsoever. `Prices` is only written for symbols something actually scanned, so a symbol nobody has touched returns a months-old row that still *looks* like a quote.

Bit the Tuser spread app on 2026-08-26: SMH's newest row was `20260603 21:33` at 638.59 while spot was 556.62 — an **84-day-old price 15% above the market**, displayed as "Current Price". Worse, it fed the strike window (`current_price ± moneyness_pct`), so the scan searched strikes 607–670 against a 557 spot and every result was meaningless. Frequently-scanned symbols (SPY, BRK B) have rows from minutes ago, which is why it never showed up in testing.

Pattern to use: take the stored row only within a few days (spread app uses 3, covering a weekend), else fall back to `getStockPrice(sym, close = TRUE)` — last Yahoo close, no TWS round-trip. Always surface the as-of timestamp and source in the UI; a silent stale quote is the whole failure mode.

Two datetime spellings in play: `Prices` stores `"20260603 21:33"`, `getStockPrice()` returns `"2026-08-25 22:00"`. Parse both.

Same exposure exists anywhere else `getStoredMetrics()` feeds a strike window or a valuation — not swept.

Related: [[reference_closed_market_option_marks_stale]], [[reference_tdata_option_fetch_internals]], [[feedback_recheck_strike_at_execution_spot]]
