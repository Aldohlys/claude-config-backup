---
name: reference_yahoo_stoxx50e_close_mismatch
description: Yahoo ^STOXX50E daily change can differ from the user's quote (Fri 2026-10-02 Yahoo +1.02% vs user +1.21%), unresolved which close is off
metadata:
  type: reference
---

2026-10-02 (Fri): Yahoo bars in intermarket_cache give Thu close 6,175.45, Fri close 6,238.5 → +1.02%. User quoted +1.21% (needs Thu ~6,164 or Fri ~6,250). Not resolved which source's close differs.

How to apply: when the user disputes a 1D index move in macro_context, pull the last bars from `intermarket_cache` (`SELECT date,Open,High,Low,Close FROM intermarket_cache WHERE ticker=? AND cache_date=(SELECT MAX(cache_date) ...)`, date stored as days since 1970) and compare closes before assuming a code bug. Related: [[feedback_macro_context_layout]], [[reference_yahoo_fx_calendar_shift]].
