---
name: project_trades_cash_identification
description: "CASH/FX trades identified by Instrument-in-currencies (not Symbol==\"CASH\"); Symbol holds the foreign (non-base) currency — TODO"
metadata: 
  node_type: memory
  type: project
  originSessionId: 513f9f5e-a71a-4efb-b612-296b2ee13a32
---

TODO #35 Phases 2 (CASH identification) + 3 (FX Instrument mismatch) — shipped 2026-06-15, Tdata 5.10.29.

**The marker changed.** CASH/FX trades no longer carry the literal `Symbol == "CASH"`. A CASH trade is now `Instrument %in% getActiveCurrencies()` (Instrument is a bare currency code). `Symbol` now holds the **foreign (non-base) currency** of the pair, resolved as `if (Instrument == BaseCurrency) Currency else Instrument` — generalizes to a non-CHF base. Migrated rows: 659→EUR, 660/704→USD, 687/720→JPY, 719→GBP (`scripts/migrate_cash_symbol.R`, DB backed up).

**Why Symbol is NOT a safe cash detector:** a currency-rooted *future* (6S) normalizes to `Symbol="CHF"` (TODO #35 Phase 0), so `Symbol %in% currencies` would false-positive a CHF futures-option leg. Always detect on `Instrument`. This is why `compute_pnl_for_group(pos, base_total, is_cash)` takes an explicit `is_cash` logical (computed `Instrument %in% active_currencies` at the call site) instead of inspecting Symbol.

**Direction asymmetry (root of the trade-687 bug):** the transacted leg sits in `Instrument`, the settlement ccy in `Currency`, and the convention flips with trade direction — 659/660/704 hold the foreign ccy in Instrument (Currency=CHF base), but 687/719/720 hold base CHF in Instrument with the foreign ccy in Currency. `Symbol` = foreign ccy normalizes this and equals the portfolio CASH row's `symbol`.

**Files:**
- `Tdata/R/cash.R`: `is_cash_trade()` → Instrument-in-currencies; `getCashTradeForCurrency()` matches `Symbol = ?` (foreign ccy); `reconcile_cash_positions()` joins `Symbol = symbol`. `resolve_cash_cost_basis()` unchanged (re-derives quote direction from Instrument vs position ccy). Portfolio side still keyed on `type == "CASH"` (unchanged).
- `RReporting`: `trade_operations.R` (create_manual_cash_trade Symbol, flex-import Symbol, forex-detect in adjust/close), `reactives.R` (cash_raw_legs via Instrument, cash_idx via cash_raw_legs$TradeNr), `server.R` (3 compute_pnl_for_group sites). RReporting `read_trade` test now needs `library(Tdata)` (read_trade calls getParam).
- **Phase 3 (687):** `Tuser/symbol/logic/symf.R::stats_one_position` symbol-branch gate `type=="Stock"` → `type %in% c("Stock","CASH")` (currency-code Symbols only match CASH portfolio rows, can't alias a future root). `Tuser/symbol/view/displaytradeUI.R` now passes `Currency` into `stats_one`/`stats_one_position` so `convert_fx_trade_amounts()` fires → 687 Risk 78 649 JPY → 389.71 CHF, MarketValue/unPnL no longer NA.

Related: [[project_trades_symbol_underlying_model]], [[reference_trades_portfolio_join_key_by_type]], [[project_fx_risk]], [[reference_savetrades_overwrite]]. (The `feedback_no_transient_migration_logic_in_code` sync_stop_risk.R pick() resolver was already collapsed 2026-06-14, commit 6e5e4a2 — not a #35 follow-up.)
