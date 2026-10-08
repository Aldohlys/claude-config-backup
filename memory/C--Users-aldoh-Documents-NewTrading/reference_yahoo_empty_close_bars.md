---
name: reference_yahoo_empty_close_bars
description: "Yahoo can return the previous session's bar with an empty Close (10-08 09:00: 35 European/Asian symbols for 10-07); IS0L.DE / GLTL.L / NUCL.L stayed empty into the evening"
metadata:
  node_type: memory
  type: reference
  originSessionId: 699892b4-1a21-4a3f-a5a9-7224c9a599df
  modified: 2026-10-08T21:37:40.775Z
---

On 2026-10-08 at 09:00 CET, Yahoo returned the 10-07 bar with Close = NULL for 35 symbols: every European and Asian index in macro_context (Euro Stoxx 50, DAX, SMI, CAC, IBEX, FTSE MIB, Nikkei, KOSPI, HSCEI), the Bund and gilt ETFs `IS0L.DE` / `GLTL.L`, and European stocks. A re-fetch at 23:30 filled 32; `IS0L.DE`, `GLTL.L` and `NUCL.L` were still empty.

Code that drops NA rows then shows the session before as the last one, silently. Check in the DB with `SELECT ticker FROM intermarket_cache WHERE date = <last session> AND Close IS NULL` (dates are stored as days since 1970, e.g. 20733 = 2026-10-07).

**How to apply:** when a report's last-session strip shows an older date for one region than for the US, suspect empty closes before a holiday. macro_context now re-fetches once and shows an amber banner (`refetch_empty_closes()`), so a banner naming symbols means Yahoo still had no close at run time. Related: [[reference_yahoo_fx_calendar_shift]], [[reference_yahoo_stoxx50e_close_mismatch]], [[project_intermarket_macro_scenarios]].
