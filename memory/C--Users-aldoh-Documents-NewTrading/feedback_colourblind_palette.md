---
name: feedback_colourblind_palette
description: No green/amber/red signal colours in reports - user asked for blue / black / light red (colour-blind safe); yellow rejected for low contrast on white
metadata:
  node_type: memory
  type: feedback
  originSessionId: b2c1dc04-7dd9-4dc9-a48d-437eadf5cf6b
  modified: 2026-10-16T20:46:58.433Z
---

Signal colours in generated reports: blue = supports risk / good, black (or dark grey) = neutral, light red = drains / bad, dark red = critical. Values used in macro_context template.html (2026-10-16): blue #0072B2 (text #005A8C), neutral #333 (fill rgba(0,0,0,.05)), light red #D9534F (text #B03A35), dark red #8E1B1B. Links and accents in slate #3F4A5A, so blue only carries the signal meaning.

**Why:** user, 2026-10-16: "amber-green-red: this is not colorblind friendly - better use light red, black and blue". They also offered yellow/black/blue; yellow text on white has about 1.5:1 contrast, so blue/black/light red was kept.

**How to apply:** any new report, xlsx fill, chart or badge uses this triad, and the visible words say BLUE/BLACK/RED, not GREEN/ORANGE. Internal zone codes ("GREEN", "ORANGE") in macro_context R code were left unchanged on purpose. Still open in macro_context: the 13-colour scenario timeline palette (TIMELINE_COLORS, summary_render.R) and the xlsx header fill. Related: [[feedback_macro_context_layout]], [[project_macro_context_redesign]].
