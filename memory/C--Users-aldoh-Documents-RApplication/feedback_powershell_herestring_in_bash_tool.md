---
name: feedback_powershell_herestring_in_bash_tool
description: "PowerShell here-string @'...'@ leaks @ delimiters into git messages when used in the Bash tool"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4152f0ec-a745-46da-8951-1e46783a5f4b
---

The `@'...'@` here-string is **PowerShell** syntax. Using it for a `git commit -m`
message inside the **Bash** tool produces a malformed message: bash treats `@`
as literal text and `'...'` as an ordinary single-quoted string, so the commit
subject gains a leading `@ ` and a trailing `@` (the delimiters leak in).

**Why:** the tool-doc guidance to use `@'...'@` for multiline messages applies to
the **PowerShell** tool only. This environment exposes both a Bash tool and a
PowerShell tool — the here-string is not portable between them.

**How to apply:** In the **Bash** tool, pass multiline commit messages via
`git commit -F <tempfile>` (Write the message to a temp file first), or a real
bash heredoc — never `@'...'@`. Reserve `@'...'@` for the PowerShell tool.
Observed 2026-06-10 committing docs/TODO.md. Related: [[feedback_git_status_before_commit]].
