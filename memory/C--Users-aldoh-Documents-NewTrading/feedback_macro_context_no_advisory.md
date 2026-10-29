---
name: feedback_macro_context_no_advisory
description: "macro_context shows what happened historically (hit rate vs base, n, source), never sizing or trade instructions"
metadata:
  node_type: memory
  type: feedback
  originSessionId: cfd01905-af04-48c7-8713-2272d5ea18b3
  modified: 2026-10-20T15:13:14.666Z
---

On 2026-10-20 the user asked where the regime × risk cell texts came from ("Full size", "Smallest size, defensive longs only"...). They warned: "beware of the thin line, no trading advisory is to be done here." Those texts were my own prescriptions. Only the VIX 77% figure came from data.

**Why:** a report that tells the user what size to trade or which side to take is advisory. Instructions mixed with measurements also hide which parts are evidence and which are opinion.

**How to apply:**
- In macro_context (and its prototypes), each cell or line states what followed historically:
  - hit rate vs base rate;
  - sample size (Fridays / episodes);
  - source script.
- Write "Not tested" where no data exists.
- Label the line "What followed historically", not "Read".
- No size words (full / reduced / smallest), and no do / don't verbs.
- The scenario texts in `archetypes.R` (`bot$note`, e.g. "Exit energy longs on the first sign…") are advisory in the same way, so flag them when porting.
- Done 2026-10-20: section 08 `events.R` action texts and the FOMC/Mag 7 rows were rewritten as neutral descriptions or measured history (RStudies 4956bd5). `archetypes.R` `bot$note` + yen-carry `bot` rewritten the same day as 7-force-analog history or "Not tested."
- Done 2026-10-20 (f17834e): the `archetypes.R` `bot$note` texts and the yen-carry BOT text now cite the closest step-2 analog's history. Still advisory-style: the VIX ≥ 25 stress banner in render_html.R ("Consider waiting for VIX to settle below 25").

Same principle as [[feedback_analyze_data_only]]. Related: [[project_macro_context_redesign]].
