---
name: reference_bot_tradable_universe_csv
description: "tradable_universe CSV is the BOT scanner's config surface — refresh_atr re-derives book_BOT and silently overwrites hand-set values; new columns survive the round-trip"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6df3dc0a-4099-43b9-b384-ce0952b42b01
  modified: 2026-09-03T18:56:05.343Z
---

`Strategies/tradable_universe_20260827.csv` (`;`-delimited) drives
`Strategies/Breakouts/bot_scan_universe.py`. It is **the** config surface — edit it
directly rather than adding code branches. Distinct from the swing scanner's DB-backed
ScannerUniverse ([[project_scanner_universe]]); they share no rows.

## The overwrite trap

`refresh_atr()` (run via `--refresh-atr`) **re-derives `book_BOT` from the rule** for every
row and rewrites the whole file:

```python
liq = yahoo.contains('=') | ((adv_chf_m >= 85) & (oi >= 5000))
book_BOT = 'Y' if atr_med5y >= 3.0 and liq else 'catalyst' if atr_med5y < 1.7 and liq else ''
```

So a **hand-set `book_BOT` is silently reverted on the next refresh.** Found 2026-09-03:
GIS was carrying `Y` with `atr_med5y` 1.90 — neither >= 3.0 nor < 1.7, so the rule yields
blank. It had been scanned for months on a stale value (always VETO, so never actionable).
To include a name, move the *rule*, not the cell.

It only recomputes `atr_med5y / p25 / p75 / n_days / atr_bucket / book_BOT`. Everything
else — `px_*`, `hv*`, `adv_chf_m`, `option_oi`, `atr_now`, `er20_*` — is left at whatever
the last universe build wrote. **The build script that populates those is not in the repo.**
Formulas that reproduce it (validated against EQT's stored row to 3 decimals on `er20_med`,
exactly on `n_days`): HV = annualised sd of log returns; `er20` = Kaufman efficiency ratio
`|C_t - C_t-20| / sum|dC|`; `adv_chf_m` = 60-day median of Close x Volume, x USDCHF.

## Columns added 2026-09-03

- **`bench`** — see [[project_bot_industry_benchmarks]]
- **`em_class`** — see [[project_bot_expected_move_band]]
- **`atr_window_y`** — truncates the ATR history for one name. Justified ONLY by a corporate
  event that makes the earlier series a different company (EXE: CHK+SWN merger Oct-2024,
  window 2). Never to make a name qualify — see [[feedback_shortened_lookback_fits_regime]].

`refresh_atr` reads with `pd.read_csv` and writes with `U.to_csv(...)`, so it round-trips the
whole DataFrame — **new columns survive `--refresh-atr` untouched.** Verified.

## Liquidity floor

`adv_chf_m >= 85` (was 100 until 2026-09-03). Sweeping it: 85 admits RRC (90), NIO (97) and
SIL (89) and **no lower floor admits anything further** — every remaining name fails on
option OI or the Gate 4 ATR band, not on turnover. The scanned set already carried TOL at
105 and URA at 112, so 100 was excluding names thinner than nothing it kept out. At this
book's per-trade size, OI and spread bind; tape does not.

Two data defects still open: **EQT and MOS share the identical `option_oi` string
`102742@2026-09-18`** — one row carries the other's number (neither is near the 5000 floor,
so nothing turns on it today). And `note` fields must not contain `;`.
