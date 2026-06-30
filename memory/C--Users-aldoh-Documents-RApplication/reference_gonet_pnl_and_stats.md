---
name: reference_gonet_pnl_and_stats
description: How Gonet PnL/cost is computed (getGonet) and the Trade-tab Gonet stats branch
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3f4e5a5e-17d7-445c-8433-e74f630f76ef
---

**getGonet PnL math** (`Tdata/R/account.R` ~836-1007): cashflow is summed **per `sym_yahoo`** from `GonetTrades.csv` legs — `cost = sum(init_cost)` (signed: − buys, + sells), `TradeNr = first()`. Then per position: `mktValue = pos*mktPrice`, `unPnL = mktPrice*pos + cost`, `avgCost = round(-cost/pos, 2)`.

Consequences:
- **`unPnL` is TOTAL cash-basis PnL** (realized + unrealized), NOT unrealized-only. Verified on OR (partial sale at a loss): snapshot unPnL = realized + unrealized.
- **Net cash invested = `mktValue - unPnL` = `-sum(init_cost)`** exactly. Use this for "Cost", NOT `pos*avgCost` (avgCost is stored pre-rounded → ~0.007% drift; `mktValue-unPnL` makes Value−Cost=PnL reconcile exactly).
- **Dividends are absent everywhere** (no dividend rows in the CSV ledger; IBKR API here only gives forward yield via `getNTMDividend`, and these are Gonet-held not IBKR-held). So Gonet PnL is ex-dividends — same basis as the IBKR view's price-based unPnL.
- Grouping by symbol (TradeNr=first) means **corporate-action TradeNr relabels are harmless**: e.g. HOLN spun off AMRZ 2025-06, user closed ledger trade 8 + reopened as 18; snapshot still labels the live 260sh as TradeNr 8 but cost sums all HOLN legs (8+18) and reconciles. AI's 10:11 annual splits likewise make CSV share counts ≠ snapshot, but cashflow-based cost is split-agnostic.

**Trade-tab 2nd table for Gonet** (shipped 2026-06-30): `symf$stats_all_gonet()` + `datatablef$stats_all_gonet()`, reached via a branch in `displaytradeUI` `output$trStats` gated on **absence of `expdate`** (Gonet is stocks-only). Columns: TradeNr, Symbol, Pos, StartPrice(avgCost), Price(mktPrice), Cost(mktValue−unPnL), Value(mktValue), PnL(unPnL), Return(PnL/|Cost|) + base-ccy TOTAL, with the same hidden `t_*_base` sort cols + `t_is_total` pin as [[reference_dt_basecurrency_sort_pin_row]]. Risk/realizedPnL/cashflow-Total **omitted** (no Gonet source).

**Why IBKR stats_all/stats_one can't serve Gonet**: `getActiveTrades("Gonet")` → "No account exists!"; Gonet TradeNrs **collide** with IBKR Trades-table numbering (`getTradeData(1)` for Gonet AI returns IBKR "VT Call"). See [[reference_account_strategy_topology]]. Shared DT-options builder = `datatablef$basecurrency_dt_opts()` (used by both stats_all and stats_all_gonet).
