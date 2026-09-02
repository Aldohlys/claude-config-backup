---
name: feedback_shell_quoting_traps
description: "Quoting rules that hold in a real terminal don't hold in these tools — Bash-tool heredocs halve backslashes, PowerShell here-strings leak @ delimiters when used in Bash, and double-backtick escaping parses unpredictably in PowerShell"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T02:23:11.682Z
---

Merged 2026-08-27 from three separate memories. Common thread: **this environment
exposes both a Bash tool and a PowerShell tool, and each adds a parsing layer that
a real terminal doesn't.** Don't assume terminal-correct quoting survives.

## 1. Bash-tool heredocs halve backslashes — use Write instead

`cat > file << 'EOF' ... EOF` in the **Bash tool** does not preserve backslashes
**even with the delimiter quoted**. Observed 2026-08-26: a Python patch script
written with `"\\\\s+"` (intended: the four characters `\\s+`, matching an R
source literal) landed as `"\\s+"`, so the comparison found 0 matches. Same
failure hit an R script using `grep("^\\.render", ...)` — R raised
`'\.' is an unrecognized escape in character string`. And a Python heredoc
containing a Windows path (`C:\Users\...`) died with
`SyntaxError: (unicode error) ... truncated \UXXXXXXXX escape`.

**Why:** the content passes through an extra shell-parsing layer before reaching
the heredoc, so one level of escaping is consumed regardless of the quoting.

**How to apply:**
- Any file content containing backslashes — R regexes (`"\\s+"`, `"\\d{2}"`),
  Python escapes, Windows paths, LaTeX — write with the **Write tool**, not a
  heredoc. In Python patch scripts, use raw strings (`r"C:\Users\..."`).
- If it must go through Bash, avoid the backslash entirely: build it as `chr(92)`
  in Python, or use `fixed = TRUE` / character classes in R.
- Symptom: an assertion counting 0 matches on text you can see in the target file,
  or R/Python complaining about an unrecognized escape. Verify with `cat -A` on
  both source and generated file before theorising.

## 2. `@'...'@` is PowerShell-only — it leaks into git messages from Bash

The `@'...'@` here-string is **PowerShell** syntax. Using it for a
`git commit -m` message inside the **Bash** tool produces a malformed message:
bash treats `@` as literal text and `'...'` as an ordinary single-quoted string,
so the commit subject gains a leading `@ ` and a trailing `@`.

**How to apply:** in the **Bash** tool, pass multiline commit messages via
`git commit -F <tempfile>` (write the message to a temp file first), or a real
bash heredoc — never `@'...'@`. Reserve `@'...'@` for the PowerShell tool.
Observed 2026-06-10 committing docs/TODO.md.

## 3. PowerShell: use `$bt = [char]96`, never double-backtick escaping

When generating SQL (or any text) needing literal backticks for identifier
quoting inside PowerShell strings, define `$bt = [char]96` at the top of the block
and use `"$bt$colname$bt"`.

**Why:** PowerShell's escape character *is* the backtick, so `` `` `` is
*supposed* to yield one literal backtick. In practice, when that pattern appears
inside a `ForEach-Object { "..." }` scriptblock, combined with `$_` expansion,
inside a multi-line here-string, or with further `-join` concatenation, the parser
fails with cryptic errors — `Argument manquant dans la liste de paramètres`,
`Le mot clé 'from' n'est pas pris en charge`, `Accolade fermante '}' manquante`.
The error cascade after the first parse failure makes it look like several blocks
broke when only one string was the culprit. Hit repeatedly in `diff_db.ps1` and
`merge_db.ps1` generating SQL for columns like `Comm.` in Trades.

```powershell
# BAD — fragile, parses unpredictably in complex contexts
$matchCond = ($keys | ForEach-Object { "s.``$_`` = t.``$_``" }) -join ' AND '

# GOOD — always parses cleanly
$bt = [char]96  # literal backtick
$matchCond = ($keys | ForEach-Object { "s.$bt$_$bt = t.$bt$_$bt" }) -join ' AND '
```

- Define `$bt` once per script that generates SQL; `[char]N` is the general escape
  hatch for any character PowerShell treats specially.
- Applies to both pwsh 7+ and Windows PowerShell 5.1.
- Symptom: a here-string compiles fine with the backtick-quoted variable
  references removed, and fails when they're added back.

Referenced indirectly in `docs/LESSONS_LEARNED.md` under the 2026-04-24 DB sync
section. Related: [[feedback_git_status_before_commit]],
[[feedback_rscript_segfault]].
