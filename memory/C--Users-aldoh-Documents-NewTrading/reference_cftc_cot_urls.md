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
| DXY (US Dollar Index) | `https://www.cftc.gov/dea/futures/financial_lf.htm` (TFF) | 098662 | Leveraged Funds + Asset Manager |
| Major currency pairs (EUR, JPY, GBP etc) | `https://www.cftc.gov/dea/futures/financial_lf.htm` (TFF) | varies | Leveraged Funds + Asset Manager |

## Wrong URLs to skip (cost cycles this session)

- `deacmesf.htm` — CME-only, **does NOT include NYMEX/COMEX/ICE contracts**. WTI is NOT here.
- ~~`financial_lf.htm` for DXY~~ — **this claim was WRONG, corrected 2026-09-03.** `financial_lf.htm` (TFF long format) *does* carry `USD INDEX - ICE FUTURES U.S. Code-098662`. Use it for the Lev-Funds + Asset-Manager split. `deanybtsf.htm` also has DXY but only in **legacy** Non-Commercial/Commercial form — a different metric, not comparable to the Lev+AM series the entries use.

## Historical archive (for multi-year percentile context)

Annual zips, one row per contract per week, parse directly with pandas:

- Disaggregated futures-only (WTI, gold, copper, grains): `https://www.cftc.gov/files/dea/history/fut_disagg_txt_<YYYY>.zip` → `f_year.txt`
- Traders in Financial Futures (DXY, VIX, rates, equity index): `https://www.cftc.gov/files/dea/history/fut_fin_txt_<YYYY>.zip` → `FinFutYY.txt` (note the different inner filename)

Columns needed: `Report_Date_as_YYYY-MM-DD`, `CFTC_Contract_Market_Code` (string, strip whitespace), `M_Money_Positions_Long_All` / `_Short_All`; TFF uses `Asset_Mgr_` / `Lev_Money_` prefixes. 2021-2026 = 295 weekly observations per contract, enough for a 5-year percentile.

This is what makes `extreme` flags reproducible without the Saxo narrative — see the refresh rule in `positioning.R`. It also cross-checks the weekly change column: the 2026-08-25 corn +136k one-week jump looked like a parse artifact and was confirmed genuine against the archive.

## Column order in the disaggregated short report (petroleum_sf.htm etc.)

11 position columns: Producer L, Producer S, Swap L, Swap S, Swap Spread, **MM L, MM S**, MM Spread, Other L, Other S, Other Spread. Validate a parse by checking that the long-side columns (including spreading once) sum exactly to Open Interest.

## Report cadence

- Released every Friday at 3:30 PM ET, covering positions as of the **previous Tuesday close**.
- Today (2026-05-27 Wed): latest available is data through 2026-05-19, released 2026-05-22.
- The next Friday release (2026-05-29) will cover positions through 2026-05-26.

## Raw CFTC vs Saxo narrative

The user's `positioning.R` historically followed Ole Hansen's Saxo digest, which adds narrative context — "15-month high", "fresh longs vs short covering", "multi-decade stockpile highs". The raw CFTC numbers alone don't tell you whether 98k MM net long crude is "extreme" — you need multi-year percentile context.

**When refreshing from CFTC raw alone (updated 2026-09-03):** the historical archive above supplies the multi-year context, so the old "default everything to FALSE" workaround is obsolete. Compute the 5-year percentile of the net series and set `extreme = TRUE` at >=90th (crowded long) or <=10th (crowded short). Record the percentile in the note. The Saxo digest is now optional colour, not the source of the flag.

## How to apply

When asked to pull or refresh COT data:
1. Go straight to the right URL from the table above — don't try generic CFTC index pages first.
2. For specs/funds positioning use the disaggregated/TFF reports, not legacy Non-Commercial.
3. Note the report-as-of date prominently. Don't conflate "released Friday" with "data through Friday" — data is Tuesday-close.
4. Cross-check Managed Money long contract count against expected scale (WTI ≈ 100k-400k; gold ≈ 80k-300k; corn ≈ 200k-500k). If a parse returns single-digit thousands, the section was wrong.
