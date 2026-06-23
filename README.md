# Claude Code Config Backup

Backup of Claude Code configuration for the trading systems (RApplication + NewTrading).
Both projects keep their `.claude/` gitignored in their own repos, so this is their only versioned copy.

## Structure

```
global/                          # ~/.claude/
  CLAUDE.md                      # Global instructions
  settings.json                  # Global settings
  commands/                      # User-level slash commands (work in ANY project)
    save.md                      # Review session -> persist to memory
    sync.md                      # This backup workflow

project-RApplication/            # RApplication/.claude/
  commands/                      # build.md, sync-DB.md
  skills/                        # git-commit, package-build, shiny-module-dev, test-shiny-app
  settings.local.json

project-NewTrading/              # NewTrading/.claude/
  commands/                      # analyze.md (neutral data report), transcribe.md
  settings.local.json
  wheel_analysis.md              # WHEEL strategy deep-dive reference

memory/                          # Persistent memory (RApplication project key)
  MEMORY.md                      # Memory index
  *.md                           # Individual memory records
```

## Restore

Copy files back to their original locations:
- `global/*`               -> `~/.claude/`
- `project-RApplication/*` -> `RApplication/.claude/`
- `project-NewTrading/*`   -> `NewTrading/.claude/`
- `memory/*`               -> `~/.claude/projects/C--Users-aldoh-Documents-RApplication/memory/`

## Sync

Run `/sync` (or say "sync claude config backup"). It mirrors the locations above and commits + pushes.
Run after significant changes to commands/skills/memory. See `global/commands/sync.md` for the exact mapping.
