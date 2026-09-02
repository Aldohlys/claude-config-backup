---
name: feedback_appdriver_ignores_config_env
description: shinytest2 AppDriver runs the app in a child R process that ignores R_CONFIG_FILE - headless tests that click through write to the PRODUCTION DB
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 68ab0317-c86c-4460-8825-45f1b6917ad1
  modified: 2026-08-31T14:36:50.807Z
---

`shinytest2::AppDriver$new()` launches the Shiny app in a **separate R process** that does **not** inherit a `Sys.setenv(R_CONFIG_FILE = ...)` set in the driving script. Since `Tdata::safe_db_connect()` resolves the DB through `config::get("DB")` (not `R_DB_PATH`), a headless test that redirects config to a scratch DB copy still reads *and writes* `data/mydb.db`.

Confirmed the hard way: a smoke test of a modal's OK button wrote two rows into the live `Account` table. The preview/read paths in the parent process used the scratch DB, so the redirect *looked* like it was working right up until the write.

**Why:** the parent's env var reaches the child only if passed explicitly; `AppDriver` builds its own process environment.

**How to apply:** in a headless AppDriver run, **drive only up to the point of a write** — open the modal, set inputs, assert on the rendered preview, then stop. Exercise the write path separately by calling the function directly in-process, where `R_CONFIG_FILE` *is* honoured and a DB copy really is isolated. If a click-through write is unavoidable, verify afterwards which DB received it (`stat` the mtime, check `max(rowid)`) and be ready to revert. Note SQLite reuses freed rowids, so a row inserted after a delete can reclaim the deleted row's id — don't use "same rowid" as proof of "same row".

Also add `Sys.setenv(NOT_CRAN = "true")` or `AppDriver$new()` aborts with `Reason: On CRAN`.

Related: [[reference_routine_app_visual_pass]], [[feedback_shinytest2_for_silent_bugs]], [[feedback_verify_db_snapshot_fresh]]
