---
name: feedback_verify_before_acting
description: "Three forms of the same discipline — re-verify a sub-agent's 'orphaned/unused' claim before destroying anything, re-run the survey behind a stale TODO before executing it, and check whether a 'NEVER do X' policy's reason still applies instead of silently deferring"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T02:23:35.661Z
---

Merged 2026-08-27 from three separate memories. Same shape each time: **a
confident-looking claim (from a sub-agent, a recorded inventory, or a written
policy) is a hypothesis, not ground truth.** Re-derive it before you act on it.

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
