---
name: reference_cotsignal_api_rejected
description: cotsignal.com has an open JSON API but serves legacy categories and no DXY — evaluated 2026-09-03 and rejected as a COT source
metadata: 
  node_type: memory
  type: reference
  originSessionId: eceac09b-8ed5-4491-b373-20a43b99833d
  modified: 2026-09-03T10:35:43.963Z
---

Evaluated 2026-09-03 as an alternative to pulling CFTC directly. **Rejected — do not re-investigate.**

The site is a React SPA (empty `<div id="root">`), so there is nothing to scrape from the HTML. It does have an open, unauthenticated JSON API, found in the JS bundle:

```
GET https://api.cotsignal.com/api/cot/raw-history/{CONTRACT}   # 347 weekly rows from 2020-01-07
GET https://api.cotsignal.com/api/cot/heatmap-landing
GET https://api.cotsignal.com/api/price/{ticker}
```

34 contracts (commodities, rates, equity indices, currency pairs). Open interest matches CFTC exactly, so it is the same underlying data, cleanly served.

**Two reasons it does not fit:**

1. **Legacy categories only** — `comm` / `large_spec` / `small_spec`. Its own tooltip calls large spec "hedge funds & CTAs — managed money", but the numbers are legacy Non-Commercial = Managed Money + Other Reportables (see [[reference_cot_trader_categories]]). Adopting it would silently swap the metric while the file still said "MM net".
2. **No dollar index.** `USD INDEX` and `DOLLAR INDEX` both 404; DXY is absent from the contract list.

History depth (6.7y) would have been fine for a 5-year percentile.

**Worth remembering:** its `large_spec_index` is the Williams COT Index = `100 * (cur - 52w min) / (52w max - 52w min)`, reproduced exactly from its own series. If a more responsive extreme trigger is ever wanted than the 5-year percentile rule in `refresh_cot.R`, that is three lines against data already pulled — no dependency needed. On 2026-08-25 both approaches agreed (Copper and Corn at 100).

Also: it is a private undocumented endpoint with no robots.txt and no reviewed terms, for data the government publishes directly and in more detail.
