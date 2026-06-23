---
name: SQLite read queries auto-approved during execution
description: Once a plan is approved, sqlite3 read queries against mydb.db must run without per-call permission prompts
type: feedback
originSessionId: 53dab338-46f2-4aba-b466-d9cda0aa1007
---
When the user has approved an implementation/analysis plan, sqlite3 read queries (`SELECT`, `.schema`, etc.) against `C:/Users/aldoh/Documents/RApplication/data/mydb.db` should execute automatically as part of the plan, not be re-prompted call-by-call.

**Why:** The DB is the primary data source for trading-strategy advisory work. Repeated permission prompts on read-only SQL during a multi-step approved task break flow and add no safety value (read-only on a local DB).

**How to apply:** Once the user has said "go" or otherwise approved a multi-step plan, run sqlite3 SELECT/schema queries as needed without pausing. Still confirm before any DB write operation, before destructive shell commands, or before plan-altering steps. Only the read-query stream is pre-authorized by plan approval.
