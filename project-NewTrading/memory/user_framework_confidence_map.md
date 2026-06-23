---
name: User confidence map across trading framework
description: Where the user feels mechanically confident (options pricing) vs where qualitative gaps remain (sector/technical) — shapes how to calibrate recommendations
type: user
originSessionId: 6777ad2b-a236-467c-b8db-01c856609c09
---
Stated explicitly by the user on 2026-04-24: **"the options pricing part is now quite clear and easy to follow for me, because it is completely data dependent. But the Sectors/technical indicators is not as satisfactory because there is some qualitative dimension into it."**

## Confidence zones

**High confidence (data-dependent, mechanical):**
- IV / HV / VRP / skew / term structure analysis
- Structure selection from surface (long call vs spread vs risk reversal)
- Greeks-based monitoring and exit triggers
- BS-based target grids for trade planning
- Vol-metrics pulls via `tdata_py.impliedvol.get_volatility_metrics`

**Lower confidence (qualitative gaps, under active work):**
- Sector regime / leadership definition (current gate is binary, not ranked — see `project_scanner_methodology_todo.md`)
- Technical pattern identification (no formalized catalog beyond BOT yet; goal is ≤3 patterns with measured edge)
- Rotation detection (today's rotation miss surfaced this gap)

## How to apply

**On options-pricing questions:** match the user's register — compute precisely, show the surface data, propose structures with clear rank-ordering. The user can read the output and decide. Less hedging language, more numbers. The user has internalized the framework and wants crisp data.

**On sector/technical questions:** don't overclaim certainty. Acknowledge the qualitative dimension. Help systematize rather than pretend to a precision that isn't there. Propose computable rules, backtest them, require statistical guardrails (e.g., ≥30% in winners, ≤15% in losers before calling a pattern edge). When the scanner or regime model produces a signal, be explicit about what it's computing and what it's missing.

**General:** when a trade goes wrong, lead with whichever side of the framework the failure came from. JPM 2026-04-24 was a sector/rotation failure — not a vol-pricing failure — so the post-mortem should focus on the scanner/gate, not on structure or pricing. Getting this attribution right matters for the user's learning loop.
