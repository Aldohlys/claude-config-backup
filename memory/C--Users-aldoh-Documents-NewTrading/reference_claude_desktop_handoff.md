---
name: reference_claude_desktop_handoff
description: /desktop CLI→Desktop handoff fails with transcript_missing when run in a new empty session; desktop logs in %LOCALAPPDATA%\Claude\logs; claude.ai/code/session_ links not readable locally
metadata:
  node_type: memory
  type: reference
  originSessionId: 73edccd0-5cb1-4d69-9320-0509f2f70101
  modified: 2026-10-19T18:56:39.677Z
---

- `/desktop` in a session whose first command is `/desktop` fails: the desktop app tries to import the transcript before the CLI writes it (2026-10-12: import 17:28:54, `.jsonl` created 17:28:55, log `Failed to import CLI session ... category: 'transcript_missing'`). Running `/desktop` inside a session that already has turns works.
- Desktop app log on this machine: `%LOCALAPPDATA%\Claude\logs\main.log`. `%APPDATA%\Claude\logs` is an older install, last written 2026-08-30.
- A `claude.ai/code/session_...` link is a cloud session: the desktop session tools (`list_events`, search) don't find it and the built-in browser pane is not signed in to claude.ai. Ask for the branch it pushed instead (e.g. `claude/<name>` on the RStudies remote) and `git fetch`.

**How to apply:** use when the user moves sessions between CLI, desktop and web, or points at another session's work. Related: [[project_rstudies_checked_out_branch]].
