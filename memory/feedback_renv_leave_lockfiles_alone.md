---
name: feedback_renv_leave_lockfiles_alone
description: "User policy on renv lockfile hygiene across the 6 app renvs. The renv \"out-of-sync\" warning is expected and must NOT be fixed with renv::snapshot(); library updates are deliberate and periodic only."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6f5786db-5489-4ea1-851d-9477ce02eb6c
---

User policy (stated 2026-06-02): the whole point of renv is that app libraries do NOT auto-track system package updates. The lockfile is the pinned source of truth and apps stay insulated from package churn. He updates libraries **manually, as an occasional deliberate global update** — not per package, not to clear a warning.

**Why:** running `renv::snapshot()` casually does the OPPOSITE of what renv is for — it blesses whatever is currently installed (often newer drifted versions) as the new pin, i.e. it bakes in the automatic-update behavior he wants to avoid.

**How to apply:**
- The renv "out-of-sync [lockfile != library]" warning is **expected and non-actionable**. Do NOT propose or run `renv::snapshot()` to silence it.
- Across the 6 app renvs the drift is **library AHEAD of lockfile** (e.g. ggplot2 4.0.3 installed vs 3.5.1 pinned, shiny 1.13 vs 1.10). So the libraries already received un-recorded updates and are NOT actually frozen at the pins right now.
- When he does a deliberate periodic global update, the reproducible sequence is: `renv::restore()` per app (pull library back to pins, undoing drift) → bump what he wants → `renv::snapshot()` to re-pin. Don't auto-run this; it's his call.
- Genuine launch blockers (a package physically missing from a lib, or a broken `activate.R` — see [[renv::restore(packages='renv') under old renv corrupts activate.R with ..md5..]]) are still worth fixing. The "leave it alone" rule is specifically about the version-drift / snapshot temptation, not about real breakage.
