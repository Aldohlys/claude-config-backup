---
name: feedback_map_all_fetch_sites_before_optimizing
description: "When optimizing fetch/perf across a pipeline, inventory ALL call sites up front — don't point-fix one log at a time"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 65083a85-2e71-48fe-97b7-db8b02bd59a8
---

During the 2026-06-09 /analyze option-fetch leaning, I fixed sites serially across many turns — Phase A spread probe, then the funnel skew/IV, then `getVolMetrics`, then Phase D OI, then the spread enumerator — each surfacing only after the user re-ran and pasted the next log. The user grew (reasonably) impatient with the whack-a-mole.

**Why:** a multi-stage pipeline has many similar call sites; fixing the loudest one just reveals the next. Chasing them one-by-one is slow and feels like thrashing.

**How to apply:** on a "too much data / too slow" complaint about a pipeline, FIRST produce a complete inventory of the relevant calls before editing — `grep` the whole flow for the fetch primitive(s), and from one full run/log enumerate every site with its magnitude (e.g. strikes qualified + quotes fetched per phase). Fix them as a batch, then verify once. I did eventually pivot to "stop point-fixing, map the whole run" — start there next time.

Pairs with [[feedback_verify_before_claiming_codebase_facts]] (grep wide first) and the concrete inventory in [[reference_tdata_option_fetch_internals]].
