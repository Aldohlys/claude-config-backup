---
name: project-swing-scanner-retired
description: Swing scanner (front-run option flow) stopped 2026-10-05; only its group-rotation rank survives, as BOT_daily detail columns
metadata:
  type: project
---
Swing scanner removed 2026-10-05. `RStudies/reports/swing_scanner/` was deleted; the last version is the git tag `swing-scanner-final` (pushed). The RunScanner task (09:00) now runs `NewTrading/scripts/run_macro_context.bat`, macro_context only. /analyze no longer reads `swing_scanner_<date>.csv`. `daily_option_fetch.R` and `test_v5_modules.R` were removed. The DB tables were kept: `option_*_history` serves as /analyze's cache, `scanner_results` as history.

**Why:**
- The P&L split of 116 BOT option trades showed spot drives P&L and vol does not (see [[project-bot-pnl-greeks-decomposition]]). So the scanner's core thesis, buying cheap vol (Phase C), was never where the money came from.
- `daily_option_fetch.R` was never scheduled. `option_skew_history` / `option_chain_oi_history` stopped on 2026-04-27 (7 symbols), so Phase D had no chain data and the scanner never produced a valid TOP PICK.
- Its trend stage and footprint duplicate BOT gates. Its target and R:R duplicate /analyze.

**What survived:** the group rotation rank (group 20-day return minus SPY, ranked). `group_rotation_test.py` measured +0.30 ATR more over 10 sessions for top-3 groups than for groups ranked below 6 (t 2.3 on non-overlapping windows). The hit rate gain is small (+3-5 pp). It became the BOT_daily `--detail` columns `group` / `grp_rs` / `grp_rank` (BOT_TOOLS_DESIGN v3.39). Reported, not gated.

On the same day, BOT S3 moved from Tickers.BOT_Bench to the correlation group: the anchor if it sits outside the group, else the median of the peers (v3.38, `rs_bench`).

**How to apply:** do not propose swing-scanner enhancements, the two-timescale v6, or option-cheapness gates as an entry screen. Option cost is a vehicle question for /analyze. The superseded design is in [[project-swing-scanner-redesign]] for history only. /analyze reads the option tables only as a cache, with a live TWS fallback.
