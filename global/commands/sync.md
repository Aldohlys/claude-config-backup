# Sync Claude Config Backup

Mirror the local Claude Code configuration into the private backup repo
`Aldohlys/claude-config-backup` (local clone: `C:/Users/aldoh/Documents/claude-config-backup`)
and commit + push.

**INSTRUCTIONS FOR CLAUDE:**

This command uses **absolute paths**, so it backs up the same known config set regardless of which
project it is invoked from. It is the shorthand for the phrase "sync claude config backup".

Two projects keep their `.claude/` **gitignored** in their own repos, so this backup is their only
versioned copy: **RApplication** and **NewTrading**.

## Path mapping (source → backup repo)

```
~/.claude/CLAUDE.md                                              -> global/CLAUDE.md
~/.claude/settings.json                                          -> global/settings.json
~/.claude/commands/                                              -> global/commands/        (user-level slash commands, incl. /save, /sync)
RApplication/.claude/commands/                                   -> project-RApplication/commands/
RApplication/.claude/skills/                                     -> project-RApplication/skills/
RApplication/.claude/settings.local.json                        -> project-RApplication/settings.local.json
NewTrading/.claude/commands/                                     -> project-NewTrading/commands/     (/analyze, /transcribe)
NewTrading/.claude/settings.local.json                          -> project-NewTrading/settings.local.json
NewTrading/.claude/wheel_analysis.md                            -> project-NewTrading/wheel_analysis.md
~/.claude/projects/<any-project-key>/memory/                     -> memory/<any-project-key>/         (AUTO-DISCOVERED — every project, see step 4)
```

Absolute source roots:
- Global:          `C:/Users/aldoh/.claude`
- RApplication:    `C:/Users/aldoh/Documents/RApplication/.claude`
- NewTrading:      `C:/Users/aldoh/Documents/NewTrading/.claude`
- Projects root:   `C:/Users/aldoh/.claude/projects`   (each `<key>/memory/` is auto-discovered)
- Backup repo:     `C:/Users/aldoh/Documents/claude-config-backup`

## How to mirror

`rsync` is NOT available in this Git Bash. Mirror directories with `rm -rf <dest> && mkdir -p <dest> && cp -r <src>/. <dest>/`
(emulates `rsync --delete` so removed files propagate). Copy single files with `cp`.

A reusable helper:
```bash
mirror() { rm -rf "$2" && mkdir -p "$2" && cp -r "$1/." "$2/"; }
```

## Steps

1. **Global:**
   - `cp ~/.claude/CLAUDE.md <backup>/global/CLAUDE.md`
   - `cp ~/.claude/settings.json <backup>/global/settings.json`
   - `mirror ~/.claude/commands <backup>/global/commands`

2. **RApplication project:**
   - `mirror <RApp>/commands <backup>/project-RApplication/commands`
   - `mirror <RApp>/skills    <backup>/project-RApplication/skills`
   - `cp <RApp>/settings.local.json <backup>/project-RApplication/settings.local.json`

3. **NewTrading project:**
   - `mirror <NT>/commands <backup>/project-NewTrading/commands`
   - `cp <NT>/settings.local.json <backup>/project-NewTrading/settings.local.json`
   - `cp <NT>/wheel_analysis.md   <backup>/project-NewTrading/wheel_analysis.md`

4. **Memory (ALL projects — auto-discovered).** Do NOT hardcode project keys. Wipe `<backup>/memory`
   and rebuild it by iterating every project that has a `memory/` subfolder. This automatically
   picks up new projects and drops ones deleted locally:
   ```bash
   PROJ="C:/Users/aldoh/.claude/projects"
   rm -rf "<backup>/memory" && mkdir -p "<backup>/memory"
   for d in "$PROJ"/*/; do
     key=$(basename "$d")
     [ -d "$d/memory" ] && mirror "$d/memory" "<backup>/memory/$key"
   done
   ```
   Result: `<backup>/memory/<full-project-key>/...` per project (key = the dir name under `projects/`,
   e.g. `C--Users-aldoh-Documents-NewTrading`).

5. **Secrets — never copy.** `config.yml`, `.Renviron`, `Renviron.site`, `*.secret`, `*.key`, `*.pem`, `*.p12`, or any file containing credentials/tokens. `settings.local.json` is allowed (it has been backed up before) but **skim it for literal secrets first** — env-var *name* references (e.g. `ALERT_EMAIL_PASSWORD`) are fine; literal keys/passwords/emails are not.

6. **Review before commit.** Run `git status --short` and inspect any **deletions** — a removed file in the backup means the local source no longer has it. Before letting `cp`-mirror propagate a deletion, confirm the file is genuinely gone locally and not just **misfiled** under the wrong project folder (this is how an obsolete `analyze.md` once sat under `project-RApplication` when the real one lives under NewTrading).

7. **Commit + push:**
   - `cd <backup> && git add -A`
   - If `git status --porcelain` is empty → report "nothing to sync" and stop.
   - Otherwise commit with a concise message summarizing what changed, ending with the required co-author trailer.
   - `git push`

8. **Report** the commit hash, push result, and a one-line summary of what was mirrored.

## Notes
- Backup layout: `global/` (user `~/.claude`), `project-<name>/` per project for hand-curated config (commands/skills/settings/loose files), and `memory/<full-project-key>/` for memory — the memory tree is auto-discovered from `~/.claude/projects/*/memory/`, so new projects' memory is captured with no spec change. Adding a new project's **config** (commands/skills/settings) still needs a new `project-<name>/` block in steps 2-3; its **memory** is automatic.
- Keep `<backup>/README.md` in sync with the layout if it changes.
- The LF→CRLF warnings from git on Windows are normal and harmless.
