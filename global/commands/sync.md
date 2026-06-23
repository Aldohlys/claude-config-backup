# Sync Claude Config Backup

Mirror the local Claude Code configuration into the private backup repo
`Aldohlys/claude-config-backup` (local clone: `C:/Users/aldoh/Documents/claude-config-backup`)
and commit + push.

**INSTRUCTIONS FOR CLAUDE:**

This command uses **absolute paths**, so it backs up the same known config set regardless of which
project it is invoked from. It is the shorthand for the phrase "sync claude config backup".

## Path mapping (source → backup repo)

```
~/.claude/CLAUDE.md                               -> global/CLAUDE.md
~/.claude/settings.json                           -> global/settings.json
~/.claude/commands/                               -> global/commands/        (user-level slash commands, incl. /save, /sync)
RApplication/.claude/commands/                    -> project-RApplication/commands/
RApplication/.claude/skills/                      -> project-RApplication/skills/
RApplication/.claude/settings.local.json          -> project-RApplication/settings.local.json
~/.claude/projects/C--Users-aldoh-Documents-RApplication/memory/  -> memory/
```

Absolute source roots:
- Global: `C:/Users/aldoh/.claude`
- Project: `C:/Users/aldoh/Documents/RApplication/.claude`
- Memory: `C:/Users/aldoh/.claude/projects/C--Users-aldoh-Documents-RApplication/memory`
- Backup repo: `C:/Users/aldoh/Documents/claude-config-backup`

## Steps

1. **Copy global files:**
   - `cp ~/.claude/CLAUDE.md <backup>/global/CLAUDE.md`
   - `cp ~/.claude/settings.json <backup>/global/settings.json`
   - Mirror commands dir (create if missing): `rsync -a --delete ~/.claude/commands/ <backup>/global/commands/`

2. **Mirror the RApplication project config** (use `rsync -a --delete` so removed files propagate):
   - `<project>/.claude/commands/`           -> `<backup>/project-RApplication/commands/`
   - `<project>/.claude/skills/`             -> `<backup>/project-RApplication/skills/`
   - `cp <project>/.claude/settings.local.json <backup>/project-RApplication/settings.local.json`

3. **Mirror memory** (use `rsync -a --delete` to drop deleted memories):
   - `<memory>/` -> `<backup>/memory/`

4. **Do NOT copy secrets.** Never sync `config.yml`, `.Renviron`, `Renviron.site`, `*.secret`, `*.key`, `*.pem`, `*.p12`, or any file with credentials/tokens. `settings.local.json` is allowed (it has been backed up before) but skim it for secrets first.

5. **Commit + push:**
   - `cd <backup> && git add -A`
   - If `git status --porcelain` is empty, report "nothing to sync" and stop.
   - Otherwise commit with a concise message summarizing what changed (e.g. `sync: +2 memory files (untagged-cash, currency-pipelines), add /save /sync commands`).
   - End the commit message with the required co-author trailer.
   - `git push`

6. **Report** the commit hash, push result, and a one-line summary of what was mirrored.

## Notes
- Backup repo layout is modeled for the RApplication trading system (`project-RApplication/`, bare `memory/`). If you ever back up a *different* project, extend the mapping with a new `project-<name>/` folder and a per-project memory folder rather than overwriting `memory/`.
- README at `<backup>/README.md` documents the restore mapping; update it if the layout changes.
