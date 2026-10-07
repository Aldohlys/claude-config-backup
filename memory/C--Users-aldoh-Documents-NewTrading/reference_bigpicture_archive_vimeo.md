---
name: reference-bigpicture-archive-vimeo
description: "Big Picture archive page → Vimeo IDs (console snippet pairing date+show+ID); two shows/day need distinct NAME arg; transcription is FIFO-queued one at a time (RAM); yt-dlp reads Vimeo titles; ID spacing by weekday; batch = sequential chain; quirks (Fri MO only, 09-22 slots shifted, 09-24 Q&A ≠ WTT)"
metadata:
  node_type: memory
  type: reference
  originSessionId: becb9fa2-81de-4d0c-a163-304ae3530fc7
  modified: 2026-10-02T07:24:09.013Z
---

Archive page: `https://secure.bigpicturetrading.com/membership/bigpictureanalytics/members_home/archives.php` (login). Each weekday = Macro Outlook (MO, carousel, left) + Where Is The Trade (WTT, right); **Fridays = MO only**.

**Getting IDs:** F12 → Console, walk `h1..h6,iframe` in document order, keep last date heading (`/(MON|...)DAY.*\d{4}/`) and section heading (`/macro|where/`), extract `vimeo.com/video/(\d+)` from `src || dataset.src`. Prints `date  show  ID`. Worked first try on 2026-10-02.

**ID ↔ date:** no formula — Vimeo IDs are one platform-wide upload counter. Only order holds: MO < WTT same day (+40-70k), ~300-400k per weekday, ~630-770k over a weekend. Use the snippet, not inference. Refined 2026-10-05 over 09-08..10-02: same-day gap depends on weekday (Tue 30-55k, Thu 62-72k); WTT → next MO 267-359k; weekend days ~230k each vs ~350k per weekday. Labor Day week (09-08) gaps run low. Spacing narrows an unknown ID to ~25k candidates, never exact — get the real ID from BPT, not by probing Vimeo.

**Embed quirks (checked 2026-10-05):** 09-22 is mislabeled on the page: its MO slot holds the 09-22 WTT (`1229198074`) and its WTT slot the 09-21 WTT (`1228862294`); the 09-22 MO is not embedded (ID unknown, ~30-55k below 1229198074). Verify any ID's title with `yt-dlp --referer https://secure.bigpicturetrading.com/ --skip-download --print "%(title)s" https://player.vimeo.com/video/<ID>` (titles like "Macro Outlook - October 02, 2026"; all other rows matched on 2026-10-05). `wherestrade_20260924_transcript.txt` (`1229952716`) is the *Master Member Q&A*, not WTT (`1229920246`).

**Pipeline naming (commit d2986b5):** `Run-Transcribe.ps1 ID YYYYMMDD [NAME]`, NAME default `wherestrade`; files `audio_<NAME>_<date>.*`, `<NAME>_<date>_transcript.txt/_summary.md`. Same-day shows MUST get distinct names (`macro_outlook` / `where_is_the_trade`) — otherwise the 2nd run finds the 1st's audio, skips download, transcribes the wrong video. `/transcribe` Δ-vs-prior matches same root, so WTT history is under `wherestrade_*`.

**Queue (commit c1d1224):** one transcription at a time, FIFO via tickets in `%TEMP%\bpt_transcribe_queue\`; dead-PID tickets auto-dropped. Reason: large-v3 int8 ≈ 2.5 GB each; PC = i7-12700T, 15.7 GB RAM, TWS ~1.4 GB → 7 concurrent runs would exhaust RAM and risk TWS/BOT batch.

**Launching from PowerShell:** `run_transcribe.bat` is not on PATH — use full path `C:\Users\aldoh\Documents\NewTrading\scripts\run_transcribe.bat`; it ends with `pause`, so use `Start-Process $bat -ArgumentList "ID DATE NAME"` for parallel launch, or the Launcher (NAME box).

Related: [[reference-transcribe-pipeline-atomic]], [[reference-ceresna-macro-tenets]].

**Script + full ID table** (09-08..10-02, with titles check) live in `Transcripts/bigpicture_transcript_procedure.md` Step 1 — the 2026-10-02 study existed only in chat until 2026-10-02 evening; keep that doc current when new IDs are read. The browser oEmbed title script is likely blocked (domain-restricted embeds); yt-dlp with the referer works.

**Batch backlogs:** the user wants a sequential chain script (scratchpad `transcribe_chain*.ps1`: foreach job → `& Run-Transcribe.ps1 ID DATE NAME`, Start-Transcript log, launched with `Start-Process powershell -File`), not N parallel launches — avoids loading CPU/RAM even with the FIFO queue. Summaries run inside each job, so they are sequential too. 2026-10-05: 12 Macro Outlooks 09-08..09-24 queued this way (09-22 MO missing).
