---
name: project_rstudies_checked_out_branch
description: "RunScanner 09:00 runs whatever branch RStudies has checked out; claude/vibrant-newton-2xvfgc still checked out but fast-forwarded into main on 2026-10-19 (identical at 1f6d535)"
metadata:
  node_type: memory
  type: project
  originSessionId: 73edccd0-5cb1-4d69-9320-0509f2f70101
  modified: 2026-10-19T19:46:54.104Z
---

On 2026-10-12 RStudies (`RApplication/RStudies`) was switched from `main` to `claude/vibrant-newton-2xvfgc` (macro_context summary band, cross-asset heatmap, scenario map). On 2026-10-19 `main` was fast-forwarded to the branch and pushed (both at `1f6d535`, scenario-map trail start dot). The branch is still the checked-out one, so new commits land on it, not on `main`.

**Why:** the scheduled RunScanner task runs the checked-out working tree, not `main`, so production reports follow whatever branch is checked out.

**How to apply:** before claiming what the morning reports contain, check `git branch --show-current` in RStudies. If commits land on the branch again, `main` falls behind; offer to switch the checkout to `main` (identical as of 10-19). `reports/macro_context/positioning.R` usually sits modified from the Saturday COT refresh; it is committed by hand and survives checkouts. Related: [[project_macro_context_redesign]], [[project_positioning_r_automation_todo]].

**Update 2026-10-20:** the checked-out branch is now `claude/macro-context-main-detail`, created from `claude/vibrant-newton-2xvfgc` for the Main/Detail/Legend port; it was uncommitted at creation. The next RunScanner runs it. Merge it into main once the user approves. Pushed to origin 2026-10-20; last commit d72d643.
