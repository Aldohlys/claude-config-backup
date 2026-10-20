---
name: reference_yfinance_earnings_dates
description: "yfinance get_earnings_dates needs lxml, which R's r-reticulate env lacks → macro_context calls conda python via system2; limit=100 reaches 2002, has dupes, 16:00 = after close"
metadata:
  node_type: memory
  type: reference
  originSessionId: 61f1c2d1-088a-42b7-bdf0-d5e79120cc6a
  modified: 2026-10-20T20:56:50.104Z
---

**yfinance `Ticker.get_earnings_dates(limit=N)`** (checked 2026-10-20, yfinance 1.2.0)
- Requires `lxml`.
- R's reticulate binds to `AppData/Local/r-miniconda/envs/r-reticulate`, which has no lxml, so the call fails there with "Missing optional dependency 'lxml'".
- `C:/Users/aldoh/miniconda3/python.exe` has lxml. macro_context `event_history.R::mag7_dates()` therefore writes a temp .py file and runs it through `system2`. The env was left as is.
- `limit=100` goes back to about 2002 for NVDA/AAPL/MSFT/AMZN; META from 2012, TSLA from 2010.
- Some dates are duplicated: dedupe by date.
- The index hour is ET. 16:00 means after the close, so the reaction is the next session.
- Future rows can carry a 15:00 placeholder and may be estimates until the company confirms.

**FOMC calendar scraping** (federalreserve.gov)
- 2008–2020: the `fomchistorical{year}.htm` pages give one `<h5>` per meeting or call. Some meetings span two months ("July 31-August 1"). There are also "(unscheduled)" and "(cancelled)" entries.
- 2021+: `fomccalendars.htm` lists "Apr/May" months and "(notation vote)" rows. Skip the notation votes.
- Parser: `NewTrading/scripts/macro_forces/event_study.py::fetch_fomc`.

Related: [[project_macro_events_study]], [[reference_python_conda_launch]].
