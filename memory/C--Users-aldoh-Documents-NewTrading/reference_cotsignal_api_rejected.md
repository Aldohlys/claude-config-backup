---
name: reference_cotsignal_api_rejected
description: "cotsignal.com JSON API — re-evaluated 2026-10-02: 36 contracts incl. DXY, legacy categories only, Williams index over 1/3/5y; user values its long/short/net × 1/3/5y + WoW view"
metadata:
  node_type: memory
  type: reference
  originSessionId: b21f6538-9f8c-45e5-a7da-87859978dacb
  modified: 2026-10-02T07:04:48.502Z
---

**Endpoints.** Open, unauthenticated JSON API, found in the JS bundle at `www.cotsignal.com/assets/index-*.js`:

```
GET https://api.cotsignal.com/api/cot/raw-history/{CONTRACT}?lookback_weeks=52|156|260
GET https://api.cotsignal.com/api/cot/heatmap-landing      # 36 rows, latest index + WoW per group
```

**Data.** Weekly rows from 2020-01-07, 351 weeks as of 2026-09-22. Each row has open interest plus long/short/net for comm, large_spec and small_spec. Open interest matches CFTC exactly.

**Contracts (36).** Equity indices, 2/5/10/30Y Treasuries, `$ DOLLAR INDEX` (added since the 2026-09-03 check, when DXY was missing), 6 currencies, gold, silver, copper, platinum, palladium, WTI, Brent, natural gas, RBOB, bitcoin, corn, soybeans, wheat, soy oil, soy meal, cocoa, coffee, sugar, cotton, orange juice, lumber.

**Index calculation.** The `*_index` fields are the Williams COT index, `100*(cur-min)/(max-min)` over the lookback window. They are min-max values, not percentile ranks: copper 52w net = 96.74 from Williams, 98.1 as a percentile rank. Long and short indices are not served; the site computes them client-side from the raw legs. WoW = difference between consecutive rows.

**History.**
- 2026-09-03: rejected, for legacy categories only and no DXY.
- 2026-10-02: user re-raised it, because the long/short/net × 1/3/5y + WoW view is valuable.
  - The DXY objection is gone.
  - The legacy objection stands: large spec = managed money + other reportables, e.g. copper 90.5k against MM 82.5k ([[reference_cot_trader_categories]]).

**Worked example of why the legs matter.** Copper on 2026-09-22: large-spec net at Williams 98 on the 3y window, but long only 68 and short 12. The extreme net came mostly from shorts leaving, not from new longs.

**Decision (2026-10-02): the user chose to build the view from CFTC.** `refresh_cot.R` now writes `NewTrading/Reports/cot_positioning_latest.csv`: 38 markets × 5 groups, with long/short/net, WoW and the Williams index over 52/156/260 rows. The macro_context report shows it as section 11. Its `[legacy] large spec` rows match COTSignal exactly on 14 markets (net, open interest, all three indices). Use the API only as a cross-check.

Ceresna's quoted COT percentiles match the 3y large-spec index (nat gas 3, cocoa 8.5, lumber 5.8, sugar 86.1). He reads COTSignal's 3-year view.
