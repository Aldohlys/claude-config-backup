---
name: reference_trades_datetime_utc
description: Trades.DateTime is UTC whatever TimeZoneSource says; date-only rows are 00:00:00; cleanup-script expiry rows are 16:00 New York
metadata:
  node_type: memory
  type: reference
  originSessionId: 3f7efce0-fd27-406b-b771-59dc993e4d3e
  modified: 2026-10-08T08:58:24.718Z
---

`Trades.DateTime` is stored in **UTC** (`parse_ibkr_datetime()` in RReporting
converts the flex time). `TimeZoneSource` ("America/New_York", "UTC_migrated")
only records where the time came from — reading DateTime as New York time puts
fills 4-5 hours late (trade 752: 18:58:21 stored = 14:58 NY).

Three exceptions to keep in mind when comparing a fill time with anything else
(snapshot `heure` is Europe/Zurich local time):
- **Date-only rows** — migrated (`UTC_migrated`), manual opens/closes, dividend
  rows: `00:00:00`. They can sit on either side of that day's snapshot;
  `Tdata::attributePnL` places them by whether the snapshot position already
  includes them.
- **Expiry rows written by `scripts/cleanup_trades_*.R`**: `16:00:00` with
  `TimeZoneSource = America/New_York`, meaning the NY close — not UTC. Price 0.
  `getUnderlyingPriceAt(..., day_close = TRUE)` handles them.
- **ManualClose before 2026-10-08** copied the selected row, so its close row
  carries the *opening* fill's DateTime (fixed in RReporting server.R).

Related: [[reference_position_snapshot_uprice_unadjusted]] (the other trap of
the P&L attribution, TODO #106).
