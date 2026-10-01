---
name: project_scanner_correlation_groups
description: "TODO 71: ScannerUniverse.Cluster/ClusterETF = correlation groups (max 10) from scripts/cluster_universe.py; Sector kept only for macro rules via SECTOR_RULE_KEY; column named Cluster not Group"
metadata:
  node_type: memory
  type: project
  originSessionId: 1c26deb6-0a13-4dd0-99ab-91d81259daed
  modified: 2026-09-30T21:44:21.632Z
---

In production since 2026-09-29 (RStudies 9d851a2, RApplication ecb6d28).

- `scripts/cluster_universe.py` (Python, yfinance): 250 completed sessions of adjusted returns,
  average-linkage on 1−ρ cut at ρ 0.4, groups >10 re-cut, lone names attached at ρ ≥ 0.3 else
  `Ungrouped` (renamed from `Unclassified` 2026-10-02: these names keep a well-defined Sector, they just have no correlation group; SR1 warning + proposal with its ρ), hysteresis 0.10 (group names and manual
  DB Browser edits persist), anchor = member ETF / nearest ETF ρ ≥ 0.7 / most central member.
  Dry run by default; `--write` updates the DB. Measured edge (ρ peers − ρ rest): Sector
  0.249 → groups 0.464 (53 groups). Run monthly by hand for now.
- **Yahoo is unreliable in batch:** one `yf.download` of 294 symbols lost 291, 10, then 2 on
  three identical calls; `fast_info` rate-limits after ~10 full runs (then no ETF is
  recognised and every anchor silently falls back to a stock). The script retries and aborts
  ("Nothing written") above 10 missing symbols — keep that guard.
- Column is `Cluster`, not the `Group` the user first approved: GROUP is an SQL keyword.
  The user looked for "Group" in DB Browser — say the real column name up front.
- Consumers: swing scanner gate/RS rank/sector_pts and /analyze Phase B (`get_groups()`,
  `get_group_anchors()`, … in `shared/universe.R`, which `stopifnot`s on the columns).
  macro_context still uses `get_sector*()`.
- **Macro rules had been dead for 6 of 10 families:** SECTOR_MACRO_RULES keys (`PreciousMetals`,
  `Financials`…) never matched the Sector labels (`Precious Metals`, `Financial`…).
  `SECTOR_RULE_KEY` maps them; groups use their majority Sector's rules.
- **Anchor rule measure:** the script scores an ETF by corr(ETF return, group's equal-weight
  return), which runs well above the ETF's mean ρ with each member (SOXX 0.93 vs 0.84). Don't
  quote the member-mean as "the 0.7 rule". The 0.70 itself is a judgement value, never calibrated.
- Production DB writes by Claude may be blocked by the permission classifier; give the user
  PowerShell-form commands (`& "path\python.exe" -c "…"`, single line).

See [[project_scanner_universe]], [[project_bot_industry_benchmarks]].

- **Hand review applied 2026-10-02** (`logs/cluster_review_20261001.sql`, local: logs/ is gitignored; TODO 71 records it): 55 groups "Theme - Group" + Ungrouped, 336 names, manual anchors (SOXX, BNK=BNK.PA, IAK, IHF, IXC, IBUY, PEJ, BOAT, XLI, COPX). User's rule: a name whose mean ρ with its group is < 0.30 goes to Ungrouped under its Sector; a move that lowers ρ by ≥ 0.02 is reverted. Never run cluster_universe.py --write as is: hysteresis does not protect these edits (dry run, then apply by hand).
- **Cluster review page** = claude.ai artifact UzXv2LdAjpcX1YeEYVU9jw, opened by `NewTrading/scripts/run_cluster_review.bat` (launcher). Saved reviews live in its db: `reviews/cluster_review` (v1, applied) and `reviews/cluster_review_v2` (current); read with ArtifactData. Page source + data build in the session scratchpad (`cluster_review.tpl.html`, groups_v2.json from ScannerUniverse, biz.json Yahoo summaries) — rebuild from the DB if lost.
