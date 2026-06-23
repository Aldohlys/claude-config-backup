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
