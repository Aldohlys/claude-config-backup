---
name: reference_gonet_pnl_and_stats
description: How Gonet PnL/cost is computed (getGonet) and the Trade-tab Gonet stats branch
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3f4e5a5e-17d7-445c-8433-e74f630f76ef
---

**getGonet PnL math** (`Tdata/R/account.R`). Since **5.14.0** the basis is an average-cost lot walk (`gonet_lots`), not a sum of cashflows, so the snapshot carries a genuine split:

- `unPnL = mktValue - basis` — **unrealized only**, the shares still held.
- `realizedPnL = realized` — gains banked by past sales **plus** cash events booked against the trade (dividends, 5.20.0).
- `avgCost = basis / pos`; **Cost = `mktValue - unPnL`** = the basis still carried (avgCost is stored pre-rounded, so `pos*avgCost` drifts).
- **Total PnL = `unPnL + realizedPnL`.** Pre-5.14.0 notes calling `unPnL` the total cash-basis PnL are wrong.

Dividends are no longer absent: they enter `realizedPnL` when recorded as a ledger row carrying the trade's TradeNr — see [[project_gonet_cash_positions]] for the row convention.

Grouping is per `sym_yahoo` (TradeNr = first leg), so **corporate-action TradeNr relabels are harmless**: HOLN spun off AMRZ 2025-06, user closed ledger trade 8 + reopened as 18; the snapshot still labels the live 260sh TradeNr 8 but the walk covers all HOLN legs. AI's 10:11 annual splits make CSV share counts ≠ snapshot; `GonetPos.csv` stays authoritative for the share count and a mismatch is logged (AI: CSV 169 vs legs 135 — a standing gap).

**Trade-tab 2nd table for Gonet**: `symf$stats_all_gonet()` + `datatablef$stats_all_gonet()`, reached via a branch in `displaytradeUI` `output$trStats` gated on **absence of `expdate`** (Gonet is stocks-only). Columns: TradeNr, Symbol, Pos, StartPrice(avgCost, blank for cash), Price, Cost, Value, ValueBase, **realizedPnL, UnrealizedPnL, PnL**, PnLBase, Return(PnL/|Cost|) + base-ccy TOTAL, with hidden `t_*_base` sort cols + `t_is_total` pin ([[reference_dt_idioms]]). Risk and a cashflow Total are omitted (no Gonet source). Cash rows show a blank realizedPnL. Test: `Tuser/tests/test_gonet_stats_pnl.R`.

**Why IBKR stats_all/stats_one can't serve Gonet**: `getActiveTrades("Gonet")` → "No account exists!"; Gonet TradeNrs **collide** with IBKR Trades-table numbering (`getTradeData(1)` for Gonet AI returns IBKR "VT Call"). See [[reference_account_strategy_topology]]. Shared DT-options builder = `datatablef$basecurrency_dt_opts()` (hides whatever `sort_map` names, so a new column needs an entry there).
