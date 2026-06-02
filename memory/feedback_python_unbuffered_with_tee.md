---
name: Use python -u when piping through tee on Windows or output buffers indefinitely
description: Python's stdout buffers when not connected to a tty (i.e. through a pipe). With `python script.py | tee log.log`, output stays empty until the script exits — masks hung scripts as still-running and stalls live diagnostics.
type: feedback
originSessionId: 110f4600-4558-489d-88b2-cdf4eb853b4a
---
When invoking Python scripts from a Bash/PowerShell pipeline like:

```bash
python script.py 2>&1 | tee log.log
```

Python detects stdout is not a tty and switches to block-buffering. The `tee` then sees nothing until the script exits or buffers fill (~8 KB). For scripts that print progress and may hang on a TWS call, this looks identical to "still running" — there's no way to tell from the log whether the script is making progress or stuck.

**Why:** CPython's stdout uses `sys.stdout` line-buffered for ttys, fully-buffered otherwise. PYTHONUNBUFFERED env var or `-u` flag override this. Hit on 2026-05-10 testing TWS scripts: first `python test_mcln6_fix.py | tee log.log` ran for 2+ minutes with empty log, killed it as "hung"; re-ran with `python -u` and saw it had been past Step 1 the whole time.

**How to apply:**
- Always use `python -u` when piping through `tee`, redirecting to a file, or running in background where you want to see live progress:
  ```bash
  "C:/Users/aldoh/AppData/Local/r-miniconda/envs/r-reticulate/python.exe" -u script.py 2>&1 | tee log.log
  ```
- Alternatively set `PYTHONUNBUFFERED=1` before invocation.
- For Rscript via reticulate: stdout from Python is already routed through R's connection, behaves differently — this rule applies only to direct `python script.py` invocations.
- Same trap exists with `subprocess.Popen` capturing stdout — pass `bufsize=1` and `universal_newlines=True` for line buffering.
