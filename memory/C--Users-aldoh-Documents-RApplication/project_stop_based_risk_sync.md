---
name: project_stop_based_risk_sync
description: scripts/sync_stop_risk.R syncs share Risk from live TWS stop orders; Trades.Stop column added
metadata: 
  node_type: memory
  type: project
  originSessionId: 9396fe68-7474-48eb-9f7a-c38651ff5dc2
---

Stop-based Risk automation for share positions (TODO #67 share half), shipped 2026-06-14.

**Why:** VALUE-strategy share trades stored Risk = full position notional (the static/wrong risk [[project_trades_signed_delta_risk]] / TODO #38 flags). Real max-loss is defined by the protective stop order, not capital outlay.

**How to apply:**
- `Trades.Stop` (REAL, nullable) column now exists — added via direct DBI ALTER, NOT a Tdata rebuild.
- `scripts/sync_stop_risk.R` (default = ALL accounts; pass an account to restrict): pulls live STP/TRAIL orders via `tdata_py$safe_ib_connect()` + inline `ib.reqAllOpenOrders()`/`openTrades()` (no Tdata function — retrieval inline), filters to secType **STK + FUT** (FOP excluded — long-option risk is the premium), matches each order to the open trade by **root `symbol` == trade match-column** (TWS `symbol` is the bare root, e.g. `5658`/`CA`/`MCL`; `localSymbol` carries `.T`/contract suffix), computes `Risk = (avg entry − stop) × qty × multiplier` native ccy (STK mult=1; FUT mult from `c.multiplier`), consolidates full Risk on the trade's **first row** (min rowid) with others zeroed, writes `Stop` on all rows.
- Match column is **`Symbol`** — the bare root that equals the IBKR order `symbol` (`5658`/`CA`/`SOFR3`). `Underlying` holds the raw localSymbol (`SR3Z6`, NULL for stocks) — do NOT match on it. Final #35 names: `Symbol` (was Ssjacent), `Price` (Prix), `Status` (Statut), `Commission` (Comm.), `Notes` (Remarques); `Status` values stay French (`Ouvert`/`Ajusté`/`Fermé`). Closed trades matched via `NOT LIKE 'Ferm%'` (dodges accented 'Fermé').
- Dry-run by default; `--commit` binary-copies the DB to `logs/mydb_pre_stop_sync_<ts>.db` first, then UPDATEs in a transaction. Re-runnable as stops move.
- Wired into `daily_portfolio_update.R` as **Step 3/4** (subprocess `--commit`, warn-and-continue).
- First run (2026-06-14) updated 10 U25343478 VALUE share trades; CRST (no stop) reported+untouched; FOREX/CASH excluded. No FUT stops exist today, so the futures path is built but inert.

**#35 complete (Tdata 5.10.28):** the temporary `pick()` migration resolver was removed (commit 6e5e4a2) — script now uses plain `Symbol`/`Price`/`Status`. (Earlier note that Ssjacent→Underlying was wrong; it was Ssjacent→**Symbol** with Underlying = raw localSymbol.)

**Modal half of #67 DONE 2026-06-14 (RReporting stable/prod 7249fea):** `compute_risk(stop, price)` branch = `(avg entry − stop) × qty × multiplier` (mirrored shorts, floored 0); Open/Adjust modals show a `Stop` input gated on Strike/Right (stocks/futures only, Multiplier defaults to 1 there); live observer recomputes Risk from stop on Open. `Underlying`+`Stop` added to `trade_fields` and written in insert/adjust/close/cash paths (also fixed a latent post-#35 rbind/save-drop mismatch). 8 tests in `tests/test_stop_risk.R`.
- **Adjust caveat:** stop is recorded but Risk stays under the #38 signed-delta resolvers (absolute stop-risk vs delta semantics = #75 residual); re-derive via sync_stop_risk.R.
- **Futures caveat:** Multiplier field must be set to the contract multiplier (e.g. MCL=100) on a futures Open; defaults to 1 (shares).

## Orphaned stops (added 2026-09-03)

The sync walks **live stop orders**, so a trade whose stop has disappeared is never visited and keeps its old stop-derived Risk — understating risk on a position that is no longer protected, i.e. failing in the *reassuring* direction. Stops vanish routinely: IBKR cancels resting orders on corporate actions, and a non-GTC order expires.

- A trade with a recorded `Stop` but no live stop order is an **orphan**, always reported.
- `--reset-orphans` clears the `Stop` and restates Risk as the unprotected max loss (`|pos| * avg entry`), consolidated on the trade's first row.
- **Futures orphans are reported, never rewritten** — `Trades` stores no contract multiplier, and the live order that would have supplied it is exactly what is missing.
- **Safety rail:** an *empty* live stop set while trades still carry a `Stop` is treated as an incomplete fetch, not as a day on which every stop was cancelled. Nothing is rewritten. Without this, one bad TWS fetch would restate every stop-managed trade to full notional.
- Orphan grouping is scoped by `Symbol` as well as `TradeNr`, matching the live-order match key — trade 741 spans SLB and T, whose positions must not be pooled.

`daily_portfolio_update.R` passes `--reset-orphans` on the **late-afternoon run only** (hour 16-19). The scheduled task fires at **10:00, 17:00 and 22:00** — note the checked-in `scripts/DailyPortfolioUpdate.xml` still declares a single 08:00 trigger and is stale relative to what is registered. Late afternoon is when the day's order book is settled; running the reset pre-open would act on an unsettled book.

First live catch: trade 697 (CA, Carrefour, 200 sh) carried `Stop = 13.5` with no live order — Risk read **487 EUR** against an unprotected exposure of **3187 EUR**.

Risk still goes stale between runs (move a stop at 10:30 and it is stale until 17:00; worst gap 22:00→10:00), when TWS is unreachable at run time (the daily job warns and continues by design), and on the two silent skips: two stop orders for one (account, symbol), or an order symbol matching no open trade. The **Orders tab** in the routine app surfaces the same gap from the position side — see [[reference_ibkr_open_orders_semantics]].

## Price-scale reconciliation (added 2026-09-03)

Stops are restated onto the position's own price scale before the risk formula runs: IBKR quotes LSE stocks in pence while reporting `currency = 'GBP'`, so an order price can be 100x the stored position (CRST: stop 45.0 pence vs a 1.03 GBP entry). The reference is the account snapshot's `mktPrice`, so nothing about exchanges or currencies is hardcoded. The old `max(0, risk)` clamp is gone — it turned that mismatch into `Risk = 0` on a live 400-share position. Full detail and the general lesson: [[reference_ibkr_minor_unit_quotes]].
