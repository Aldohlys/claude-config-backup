---
name: feedback_changelog_before_build
description: build_package.R generates descriptive commit messages from CHANGELOG.md; without an entry for the new version it falls back to a generic "Update <files>" file-list
type: feedback
originSessionId: 6179772a-4f40-4f79-be76-374d711c6e2b
---
Add a `## [x.y.z] - YYYY-MM-DD` section to `<Package>/CHANGELOG.md` for the *target* version BEFORE running `/build <package> auto`.

**Why:** `build_package.R` reads CHANGELOG.md to generate the git commit title/body. When no entry exists for the version it's about to write, it falls back to summarizing changed file names — producing generic messages like "Build Tdata v5.10.9: Update contract.py, parquet_storage.py" instead of explaining *why* the changes were made. Observed in this session on two consecutive Tdata builds (5.10.9, 5.10.10).

**How to apply:**
- For `auto patch`: before running, increment in your head and add the matching `## [x.y.z+1]` section.
- If user runs `/build` without a pre-written CHANGELOG entry, flag it and offer to draft one before kicking off the build. Don't silently let the generic message ship.
- The CLAUDE.md "Changelog Workflow Summary" already says "BEFORE build: Update CHANGELOG.md" — this memory is a reminder that *I* should enforce it, not assume the user did.

**UPDATE 2026-06-13 (Tdata 5.10.24):** A correct `## [5.10.24]` CHANGELOG entry was present BEFORE the build, yet the commit STILL came out generic ("Build Tdata v5.10.24: Package updates"). So a CHANGELOG entry is necessary but NOT sufficient — the message-generation step appears to need an interactive Claude/LLM call that isn't available in the headless Rscript `/build` run, and falls back to generic regardless. **Plan to inspect `git log -1` after every `/build` and proactively `git commit --amend -F tempfile` + `git push --force-with-lease` to fix the message.** (Each app/package is its own repo — amend in the package dir; `--amend` ignores unstaged unrelated files like a dirty atr_move.R.)

**UPDATE 2026-06-15 (Tdata 5.10.29):** the amend remediation works, but the `git push --force-with-lease` is **not silent/proactive** — the auto-mode classifier BLOCKS force-pushing an already-pushed commit unless the user explicitly asked for it (it rewrites remote history). So: surface the generic message, propose the amend, and get an explicit "amend + push" before force-pushing. The local `git commit --amend` alone is harmless (no remote rewrite) but leaves local/remote diverged until the approved push — verify with `git log -1 origin/<branch>` if a push was denied mid-sequence.
