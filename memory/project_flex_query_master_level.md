---
name: project_flex_query_master_level
description: Flex Query configured at master/F-account level ignores sub-account context; both TradeU1804173.csv and TradeU25343478.csv are byte-identical unless ClientAccountID is added
type: project
originSessionId: f923bfb7-376f-4fb9-8735-ba041515b3a2
---
**Finding (2026-04-24):** Downloaded `TradeU1804173 (1).csv` and `TradeU25343478 (1).csv` expecting sub-account-scoped data. Both files were 193 lines and `diff` returned zero output — byte-for-byte identical. The current Flex Query is configured at master-level and returns consolidated trade activity across all sub-accounts regardless of which sub-account was active in the Portal at download time.

**Why:** IBKR master-level Flex Queries scope their query to the master account by default. The downloaded filename picks up the current Portal context, but the data is identical. The current Flex also omits `ClientAccountID` from its selected columns, so even merging two downloads can't reconstruct per-sub-account attribution.

**How to apply:**
- Don't trust Flex Query filenames to mean "this file is scoped to that sub-account." Checksum or diff after downloading to verify.
- Two valid fixes: (1) add `ClientAccountID` column to the master-level query and filter client-side; (2) create separate per-sub-account Flex Queries via the "Accounts" selector in the Flex config. Option 2 is simpler for `getIBKR()` ingestion.
- Operational sign this is happening: `getIBKR()` reports different unmatched instruments per sub-account, but the same symbol can be grep'd successfully in the "other" sub-account's CSV. If both CSVs show the same hit, they're the same data.
