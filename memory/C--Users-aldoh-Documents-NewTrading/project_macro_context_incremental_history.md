---
name: project_macro_context_incremental_history
description: "macro_context re-downloads 450 days × 398 symbols daily; if Yahoo gets unstable (throttling, failures, slow runs), propose incremental history"
metadata:
  node_type: memory
  type: project
  originSessionId: 9768c881-e087-47ba-8fe2-7db428bfa064
  modified: 2026-10-07T23:47:22.225Z
---

macro_context's intermarket fetch pulls 450 days of history for 398 symbols every morning, about 124k rows (`intermarket_cache` keeps only the current day). The breadth step separately pulls 80 days for the 503 S&P 500 members. 84 symbols serve the panels, drivers and fingerprints. 314 are sector-map group members, read as equal-weight indices since 10-05 so the map and BOT's S3 / group rank compare against the same peers. That design stays. Full run was 3.5 min on 2026-10-07, so there is no problem today.

**Why:** the user decided on 2026-10-08 to leave the fetch as is while it is stable.

**How to apply:** if Yahoo becomes unstable (rate limits, failed or partial fetches, "Intermarket: no data for ..." messages, run time growing), propose incremental history. That means a persistent price table, fetching only the last few sessions per symbol, and re-fetching a symbol's full history when its adjusted closes change (splits, dividends). A second step would share one price store between breadth and the sector map. Related: [[project_intermarket_macro_scenarios]].
