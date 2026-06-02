---
name: reference-ibkr-activity-statement
description: "IBKR Activity Statement CSV layout - how to source official TWR, cashflows, and in-kind transfers for reconciliation"
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
All sections take **this account's perspective**: positive = IN, negative = OUT. The mirror account's statement will have opposite signs. The DB `Account.CashFlow` follows the same convention (per `[[project-account-transfer-cashflow-signs]]`).

### Statement period
Header: `Statement, Data, Period, "<start> - <end>"`. The TWR is computed over this exact range — match your DB query window to it for clean comparisons.

### When to ask the user for a statement
- TWR mismatch suspected (compare against `Time Weighted Rate of Return` row)
- Missing/incorrect CashFlow rows in `Account` table
- Verifying inter-account transfers
- Auditing realized vs unrealized P&L per symbol
- Any time the system's portfolio-level numbers conflict with the user's recollection
