---
name: reference-transcribe-pipeline-atomic
description: transcribe.py is atomic — no resume / checkpointing. Crash mid-run forces full re-transcribe. Re-run with -SkipDownload to skip the 35MB Vimeo fetch. Wall-clock ~80 min for 50min audio at large-v3 int8.
metadata: 
  node_type: memory
  type: reference
  originSessionId: 0bb4ef14-7deb-4012-92d7-6a1eda0ffb8a
  modified: 2026-10-15T17:32:19.549Z
---

`C:\Users\aldoh\Documents\NewTrading\Transcripts\transcribe.py` (called by `Run-Transcribe.ps1`) processes the audio in a single pass and writes the .txt file line-by-line as Whisper emits each segment. There is **no checkpoint, no resume**. If the script crashes (e.g. UnicodeEncodeError on a stdout print, OOM, ctrl-C), the partial .txt is overwritten on the next run via `open(output_file, "w", ...)` and the entire audio is re-transcribed from zero.

**Cost of a crash:** ~80 min wall-clock per 50 min audio on `large-v3 int8` (~0.6× real-time). A mid-run crash followed by a full re-run = ~160 min total. The 2026-05-12 wherestrade run cost ~3.2 h instead of ~1.4 h because of a mid-run UnicodeEncodeError at audio position 10:09.

**Mitigation if a crash happens:**
- The audio file (`audio_<NAME>_<YYYYMMDD>.mp4` from yt-dlp; see [[reference-bigpicture-archive-vimeo]]) is preserved — it's the .txt that gets clobbered, not the source.
- Re-launch with `-SkipDownload` so the 35MB Vimeo fetch doesn't repeat:
  ```powershell
  pwsh -File Run-Transcribe.ps1 <VIDEO_ID> <YYYYMMDD> [NAME] -SkipDownload
  ```
- Don't sleep / hibernate the machine mid-run.

**Crashes fixed in code (no longer a concern):**
- stdout UnicodeEncodeError on Whisper non-cp1252 glyphs → transcribe.py now reconfigures stdout/stderr to UTF-8 with `errors="replace"`.
- CWD mismatch (output landed in caller's directory) → Run-Transcribe.ps1 now self-anchors via `Set-Location -LiteralPath $PSScriptRoot`.

**If future improvement is desired:** add periodic checkpoint write to a sidecar file (e.g. last segment index) and resume from there on re-launch with `--resume`. Not done as of 2026-05-13.

See [[reference-tdata-install-topology]] for the analogous "rebuild requires restart" pattern in the R-side Python bridge.

**Throughput observed 2026-10-02..03 (incl. headless summary):** ~2.3-3.5 min wall per audio minute — e.g. MO 10-02 (24:48) ≈ 58 min, MO 09-29 (17:51) ≈ 80 min, WTT ~55 min audio ≈ 2.5-3 h. Plan overnight batches from that, not from the 0.6× figure above.

**Sleep:** the PC (HP Optimized, Modern Standby) had AC standby = 300 s and the pipeline sets no keep-awake, so unattended runs suspend after 5 min idle. Set `powercfg /change standby-timeout-ac 0` on 2026-10-02 (no admin needed); hibernate AC already 0. Original value was 5 min (the procedure doc's "restore 30" is not the user's setting). Check `powercfg /query SCHEME_CURRENT SUB_SLEEP STANDBYIDLE` before leaving a batch running.

**Screen-off = Modern Standby (2026-10-15):** with standby-timeout AC 0, the display timeout (VIDEOIDLE AC 300 s) still puts this HP into Modern Standby (event 506). On 10-15 the PC hard-hung in Connected Standby at ~11:25 mid-batch (Kernel-Power 41, BugcheckCode 0, ConnectedStandbyInProgress true; earlier CS crashes 04-18 and 07-30, non-CS 10-12), killing all 4 queued jobs. Chain scripts now hold `SetThreadExecutionState(ES_CONTINUOUS|SYSTEM|DISPLAY_REQUIRED)` for the run (scratchpad `transcribe_chain_20261015.ps1` is the template). Stale queue tickets in `%TEMP%\bpt_transcribe_queue\` contain `ID DATE NAME`, so they recover video IDs after a crash; delete them before relaunching, because PIDs can be reused after a reboot.

**Memory pressure (2026-10-08):** Whisper large-v3 used ~4.6 GB while transcribing; Claude Code reaped an idle background watcher (`until [ -f summary ]` loop) for low system memory. The transcription itself survived. Do not rely on a background shell to wait for a summary during a transcribe run; ask the user to say when it is done, or check the files on the next turn. Observed timing that evening: MO 15 min audio started 20:37, summary 21:31; WTT 30 min audio started 21:34, summary 22:48.

**Audio cleanup (user preference, 2026-10-16):** once `<NAME>_<DATE>_transcript.txt` and `_summary.md` both exist and are non-empty, the user wants `audio_<NAME>_<DATE>.mp4` deleted (asked twice that day). Offer it when reporting finished jobs; never delete audio of a job still running or queued (`%TEMP%\bpt_transcribe_queue\` tickets) — a crash re-run with `-SkipDownload` needs it.
