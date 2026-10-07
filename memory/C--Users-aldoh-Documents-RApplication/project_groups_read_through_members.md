---
name: project_groups_read_through_members
description: "Since 2026-10-05 every consumer reads a correlation group through its members (median / equal-weight), never its anchor ETF: macro sector map, BOT S3 rs_bench, grp_rank, /analyze Phase B"
metadata:
  node_type: memory
  type: project
  originSessionId: 40261db9-cf1a-4c33-b2f5-d91f2eec6c7b
  modified: 2026-10-06T06:41:54.429Z
---

User decision 2026-10-05: a correlation group (`ScannerUniverse.Cluster`) is read through its members everywhere; `ClusterETF` anchors are labels only (`get_group_anchors()` has no caller left).

- **Macro report section 10 (BOT sector map)**, `macro_context/intermarket.R::group_map()` / `ew_close()`: one row per group holding a `Tickers.BOT_Eligible` name (55), equal-weight index of all members (mean daily adjusted return, compounded). Replaced 33 hand-listed ETFs keyed on `tradable_universe_20260827.csv` `bench`, which split groups across rows (spec growth over IAI/IGV/CARZ/QQQ/IBIT) and merged groups into rows (SMH = semis hardware + equipment + networking). Drivers are hand-written in `GROUP_DRIVERS`, keyed by group name. A group renamed or added by a review stops the map until it gets drivers. Parent sectors moved to a "US sectors (SPDR)" panel.
- **S3** (`shared/bot_read.R`): `rs_bench = peers:<group>`, the median ret20 of the other members (±50% cut); `grp_rs`/`grp_rank` use the members' median as well.
- **/analyze Phase B** (`live_sources.R::compute_sector_rs_context()`): same member medians. It now fetches ~340 names and takes ~2 min. It could read the macro report's `intermarket_cache` instead, if the macro report ran that day.

**Why:** an outside anchor is the nearest universe ETF, a proxy that can serve two groups (ITA: defence primes and commercial aerospace read identically) or another industry (ITB for machinery). Members-only kept the rotation-rank edge: top 3 vs rest +0.32 ATR, t 3.4, against +0.28, t 3.1, for the anchor rule (`group_rotation_test.py`; the old rule is `--anchor-rule`).

**How to apply:** don't reintroduce anchor ETFs as benchmarks. When a cluster review renames groups, update `GROUP_DRIVERS` in `macro_context/intermarket_config.R`. The map's verdicts are untested until the TODO 105 backtest is done.

Related: [[project_scanner_correlation_groups]], [[project_bot_industry_benchmarks]].
