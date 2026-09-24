---
name: feedback_check_memory_before_absence_claims
description: Before telling the user something is "not set / not configured / missing", grep memory and test from the runtime that actually uses it (R for R env vars)
metadata:
  type: feedback
---

On 2026-09-23 I reported "ALERT_EMAIL_PASSWORD is not set anywhere on this machine, email is not configured, and check_alerts.R only looks fine because it exits early". The user corrected it: the password is in `RApplication/Renviron.site`. Memory already said so (project_rapplication_repo_policy: "Renviron.site contains ALERT_EMAIL_PASSWORD"). I had checked only the Windows env and `.Renviron`, and I added an unverified claim about check_alerts.

**Why:** an absence claim sends the user off to "fix" something that works, and it contradicted a fact already saved in memory.

**How to apply:** before asserting that a config, secret or setup is missing:
1. Grep the memory directory.
2. Test from the consuming runtime (`Rscript -e Sys.getenv(...)`, the app's own config loader), not from the OS shell.
3. Don't generalise to other scripts without checking them.

Related: [[reference_r_environment]], [[feedback_verify_policy_assertions]].
