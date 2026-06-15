---
name: project_ibkr_subaccounts
description: Critical API differences when using IBKR sub-accounts vs single account - portfolio subscription, accountSummary split
type: project
originSessionId: 5ceb604c-8e70-4a7f-a505-a5a16612d921
---
U1804173 and U25343478 are IBKR sub-accounts under the same master.

**Key API findings (tested 2026-04-17):**

1. `ib.managedAccounts()` returns both: `['U1804173', 'U25343478']`

2. `ib.accountSummary()` returns THREE account groups:
   - `U1804173`: only margin/equity tags (NetLiquidation, FullInitMarginReq, etc.)
   - `U25343478`: same margin/equity tags
   - `All`: only market value/PnL/cash tags (StockMarketValue, OptionMarketValue, UnrealizedPnL, TotalCashBalance)
   - Tags are SPLIT across accounts — no single account has all tags

3. `ib.portfolio()` returns EMPTY with sub-accounts. Must use:
   ```python
   ib.reqAccountUpdates(account='U25343478')
   ib.sleep(2)
   items = ib.portfolio('U25343478')
   ib.reqAccountUpdates(account='')  # cancel subscription
   ```

4. `ib.positions()` works without subscription but lacks marketValue/unrealizedPNL

**Why:** Per-account StockMarketValue/OptionMarketValue/UnrealizedPnL are computed from portfolio items in `getIBKRData()`, not from accountSummary.

**How to apply:** When extending IBKR data retrieval, always account for sub-account API splits. Never assume accountSummary has all tags for a single account.
