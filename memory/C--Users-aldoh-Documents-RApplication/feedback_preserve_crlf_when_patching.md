---
name: feedback_preserve_crlf_when_patching
description: "Patching a CRLF source file with a Python script: io.open in text mode converts to LF on write, turning a 5-line edit into a whole-file diff — open with newline='' and translate the search strings"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6df3dc0a-4099-43b9-b384-ce0952b42b01
  modified: 2026-09-03T18:57:29.705Z
---

2026-09-03, patching `Strategies/Breakouts/bot_scan_universe.py`. Eight small edits produced
a `diff` showing all 661 lines replaced. Cause: the file is CRLF; `io.open(p, encoding=...)`
reads with universal newlines (CRLF -> `\n`) and `io.open(p, 'w', ..., newline='')` then
writes the bare `\n` back. Content was correct; every line ending had flipped.

**The fix — open with `newline=''` on BOTH sides and translate the search strings:**

```python
s = io.open(p, encoding='utf-8', newline='').read()      # keeps \r\n

def sub(old, new):
    old, new = old.replace('\n', '\r\n'), new.replace('\n', '\r\n')
    assert s.count(old) == 1, f'expected 1 hit, got {s.count(old)}'
    ...

io.open(p, 'w', encoding='utf-8', newline='').write(s)
```

Without the `.replace` the assertions fail with 0 matches on text that is plainly in the
file — the same confusing symptom as the backslash-halving in
[[feedback_shell_quoting_traps]], different cause. Recovering afterwards works
(`data.replace(b'\r\n', b'\n').replace(b'\n', b'\r\n')`) but only if you notice; a
whole-file diff is easy to wave through as "just line endings", which is exactly how a real
change hides in one.

**How to apply:** check `git diff --stat` after every scripted patch. If the changed-line
count matches the file length, fix the line endings before doing anything else. Distinct
from [[reference_mydb_sql_dump]], where a full-file CRLF diff genuinely IS normal — that is
a generated dump, this is hand-edited source where it never is.

The single-hit `assert` on every `sub()` is worth keeping regardless: it caught a stale
search string mid-run and left the file untouched, because the write happens once at the end
rather than per-edit.
