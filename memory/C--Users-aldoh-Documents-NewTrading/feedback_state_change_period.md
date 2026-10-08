---
name: feedback_state_change_period
description: "Every % or bp change in reports and chat must state the period it is computed over, unless obvious (e.g. a daily change)"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 9768c881-e087-47ba-8fe2-7db428bfa064
  modified: 2026-10-07T23:55:11.908Z
---

Always state the window of a variation: "Gold −6.9% over the last 21 sessions (1 month)", not "Gold −6.9%". Applies to generated report text (macro_context section 00 uses `P1M` in `intermarket_reads.R`; mismatch RS is labelled "20d"), tables and chat. A shared window may be stated once per sentence ("Wheat −6.1%, corn −1.8%, all over the last 21 sessions").

**Why:** the user asked on 2026-10-08 after section 00 printed "Gold −6.9%, a clear move" with no period.

**How to apply:** when writing or generating any change figure, attach its window (sessions or calendar span). Skip it only where it is obvious, such as a column headed 1D or a "last session" strip.
