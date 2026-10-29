---
name: reference_macro_report_empty_close_fill
description: "At 09:00 Yahoo has no previous-day close for futures (=F) and European listings; macro report fills them from the =F's own contract series, else IBKR — never IBKR's continuous future"
metadata:
  node_type: memory
  type: reference
  originSessionId: b1df7bc4-c85e-4ae6-818f-337366774300
  modified: 2026-10-22T15:32:49.726Z
---

The 09:00 macro_context run (RStudies `reports/macro_context/intermarket.R`) regularly meets
an empty or missing previous-session close on Yahoo. On 2026-10-22: 30 symbols for 21 Oct
(6 futures `=F`, 24 European stocks/ETFs). European closes appeared ~10:45; BZ=F and GLTL.L
were still empty at 10:48. A one-shot re-fetch at 09:00 cannot fix it.

Fix (RStudies `653c399`):
- `fill_futures_closes()`: an `=F` close comes from the contract the series tracked that day
  (`fut_front_ym()`), Yahoo per-contract series first (e.g. `BZZ26.NYM`), else IBKR daily bars
  chosen by `contractMonth` (`ibkr_bars.py`). **Never IBKR's ContFuture**: it had PL on Jan
  while PL=F was on Oct (1635.9 vs 1625.5) and HO on Dec while HO=F was on Nov.
  Coffee and lumber were added to `FILL_ROOTS` (KC .NYB, LBR .CME; roll dates are from the
  exchange rules, not yet seen on Yahoo).
- `fill_stock_closes()`: IBKR **all-session** daily bar (`useRTH=False`): the regular-hours
  bar misses the European closing auction (Nestle 83.43 vs 83.11). Used only when IBKR's
  previous close agrees with Yahoo's within 0.25%; that rejected TotalEnergies (a late
  off-exchange print: 79.75 vs official 80.90) and NUCL resolved to the wrong stock (fixed
  by passing `Tickers.Exchange` as primaryExchange). Primary exchange instead of SMART
  returns no bars for continental listings. Validated: 21 IBKR closes matched Yahoo's later
  official close to 0.004%.
- Yahoo's `=F` continuous series can also be WRONG when present: KC=F showed 316.65 for
  21 Oct at 10:28, the contract KCZ26 closed 316.15.
- Stooq returns 404 for futures; not a source.

Banner "CLOSES FROM OTHER SOURCES" names each filled close and its source. Needs TWS at
09:00 for the IBKR legs. Related: [[feedback_ibkr_lasttradedate_yyyymm]], [[reference_ibkr_minor_unit_quotes]].
