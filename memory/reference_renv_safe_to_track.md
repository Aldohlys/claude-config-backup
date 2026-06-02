---
name: renv state is safe to git-track because of the renv-managed sub-gitignore
description: Each renv/ ships a renv/.gitignore that excludes library/staging/cellar/etc., so `git add renv/` won't drag in hundreds of MB. Track .Rprofile, renv.lock, renv/activate.R, renv/settings.json.
type: reference
originSessionId: 66254968-8c04-4ad8-bfc3-2b1d56f655a0
---
Each renv-managed project ships a `renv/.gitignore` that already excludes the heavy/derived directories. Confirmed identical content across all 6 RApplication R apps (Tuser, RReporting, RJournal, RPreTrade, ROrder, RStudies):

```
library/
local/
cellar/
lock/
python/
sandbox/
staging/
```

So `git add renv/` is safe — it picks up `renv/activate.R`, `renv/settings.json` (if present), and `renv/.gitignore` itself, but NOT the multi-hundred-MB package library. No need to manually enumerate paths to avoid library bloat.

**Standard renv tracking convention (use across all R apps):**
- TRACK: `.Rprofile`, `renv.lock`, `renv/activate.R`, `renv/settings.json` (if exists), `renv/.gitignore`
- IGNORE (auto-handled by `renv/.gitignore`): `renv/library/`, `renv/staging/`, `renv/local/`, `renv/cellar/`, `renv/lock/`, `renv/python/`, `renv/sandbox/`

**RApplication-wide policy (set 2026-04-30 via TODO #55):** all 6 R apps follow this convention. If a future repo gitignores `renv/` or `renv.lock` at the top level, that's a violation of the unified policy — un-ignore and bring into git.
