---
name: feedback_net_pos_per_instrument
description: "Never net Trades.Pos across a trade's legs — a fully open +1/-1 vertical sums to 0 and reads as flat; group by Instrument first"
metadata:
  node_type: memory
  type: feedback
  originSessionId: b5594f9f-5c16-4d1e-914d-5892ae66634f
  modified: 2026-09-24T08:46:35.380Z
---

Summing `Pos` over all rows of a TradeNr is wrong for multi-leg trades: a fully open
vertical (+1 long leg, -1 short leg) nets to 0 and is read as "flat/closed".

The same bug has shipped twice in RReporting, both on trade 746 (BRK B 520/525C):
- `compute_open_realized()` treated net 0 as fully closed and booked the whole debit as realized (fixed: `instrument=` arg, split per contract).
- `register_tradenr_echo()` in the Adjust/Close modal showed `net +0` (fixed 2026-09-24, commit 7fe1256: `format_open_position()` nets per Instrument, shows `legs +1/-1`).

**Why:** each check that nets Pos across legs looks correct on the single-leg trades it is usually tested with. It fails only on spreads, and the output looks like a plausible value.

**How to apply:** whenever code aggregates `Pos` per TradeNr, group by `Instrument` first, then combine the per-leg nets. Include a multi-leg spread in the test fixture. A roll (old leg flat, new leg open) is the other case to cover. For what is actually held, see [[feedback_open_positions_from_portfolio_not_trade_status]].
