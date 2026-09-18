---
name: feedback_verify_before_acting
description: "Four forms of the same discipline — re-verify an 'orphaned/unused' claim before destroying anything, re-run the survey behind a stale TODO, check whether a 'NEVER do X' policy's reason still holds, and check whether a TODO's stated residual was already closed by a spawned item"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T02:23:35.661Z
---

Merged 2026-08-27 from three separate memories; a fourth added 2026-09-03. Same
shape each time: **a confident-looking claim (from a sub-agent, a recorded
inventory, a written policy, or a TODO's own Status line) is a hypothesis, not
ground truth.** Re-derive it before you act on it.

## 1. Re-grep "orphaned / unused / write-only" before destroying anything

Before executing destructive DB/filesystem operations (`DROP TABLE`, `rm`,
`git clean`) based on a sub-agent's classification, **personally re-verify** with
a direct Grep across all downstream consumers — not just the writer's own module.

**Why:** 2026-04-20, an Explore sub-agent classified `macro_context_results` and
`macro_context_mismatches` as "write-only" (it found `dbWriteTable` in
`macro_context/main.R` only). Both tables were dropped without verification. The
scanner's `swing_scanner/main.R:45` and `:49` **read** them — the reads were
`tryCatch`-wrapped, so there was no visible crash, just silent degradation of the
macro overlay. Had to restore from backup.

**How to apply:**
- Treat "orphaned"/"write-only"/"unused" as hypotheses.
- Run a fresh targeted Grep for the table/file/symbol across *all* relevant file
  types (`.R`, `.py`, `.sql`, `.ps1`, `.md` templates), not just the owning module.
- Watch especially for `tryCatch`-wrapped reads (they look unused when they return
  NULL), indirect reads via views/joins, and reads in downstream dashboards.
- If in doubt, keep it. Dropping is cheap to reverse *only if* a backup exists.

## 2. Re-run the survey behind a stale TODO

For any TODO whose body is "here is an inventory, now execute against it", re-run
the underlying survey before staging anything. The recorded inventory is a
snapshot, not ground truth.

**Why:** TODO #55 (clean uncommitted state across 9 repos) was surveyed
2026-04-21 and executed 2026-04-30. In those 9 days `mydb.sql` and
`update_missing_iv_safe.R` had already been committed, RStudies had a pile of new
research scripts (catalyst studies, IV-vs-spot-lag, ivol probe), Tdata had new
in-progress work, and new untracked items had appeared (`scripts/Tuser-ticker.bat`,
`quotes/`, `Tuser/ticker/`). Trusting the snapshot would have produced wrong
commits **and** missed real work.

**How to apply:** run the equivalent of the original survey (`git status` across
all repos, file-existence checks), diff against the recorded inventory, and
surface the deltas before staging. Tighter the older the TODO; non-negotiable past
about a week.

## 3. Check whether a "NEVER do X" policy's reason still holds

When a memory, CLAUDE.md, or TODO says "NEVER do X", don't silently defer. Verify
whether the underlying reason still applies, surface the evidence, and let the
user re-decide.

**Why:** TODO #55 and CLAUDE.md both said `config.yml` must NEVER be committed
because of secrets. A full grep + read across all 7 `config.yml` files showed they
were entirely secret-free — the real secrets (`ALERT_EMAIL_PASSWORD` etc.) live in
`Renviron.site`, separated by design. The rule was a defensive over-generalization;
silently honouring it would have left 7 byte-identical configs un-versioned and
drifting. Surfacing the evidence let the user flip the policy in one round-trip.

**How to apply:**
1. Identify the stated reason (secrets, blast radius, past incident).
2. Check whether it still holds for the current files — read them, grep the pattern.
3. If it no longer applies, present the evidence and ask whether to keep or relax.
4. If it DOES still hold, follow the rule.

Never just nod and skip the work the rule is blocking — that's how policies outlive
their justification. Distinct from asserting *code* facts without checking; this is
about deferring to *policy* facts without checking. See
[[project_rapplication_repo_policy]], [[feedback_git_status_before_commit]].

## 4. Check whether a TODO's residual was already closed by a spawned item

A TODO's `Status` / `Priority` line is maintained by hand and is not updated when
the work migrates into a **spawned** TODO that then closes on its own. Before
picking up a long-lived item — especially a high-priority one — read its residual
against `docs/TODO_COMPLETED.md` and against the DB, not just the entry itself.

**Why:** 2026-09-03, #38 "Revisit Risk Data Management" was the *only* HIGH item
in the file. Both residuals keeping it there had in fact been resolved by **#75**
on 2026-06-15 — three months earlier: the `modify_trade` signed-delta invariant
(#75 item 1, where keeping Risk editable + warning was chosen over locking the
field) and the MaxRisk editor UI (#75 item 3, resolved as "edit in DB Browser").
Working #38 as written would have re-done decided work. Meanwhile the *real*
remaining scope was something the residual never mentioned: Phase 5 was
going-forward only, so 256 of 674 closed trades still fail `sum(Risk) = 0`.

**How to apply:**
- Grep `TODO_COMPLETED.md` for the item number and for its spawned children before
  starting; a "Spawned TODOs" list at the bottom of an entry is the tell.
- Re-measure the claim against live data where it is measurable (one SQL query
  settled both what was done and what was left).
- Rewrite the entry to the work that actually remains and re-rate it, rather than
  leaving a stale priority to keep mis-ranking the backlog. Note in the entry
  which residuals closed where, so the next reader does not repeat the check.

Same failure mode as a stale memory: [[feedback_todo_done_to_completed_section]]
pointed at a COMPLETED section that no longer existed, and
[[reference_live_virtual_account_no_table]] described `getAccountLive()` behaviour
that had been rewritten. Records rot; re-verify the ones you are about to act on.

## UPDATE 2026-09-15 — an old TODO's plan and "current state" age with the code

- **#16** (plan from 2025-12) said to remove `accountUI$input()`; by now `input()` also held the Update and Cash Flow buttons and the module served two Tuser apps. Followed literally it would have removed the buttons and broken both apps — read the module in full before designing.
- **#18** said date-only DTE "defaults to 16:00"; the code reads midnight (~10% error a week out), so its "current behaviour is reasonable" premise was false. See [[reference_getdte_date_midnight]].
- **#54 and #60** were effectively done: 81 logged scheduled runs and 52 real `/analyze` reports already satisfied their done-when lists. Closing them needed evidence from logs and outputs, not new runs.
- **#53**'s premise ("the time limit failed to kill a hang") dissolved once the long runs were matched to PC sleep.

**How to apply:** before implementing or re-prioritising an old TODO, check its "current state" claims against the code, and look for logs or generated reports that already answer its done-when criteria.

## UPDATE 2026-09-15 (b) — scanning for dead code without false positives

The TODO #74 pass (49 functions removed) worked by token-scanning every `.R`/`.py`/`.Rmd`/script/slash-command file in RApplication and NewTrading for names referenced nowhere but their own definition. What made it trustworthy:
- **Re-scan after each removal** — every pass orphaned more helpers (`get_swiss_bond_yields`, then `.d2`/`.nd1`, then `.d1`); stop only when a scan finds nothing new.
- **Skip `.Rproj.user`** when checking whether a deleted file is still referenced: RStudio's editor history names every file ever opened and produced a dozen false "references".
- **Scripts without an entry point may be console tools, not dead code** — `complete_cache_reset.py` and `RReporting/scripts/validation_functions.R` had no callers by design; the first also turned out to be unimportable. Ask before deleting a whole utility script.
- **String-built calls** (`do.call`, `match.fun`, `get(paste...)`, `getattr`) defeat a token scan — grep for them near each candidate.
- Check the emptied-file deletions and the second-wave orphans with the user's "unreferenced, keep exports" rule, and keep templates/archive folders out.
