---
name: newtrading-repo-policy
description: "What the NewTrading git repo tracks vs ignores; canonical \"should I commit X here?\""
metadata: 
  node_type: memory
  type: project
  originSessionId: 8a79388b-e664-4358-bbbe-befceb1eaf17
---

NewTrading (`Aldohlys/NewTrading`, branch `master`, see [[project_newtrading_remote_todo]]) is the trading-advisory workspace. As of 2026-06-09 the user chose to back up content **including binary office docs**, not just text.

**Tracked:** `.md`, `.R`, `.py`, `.txt`, `.bat`, and binary docs `.docx` `.odt` `.xlsx` `.xlsm` `.ods` `.pdf` `.csv`, plus reference screenshots (`.png .jpg .jpeg .url`).

**Ignored (.gitignore):**
- Generated/regenerable: `*.html` (analyze_/macro/scanner reports), `*.json`, `*.mp4` (Transcripts audio ~297MB), `*.parquet`, `*surface.log`, `*.stdout`, `__pycache__/`, `*.pyc`
- Data dirs: `quotes/ logs/ chains/ strikes/ Reports/reports/ Archive/ Téléchargements/`
- `Discussions/` — personal correspondence, deliberately kept off GitHub
- Office lock/temp files `~$*`, `~WRL*`, `~BROMIUM/`, `desktop.ini`, `*.tmp`
- `.claude/`
- **5 large third-party textbooks (>15MB)** excluded by explicit path to avoid permanent history bloat: McMillan "Options as a Strategic Investment" (×2 editions), Hull 4th, Murphy "Technical Analysis", "In Gold We Trust 2022". They remain on disk, re-downloadable. Smaller third-party PDFs (Natenberg, Sinclair, Lang, Hull 9th) ARE tracked.

**Why:** user wanted off-machine backup of the whole knowledge base (formation notes, trade studies, calculators), accepting binary-doc tracking. `.git` is ~372MB as a result — fine for GitHub but every clone pulls it. If it ever feels heavy, lean alternative = cloud-store reference binaries, track only own analysis.

**How to apply:** before committing a new file type, check it isn't a regenerable artifact or a >15MB third-party book. `change.log` is a tracked `.log` — never add a blanket `*.log` ignore (would shadow it); target scratch logs by name. See [[reference_changelog_convention]].
