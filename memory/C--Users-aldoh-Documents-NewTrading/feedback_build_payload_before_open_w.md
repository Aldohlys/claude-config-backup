---
name: feedback_build_payload_before_open_w
description: "open(path,'wb') truncates before the write argument is evaluated — build the payload first, write to .tmp, os.replace; back up user data files before mutating them"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: bcd2f874-bc6c-429b-a57e-f3de0e852923
  modified: 2026-09-22T07:58:14.212Z
---

When a script rewrites one of the user's data files, build the complete payload into a variable
first, write it to a temporary file, then `os.replace()` it over the target. Never evaluate an
expression inside the `write()` call of an already-opened handle.

**Why:** `open(path, 'wb')` truncates the file the moment it is opened. If the expression passed to
`write()` then raises, the file is already at zero bytes and the original content is gone. On
2026-09-22 this destroyed `GonetTrades.csv` — a `TypeError` from joining `str` lines with a `bytes`
separator fired after the handle had opened, and the next run asserted on a 0-line file. It was
recoverable only because a copy had been taken one command earlier.

**How to apply:**

- Before the first mutation of any file the user maintains by hand, copy it to the scratchpad. Do
  this in the same command that starts the work, not as a later step.
- Write as: build `payload` -> `open(tmp,'wb').write(payload)` -> `os.replace(tmp, path)`. The
  replace is atomic, so a failure anywhere earlier leaves the original untouched.
- Assert the expected shape after reading and before writing (`assert len(lines) == 46`), so a
  half-written file fails loudly on the next run instead of being silently extended.
- After restoring from a backup, prove it: compare `md5sum` against the copy and check `git diff`
  shows only the user's own pre-existing changes. Say plainly that it was restored and verified —
  do not bury it.

Same family as [[feedback_verify_price_move_vs_close]]: verify against the primary artifact rather
than assuming the operation did what it looked like it did. Ledger context:
[[reference_gonet_csv_bookkeeping]].
