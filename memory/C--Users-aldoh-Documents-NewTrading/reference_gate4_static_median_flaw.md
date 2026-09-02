---
name: reference_gate4_static_median_flaw
description: "Gate 4 reads a static 5y median ATR, so it cannot see a vol-regime change — GDX and GIS are the worked cases; input for the gate/indicator simplification work"
metadata:
  node_type: memory
  type: reference
---

Found 2026-08-31 while screening universe candidates. Direct input to the **breakout gate/indicator simplification** project.

**The flaw.** `book_BOT` and Gate 4 both read `atr_med5y` — the median of ATR14/price over 5 years (`bot_scan_universe.py:332`, band logic at :261). It is a **static property of the last five years**, so a name whose volatility regime has changed is classified by the regime it has left. The scan already computes `atr_now` and `atr_pctile` per row, and the universe CSV already carries `atr_p25`/`atr_p75` — none of them reach the gate. They are display-only.

**Case 1 — GDX vs GDXJ: the same trade, opposite verdicts.**
GDX `atr_med5y` 2.77 → veto band → not scanned. GDXJ 3.09 → RICH → scanned. They are both gold miners, same direction, same driver, separated by 0.32 of a median across a 3.00 cutoff. On the evidence the split is backwards: GDX is positive in **all four** extension buckets (54.5-58.7% P(+2 ATR before -2 ATR), median MFE 2.6-3.3 ATR) while GDXJ has a negative bucket and falls to 48.2% when extended — and GDX has 3.2x the liquidity (ADV 1,393 vs 432 CHF m). See [[reference_bot_name_continuation_character]].

**Case 2 — GIS: the gate reading a company that no longer exists.**
GIS fell 68 -> 31 (Sep-2024 to Jun-2026) and re-rated into a much higher vol regime:

| GIS ATR14/price | |
|---|---:|
| 5y median (what the gate reads) | 1.90 |
| last 252 sessions | 2.50 |
| last 63 sessions | 2.97 |
| today (75th pctile of its own 2y) | 2.75 |

Scanned 2026-08-31 it prints **S 5 / BK 3, both F1 and F2 firing, rs20 +15.8, rs60 +25.2, rng_pct 98, adx10 29** — the strongest sub-scores in the 75-name scan — and comes out **VETO**, on a median measuring the pre-crash staple.

**Candidate fixes** (none implemented; backtest against the full universe before adopting):
- Gate on `max(atr_med5y, atr_med252)` — cheapest change, catches both cases.
- Gate on `atr_p75` instead of the median — GDX's p75 is 3.48, GIS's 2.18.
- Keep the median but raise a `regime_shift` flag when `|atr_med252 - atr_med5y| > ~0.5`, and let a flagged row bypass the veto.

**The deeper question for the simplification work.** The 1.7-3% veto was calibrated on **realized BOT trade P&L** (~32% win rate, net negative — [[reference_atr_move_multiples]]). That statistic bundles vehicle choice and premium cost together with direction. But GDX and GIS both *continue* fine directionally. So the mid-ATR veto may be measuring **an option-affordability problem masquerading as an entry gate** — mid-ATR names move enough to be right and not enough to pay for the call. If so the fix is not a better ATR gate but moving the test to the vehicle step, which is where [[project_bot_universe_scanner]] already says option-level judgements belong.

Related: [[feedback_price_action_rr_gate]], [[reference_first_touch_barrier_test]], [[feedback_no_static_option_judgements_in_screens]].
