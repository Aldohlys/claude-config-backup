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

memory/                          # Persistent memory — AUTO-DISCOVERED, one subfolder per project key
  C--Users-aldoh-Documents-RApplication/            # RApplication memory
  C--Users-aldoh-Documents-RApplication-RStudies/   # RStudies memory
  C--Users-aldoh-Documents-NewTrading/              # NewTrading memory
    MEMORY.md                    # Memory index (per project)
    *.md                         # Individual memory records
```

`memory/` is rebuilt from scratch on every `/sync` by iterating `~/.claude/projects/*/memory/`, so any
new project's memory is backed up automatically (no spec change needed).

## Restore

Copy files back to their original locations:
- `global/*`               -> `~/.claude/`
- `project-RApplication/*` -> `RApplication/.claude/`
- `project-NewTrading/*`   -> `NewTrading/.claude/`
- `memory/<project-key>/*` -> `~/.claude/projects/<project-key>/memory/`

## Sync

Run `/sync` (or say "sync claude config backup"). It mirrors the locations above and commits + pushes.
Run after significant changes to commands/skills/memory. See `global/commands/sync.md` for the exact mapping.
