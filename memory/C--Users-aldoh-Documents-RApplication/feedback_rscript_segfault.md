---
name: feedback_rscript_segfault
description: Multiline R code passed via Rscript -e causes segfaults on Windows — use temp script files instead
type: feedback
---

Never pass multiline R code inline with `Rscript -e "..."` — it causes segmentation faults on this Windows environment.

**Why:** Windows shell quoting + Rscript -e interaction causes segfaults with complex multiline code (quotes, special chars).

**How to apply:** Always write R code to a temp file first, then run with `Rscript path/to/temp.R`. Clean up the temp file after use.
