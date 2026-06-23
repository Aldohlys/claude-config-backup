---
name: positioning-r-automation-todo
description: "positioning.R is hand-curated from Saxo digest, drifts stale silently — feeds regime crowding score"
metadata: 
  node_type: memory
  type: project
  originSessionId: 8c65e4a9-c01b-4a86-95fe-f6292e3db0e8
---

# positioning.R hand-curation drift — automation TODO

Opened 2026-05-27 after the file went 10+ weeks without refresh and was discovered stale during a XOP/crude follow-up question.

## Current state

`RStudies/reports/macro_context/positioning.R` is a hand-edited list of 5 COT positioning entries (Crude, USD, Gold, Grains, Copper). The file header says "EDIT THIS FILE EVERY MONDAY after reading Ole Hansen's COT report" — but there's no reminder, alert, or staleness check.

It feeds `compute_positioning_stress()` in `scenarios.R`, which counts `extreme = TRUE` flags to compute `crowding_score` (up to 0.5 contribution). A stale file with old `extreme` flags can either over-weight crowding (regime stays "fragile" past the actual extreme) or under-weight it (missed new extreme).

## Why

- 2026-03-16 entry sat unchanged through 2026-05-27 (10 weeks). Crude extreme=TRUE was no longer true (specs unwound 80% from peak). Grains extreme=TRUE was also stale. Copper net=short was inverted (now net long).
- No CI/scheduled task ever touched it. No staleness banner in the macro_context HTML report.

## How to apply

When the user wants regime-scoring output and `COT_DATE` is more than ~10 days old:
- **Flag it before deriving anything** — don't silently use stale crowding contributions.
- Offer to refresh from raw CFTC ([[cftc-cot-urls]]) if Saxo digest unavailable.

## Possible fixes (not shipped)

1. **Staleness warning at read time** — `scenarios.R` could emit a `message()` when `COT_DATE` is >10d old; render that as a banner in the HTML.
2. **Auto-refresh from CFTC structured data** — CFTC publishes CSV/XLS feeds; could pull weekly via cron and update the 5 entries. Loses Ole Hansen's narrative judgment on `extreme` flag.
3. **Hybrid**: auto-pull raw numbers each Friday after CFTC release, leave `extreme` flag manually editable Monday after reading Saxo.

Option 3 is best — preserves the narrative judgment that makes the file useful while removing the silent-drift failure mode.

## Sources used in 2026-05-27 refresh

See [[cftc-cot-urls]] for the URL map.
