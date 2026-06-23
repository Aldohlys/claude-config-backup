---
name: reference_claude_headless_slash_command
description: Run a project slash command headlessly (scripts/automation) with auto file-writes
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6521c13e-ba17-4066-985c-d891cb840ed3
---

To invoke a Claude Code **project slash command** from a script (non-interactive, auto-writing files):

```
claude -p "/<command> <args>" --permission-mode acceptEdits
```

- `-p` / `--print` = headless (print result and exit). `--permission-mode acceptEdits` lets it create/edit files without prompting (needed for unattended runs).
- **cwd must contain (or be under) the `.claude/commands/` dir** that defines the command — Claude Code resolves project commands from cwd and ancestors. A single existing-file arg routes to "summarize-only"-style modes if the command branches on that.
- The `claude` CLI on this machine: `/c/Users/aldoh/.local/bin/claude` (v2.1.x), authenticated. Headless `claude -p "..."` works and respects per-project `.claude/settings.local.json`.

Reference impl: `NewTrading/Transcripts/Run-Transcribe.ps1` Step 5 runs `claude -p "/transcribe <transcript>" --permission-mode acceptEdits` from the Transcripts dir to auto-write the summary — see [[project_transcribe_autosummary]]. Validated 2026-06-02 (produced an 18KB summary).

Verify flags exist with `claude --help | grep -E "print|permission-mode"`.
