---
name: reference_claude_config_backup
description: "claude-config-backup repo + /save & /sync user-level commands; layout, auto-discovered memory, gotchas"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 672ba0ce-0bbd-4810-bd8c-9c6150cb270e
---

Claude Code config is backed up to private repo **`Aldohlys/claude-config-backup`** (local clone: `C:/Users/aldoh/Documents/claude-config-backup`, remote `master`).

**Shorthand commands (user-level, work in ANY project — live in `~/.claude/commands/`):**
- **`/save`** — review the session and persist memory-worthy facts to the current project's memory dir + update its `MEMORY.md`.
- **`/sync`** — mirror config into the backup repo and commit+push (the shorthand for "sync claude config backup").
- Created 2026-06-23. User-level (`~/.claude/commands/`) = available everywhere; project-level (`<proj>/.claude/commands/`, e.g. RApplication's `/build` `/sync-DB`, NewTrading's `/analyze` `/transcribe`) = that project only.

**Backup layout:**
- `global/` ← `~/.claude/` (`CLAUDE.md`, `settings.json`, `commands/`)
- `project-<name>/` ← that project's `.claude/` (commands/skills/settings.local.json/loose files) — **hand-curated**, one block per project in `/sync` steps. Currently `project-RApplication/` + `project-NewTrading/`.
- `memory/<full-project-key>/` ← **auto-discovered** by iterating `~/.claude/projects/*/memory/` (wiped+rebuilt each `/sync`), so any project's memory is captured with no spec change. Key = dir name under `projects/`, e.g. `C--Users-aldoh-Documents-NewTrading`.

**Why both `RApplication/.claude` and `NewTrading/.claude` are backed up here:** both are **gitignored** in their own repos (`NewTrading/.gitignore:9 = .claude/`), so this backup is their only versioned copy. See [[reference_app_subdirs_are_separate_repos]].

**Gotchas:**
- `rsync` is NOT in the Bash-tool Git Bash → `/sync` mirrors dirs with `rm -rf <dest> && cp -r <src>/. <dest>/` (emulates `--delete`).
- Before committing, inspect **deletions** in `git status` — a removed file may be **misfiled** (an obsolete `analyze.md` once sat under `project-RApplication` when the live one is in NewTrading), not genuinely gone. Refile, don't drop.
- LF→CRLF git warnings on Windows are normal/harmless.
- Sync triggered manually: `/sync` or "sync claude config backup".
