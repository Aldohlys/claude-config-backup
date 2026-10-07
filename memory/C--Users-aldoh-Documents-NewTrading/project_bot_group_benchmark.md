---
name: project-bot-group-benchmark
description: BOT S3/rs20 benchmark = correlation group (anchor ETF outside group, else peer median ±50%), BOT_Bench only for ungrouped; group rotation rank grp_rank in BOT_daily detail
metadata:
  type: project
---
Since 2026-10-05 (RStudies 0cc5879, 4b24ca8; BOT_TOOLS_DESIGN v3.38-3.39):
- **S3 / `rs20` benchmark** (`rs_bench`, `shared/bot_read.R::.bot_group_bench`):
  - group anchor (`ScannerUniverse.ClusterETF`) when it sits outside the group, which by construction is a universe ETF;
  - else the median 20-day return of the other members, each within ±50%. This covers member ETFs (SMH, GDX) and stock anchors (CF, TMO).
  - `Tickers.BOT_Bench` only for ungrouped names.
  - First run: 149 names on an anchor or BOT_Bench, 171 on a peer median, 4 without a benchmark.
- **Rotation rank** (`group`, `grp_rs`, `grp_rank`, BOT_daily `--detail` only, NULL in /analyze): the group's 20-day return minus SPY's, ranked across 55 groups.
  - `group_rotation_test.py`: top-3 groups +0.30 ATR over 10 sessions (t 2.3 on non-overlapping windows), hit rate +3-5 pp only.
  - Stock rs20 against its group: no effect (t −0.4).
  - Reported, not gated.

**Why:** the group fits better than BOT_Bench: median correlation 0.70 anchor / 0.72 peers vs 0.63 (in sample). 61 grouped names had no BOT_Bench, and 36 member anchors were their own benchmark.

**How to apply:** the anchor type (ETF or stock) is not stored in the DB. A member ETF anchor is therefore treated as a basket, a deliberate simplification. Do not score grp_rank into tiers without an out-of-sample check, because the last year is in sample for group membership.

Open data issue: Yahoo CTVA read 11.92 / ret20 −86.5% on 10-02, probably an unadjusted corporate action. Its BOT row has a negative stop. It needs fixing at the price source, not in BOT.

Related: [[project-swing-scanner-retired]], [[project-bot-universe-scanner]]
