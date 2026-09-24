---
name: reference_index_rebalance_volume_signature
description: "How to recognise an index-rebalance volume spike (prints the session BEFORE the effective date, flat close, decays to baseline next day) vs information-driven volume"
metadata: 
  node_type: memory
  type: reference
  originSessionId: bcd2f874-bc6c-429b-a57e-f3de0e852923
  modified: 2026-09-22T00:58:49.414Z
---

A volume spike of 10-20x normal with a near-flat close is an index cross, not information. Check
this before treating the day as a signal.

Mechanics to check:

- The rebalance trade prints at the close of the last session BEFORE the index effective date, not
  on the effective date itself. Sandoz SMI inclusion effective Monday 2026-09-21 traded on Friday
  2026-09-18.
- Exchanges schedule rebalances on the quarterly derivatives expiry (third Friday, EUREX/SIX), so
  expiry volume and rebalance volume stack on the same session.
- Price impact is near zero by construction: trackers must own a fixed weight at a fixed price, and
  the arbitrage community that accumulated since the index announcement sells into them. Any real
  move happens in the front-running days beforehand, not on the day.

The decay shape is the tell:

| | Volume vs normal |
|---|---|
| 2 sessions before | 1.5-2x (pre-positioning) |
| Rebalance session | 10-20x |
| Next session | ~1.5x, then baseline |

Information-driven volume decays slowly over days; rebalance volume returns to baseline
immediately. Worked case — SDZ.SW, normal ~0.85m shares/day:

```
2026-09-16  close 65.72   1,457,343   1.7x
2026-09-17  close 67.88   1,518,295   1.8x
2026-09-18  close 68.62  17,326,209  ~20x   range 67.92-70.44, +1.1%
2026-09-21  close 69.72   1,271,805   1.5x
```

17.3m shares was 4.0% of shares outstanding and ~CHF 1.19bn in a name doing CHF 55-60m/day.

Trading consequence: after the effective date the passive bid is spent, removing a mechanical
support. Do not read the run-up into an inclusion as a fundamental re-rating, and do not size a
breakout off a rebalance bar — the range is a cross, not a thrust. Related:
[[feedback_portfolio_drawdown_vs_index_move]], [[project_sandoz_coverage]].
