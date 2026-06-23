---
name: reference_tdata_ir_refresh_system
description: "How Tdata interest rates are stored, fetched per-currency, dispatched off active currencies, and the Bank of Canada Valet series IDs"
metadata: 
  node_type: memory
  type: reference
  originSessionId: c63ff519-1cac-4027-a4b5-d2730bebb486
---

Interest rates live in the DB **`Currencies`** table, one row per currency, columns `ir1week/ir1month/ir3months/ir6months/ir1year/ir2years` + `last_ir_update` (YYYYMMDD int), stored in **percent**. `Tdata::getLastRate(currency, DTE)` reads that row, returns **decimal** (÷100), warns if `last_ir_update` > 7 days old, and buckets by DTE: `<18`→1week, `<60`→1month, `<135`→3months, `<270`→6months, `<545`→1year, else 2years. See [[reference_tdata_interest_rate_utils]].

`Tdata::getInterestRates(update_db=TRUE, currencies=NULL)` (in `Tdata/R/interest_rate_utils.R`) was refactored 2026-06-05 to a **per-currency fetcher registry driven by `getActiveCurrencies()`** (Currencies.Active='Yes'). A currency with no fetcher is skipped. Fetchers: USD=FRED (DTB3/DTB6/DGS1/DGS2 via quantmod), EUR=EURIBOR scrape, CHF=SARON/SNB (NASDAQ key), JPY=MOF JGB CSV + FRED TIBOR, **CAD=Bank of Canada Valet API**. **GBP/HKD have no fetcher** (active but skipped). Daily refresh: Windows Task `\RApplication\RefreshInterestRates` (08:00) → `RApplication/scripts/refresh_interest_rates.R`.

**Bank of Canada Valet series IDs** (no API key; endpoint `https://www.bankofcanada.ca/valet/observations/<id,id,...>/json?recent=N`, observations returned **newest-first** so pick by max date, not list position): CORRA overnight = `AVG.INTWO`; GoC T-bill yields 1m=`V80691342` 3m=`V80691344` 6m=`V80691345` 1y=`V80691346`; 2y benchmark bond = `BD.CDN.2YR.DQ.YLD` (group `bond_yields_benchmark` also works). The rendered pages `/rates/interest-rates/t-bill-yields/` and `/corra/` are the human fallback.

**Deploy caveat:** Tdata's `quick_build` is broken — deploy via direct `R CMD INSTALL` ([[project_build_package_renv_prune_todo]], [[reference_tdata_install_topology]]); reticulate caches the package per R session so a fresh process is needed to see changes.
