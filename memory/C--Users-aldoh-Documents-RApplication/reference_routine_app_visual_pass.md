---
name: reference_routine_app_visual_pass
description: Headless visual-pass recipe for the routine Shiny app via shinytest2 AppDriver (+ IBKR-hangs caveat)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3f4e5a5e-17d7-445c-8433-e74f630f76ef
---

To screenshot the routine app headlessly (a real PNG you can Read): set `SHINY_PORT=1` so `Tuser/routine/routine.R` returns the `shinyApp` object instead of calling `runApp()`, grab it with `source(".../routine/routine.R", local=new.env())$value`, then `shinytest2::AppDriver$new(app_obj, timeout=90000, load_timeout=90000)`. Switch account via `app$set_inputs(account="Gonet")`; the `tabsetPanel` has no id, so activate a tab with `app$run_js('$(\'a[data-value="Trade"]\').tab("show")')`; `app$wait_for_idle()`; `app$get_screenshot(png)`; then Read the PNG. chromote + shinytest2 are installed; Chrome at `C:\Program Files\Google\Chrome`. Run via Rscript with `R_BOX_PATH` set; bump the Bash tool timeout (each run ~1–2 min).

Caveat: **Gonet account loads fine headless** (reads the stored DB snapshot). **IBKR accounts (U1804173 etc.) HANG** — the account module fetches live data and `set_inputs` times out at ~90s ("Server did not update any output values within 90 seconds"). So validate shared display logic on Gonet; for IBKR-only columns drive the render functions directly on a stored `readPortfolio()` snapshot instead (no app, no TWS — see [[reference_dt_basecurrency_sort_pin_row]]). Pre-existing boot warnings (globals 'primer'; duplicate ids `displaysym-symbolsPortfolio`/`getPortfolio-timePortfolio`) are harmless. Each AppDriver run also churns `renv/activate.R` — revert it, [[feedback_build_renv_churn_revert]]. Related: [[feedback_shinytest2_for_silent_bugs]].
