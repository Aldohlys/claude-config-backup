---
name: reference_gonet_csv_bookkeeping
description: "How GonetTrades.csv / GonetPos.csv feed getGonet — sign conventions, free-share handling, and the user's recurring sign errors to verify"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 9086880f-22ad-4f6d-a57f-6eda3257a22b
---

The user maintains two semicolon-delimited CSVs (dates `DD.MM.YYYY`) in `NewTrading/` that feed `getGonet()` (Tdata `account.R`):
- **GonetTrades.csv** — cashflow ledger → cost basis. Cols: `TradeNr;orig_date;sym_yahoo;sym_ibkr;init_position;init_price;init_cost;currency`.
- **GonetPos.csv** — current positions. Cols: `sym_yahoo;sym_ibkr;position;exchange;type`.

**How getGonet uses them:** POSITION comes from GonetPos.csv; COST from `sum(init_cost)` per symbol in GonetTrades.csv; `avgCost = -sum(init_cost)/position`. `init_position` in the trades file is NOT used by getGonet (documentary only) — the position driver is GonetPos.

**Sign conventions (critical):**
- BUY = **positive** position, **negative** init_cost (cash out).
- SELL = **negative** position, **positive** init_cost (cash in).
- `init_cost` = actual net cashflow (may embed commission → can differ from `|pos|×price`; e.g. FXC buy cost 20,261.52 vs 177×112.84).
- Sales reuse the **original buy's TradeNr**.

**Free-share attributions** (e.g. Air Liquide loyalty ~1:10 "10→11"): zero cash cost → just **bump the GonetPos position**, and (recommended for audit) add a **zero-cost** line in GonetTrades: `TradeNr;date;sym;ibkr;<n>;0;0;CCY`. avgCost auto-drops (free shares dilute cost). This is how prior AI attributions were absorbed (trades net 120 sh but Pos was 154). 2026-06: AI 154 → 169 (+15 free), avgCost 75.09 → 68.43 €.

**Verify the user's manual sell entries — recurring SIGN ERRORS.** Twice in one session: ABBN sale had the cost sign wrong (negative not positive); FXC sale had the position sign wrong (+177 not −177). Always sanity-check that sells are `-qty / +cost` before accepting their edits.

**Gonet account note:** Swiss private bank, CHF base, holds the equity book (ABBN/HOLN/RO/SLHN/AMRZ/AI/TTE/OR/GTT + ETFs), buys UCITS ETFs. **Charges hefty per-order commission → prefer a single order over scaling into tranches** when advising entries/exits. Related: [[project_savetrades_overwrite_bug]], [[reference_trades_right_column_sparse]].
