# Claude Code Config Backup

Backup of Claude Code configuration for the RApplication trading system.

## Structure

```
global/                          # ~/.claude/
  CLAUDE.md                      # Global instructions
  settings.json                  # Global settings

project-RApplication/            # RApplication/.claude/
  commands/                      # Slash commands (/analyze, /build)
    analyze.md                   # BOT candidate analysis
    build.md                     # Package build workflow
  skills/                        # Auto-detected skills
    git-commit.md
    package-build.md
    shiny-module-dev.md
    test-shiny-app.md
  settings.local.json            # Project-specific settings

memory/                          # Persistent memory files
  MEMORY.md                      # Memory index
  *.md                           # Individual memory records
```

## Restore

Copy files back to their original locations:
- `global/*` -> `~/.claude/`
- `project-RApplication/*` -> `<project>/.claude/`
- `memory/*` -> `~/.claude/projects/<project-key>/memory/`

## Sync

Run manually or after significant changes to commands/skills/memory.
