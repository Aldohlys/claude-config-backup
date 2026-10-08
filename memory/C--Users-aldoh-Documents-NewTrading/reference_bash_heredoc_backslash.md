---
name: reference_bash_heredoc_backslash
description: "In this Git Bash setup, heredocs collapse double backslashes, which breaks R and Python regex written through cat <<'EOF'; write such files with the Write/Edit tools"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9768c881-e087-47ba-8fe2-7db428bfa064
  modified: 2026-10-08T08:17:49.725Z
---

On this machine (Windows, Git Bash tool), text passed through a heredoc (`cat > f <<'EOF'` or `python - <<'EOF'`) loses one backslash of each `\\` pair, even with a quoted delimiter. Seen on 2026-10-08 in three cases:
- an R test script: `"^.tabs\\{"` became `"^.tabs\{"`, giving "'\{' is an unrecognized escape"
- an R regex with `\\^`, which failed the same way
- a Python patch that wrote `\\n` into `calibrate_scenarios.py` as a real newline inside a regex

How to apply: write or patch any file containing regex escapes or backslashes with the Write or Edit tool, not a shell heredoc. Inside heredoc scripts, avoid backslashes entirely: use `fixed = TRUE`, `startsWith()`, or `str.replace` on plain substrings.
