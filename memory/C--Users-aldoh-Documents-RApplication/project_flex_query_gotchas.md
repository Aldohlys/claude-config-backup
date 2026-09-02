---
name: project_flex_query_gotchas
description: "IBKR Flex Query traps — a Trade-only Flex silently omits transfers and corporate actions (use the Monthly Activity Statement as ground truth), and a master-level Flex returns byte-identical CSVs for every sub-account"
metadata: 
  node_type: memory
  type: project
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T02:23:53.074Z
---

Merged 2026-08-27 from two separate memories. Both found 2026-04-24 while
debugging `getIBKR()` unmatched-position reports.

## 1. A Trade-only Flex omits transfers and corporate actions

`getIBKR()` flagged 5982.T as unmatched in U25343478 Trades. The position
(100 sh, cost basis 3,983.1840 JPY) existed in the IBKR portfolio but was absent
from both the Flex Query Trades CSV **and** the Activity Statement's Transfers
section. It was eventually found in the Activity Statement's **"Trades"** section
— same name, broader coverage than the Flex Query's "Trades" section.

**Why:** a Flex Query is section-configurable (Trades, Transfers, Corporate
Actions, Cash Transactions, Non-Trade Activity). A Trade-only Flex returns pure
execution records, so positions arriving via ACATS/FOP external transfer, internal
sub-account journal, or a corporate action (spin-off, ticker change, allocation)
never appear — **regardless of date range**. The Activity Statement's "Trades"
section includes some events IBKR booked as trades even when they weren't pure
executions.

**How to apply:**
- When `getIBKR()` reports "could not be matched in DB Trades table", don't trust
  the Flex CSV as authoritative. Debug order: (1) grep the Flex Trade CSV; (2) if
  missing, check the Monthly Activity Statement "Trades"; (3) then its "Transfers"
  and "Corporate Actions" sections.
- Cost-basis numbers with unusual fractional decimals (3,983.1840) signal an
  adjusted/carry-in basis, not a fresh execution.
- For a single authoritative Flex CSV for DB ingestion, re-configure the Flex to
  include Transfers + Corporate Actions, not just Trades.
- Manual Trade-row reconstruction: `docs/LESSONS_LEARNED.md`, "Flex Query Trades
  Section Silently Excludes Transfers and Corporate Actions" (SQL template used
  for 5982, TradeNr 714).

## 2. A master-level Flex returns identical CSVs per sub-account

`TradeU1804173 (1).csv` and `TradeU25343478 (1).csv` were downloaded expecting
sub-account-scoped data. Both were 193 lines and `diff` returned **zero output** —
byte-for-byte identical. The Flex is configured at master level and returns
consolidated activity across all sub-accounts regardless of which sub-account was
active in the Portal at download time.

**Why:** master-level Flex Queries scope to the master account by default. The
downloaded filename picks up the current Portal context, but the data is the same.
The current Flex also omits `ClientAccountID` from its columns, so even merging two
downloads can't reconstruct per-sub-account attribution.

**How to apply:**
- Never trust a Flex filename to mean "scoped to that sub-account" — checksum or
  `diff` after downloading.
- Two valid fixes: (1) add the `ClientAccountID` column to the master-level query
  and filter client-side; (2) create separate per-sub-account Flex Queries via the
  "Accounts" selector. Option 2 is simpler for `getIBKR()` ingestion.
- Operational tell: `getIBKR()` reports different unmatched instruments per
  sub-account, yet the same symbol greps successfully in the "other" sub-account's
  CSV. If both CSVs show the same hit, they are the same data.

See [[project_ibkr_subaccounts]].
