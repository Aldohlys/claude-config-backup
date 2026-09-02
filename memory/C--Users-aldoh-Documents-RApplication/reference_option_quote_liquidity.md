---
name: reference_option_quote_liquidity
description: "How to judge option liquidity from Tdata — getOptValue served FROZEN quotes by default until 5.14.3, the `spread` column is the relative bid-ask width and is NaN exactly when there is no true mid, and compute_spread_risk_reward now returns both leg deltas and both leg spreads"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T04:12:58.889Z
---

Added Tdata **5.14.3** (2026-08-27), prompted by the user's rule: *"if liquidity
is bad in a weekly chain (20% or more) it should not be considered — and we need
live markets to really appreciate liquidity; frozen option prices do not mean
much."*

## The trap that made liquidity meaningless

`contract.py::getOptValue()` **hardcoded `ib.reqMarketDataType(2)`** — frozen —
with the comment "for consistent pricing". Every caller, including all option
spread pricing, was therefore scoring bid/ask width off last-close snapshots.
Those numbers look perfectly healthy and mean nothing.

`getOptValue(..., market_data_type = )` now exists. **Default `2` preserves the
old behaviour for every existing caller** (/analyze, swing scanner, RPreTrade,
Tuser); pass `1` for live with IBKR's own frozen fallback outside RTH. Precedent:
`chains_manager.py:1226` already used `reqMarketDataType(1)`.

Asking for live is not the same as getting it, so each row now carries
**`mktdata_type`** (`ticker.marketDataType`: 1=live, 2=frozen, 3=delayed,
4=delayed-frozen). Threaded through both quote-cache read paths. Check it before
presenting any liquidity figure.

## `spread` is the relative bid-ask width — and NaN is meaningful

`contract.py` computes `spread = 2*(ask-bid)/(ask+bid)`, i.e. `(ask-bid)/mid`.
So a 20% liquidity gate is literally `spread >= 0.20`; no new maths is needed.

`value` is `ticker.marketPrice()`, which prefers the **midpoint**, then falls
back to last, then close — silently. But `spread` is `NaN` **exactly when** bid
or ask is missing, which is also exactly when `value` took that fallback.
Therefore:

- `spread` is `NaN` → **no true mid**; the price is a last/close artefact
- `spread` is large → quoted, but too wide to trade

Two distinct exclusions, distinguishable with one column. Report them separately.

## compute_spread_risk_reward returns (5.14.3)

`short_delta`, `long_delta`, `short_spread`, `long_spread`, `mktdata_type` — on
top of the pre-existing strikes/premium/risk/reward/probability/edge/EV fields.
The leg deltas were always read from `opt_df` but only `prob_success_delta` was
emitted, so **no caller could filter on a leg's delta**. Returned unfiltered by
design: the caller sets its own threshold and can report what it dropped.

Also fixed there: `force_refresh` now reaches `getOptValue`, not just
`getOptionStrikes`. Spread legs were priced from the parquet quote cache (TTL
`CONFIG['quotes_ttl_minutes']`, default 30 min) even when the caller asked for a
refresh. **A half-hour-old bid/ask cannot support a liquidity decision** — force
refresh whenever liquidity is the question.

## Consequence: results stop being reproducible outside RTH

Verified live 2026-08-27 05:29 CEST, US market closed, SPY 20260916 put
verticals width 5:

| `market_data_type` | `mktdata_type` | worst-leg spread | usable |
|---|---|---|---|
| 2 (frozen) | 2 | 0.49% – 15.5% | 144/144 |
| 1 (live) | 1 | all `NaN` | **0/144** |

Both returned the same 144 rows. The frozen run looks fine and is not tradable
information. Anything gated on live quotes correctly yields **nothing** outside
market hours — that is the honest answer, not a bug, and the UI must say so
rather than showing plausible numbers. It also means such a feature cannot be
fully tested outside RTH.

Consumed by `Tuser/spread` (top-10-by-EV with a configurable bid-ask threshold,
default 20%), which therefore **requires Tdata >= 5.14.3**. Note
`compute_spread_risk_reward` still takes `trading_class` as a required
positional with no auto-resolution — see [[reference_ibkr_symbol_with_space]].
The runtime parquet cache it writes lands in the launch CWD as `quotes/`, now
gitignored in every repo that can produce one.
