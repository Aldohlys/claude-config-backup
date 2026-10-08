---
name: reference_ibkr_activity_statement
description: "IBKR Activity Statement CSV layout - official TWR, cashflows, in-kind transfers; MULTI two-account file; decomposing a NAV drop into withdrawals vs P&L"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 13d07ea0-0f5c-4fb7-8397-97b2e1c8249a
---

When reconciling DB state against IBKR ground truth (TWR off, cashflows missing, transfer events to verify), the IBKR Activity Statement CSV is authoritative.

### Cannot be auto-retrieved
**IBKR Activity Statements must be manually exported by the user from Account Management.** Claude Code has no API/portal access. Ask the user to export and share the path (typical: `C:\Users\aldoh\Downloads\<account>_<YYYYMMDD>_<YYYYMMDD>.csv`).

### Always pull BOTH sides of a transfer
The user's mental model ("I moved everything from A to B") can be incomplete — e.g., in-kind securities went A→B but the matching FX cash legs went B→A as part of the rebalance. Reconcile against both source and destination statements before drawing conclusions.

### Key sections (by CSV row prefix)

| Section | What to read |
|---|---|
| `Net Asset Value, Data, <pct>%` (after `Time Weighted Rate of Return` header) | **Official TWR for the period** — the number to match against `Tdata::twr` output |
| `Change in NAV, Data, Starting Value / Ending Value` | Period anchor NAVs (in base currency) |
| `Change in NAV, Data, Deposits & Withdrawals` | Net cash transfers in/out (CHF-equivalent) |
| `Change in NAV, Data, Position Transfers` | Net in-kind security transfers (CHF-equivalent) — **NOT recorded in DB historically; required for TWR accuracy** |
| `Cash Report, Data, Account Transfers, <CCY>` | Per-currency cash transfer amounts. Sign = direction from this account's perspective (positive = IN) |
| `Transfers, Data, Stocks, <CCY>, <Symbol>, <date>, Internal, Out/In, --, <OtherAccount>, ...` | Per-symbol in-kind transfers with direction and native-currency Market Value. Has its own date (may differ from cash leg date) |
| `Deposits & Withdrawals, Data, <CCY>, <date>, <Description>, <amount>` | Per-event log of cash deposits/withdrawals (description text identifies counterparty account) |
| `Open Positions` | Current holdings with Cost Basis vs Value vs Unrealized P/L |
| `Forex Balances` | Per-currency cash balances at period end |

### Sign convention
All sections take **this account's perspective**: positive = IN, negative = OUT. The mirror account's statement will have opposite signs. The DB `Account.CashFlow` follows the same convention (per `[[project_account_transfer_cashflow_signs]]`).

### Statement period
Header: `Statement, Data, Period, "<start> - <end>"`. The TWR is computed over this exact range — match your DB query window to it for clean comparisons.

### When to ask the user for a statement
- TWR mismatch suspected (compare against `Time Weighted Rate of Return` row)
- Missing/incorrect CashFlow rows in `Account` table
- Verifying inter-account transfers
- Auditing realized vs unrealized P&L per symbol
- Any time the system's portfolio-level numbers conflict with the user's recollection

### MULTI statements (both IBKR accounts in one CSV)
`MULTI_<start>_<end>.csv` holds U1804173 and U25343478 as consecutive blocks, each beginning again with `Statement,Header`. Sections repeat per account, so a `grep` returns each section twice — attribute rows to an account by line order. `Account Summary` (once, at top) gives Prior/Current NAV per account plus Total.

### Decomposing a NAV change ("why is the account down X?")
Sum `Change in NAV` across both accounts. Internal flows between the two accounts (CHF cash, GBP/JPY legs, `Position Transfers`) cancel in the sum. What remains = external Deposits & Withdrawals + P&L lines (MTM, dividends, interest, fees, commissions). Check the withdrawals first. Example (2026-10-07, period 25.12.2025–06.10.2026): NAV 104,148 → 78,169 CHF; −24,431 was net external withdrawals (EUR 21,000 disbursed in 4 lots + CHF −5,000 net); only −1,548 was trading/costs. U1804173 alone showed −44,683 D&W because its transfers to U25343478 look like outflows.
Per-symbol attribution: `Mark-to-Market Performance Summary`, second-to-last field = MTM P/L Total; each asset class ends in a `Total` row (the asset-class name is on the row above).
The user's own NAV figures can differ by ~0.1–1.5k from the statement (different valuation day or source). Say that the statement can't explain that gap; don't force a reconciliation.
