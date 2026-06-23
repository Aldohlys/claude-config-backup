---
name: NewTrading GitHub remote
description: NewTrading repo is now on Aldohlys/NewTrading (HTTPS); master tracks origin/master
type: project
originSessionId: dbd241e8-773c-4681-bb7b-e2c19d1a98e3
---
DONE 2026-06-09: the user created `Aldohlys/NewTrading` on GitHub and I wired up the remote. `origin` = `https://github.com/Aldohlys/NewTrading.git` (HTTPS via credential manager, same scheme as RApplication/RStudies/Tdata). `master` set to track `origin/master`; pushed up to `1f488f9`.

**How to apply:** Push tracked-file commits with `git push` as normal. Default/working branch is `master` (local "main" label is just git-status metadata). Working tree often carries uncommitted edits (CLAUDE.md, change.log, Trades/, untracked Reports/*.html) — only commit/push these when the user asks.

Opened 2026-05-07 as a TODO; resolved 2026-06-09.
