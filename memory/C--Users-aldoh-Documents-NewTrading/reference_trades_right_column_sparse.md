---
name: reference_trades_right_column_sparse
description: "mydb.db Trades.Right/Strike are null on older rows — parse vehicle from the Instrument string, not the Right column"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 2bd4384e-3b59-46f3-a4cf-4b6e7212fea0
---

In `mydb.db` Trades table, the `Right` (C/P) and `Strike` columns are populated only on newer trades — null/blank on older ones (e.g. **182 of 339 BOT legs**). Aggregating "option vs stock" off `Right IN ('C','P')` silently misclassifies every pre-2025 option trade as stock. Surfaced 2026-06-08: both AAPL OTM-option BOT winners (+\$1,346 call, +\$1,277 put) were flagged as stock, which flipped a backtest conclusion until caught.

**How to apply:**
- Detect vehicle by parsing the **`Instrument`** string: option legs end in strike + ` C`/` P` (e.g. `AAPL 26JAN24 190 C`, `SMH 18JUN26 547.5 P`); futures look like `MCL JUL26` / `GCM5`; stock = bare ticker. Last whitespace token in {C,P} ⇒ option.
- Underlying ticker = **`Ssjacent`**; a trade = group by **`TradeNr`**; realized **`PnL` lands on the closing fill** (sum over TradeNr; opening legs are 0).
- General rule: before grouping by any convenience column, check it's actually populated (`SELECT COUNT(*) WHERE col IS NULL`) — don't trust a sparse flag.

Used by `Strategies/Breakouts/bot_atr_validation.py`.
