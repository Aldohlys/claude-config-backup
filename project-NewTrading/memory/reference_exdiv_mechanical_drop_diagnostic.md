---
name: reference-exdiv-mechanical-drop-diagnostic
description: How to tell ex-div mechanical drop from real selloff (Boursorama / TradingView / Yahoo)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4e0cd598-88e0-49a6-9c57-1e03df76a151
---

# Ex-dividend mechanical drop vs real selloff — diagnostic

When a European dividend stock prints what looks like a large gap-down, the question is always: how much is dividend detachment vs real selling? Three independent cross-checks, fastest first.

## 1. Boursorama "previous close" (fastest, definitive)

Boursorama (and Euronext's official quote feed) **already re-base "previous close" to the ex-div adjusted reference** on the ex-day. So:

- If displayed `% change ≈ 0` and `previous close` is **lower** than yesterday's actual close → 100% mechanical, no real selling.
- If displayed `% change` is materially negative on top of the rebased reference → that's the real selling component.

Worked example (Carrefour 2026-05-26, dividend €1.18):
- Friday cum-div close: €17.28
- Theoretical ex-open: €17.28 − €1.18 = €16.10
- Boursorama "previous close": **€16.145** (already adjusted)
- Boursorama intraday: €16.17 = +0.15%
- Conclusion: zero residual selling.

## 2. TradingView toggle

Chart Settings (gear, bottom-right) → **Symbol** tab → **Adjust data for dividends**. With it ON, the gap disappears; with it OFF, you see the optical drop. Caveats:
- Per-chart, not global (set in defaults to make sticky).
- Exceptional / special dividends are sometimes not classified for auto-adjustment by the data vendor — if a residual step remains after toggling ON that matches the exceptional component, that's the toggle's blind spot, not real selling.

## 3. Yahoo Adj Close column

`Adj Close` recomputes the historical series backward from each ex-date. On the ex-day, `Close − Adj Close ≈ dividend` for prior rows. Useful when Boursorama/TradingView aren't available, or as a third confirmation.

## When the question matters

- Position-add decisions on the ex-day — buying after detachment skips paying for a coupon you won't receive.
- Trend-following screeners running on **unadjusted** daily series will produce false breakdown signals on ex-days. Always check whether your scanner's data source is adjusted.
- Chart pattern reads — a "support break" on ex-day is fake.

## Related

- [[project_carrefour_position]] — concrete worked example, Carrefour 2026-05-26.
