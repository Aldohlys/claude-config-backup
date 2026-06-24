---
name: reference_trades_right_column_sparse
description: "mydb.db Trades migrated French->English cols + Underlying/Right empty for BOT — parse ticker AND vehicle from the Instrument string"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 2bd4384e-3b59-46f3-a4cf-4b6e7212fea0
---

**Schema migrated French→English (seen 2026-06-24):** Trades columns are now `Underlying` (was `Ssjacent`), `Status` (was `Statut`), `Notes` (was `Remarques`), `Price` (was `Prix`), `Commission` (was `Comm.`). `Status` VALUES are still French: `'Fermé'` (closed), `'Ouvert'` (open), `'Ajusté'` (adjusted) → filter `Status LIKE 'Ferm%'`. Full col list: TradeNr, Account, TradeDate, DateTime, TimeZoneSource, Strategy, Instrument, Symbol, Pos, Price, Commission, Total, `Exp.Date`, Risk, Reward, PnL, Status, Currency, Notes, Strike, Right, EventType, Return, Underlying, Stop.

**`Underlying` AND `Right` are EMPTY for BOT rows** — not just sparse on old ones. For BOT closed legs: `Underlying` blank (only futures like `GCM5` populated); `Right` = 182 empty / 141 C / 19 P. So BOTH the ticker and the call/put rights must be parsed from the **`Instrument`** string. (The older sparse-`Right` note from 2026-06-08 was the precursor; the migration made `Underlying` unusable too.)

**How to apply (parse from `Instrument`):**
- Ticker = first whitespace token if it matches `^[A-Z]{1,5}\b` (e.g. `AMGN 16JUN23 225 C`, `SPY Vertical Spread 410/420 -1C/1C` → `SPY`). Combos like `+1 Call @158 +1 Put @155` have no ticker → skip.
- Vehicle/direction: option legs end in strike + ` C`/` P`; verticals embed `1C`/`-1C`. Count calls vs puts with `\dC|\bCall\b|\sC\b` vs `\dP|\bPut\b|\sP\b`; puts-dominant = short/bearish. Futures: `MCL`/`GCM`/`MSF` or `GC[F-Z]\d`.
- A trade = group by **`TradeNr`**; realized **`PnL` lands on the closing fill** (sum over TradeNr; opening legs are 0).
- General rule: before grouping by any convenience column, check it's actually populated — don't trust a sparse/emptied flag.

Used by `Strategies/Breakouts/bot_retroactive_test.py` (now does this parse) and `bot_atr_validation.py` (still references old cols — would break, needs the same fix). See [[project_bot_trading_plan_review]].
