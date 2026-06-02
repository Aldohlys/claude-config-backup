---
name: Use $bt = [char]96 instead of escaped backticks in PowerShell strings
description: Double-backtick escaping in here-strings with variable expansion triggers cryptic parser errors; literal [char]96 is always safe
type: feedback
originSessionId: 20890e27-e237-4b5b-ad56-028d4f1ab618
---
**Rule:** When generating SQL (or any text) that needs literal backticks for identifier quoting inside PowerShell strings, define `$bt = [char]96` at the top of the block and use `"$bt$colname$bt"` — NEVER write `` "`\`$var\`\`" `` style double-backtick escaping.

**Why:** PowerShell's escape character is the backtick `` ` ``. Inside double-quoted strings (including here-strings), `` ` `` followed by another `` ` `` is *supposed* to produce a single literal backtick. In practice, when this pattern appears:
- inside a `ForEach-Object { "..." }` scriptblock
- combined with variable expansion like `$_`
- inside a multi-line here-string (`@"..."@`)
- with further concatenation or `-join`

the parser can fail with cryptic messages like `Argument manquant dans la liste de paramètres` (missing argument in parameter list), `Le mot clé 'from' n'est pas pris en charge` (keyword 'from' not supported), or `Accolade fermante '}' manquante` (missing closing brace). The cascade of errors after the first parse failure makes it look like you broke multiple blocks when only one string was the culprit. Happened multiple times this session in diff_db.ps1 and merge_db.ps1 while generating SQL with backtick-quoted SQL identifiers (needed to handle columns like `Comm.` in Trades).

**How to apply:**

```powershell
# BAD — fragile, parses unpredictably in complex contexts
$matchCond = ($keys | ForEach-Object { "s.``$_`` = t.``$_``" }) -join ' AND '

# GOOD — always parses cleanly
$bt = [char]96  # literal backtick
$matchCond = ($keys | ForEach-Object { "s.$bt$_$bt = t.$bt$_$bt" }) -join ' AND '
```

- Define `$bt` once at the top of the function or script that generates SQL.
- Apply the same pattern for any character PowerShell treats specially in string contexts — `[char]N` is the escape hatch.
- Same applies to PowerShell-Core (pwsh) and Windows PowerShell 5.1 — neither handles the double-backtick pattern reliably in all contexts.
- Symptom pattern: if a here-string compiles fine when you remove the backtick-quoted variable references, but fails when you add them back, this is the issue.

Referenced in `docs/LESSONS_LEARNED.md` indirectly under the 2026-04-24 DB sync section (backtick quoting was needed to handle `Comm.` column in Trades).
