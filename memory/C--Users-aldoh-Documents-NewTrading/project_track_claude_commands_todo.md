---
name: TODO — track .claude/commands in NewTrading git
description: NewTrading's .claude/ directory is gitignored, so slash-command spec files (e.g. analyze.md) live local-only and can't be shared across machines or versioned. Future cleanup task.
type: project
originSessionId: 9cbc474c-d67a-496b-92d1-fb42a1346715
---
## Status: Future TODO (opened 2026-04-27)

`.claude/commands/analyze.md` and other slash-command specs in `NewTrading/.claude/` are currently **gitignored** (per `git check-ignore .claude/commands/analyze.md`). Confirmed by `cat .gitignore` — the `.claude` pattern matches.

**Why:** Edits to these files (e.g. the 2026-04-27 VRP log-ratio vs vol-points fix in `analyze.md`) are local-only. They can't be shared between machines, can't be reviewed in PRs, and a fresh clone won't have them.

**Why:** The slash commands are now meaningful work product — the `analyze.md` spec drives every `/analyze` run with non-trivial methodology (5-phase pipeline, vol-funnel grid, edge-source taxonomy). Losing them would lose hours of refinement.

**How to apply:** When ready to do the cleanup, remove the `.claude/` rule from `NewTrading/.gitignore` (or scope it to `.claude/settings.local.json` only — see Claude Code docs on which `.claude/` files are user-private vs project-shared). Then `git add .claude/commands/` and commit.

Open questions to resolve before doing it:
- Are there project-private settings in `.claude/` (API keys, user-specific paths) that should stay ignored?
- Standard pattern from Claude Code is to ignore `settings.local.json` only; commit `settings.json`, `commands/`, `agents/`. Verify before bulk-adding.

Trigger: revisit when next editing a slash command, or when setting up NewTrading on a second machine.
