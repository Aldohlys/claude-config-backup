---
name: feedback_roxygen_helper_splits_block
description: "Don't insert a helper between a function's roxygen description and its @tags — it splits the block and drops the function's help page"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8392b3e4-5d8b-457c-9eab-5c60ef858317
---

Inserting `gonet_realized_fx` (with plain `##` comments) **between** getGonet's `#'` description and its `#'@returns/@export` tags split the roxygen into two blocks. The tag block had no title → `document()`: "Skipping; no name and/or title" and **getGonet.Rd was dropped** (the misplaced title block instead attached to the helper).

**Why:** roxygen attaches a *contiguous* `#'` block to the next object; any function or plain comment in the middle breaks the association.

**How to apply:** place new helpers **above the whole roxygen block** (or after the documented function), keeping each function's `#'` block immediately contiguous with it. Verify with a quick `devtools::document()` and check for "Skipping; no name/title".

Note: Tdata `man/` is **gitignored**, so `.Rd` churn is filesystem/installed-package only (not committed) and self-heals on the next build — the regression is cosmetic (`?getGonet` fails until rebuilt), not a repo problem.
