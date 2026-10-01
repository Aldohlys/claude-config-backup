---
name: reference_scanner_removal_is_deactivation
description: "To drop a name from ScannerUniverse but keep its Tickers row, set IsActive = 0; a DELETE is undone by syncTickersToScanner()"
metadata:
  node_type: memory
  type: reference
  originSessionId: 1c26deb6-0a13-4dd0-99ab-91d81259daed
  modified: 2026-10-01T22:05:48.075Z
---

`RStudies/reports/shared/universe.R::get_universe()` calls `Tdata::syncTickersToScanner(max_price = 500)` (Tdata/R/ticker.R) on every load. It re-inserts, as active, every `Tickers` row with `IV = 'YES'`, `Type = 'STK'` and no `ScannerUniverse` row (role `etf` for names matching its ETF patterns, e.g. VXX, else `scanner`).

So removing a name from the scanner while keeping its Tickers row (trades, holdings) must be `UPDATE ScannerUniverse SET IsActive = 0, Notes = '... [deactivated <date>: reason]'` — the convention already used for LLY, NOC, CA.PA. A DELETE only sticks when the Tickers row goes too, or when Tickers.IV = 'NO'.

Seen 2026-10-02: the cluster review deleted DBA, OR, NLR, VXX from ScannerUniverse; the first get_universe() brought them back as Ungrouped. Fixed by deactivation. Related: [[project_scanner_correlation_groups]].
