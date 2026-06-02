---
name: Update CHANGELOG.md before running /build
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
