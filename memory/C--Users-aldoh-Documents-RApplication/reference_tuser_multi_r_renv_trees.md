---
name: reference_tuser_multi_r_renv_trees
description: "Tuser/renv/library holds multiple per-R-version trees; the active R 4.4.3 uses windows/R-4.4 — verify deployed package versions THERE, not the stale R-4.3/R-4.5 trees."
metadata: 
  node_type: memory
  type: reference
  originSessionId: e7c37a4a-46bb-4b8f-a2ed-8b13b66f3b38
---

`Tuser/renv/library/` contains several per-R-version subtrees, only one of which is live:
- `R-4.3/x86_64-w64-mingw32/` — STALE (e.g. held Tdata 2.4.5)
- `windows/R-4.4/x86_64-w64-mingw32/` — **ACTIVE** for the current R 4.4.3 (`Rscript.exe` at `C:\Program Files\R\R-4.4.3`)
- `windows/R-4.5/x86_64-w64-mingw32/` — STALE (e.g. held Tdata 5.5.1)

When verifying that `/build` actually deployed a new package version, check the `windows/R-4.4` tree (`grep ^Version: .../R-4.4/x86_64-w64-mingw32/<pkg>/DESCRIPTION`). A naive `find -name DESCRIPTION -path '*<pkg>*' | head -1` can grab a stale tree first and mislead (it returned Tdata 2.4.5 once while the real install was 5.12.0). The other apps (RPreTrade etc.) and the system library `C:/Users/aldoh/Documents/RLibrary` are single-tree. See [[feedback_build_renv_churn_revert]], [[project_tdata_install_locations]].
