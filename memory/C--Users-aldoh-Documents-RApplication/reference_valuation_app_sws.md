---
name: reference_valuation_app_sws
description: "Valuation calculator (Tuser/valuation, 00_valuation.bat) mirrors Simply Wall St's six checks; GitHub model is outdated; their pages are Cloudflare-blocked, read their news articles; Excess Returns formula verified on JPM; cost-of-equity gap explains score differences"
metadata:
  node_type: memory
  type: reference
  originSessionId: 1c22afef-2f88-43f7-b806-e8014e3af784
  modified: 2026-10-06T15:07:11.240Z
---

Built 2026-10-06. `Tuser/valuation/app.R` + `logic/valuation.R` (calcs) + `logic/fetch_valuation.py`
(yfinance, called via reticulate `import_from_path`, no Tdata/TWS). Launcher `scripts/00_valuation.bat`.
Test: `Rscript tests/test_valuation.R` (from Tuser).

**Simply Wall St's current six valuation checks** (not the ones in their GitHub
Company-Analysis-Model, which still lists P/E vs market, PEG, P/B — outdated):
price ≥20% below and ≥40% below the cash-flow value (2-stage FCFE DCF; Excess Returns for
banks/insurers), P/E vs peers, P/E vs industry, P/E vs Fair Ratio, analyst mean target >20%
above price with analysts in agreement. Loss-makers get P/S instead of P/E.
"X% below fair value" means 1 − price/value.

**Verified from their published JPM figures (2026-06-21):** Excess Returns =
book + (ROE_fwd − r) × stable book / (r − g), perpetuity not discounted further; g = 5y mean
of the US 10y (3.79%). 128.38 + 13.00/(0.0802−0.0379) = 435.6.

**Known gaps the app can't close:** peer P/E, industry P/E and Fair Ratio are typed in from
their site (the app prefills the US sector ETF P/E — XLK/XLF run far above their industry
averages: 35.6 vs 23.9, 15.3 vs 11.9). Their cost of equity uses an industry beta and an
unpublished ERP (JPM 8.02% vs app 8.41% with Yahoo beta × 4.5%), which flips JPM's 20% check
and makes the app 1/6 against their 2/6.

**Access:** simplywall.st returns a 403 (Cloudflare challenge) to WebFetch and curl. Their
`/news/...` article pages and WebSearch snippets do come through, and carry the DCF/ER inputs,
P/E, peer/industry averages and Fair Ratio. `wall.st` is a dead Apache test page, not Simply Wall St.

**Yahoo FCF trap:** `info["freeCashflow"]` is a levered measure (AAPL 107.7b) ≠
operating CF − capex (136.7b from `quarterly_cashflow` "Free Cash Flow", last 4 quarters),
which is what Simply Wall St uses. Using the info field understated AAPL's DCF by ~20%.
See also [[reference_yahoo_ticker_and_oi_traps]], [[reference_ibkr_data_entitlements]].

**Share-class / ADR basis traps in Yahoo (2026-10-06).** EPS, DPS and trailing P/E are on the
quoted line's basis, but `sharesOutstanding` counts that line only (GOOG 5.5bn of 12.2bn; UHR
28.9m of 52.0m bearer-equivalents; NVO ADR), `bookValue` is copied across classes (UHR = UHRN;
BRK-B shows BRK-A's 522k), and `forwardEps` can be another class's (UHRN 19.94 at price 35).
The fetcher derives a line-basis share count (net income / EPS when |EPS| >= 0.5, else
`impliedSharesOutstanding`; each fails once: UHRN rounding, LISN market cap) and divides
balance-sheet equity and FCF by it; it drops forward EPS when the forward P/E is outside 3-300,
and warns after a split (<= 30 days, price/EPS vs trailingPE off > 10%, target/price outside 1/3-3).
