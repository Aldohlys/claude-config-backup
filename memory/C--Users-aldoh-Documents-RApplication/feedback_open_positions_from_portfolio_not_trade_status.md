---
name: feedback_open_positions_from_portfolio_not_trade_status
description: "'What do I hold?' comes from the portfolio snapshot, never from Trades.Status != 'Fermé' — and the portfolio's TradeNr outranks the Trades table's attribution"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1e2ceadf-edaa-4f9c-82d1-d1e2a412227d
  modified: 2026-09-01T08:56:53.996Z
---

When a feature needs the positions currently held, read the **portfolio snapshot** (`Tdata::getLatestPositions(account)` — latest `heure` of the latest `date`), not `Trades WHERE Status != 'Fermé'`.

**Why:** `Trades.Status` is maintained by hand and goes stale. Trade 511 (URA) still read `Ouvert` with legs that expired 18OCT24 / 15NOV24 and no position anywhere — the first Orders-tab build listed it as an unprotected position. The user caught it immediately: *"Why an URA missing order? There is no URA position."* A false alarm in a risk view is worse than a missing row, because it trains you to ignore the amber.

Two corollaries from the same incident:

- **The portfolio's `TradeNr` is the authoritative position→trade attribution.** The Trades table carried an SLB leg recorded under trade 741 (a `T` trade), which made a single SLB order look contested between 741 and 745; the portfolio assigns the held contract to 745 and settles it. Use Trades only to *recover* a `TradeNr` the snapshot left unset (see [[project_tradenr_backfill_stale]]), unambiguously or not at all.
- **Don't derive a position size by summing `Trades.Pos`.** The same mis-tagged row produced a phantom `-1` SLB short that is not held anywhere. `pos` from the snapshot is the real number.

**How to apply:** positions ⇒ portfolio; trade metadata (Strategy, notes) ⇒ Trades, joined on `TradeNr`. Exclude cash/FX with `is_cash_position()` — a currency balance is not a position an order acts on. When the two sources disagree, the portfolio wins and the disagreement is worth reporting to the user, not silently resolving.

Related: [[reference_trades_portfolio_join_key_by_type]], [[reference_ibkr_open_orders_semantics]], [[feedback_verify_before_acting]].
