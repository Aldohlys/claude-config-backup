# /transcribe — Audio/video transcript → summary MD

Two modes:
- **Pipeline mode** — download a Vimeo video, transcribe, then summarize.
- **Summarize-only mode** — skip download/transcription; just summarize an
  existing transcript file (e.g. exported from Spotify, Otter, manual).

The summary format (narrative + every-ticker table + notable levels) is
identical in both modes. The summary is a **technical record**: every
trading/technical/macro idea in the transcript must survive into the
narrative. Only strictly personal/biographical content may be cut (see P5b).

**INSTRUCTIONS FOR CLAUDE — pick the mode first:**

- If **exactly one** argument is provided **and** it resolves to an existing
  file (absolute path, relative path, or bare filename found in
  `C:\Users\aldoh\Documents\NewTrading\Transcripts\`), use **Summarize-only
  mode** (jump to Step S1).
- Otherwise, treat as **Pipeline mode** (Step P1).
- If arguments don't fit either pattern, stop and ask.

---

## Pipeline mode

P1. Parse `$ARGUMENTS` as `VIDEO_ID OUTPUT_ROOT`.
   - `VIDEO_ID` (required): Vimeo numeric ID, e.g. `1189778472`.
   - `OUTPUT_ROOT` (required): filename root for outputs, e.g. `wherestrade`, `bigpicture`.
   - If either is missing, stop and ask the user for it.

P2. Compute today's date in two formats:
   - `YYYYMMDD` (for the existing pipeline, e.g. `20260511`)
   - `YYYY_MM_DD` (for the output filenames, e.g. `2026_05_11`)

P3. Run the pipeline from the `Transcripts` directory:
   ```powershell
   pwsh -File C:\Users\aldoh\Documents\NewTrading\Transcripts\Run-Transcribe.ps1 <VIDEO_ID> <YYYYMMDD>
   ```
   The CWD must be `C:\Users\aldoh\Documents\NewTrading\Transcripts\` so audio and transcript files land there. Run in the background (`run_in_background: true`) and let the harness notify on completion — transcription is CPU-bound at ~0.6× real-time on `large-v3 int8`, so a 50-minute audio takes ~80 min wall-clock.

   **MOTW gotcha — handle preemptively if it fires.** If the pipeline exits within seconds with `SecurityError: ... is not digitally signed`, the `.ps1` carries a Mark-of-the-Web Zone.Identifier stream (common after pulling files via browser/sync). `LocalMachine=RemoteSigned` blocks MOTW-flagged unsigned files. Fix with one local command (no policy change, no `-ExecutionPolicy Bypass` — that triggers the auto-mode classifier):
   ```powershell
   Unblock-File C:\Users\aldoh\Documents\NewTrading\Transcripts\Run-Transcribe.ps1
   ```
   Then re-run P3. Do **not** retry with `-ExecutionPolicy Bypass`; it is auto-mode-blocked and the classifier may flag follow-on commands in the same shell.

   To monitor progress mid-run, tail the background shell's output file: the last `[MM:SS]` line is the position in the audio, and the `Audio duration:` line near the top is the total. % progress = last_timestamp / Audio_duration.

P4. Once the pipeline finishes, rename the outputs to the requested root:
   - `wherestrade_<YYYYMMDD>_transcript.txt` → `<OUTPUT_ROOT>_<YYYY_MM_DD>_transcript.txt`
   - Leave `audio_<YYYYMMDD>.*` as-is (intermediate file).

P5. Read the renamed transcript (in chunks if it exceeds the Read tool's per-call limit — typical 1-hour podcast ≈ 2000 lines / >25k tokens; read in two passes via `offset` / `limit`) and write a summary at
   `C:\Users\aldoh\Documents\NewTrading\Transcripts\<OUTPUT_ROOT>_<YYYY_MM_DD>_summary.md`
   with this structure:

   ```markdown
   # <OUTPUT_ROOT> — <YYYY-MM-DD>

   **Source:** Vimeo <VIDEO_ID>
   **Transcript:** `<OUTPUT_ROOT>_<YYYY_MM_DD>_transcript.txt`

   ## Summary
   <Narrative covering every technical/trading idea raised — see P5b for the
   completeness rule. Organize by topic (or by interview segment if the
   transcript has clear segments). Length is determined by content density,
   not a paragraph cap.>

   ## Tickers / Assets Mentioned

   | Ticker / Asset | Type | Direction / Bias | Δ vs <prior_date> | Context |
   |---|---|---|---|---|
   | SPY | ETF | long | held | ... |
   | ... | ... | ... | ... | ... |

   <!-- The "Δ vs <prior_date>" column is included only when a prior summary
        with the same root exists (see P6b). Omit it entirely otherwise. -->

   **Dropped from prior coverage (<prior_date>):** <comma-sep list of tickers in
   the prior table but absent here; omit this line entirely if there are none
   or if there is no prior summary>.

   ## Notable Trade Ideas / Levels
   <bullet list of specific setups, strikes, levels, or trade structures
   discussed — only if explicitly mentioned in the transcript.>
   ```

P5b. **Summary depth rule — strict.** The summary is a *technical record*, not a tldr. Preserve every:
   - macro view, sector view, single-name view;
   - hedging philosophy, framework, mental model, decision rule;
   - structure mechanic (legs, strikes, ratios, target P&L, max risk, cost basis);
   - vol / skew / IV-percentile / term-structure observation;
   - regime call (trending / mean-reverting / corrective / breakout / consolidation);
   - specific level, target, stop, retracement, support/resistance;
   - historical analog (e.g. "2007 Bear Stearns", "2000 dotcom") with the reasoning;
   - book / paper / model reference, with the author and what the speaker said about it;
   - empirical claim or quoted statistic (breadth %, short interest, inventory draw rate, vol percentile, correlation %, Sharpe).

   Permitted omissions — and only these:
   - Strictly biographical detail with no trading implication (where someone went to school, family, hobbies, social anecdotes, sponsor reads, birthday wishes, beer reviews, sign-off banter).
   - Contact info (Twitter handle, email, LinkedIn) unless it informs a trade.

   If in doubt, *keep it*. Compression that drops a technical insight to save tokens is a failure mode — the user has explicitly flagged it. Length follows content; do not target a word count.

P6. **Ticker table rules — strict:**
   - Include **every** ticker/asset mentioned (no exception): equities, ETFs, indices, futures, FX, crypto, commodities, rates.
   - `Type`: Stock / ETF / Index / Future / FX / Crypto / Commodity / Rate / Sector.
   - `Direction / Bias`: long / short / neutral / watch / N/A — only what the speaker stated. Do not infer.
   - `Context`: one short phrase quoting or paraphrasing what was said (level, setup, thesis).
   - Sort alphabetically by ticker.
   - If the same ticker is discussed multiple times, consolidate into one row with the dominant bias.
   - Do **not** add tickers that were not mentioned, even if they are obvious "neighbors" (e.g. don't add QQQ just because Nasdaq was discussed — only add QQQ if QQQ was named).
   - When a ticker is genuinely ambiguous (e.g. `DJT` = Dow Jones Transports vs. Trump Media), give each meaning its own row.
   - Flag obvious ASR mistranscriptions in the Context column without "correcting" them silently (e.g. Whisper/Spotify renders "SOFR" as "sulfur futures" — note "transcript renders 'sulfur'").

P6b. **Comparison column — add when prior summaries exist with the same root.** Before writing the table, glob the output directory for `<OUTPUT_ROOT>_*_summary.md` (Pipeline mode) or `<basename_root>_*_summary.md` (Summarize-only mode) and pick the **most recent** prior-dated one (parse the date out of the filename). If at least one is found, add a column **`Δ vs <prior_date>`** (place it after `Direction / Bias`). Cell content per ticker, terse:
   - `new` — ticker was not in the prior summary's table.
   - `held` — same direction/bias as prior, no material change in level/target.
   - `was <prior bias>` — direction/bias changed (e.g. `was long`, `was watch`).
   - `target X → Y`, `level X → Y` — specific level/target moved.
   - `<short note>` — combine when both bias and level changed (e.g. `was long; target 480 → 520`).

   After the table, add a one-line section **`Dropped from prior coverage (<prior_date>):`** listing any tickers that appeared in the prior summary but are absent from this one. If none, omit the section.

   If **no** prior summary with the same root exists, omit the comparison column entirely — do not emit an empty column. Compare against the *single most recent* prior, not all priors (chain comparisons get noisy).

P6c. **MarkText / LaTeX dollar-sign gotcha — mandatory escape.** Many Markdown editors (MarkText in particular) parse `$...$` as inline LaTeX math, which mangles any sentence containing two dollar-quoted prices (e.g. `the call is worth $25 ... values it at $35` renders the whole span in italic math with spaces eaten). Always escape literal dollar signs as `\$` in the summary — applies to currency in narrative, table cells, and the levels list. Concretely:
   - `$25` → `\$25`
   - `~$3.30/gallon` → `~\$3.30/gallon`
   - `$200 oil` → `\$200 oil`
   - Bare `$` followed by a non-digit (e.g. variable references) does not need escaping, but the safe default in a summary file is to escape every `$`.

---

## Summarize-only mode

Use when the user already has a transcript file (e.g. Spotify export of a
podcast like `markethuddle_20260425_transcript.txt`) and only wants the
summary MD.

S1. Resolve the argument to a transcript file:
   - If absolute path → use as-is.
   - If relative → resolve from CWD, then from `C:\Users\aldoh\Documents\NewTrading\Transcripts\`.
   - If bare filename → look in `Transcripts\`.
   - File must exist; if not, stop and ask.

S2. Derive the summary path:
   - `<dir>\<basename_without_ext>_summary.md`
   - Strip a trailing `_transcript` from the basename before appending `_summary.md`.
     E.g. `markethuddle_20260425_transcript.txt` → `markethuddle_20260425_summary.md`.

S3. Read the transcript. Format is unknown — handle both:
   - **Whisper style:** `[MM:SS] line text` per line, no speaker tags.
   - **Spotify / Otter style:** `Speaker N` blocks, free-text lines, bare-number
     timestamps (`6:44`, `12:30`) on their own lines.
   - Speaker tags and timestamps are metadata; the ticker/summary extraction
     is based purely on content lines.

S4. Write the summary to the derived path, using the same template as Pipeline
   mode Step P5 (narrative + ticker table + notable levels). Adjust the header:
   - Title: basename root (e.g. `markethuddle`).
   - Date: infer from filename if possible (e.g. `20260425` → `2026-04-25`); omit if not inferable.
   - Replace `**Source:** Vimeo <VIDEO_ID>` with `**Source:** External transcript (\`<filename>\`)`. Do not invent a podcast/host name; if the speaker names the show inside the transcript, you may parenthesize it.

S5. Apply the **summary depth rule (P5b)**, the **ticker table rules (P6)**, the **comparison column rule (P6b)**, and the **dollar-escape rule (P6c)** verbatim — they hold equally here. The format of the input transcript (Whisper vs Spotify vs Otter) does not change what content survives into the summary. For P6b, the "root" is the basename root after stripping the date and `_transcript` (e.g. `markethuddle` from `markethuddle_20260425_transcript.txt`); glob for sibling `<root>_*_summary.md` files in the same directory.

---

## What this command IS NOT

- **Not a recommender.** No GO / NO-GO verdicts, no conviction labels, no edge commentary. Report what the speaker said; do not editorialize.
- **Not an interpreter.** If the speaker is ambiguous, mark `N/A` rather than guessing.

## Notes

- The underlying pipeline (`Run-Transcribe.ps1`) is the same one used for Big Picture Trading "Where's the Trade" videos — see `Transcripts/bigpicture_transcript_procedure.md` for setup, troubleshooting, and how to find the Vimeo video ID.
- If a transcription is already running, the wrapper queues this one behind it automatically.
- If the audio file `audio_<YYYYMMDD>.*` is already present, download is skipped.
