---
name: reference_child_rscript_env_override
description: A child Rscript re-reads Renviron.site and overwrites inherited R_DB_PATH — only R_ENVIRON can redirect it; system2(env=) is ignored on Windows
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1e2ceadf-edaa-4f9c-82d1-d1e2a412227d
  modified: 2026-09-03T08:28:21.052Z
---

To make a **child `Rscript`** (e.g. a script under test via `system2`) read a different DB, neither obvious approach works on this machine:

- **`Sys.setenv(R_DB_PATH = copy)` then `system2(...)`** — the child inherits it, then R startup processes `Renviron.site`, whose `R_DB_PATH=<project>/data/mydb.db` assignment **overwrites** the inherited value. The script silently reads production.
- **`system2(..., env = "R_DB_PATH=...")`** — not honoured on Windows; the assignment is passed as a command *argument* (`Rscript.exe R_DB_PATH=... script.R`) and the run fails with status 5.

What works: point `R_ENVIRON` at a replacement site file, which R loads **instead of** `Renviron.site`. Restate every variable the script needs, since the real site file is then skipped:

```r
envf <- file.path(tmp, "Renviron.test")
fs <- function(x) gsub("\\\\", "/", x)      # see below
writeLines(c(paste0("R_DB_PATH=",     fs(db)),
             paste0("R_CONFIG_FILE=", fs(Sys.getenv("R_CONFIG_FILE"))),
             paste0("R_BOX_PATH=",    fs(Sys.getenv("R_BOX_PATH"))),
             paste0("R_LOG_DIR=",     fs(Sys.getenv("R_LOG_DIR")))), envf)
Sys.setenv(R_ENVIRON = envf)
```

**Forward slashes are mandatory.** A Windows temp path (`tempdir()` returns backslashes, `file.path()` then mixes in `/`) does not survive Renviron parsing — the child comes back with a path that fails `file.exists()`.

This is only needed for a **child process**. Code running *in* the current session is redirected far more simply by pointing `R_CONFIG_FILE` at a temp config, because `Tdata::safe_db_connect()` resolves the path with `config::get("DB")` — see [[reference_testing_tuser_box_internals]].

Used to test `scripts/sync_stop_risk.R`'s orphaned-stop safety rail against a DB copy, since that script writes and must never be exercised on production — same principle as [[reference_live_virtual_account_no_table]].
