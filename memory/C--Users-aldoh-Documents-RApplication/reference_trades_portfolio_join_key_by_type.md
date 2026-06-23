---
name: reference_trades_portfolio_join_key_by_type
description: "Joining Trades to portfolio — the join key depends on instrument type (stocks via Ssjacent==symbol, options/futures via Instrument)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a8bd74cc-aa12-4020-a70e-3106b3a00888
---

When joining the **Trades** table to **portfolio** rows for market data (mktValue, unPnL), the join key depends on instrument type — there is no single universal key:

- **Stocks**: `Trades.Instrument` is the IBKR company name (e.g. `"CARREFOUR SA"`), but the portfolio stores the **ticker** (e.g. `"CA"`) in both `symbol` and `Instrument` (via `Tbasics::buildInstrumentName`, which returns `sym` for non-option/future types). So join on `Trades.Ssjacent == portfolio.symbol`. Joining on `Instrument` silently fails → mktValue/unPnL come back NA → downstream values collapse to 0.
- **Options / Futures / TreasuryBill**: `Trades.Instrument` *does* match `portfolio.Instrument` (buildInstrumentName produces the same string). Join on `Instrument`. Several option legs share one `Ssjacent`, so an Ssjacent/symbol join would multiply rows or assign one leg's value to all — keep the per-leg Instrument match here.

Canonical statement of this convention lives as a code comment at `Tdata/R/account.R` (~line 507) where the portfolio is built. The robust pattern for consumers: per-leg `Instrument` join as primary + `Ssjacent==symbol` as a coalesce fallback (covers both without regressing multi-leg trades).

This bit `Tuser/symbol/logic/symf.R::stats_one_position` (routine Trade tab Position summary) — fixed 2026-06-13 (trade 697 Carrefour, all-zero Position table). Related cash/FX variant of the same mismatch: [[project_trades_signed_delta_risk]] and the TODO #36 trade-687 CHF/JPY case.
