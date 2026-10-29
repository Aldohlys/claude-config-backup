---
name: feedback_macro_context_layout
description: macro_context HTML layout wanted by user - last-session index strip at top of 00, every section collapsible (only 00 open since 2026-10-27), each 09 panel collapsed; label return windows up front
metadata:
  type: feedback
---

Layout set 2026-10-05 (RStudies 4c843b1), keep it when changing the report:
- Section 00 starts with a "Last session" strip: 1-day change from previous close per index (US, Europe, Asia, LatAm, EM), each with its bar date (markets close on different days; China holidays show an older date). `daily_strip_html()` + `DAILY_STRIP` in intermarket_render.R; `c1d` in `series_metrics()`.
- All sections are `<details>`; only 00 open on load (user 2026-10-27: "all panels closed except panel 0"; was 00 + 09, and Detail/Legend sections were open). Section 11 COT overview rows need visible separators. Inside 09 every panel is its own collapsed `<details>`. Panel tables have a 1D column.
- Any prose with returns states the window first ("Over 1 month, local currency. Europe: ...").

**Why:** user read the World markets line (1M, label at the end of the sentence) as daily: "Euro Stoxx 50 -2.3% although Friday's close was up 1.21%". The page was also too long to scan.

**How to apply:** new sections in template.html use `<details class="section">` + `<summary class="stitle">`, closed by default. Never put a return figure in prose without its window before the numbers. Related: [[project_intermarket_macro_scenarios]].
