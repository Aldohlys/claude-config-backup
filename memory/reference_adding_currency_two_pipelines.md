---
name: reference_adding_currency_two_pipelines
description: Activating a currency in the Currencies table auto-enrolls FX conversion but NOT interest rates (needs a fetcher)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 672ba0ce-0bbd-4810-bd8c-9c6150cb270e
---

Adding/activating a currency in the `Currencies` table (`Active='Yes'`) wires it into the two refresh pipelines **asymmetrically**:

- **FX conversion (ConvertToCHF / ConvertToUSD): automatic.** `getIBKRActiveCurrencyValues()` and `getLastCHFValue()`/`getLastUSDValue()` are generic — they read active currencies and compute the cross-rate from `YahooName` × `CHFUSD=X` honoring `DirectConversion`. No code change needed; it self-maintains daily. (KRW worked the moment its row had `YahooName=KRW=X`, `DirectConversion="No"` → `chf_value = 1/(USDKRW·CHFUSD)`.)
- **Interest rates: needs a per-currency fetcher.** `getInterestRates()` (`Tdata/R/interest_rate_utils.R`) has a `fetchers` registry; a currency with no `get_<ccy>_rates()` is silently skipped → its `ir*` columns stay NA. Add `get_<ccy>_rates()` (return one row: Name, last_ir_update, ir1week..ir2years in **percent**) and register it.

Fetcher coverage after Tdata 5.11.0 (2026-06-23): USD/EUR/CHF/JPY/CAD/GBP/KRW. Only inactive HKD lacks one (by design). Sources: USD=FRED, EUR=EURIBOR scrape+MarketWatch, CHF=SARON+hardcoded gilts, JPY=MOF JGB+FRED TIBOR, CAD=BoC Valet, GBP=FRED SONIA `IUDSOIA`+gilt fallbacks, KRW=FRED `IR3TIB01KRM156N`+KTB fallbacks.

Symptom that flags the gap: Python `tdata_py.account._get_chf_rates()` logs `No ConvertToCHF rate for <ccy>; defaulting to 1.0 (sum will be inaccurate)` — that's a missing-DATA warning (currency activated but pipeline never ran), not a code bug. See [[project_trades_cash_identification]].
