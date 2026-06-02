---
name: reference_mydb_sql_dump
description: "data/mydb.sql dump — generator, CRLF full-file churn, and how to commit it cleanly"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6521c13e-ba17-4066-985c-d891cb840ed3
---

`data/mydb.sql` is a git-tracked text dump of the live `data/mydb.db`.

- **Generator (canonical):** `Rscript scripts/backup_database.R` (export only) or `... commit` (export + commit + push). It runs `sqlite3 mydb.db .dump` then post-processes `unistr(...)` → plain strings for the VM's older SQLite (3.37.2 has no `unistr()`).
- **Huge diffs are NORMAL, not real change:** `core.autocrlf=true` + the script's `writeLines` (writes CRLF on Windows) make the dump flip line-endings, so `git diff` shows the *entire file* rewritten (~110k+ lines: ~half insert, ~half delete) on essentially every dump. Don't panic-diff it looking for content changes.
- **Committing:** the dump is often pre-staged in the index. A plain `git commit` (after `git add` of other files) will BUNDLE the whole dump into your commit. To commit something else cleanly, use a pathspec commit: `git commit <path> -m "..."` (commits only that path, ignores the staged dump). The user's own commits routinely bundle a fresh dump alongside doc changes — that's their pattern.
- To resync the dump after a live-DB schema change (e.g. a column rename), just re-run `backup_database.R`; the dump regenerates from the DB (the source of truth).

Related: [[gcloud-vm]] (the VM keeps its own DB at a different path), [[reference_savetrades_overwrite]].
