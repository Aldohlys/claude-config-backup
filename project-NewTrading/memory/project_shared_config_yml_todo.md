---
name: TODO #58 — single shared config.yml across all RTrading apps
description: Open work program to consolidate the 7 per-repo config.yml files (6 byte-identical) into a single canonical config sourced via R_CONFIG_FILE env var.
type: project
originSessionId: 66254968-8c04-4ad8-bfc3-2b1d56f655a0
---
Open as RApplication TODO #58 (opened 2026-04-30, MEDIUM priority). Follows from TODO #55 closure: 7 config.yml files are now tracked in git — 6 are byte-identical, only RStudies adds an `analyze:` namespaced section.

**Goal:** one canonical config.yml, consumed by all 7 R apps. Eliminates the recurring drift risk (e.g. Tuser vs Tuser/scan/app/ already drifted on indentation, account list, log level until the 2026-04-30 commit).

**Suggested first cut (Option B in TODO #58):**
- Canonical file at `RApplication/config.yml` or `~/config.yml` (same pattern as Renviron.site).
- Each app's `config::get()` honours `R_CONFIG_FILE` env var, with fallback to in-repo path for dev.
- Set `R_CONFIG_FILE` in `Renviron.site` (already loaded everywhere).
- Delete per-repo `config.yml`; gitignore them again to prevent accidental drift.

**Why MEDIUM, not HIGH:** the 7 copies are now in sync and tracked in git, so drift is at least visible in PRs. But the duplication is real and will rot again over time.

**Effort:** ~2-3 h.

**How to apply:** when the user asks "anything we can simplify in our R app config?" or works on multi-app config edits, surface this TODO. Full design in `RApplication/docs/TODO.md` #58.

Related: `project_rapplication_repo_policy.md` (umbrella that this consolidation refines).
