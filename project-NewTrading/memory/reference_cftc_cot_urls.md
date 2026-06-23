---
name: cftc-cot-urls
description: CFTC COT report URL routing by asset class — saves wrong-page fetches when refreshing positioning data
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8c65e4a9-c01b-4a86-95fe-f6292e3db0e8
---

# CFTC COT report URLs by asset class

Discovered 2026-05-27 while refreshing `positioning.R` from raw CFTC after the Saxo digest had gone 10 weeks stale.

## URL routing

| Asset | URL | CFTC code | Category to read |
|---|---|---|---|
| WTI Crude (NYMEX) | `https://www.cftc.gov/dea/futures/petroleum_sf.htm` (disaggregated) or `petroleum_lf.htm` (long-format) | 067651 | Managed Money |
| WTI Crude (legacy format) | `https://www.cftc.gov/dea/futures/deanymesf.htm` | 067651 | Non-Commercial |
| Gold, Copper, other metals | `https://www.cftc.gov/dea/futures/other_lf.htm` | Gold 088691, Copper 085692 | Managed Money |
| Corn, Soybeans, Wheat | `https://www.cftc.gov/dea/futures/ag_lf.htm` | Corn 002602, Soy 005602, SRW 001602 | Managed Money |
| DXY (US Dollar Index) | `https://www.cftc.gov/dea/futures/deanybtsf.htm` (ICE Futures U.S.) | 098662 | Leveraged Funds + Asset Manager |
| Major currency pairs (EUR, JPY, GBP etc) | `https://www.cftc.gov/dea/futures/financial_lf.htm` (TFF) | varies | Leveraged Funds + Asset Manager |

## Wrong URLs to skip (cost cycles this session)

- `deacmesf.htm` — CME-only, **does NOT include NYMEX/COMEX/ICE contracts**. WTI is NOT here.
- `financial_lf.htm` for DXY — covers CME financials (currencies, equities, rates) but **does NOT include ICE Futures U.S. contracts** like DXY.

## Report cadence

- Released every Friday at 3:30 PM ET, covering positions as of the **previous Tuesday close**.
- Today (2026-05-27 Wed): latest available is data through 2026-05-19, released 2026-05-22.
- The next Friday release (2026-05-29) will cover positions through 2026-05-26.

## Raw CFTC vs Saxo narrative

The user's `positioning.R` historically followed Ole Hansen's Saxo digest, which adds narrative context — "15-month high", "fresh longs vs short covering", "multi-decade stockpile highs". The raw CFTC numbers alone don't tell you whether 98k MM net long crude is "extreme" — you need multi-year percentile context.

**When refreshing from CFTC raw alone:**
- Set `extreme = FALSE` by default unless you can establish multi-year context.
- Note in the entry that the refresh was raw CFTC, not Saxo narrative.
- The user can lift `extreme = TRUE` flags after reading Ole's digest.

## How to apply

When asked to pull or refresh COT data:
1. Go straight to the right URL from the table above — don't try generic CFTC index pages first.
2. For specs/funds positioning use the disaggregated/TFF reports, not legacy Non-Commercial.
3. Note the report-as-of date prominently. Don't conflate "released Friday" with "data through Friday" — data is Tuesday-close.
4. Cross-check Managed Money long contract count against expected scale (WTI ≈ 100k-400k; gold ≈ 80k-300k; corn ≈ 200k-500k). If a parse returns single-digit thousands, the section was wrong.
