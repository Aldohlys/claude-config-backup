---
name: reference_r_environment
description: "R install location, version, library path, and PATH status on this Windows machine"
metadata: 
  node_type: memory
  type: reference
  originSessionId: ad00b5eb-2764-446d-8fbc-e764925edee4
---

R environment on this machine (verified 2026-06-05):

- **Only R version installed:** R-4.4.3
- **Rscript.exe:** `C:\Program Files\R\R-4.4.3\bin\Rscript.exe`
- **`.libPaths()`:** `C:\Users\aldoh\Documents\RLibrary` (primary user lib) then `C:\Program Files\R\R-4.4.3\library` (base). Install packages with `lib='C:/Users/aldoh/Documents/RLibrary'`.
- **PATH:** `C:\Program Files\R\R-4.4.3\bin` was NOT on PATH originally — added to **user PATH** 2026-06-05, so bare `Rscript`/`R` now resolves in *new* processes. The Bash tool's environment may still not see it; PowerShell new sessions will.
- **Stale leftover:** `%LOCALAPPDATA%\R\win-library\4.3` exists but is unused (not on `.libPaths()`, wrong version) — harmless.

**`r-btw` MCP server** (user scope, `C:\Users\aldoh\.claude.json`): runs `btw::btw_mcp_server(tools='docs')` for R-package documentation lookup during coding. Configured with the **full Rscript.exe path** (not bare `Rscript`) for robustness; `btw` v1.2.1 installed in RLibrary. Related: [[feedback_verify_before_claiming_codebase_facts]].
