---
name: Scanner methodology improvement TODO
description: Open work program for tightening scanner sector gate and building computable technical pattern catalog; opened after JPM rotation miss
type: project
originSessionId: 6777ad2b-a236-467c-b8db-01c856609c09
---
Work program opened 2026-04-24 to reduce qualitative guesswork on the scanner side of the framework.

**Trigger:** JPM trade (May 15 330/340 call spread, $262, entered 2026-04-24) surfaced from a scanner run that put BAC and JPM in TOP PICK at 10/14 while Technology was the clear sector leader (+6.1% RS vs SPY) and Financial sat at −2.9%. User's read: options-pricing side of framework is clean and data-dependent; sector+technical side is qualitative and where mistakes cluster.

**Two workstreams** (details in `scanner_methodology_TODO.md`):

1. **Sector gate ranking overlay.** SECTOR GATE already computes RS vs SPY per sector but the gate is binary (LONG/SHORT/BLOCKED). Quick-win: rank LONG-passing sectors by RS, down-weight bottom-half sectors in stock scoring, cap TOP PICK to top-3 sectors. Same-sector concentration penalty as secondary. Sector gate itself stays wide — ranking layer sits on top.

2. **Computable technical pattern catalog.** Review closed winning trades (top quartile), extract recurring setups, write deterministic detection functions, backtest 5Y on ScannerUniverse, filter by trader comfort. Target ≤3 patterns with measured edge (≥30% appearance in winners AND ≤15% in losers). BOT checklist is the template for format.

**Why:** Path-of-trade failures like JPM today (correct stock selection within a lagging sector) are structurally caused by the scanner not consuming its own sector RS data. Fixing the data pipeline first is higher leverage than any individual pattern work.

**How to apply:** When user asks for scanner enhancements, new technical indicators, or methodology improvements, reference this TODO and its ordered workstreams. Don't propose item 2 (pattern backtest) until item 1 (sector overlay) is wired in and observed for 2–3 weeks. Options-pricing side is considered working and doesn't need methodology work — focus suggestions on the two items above.
