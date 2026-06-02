---
name: Flex Query Trades section excludes transfers and corporate actions
description: Positions arriving via external transfer or corporate action are invisible to a Trade-only Flex Query, regardless of date range — use Monthly Activity Statement as ground truth
type: project
originSessionId: f923bfb7-376f-4fb9-8735-ba041515b3a2
---
**Finding (2026-04-24):** `getIBKR()` flagged 5982.T as unmatched in U25343478 Trades. Position (100 sh, cost basis 3,983.1840 JPY) existed in IBKR portfolio but was absent from both the Trades CSV exported from Flex Query and from the Activity Statement's Transfers section. Eventually found in the Activity Statement's "Trades" section — same name, different coverage than the Flex Query "Trades" section.

**Why:** IBKR Flex Query is section-configurable (Trades, Transfers, Corporate Actions, Cash Transactions, Non-Trade Activity). A Trade-only Flex returns pure execution records. Positions that arrived via ACATS/FOP external transfer, internal sub-account journal, or corporate action (spin-off, ticker change, allocation) will not appear. The Activity Statement's "Trades" section is broader and includes some events IBKR booked as trades even when they weren't pure executions.

**How to apply:**
- When `getIBKR()` reports "could not be matched in DB Trades table" for a position, don't trust the Flex Query CSV as authoritative. Debug order: (1) grep Flex Trade CSV; (2) if missing, check Monthly Activity Statement "Trades"; (3) if still missing, check its "Transfers" and "Corporate Actions" sections.
- Cost-basis numbers with unusual fractional decimals (e.g. 3,983.1840) are signals of adjusted/carry-in basis, not fresh executions.
- If you want a single authoritative Flex CSV for DB ingestion, re-configure the Flex to include Transfers + Corporate Actions sections, not just Trades.
- Manual Trade-row reconstruction pattern: see `docs/LESSONS_LEARNED.md` "Flex Query Trades Section Silently Excludes Transfers and Corporate Actions" for the SQL template used for 5982 (TradeNr 714).
