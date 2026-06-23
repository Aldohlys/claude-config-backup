---
name: feedback_no_tail_on_background_bash
description: tail -N (and similar) buffer until stdin closes, leaving the .output file empty for the entire run — lose all progress visibility
type: feedback
originSessionId: 29cdc5d5-050d-4ed2-a819-c20c72b37017
---
When launching a long-running command via `Bash(run_in_background: true)`, do
NOT pipe its output through `tail`, `head`, `grep`, or any other utility that
line-buffers waiting for EOF. The harness writes the *piped* result to
`.output`, so until the upstream process exits, the file stays at 0 bytes —
you have zero progress visibility, and `TaskOutput` only tells you `running`
without any partial content.

**Why:** observed during the Tdata 5.10.11 build on 2026-04-30. I ran
`... build_package(...) 2>&1 | tail -120` in the background to keep the final
output bounded. The build then hung in tests for 30+ minutes; the entire
time the .output file was 0 bytes, forcing me to debug via side-channels
(tasklist, `logs/t-*.log`, `git status`, `find -mmin`). Without `tail` the
.output would have shown the SPY HAR-RV log line where it hung in real time.

**How to apply:**
- For background bash tasks, stream output raw — no `| tail`, `| head`, `|
  grep`, `| awk` between the command and the implicit redirect.
- If you only want the *end* of the output, read the .output file with
  `Read` after completion, or use `tail -N` on the file path (not in the
  pipeline).
- For interactive monitoring, use `Monitor` on the .output file path.
- The same applies to any wrapper that buffers — `unbuffer` (from expect)
  or `stdbuf -oL` would work around it, but the simpler fix is "don't pipe."
