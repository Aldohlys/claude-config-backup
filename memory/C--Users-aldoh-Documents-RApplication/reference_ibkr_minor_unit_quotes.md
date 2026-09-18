---
name: reference_ibkr_minor_unit_quotes
description: "IBKR quotes LSE stocks in PENCE while reporting currency 'GBP' — order prices can be 100x the scale of the stored position; and never clamp a risk metric to zero"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1e2ceadf-edaa-4f9c-82d1-d1e2a412227d
  modified: 2026-09-03T09:52:26.408Z
---

**IBKR reports LSE stocks with `currency = "GBP"` but prices in pence (GBX).** A live order on such a name comes back on a scale 100x the position it protects, and the DB stores pounds.

Measured 2026-09-03 on CRST (Crest Nicholson), 400 shares in U25343478:

| source | value | unit |
|---|---|---|
| live order | STP `45.0`, LMT `225.0`, `currency='GBP'` | pence |
| portfolio snapshot | `mktPrice 0.5403`, `avgCost 1.0381` | pounds |
| `Trades.Price` | `1.03` | pounds |

So `(1.03 - 45.0) * 400` is negative, and `scripts/sync_stop_risk.R` — which did `risk <- max(0, ...)` — wrote **Risk = 0**: *"this position cannot lose money"*, on a position with £232 of exposure to that stop. The true figure is `(1.03 - 0.45) * 400 = 232`.

**How to reconcile, without hardcoding an exchange or currency list:** compare the order price against the position's own `mktPrice` from the account snapshot table — by construction the same scale as `Trades.Price`. A ratio near 100 is the pence convention (rescale, and say so in the output); a ratio outside a plausible band is refused rather than guessed. `rescale_stop()` in `sync_stop_risk.R` is the implementation. EUR/JPY/KRW/CAD names in the same book are unaffected — CA's order and position are both EUR.

**The wider lesson, which is the reason this went unseen:** *never clamp a risk metric into its safe range.* `max(0, risk)` turned a unit mismatch into a confident, wrong, reassuring zero. A value outside its possible range means the inputs disagree — report and skip, so the number is absent rather than false. The only zero left in that code is the legitimate one: a stop through the entry, which locks in a profit. Same family as [[feedback_max_cumsum_na_rm_minus_inf]] (`na.rm` on a running total fabricates values).

Applies to anything comparing an IBKR order/quote price against a stored position: [[project_stop_based_risk_sync]], and any future stop/limit-driven sizing. Related: [[reference_ibkr_open_orders_semantics]], [[reference_gonet_foreign_etf_pricing]] (the other "IBKR gave the wrong scale/contract" trap).
