---
name: reference-marketlife-transcribe
description: "MarketLife Trading videos (marketlifetrading.com/videos/show?id=N, login) are Vimeo embeds; Run-Transcribe -Referer marketlifetrading; SE → marketlife_stocks_options, FE → marketlife_futures_forex; .bat REFERER box pending"
metadata:
  node_type: memory
  type: reference
  originSessionId: f580ecee-39df-4dfe-ac2b-699cec815aeb
  modified: 2026-10-16T05:32:28.746Z
---

`https://www.marketlifetrading.com/videos/show?id=N` needs a login (redirects to /users/sign_in). The player is a Vimeo embed: the user gets the ID from F12 (iframe src `player.vimeo.com/video/<ID>`).

Vimeo only serves it with `--referer https://www.marketlifetrading.com/`. Since 2026-10-16 `Run-Transcribe.ps1` takes a 4th positional `-Referer` = site key (`bigpicturetrading` default, `marketlifetrading`; map `$RefererSites`) or full URL; NAME must precede it positionally:
`.\Run-Transcribe.ps1 <ID> <YYYYMMDD> marketlife_stocks_options marketlifetrading`
Committed b212ca6 (pushed). **Pending:** `scripts/run_transcribe.bat` `REM @args` line still lacks a `[REFERER]` token, so the launcher has no REFERER box; add it (+ usage echo), then commit/push. Do not edit a .bat while any job runs from it (cmd reads .bat from disk by byte offset) — check `Get-CimInstance Win32_Process` for `run_transcribe` first.

Two series, distinct NAMEs (user's choice, 2026-10-16): title prefix "SE" → `marketlife_stocks_options`, "FE" → `marketlife_futures_forex`. Titles end MMDDYY ("SE 101626"). First pair 2026-10-16: SE = Vimeo 1236369995 (11:11, page id=5785), FE = 1236369996 (11:37) — consecutive IDs, so the FE ID is probably SE+1 (confirm the title with yt-dlp --print before downloading).

Related: [[reference-bigpicture-archive-vimeo]], [[reference-transcribe-pipeline-atomic]].
