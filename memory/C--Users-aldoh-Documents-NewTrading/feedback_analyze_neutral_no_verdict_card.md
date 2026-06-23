---
name: /analyze — no verdict card, no edge commentary
description: Strip the GO/NO-GO verdict card and "Edge commentary" narrative from /analyze output; replace with phase-result table.
type: feedback
originSessionId: a5c2009f-c1bb-4d78-aafa-fafb8bd015a9
---
`/analyze` reports must NOT include:
- A `verdict-card` HTML block with `Verdict: NO-GO`, `Conviction: LOW`, `Edge source:`, `Reason:`, or `Counterpoint:` lines.
- An "Edge commentary" prose paragraph at the end of the terminal output.
- Words like "favors", "viable only if", "lower-high failure" — these are directional reads dressed as facts.

Replace with a neutral **Phase Result Summary** table: phase name, result badge, single driver fact per row. End the report there.

**Why:** User has corrected this twice now (2026-04-28). Memory `feedback_analyze_neutral_stance.md` already specifies "bare facts only", but I kept slipping verdict-card and edge-commentary back in via the spec's Step 4 template. The template is wrong — neutrality wins.

**How to apply:**
- In Step 4 (terminal verdict), produce the phase-result block but skip the `VERDICT: / Conviction: / Edge source:` lines entirely.
- In Step 5 (HTML), replace `<div class="verdict-card">…</div>` with a `<table>` of phase results.
- In Step 6 ("Edge Source Commentary"), skip the section. Spec says "brief narrative" but neutrality overrides.
- The user reads tables, draws conclusions, takes the trade. /analyze is a data presenter, not an advisor.
