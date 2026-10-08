---
name: reference_yahoo_futures_contracts
description: "Yahoo serves listed futures months (CLX26.NYM, BZZ26.NYM...) with ~17 months of history but no expired contracts; =F series roll the day after the last trading day, so they carry roll gaps"
metadata:
  node_type: memory
  type: reference
  originSessionId: 699892b4-1a21-4a3f-a5a9-7224c9a599df
  modified: 2026-10-08T21:37:28.456Z
---

Yahoo Finance quotes individual futures months as `<root><month code><yy>.NYM` (CL, BZ, NG, HO, RB; e.g. `CLG27.NYM`). Checked 2026-10-08:
- Each still-listed contract has about 360 daily bars, back to 2025-05-06, even far months.
- Expired contracts return nothing (CLV26, CLU26, CLZ25 all empty), so a fully back-adjusted history cannot be rebuilt from Yahoo.
- The continuous `=F` series switch to the next contract the first trading day after the last trading day: CL=F on 09-23, NG=F on 09-29, BZ=F / HO=F / RB=F on 10-01 (2026). In backwardation each roll reads as a fall (CL=F 1M −5.1% vs its contract −1.8% to 10-08), in contango as a rise (NG).
- Today's bar of a =F series can sit on the wrong contract (RB=F on 10-08 equalled Dec, not Nov).

Last-trading-day rules that matched those dates (holidays ignored): WTI 3 business days before the 25th of the month before delivery (4 if the 25th is not a business day); Brent the last business day of the second month before delivery; Henry Hub 3 business days before the delivery month; ULSD and RBOB the last business day of the month before delivery.

**How to apply:** for any futures analysis on Yahoo data, read a fixed contract or a roll-free series, never a raw `=F` 1M change across an expiry. macro_context already does this for the five energy roots (`roll_adjust()`, `FUT_LTD` in RStudies `intermarket_config.R`); metals, grains and softs still use `=F`. Related: [[feedback_verify_futures_contract_month]], [[reference_futures_expiry_week_dynamics]], [[project_intermarket_macro_scenarios]].
