---
name: feedback_verify_audit_claims
description: Before dropping DB tables, files, or config based on a sub-agent's "unused/orphaned" classification, re-grep directly for reads across all downstream consumers
type: feedback
originSessionId: 1347083d-d9f2-4281-99ce-26cad155e9e3
---
Before executing destructive DB/filesystem operations (DROP TABLE, rm, git clean, etc.) based on a sub-agent's "unreferenced/orphaned/write-only" classification, **personally re-verify the claim** with a direct Grep across all downstream consumers — not just the writer's own module.

**Why:** In 2026-04-20 session, an Explore sub-agent classified `macro_context_results` and `macro_context_mismatches` as "write-only" (found `dbWriteTable` in `macro_context/main.R` only). I dropped both tables without verifying. The scanner's `swing_scanner/main.R:45` and `:49` read them — the reads were wrapped in tryCatch, so no visible crash, just silent degradation of the macro overlay. Had to restore from backup.

**How to apply:**
- "Orphaned" / "write-only" / "unused" classifications from sub-agents are **hypotheses, not conclusions**
- Before any destructive operation, run a fresh targeted Grep: the table/file/symbol name across *all* relevant file types (.R, .py, .sql, .ps1, .md templates), not just the module that owns it
- Particularly watch for: `tryCatch`-wrapped reads (look used even when they return NULL), indirect reads via views/joins, reads in dashboards/reports that are downstream of the writer
- If in doubt, keep the thing. Dropping is cheap to reverse *if you still have a backup*; forgetting to make one is expensive
