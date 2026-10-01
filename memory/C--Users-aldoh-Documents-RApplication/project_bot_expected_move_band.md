---
name: project_bot_expected_move_band
description: "em10 = 90th pct of SIGNED 10-session moves, chosen because it lands on the WINNERS' travel (0.91), not a typical move; C*ATR*sqrt(N) alone is a rescaling of atr_now; N=5 is the losing horizon"
metadata: 
  node_type: memory
  type: project
  originSessionId: 6df3dc0a-4099-43b9-b384-ce0952b42b01
  modified: 2026-09-03T18:56:45.495Z
---

## Current state (2026-09-30) — read this first

Since spec v3.23 (2026-09-29) `em10_hi/lo = atr_pct * sqrt(10) * Tickers.ATR_MoveCoefHi/Lo`,
coefficients written by BOT_monthly with `atr_expected_move(conf = 0.80)` = the **10th / 90th
percentile of SIGNED** standardised 10-session moves (not the c20/c80 of the original below).
Stored `ATR_MoveCoefHi`: median **0.96**, IQR 0.91-1.03 (326 names).

**Why the 90th percentile (user asked "it's an unlikely event, why?"; now spec §3.7 notes, v3.31):**
em10 is a target CAP and the unit of `res_pct_of_em10`, not a forecast. It is set where the
WINNING trades go (winners 2.87 ATR = 0.91 in ATR*sqrt(10) units at N=10; losers = the median of
|move|, 0.42-0.48). For a symmetric distribution the signed p90 = the |move| p80, which is why
"~p80" in the table below and 0.96 agree. So `res_pct_of_em10 = 100` = "a winner-sized move",
~10% of arbitrary windows. It is a distance ratio, NOT sessions-to-target; under sqrt-time,
sessions to the same quantile ~ 10 * (pct/100)^2. Never call it a "typical" move (fixed in v3.31).
The old "ATR_MoveCoefHi typically 1.1-1.6" was percentiles of |move| — a different quantity.

## Original design (2026-09-03)

Shipped 2026-09-03. Columns `em10_lo` / `em10_hi` in `bot_scan_universe.py`, placed next to
`px` because they size a strike. Reported, not gated.

```
em10_lo = px * (1 + c20 * atr_now/100 * sqrt(10))
em10_hi = px * (1 + c80 * atr_now/100 * sqrt(10))
```

## Why the naive version is worthless — check this before proposing it again

The user asked for `C * atr_now * sqrt(N)` at N=5. With C and N constant that is a **positive
scalar multiple of `atr_now`**: Spearman across the 75 names was **1.000000, exactly**. It
cannot rank, gate or separate anything. Worse, **0.46 * sqrt(5) = 1.029** — at N=5 the column
numerically reprints `atr_now` to within 3%.

Its only value is as a **unit conversion into the units a decision is made in** (price, for
strike selection and the debit<=33%-of-width rule). That belongs to the vehicle step against
a live chain, which `bot_scan_universe.py:68-74` says explicitly.

A variant that escapes the rescaling — `n_em = to_40d_high / EM(N)`, moves-to-trigger —
correlates only +0.125 with `atr_now`, but +0.895 with `to_40d_high` and +0.879 with
`rr_meas`. Mostly a repackaging. The user rejected it. (It did produce one usable read: at
N=5 on a typical move only 18 of 75 names could reach their own 40d high.)

## N=10, not 5 — the horizon evidence

Fresh run of `atr_C_horizon_invariance.py` (12 tickers, 5y): C_med 0.420 / 0.460 / 0.474 /
0.491 and C_p90 1.179 / 1.223 / 1.257 / 1.278 at N = 1 / 5 / 10 / 20. N=5 is the
**best-conditioned** horizon (sd 0.028). But winners travel 2.87 ATR, so:

| N | C needed | vs measured | Observed WR |
|---|---|---|---|
| 5 | **1.284** | above C_p90 1.223 | **32%** |
| 10 | 0.908 | ~p80 | 57% |
| 15 | 0.741 | ~p70 | 63% |

And `C_med` **is the losing case** — the book measures losers realising C = 0.42-0.48. An
expected-move column on C_med at N=5 displays the losing outcome at the losing horizon.

## Signed, per-class coefficients (`em_class` column)

Symmetric bands are wrong; book 9e's asymmetry had never been consumed anywhere until this.

| em_class | c20 | c80 | asym80 | source |
|---|---|---|---|---|
| US-stk | -0.44 | +0.75 | 1.69 | book 9e |
| ETF-eq | -0.46 | +0.82 | 1.78 | book 9e |
| ETF-gold | **-0.46** | **+0.89** | **1.93** | measured 2026-09-03 |
| ETF-crude | -0.54 | +0.68 | 1.28 | measured 2026-09-03 |
| FX | -0.48 | +0.47 | 0.98 | book 9e |

`atr_move_quantiles.py` pools GLD+SLV+USO into one `ETF-cmdty` class; that is wrong — gold
reaches c80 +0.89 where crude reaches +0.68, and the pooled class lands at +0.80 fitting
neither. GLD and IAU agree to two decimals independently. **Gold is the most skewed thing in
the universe at 1.93**, so a symmetric band is more wrong on bullion than on anything else.

Blank `em_class` = no band, rendered `-`, never an equity coefficient borrowed onto crude or
bullion. Still blank: the 9 outright futures and TLT (uncalibrated; the script's `Future`
class is a single ticker, too thin to adopt).

C is horizon-invariant, so using 9e's N=20 coefficients at N=10 is legitimate — that is what
invariance means. See [[project_bot_three_class_framework]], [[reference_bot_tradable_universe_csv]].
