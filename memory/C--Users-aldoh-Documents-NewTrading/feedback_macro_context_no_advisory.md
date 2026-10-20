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

Same principle as [[feedback_analyze_data_only]]. Related: [[project_macro_context_redesign]].
