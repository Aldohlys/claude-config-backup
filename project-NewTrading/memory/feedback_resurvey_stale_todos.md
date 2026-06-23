---
name: Re-survey state before executing a stale TODO
description: For any "execute this plan" TODO older than a few days, re-run the underlying survey before acting on the inventory.
type: feedback
originSessionId: 66254968-8c04-4ad8-bfc3-2b1d56f655a0
---
For any TODO whose body is "here is an inventory, now execute against it", re-run the underlying survey before staging anything. The recorded inventory is a snapshot, not ground truth.

**Why:** TODO #55 (clean uncommitted state across 9 repos) was surveyed 2026-04-21 and executed 2026-04-30. In those 9 days: `mydb.sql` and `update_missing_iv_safe.R` had been committed already, RStudies had a pile of new research scripts (catalyst studies, IV-vs-spot-lag, ivol probe), Tdata had new in-progress work, and several new untracked items appeared (`scripts/Tuser-ticker.bat`, `quotes/` cache directories, `Tuser/ticker/`). Trusting the snapshot would have produced wrong commits AND missed real work.

**How to apply:** When a TODO says "execute the plan" or "do the actions in this inventory", first run the equivalent of the original survey (`git status` across all repos, file existence checks, etc.) and diff against the recorded inventory. Surface the deltas to the user before staging. Tighter the older the TODO; non-negotiable past ~1 week.
