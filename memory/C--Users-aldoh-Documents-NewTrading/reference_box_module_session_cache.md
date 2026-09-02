---
name: reference-box-module-session-cache
description: box::use() caches modules for the whole R session — edits to a Tuser module don't take effect on re-runApp until purge_cache() or an R restart
metadata:
  type: reference
---

`shiny::runApp()` re-sources `app.R` every time, but `box::use(spread/view/spreadUI)` just hands back the module already loaded in that R session. So editing a `*/view/*UI.R` module and re-running the app silently keeps executing the **old** code.

Cost real time on 2026-08-26: a fixed module kept throwing the pre-fix error, and the giveaway was the traceback line numbers pointing at where the handler used to be, not where it now is. A long-lived console makes it worse — a high `runApp` port (7815) means many prior runs in the same session.

Fix: `box::purge_cache()` then re-run, or restart R (Ctrl+Shift+F10 in RStudio). Prefer the restart when reticulate has already initialised Python in that session. `box::unload()` / `box::reload()` also exist.

Corollary for debugging: a fresh `Rscript -e ...` gets a clean session every time, so a bug that reproduces in the user's console but not in an Rscript test is very likely this, not a real difference.

Related: [[reference_tuser_repo]], [[reference_r_environment]], [[feedback_tdata_rebuild_restart_r]]
