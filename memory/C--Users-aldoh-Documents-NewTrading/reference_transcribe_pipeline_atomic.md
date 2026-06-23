---
name: reference-transcribe-pipeline-atomic
description: transcribe.py is atomic — no resume / checkpointing. Crash mid-run forces full re-transcribe. Re-run with -SkipDownload to skip the 35MB Vimeo fetch. Wall-clock ~80 min for 50min audio at large-v3 int8.
metadata: 
  node_type: memory
  type: reference
  originSessionId: 0bb4ef14-7deb-4012-92d7-6a1eda0ffb8a
---

`C:\Users\aldoh\Documents\NewTrading\Transcripts\transcribe.py` (called by `Run-Transcribe.ps1`) processes the audio in a single pass and writes the .txt file line-by-line as Whisper emits each segment. There is **no checkpoint, no resume**. If the script crashes (e.g. UnicodeEncodeError on a stdout print, OOM, ctrl-C), the partial .txt is overwritten on the next run via `open(output_file, "w", ...)` and the entire audio is re-transcribed from zero.

**Cost of a crash:** ~80 min wall-clock per 50 min audio on `large-v3 int8` (~0.6× real-time). A mid-run crash followed by a full re-run = ~160 min total. The 2026-05-12 wherestrade run cost ~3.2 h instead of ~1.4 h because of a mid-run UnicodeEncodeError at audio position 10:09.

**Mitigation if a crash happens:**
- The audio file (`audio_<YYYYMMDD>.mp4` from yt-dlp) is preserved — it's the .txt that gets clobbered, not the source.
- Re-launch with `-SkipDownload` so the 35MB Vimeo fetch doesn't repeat:
  ```powershell
  pwsh -File Run-Transcribe.ps1 <VIDEO_ID> <YYYYMMDD> -SkipDownload
  ```
- Don't sleep / hibernate the machine mid-run.

**Crashes fixed in code (no longer a concern):**
- stdout UnicodeEncodeError on Whisper non-cp1252 glyphs → transcribe.py now reconfigures stdout/stderr to UTF-8 with `errors="replace"`.
- CWD mismatch (output landed in caller's directory) → Run-Transcribe.ps1 now self-anchors via `Set-Location -LiteralPath $PSScriptRoot`.

**If future improvement is desired:** add periodic checkpoint write to a sidecar file (e.g. last segment index) and resume from there on re-launch with `--resume`. Not done as of 2026-05-13.

See [[reference-tdata-install-topology]] for the analogous "rebuild requires restart" pattern in the R-side Python bridge.
