---
name: project-macro-context-cot-section11
description: "macro_context section 11 (COT) redesigned 2026-10-27 into COTSignal-style charts: specs long/short split, 4-week driver, other reportables apart, hover values; history in output/cot_history.csv"
metadata:
  node_type: memory
  type: project
  originSessionId: dd8f0bea-ff17-47d3-80cd-59333ebf1b0c
  modified: 2026-10-27T11:42:52.701Z
---

On 2026-10-27 section 11 of the macro_context report (RStudies `reports/macro_context/cot_render.R`, commits 975ebd3 → 540c616 on main) was rebuilt on the user's request: "a plot like cotsignal.com, to grasp in a few seconds who is long what and how it evolved", then "displaying only net is not enough".

- Overview, one row per market. 3y net index cells for Specs / Other rept. / Other side / Small, coloured blue (more long than usual) to red (less long). Specs long / short / net contracts. 4-week change per leg, plus "Driven by", the larger leg: buying, long selling, short selling, short covering. A 5-year chart of specs longs above zero, shorts below zero and net.
- Below it, collapsed per-sector cards with a bigger long/short/net chart and a 3y index panel (15/85 bands). The old 1y/3y/5y tables are kept, collapsed.
- Every chart has hover values (data-cot JSON + script in template.html).
- Specs = managed money (commodities) and leveraged funds (TFF financials). The user asked: specs vs commercials vs MM + other reportables? Kept MM, because other reportables trade against MM in commodities ([[reference-cot-trader-categories]]). Commercials = hedgers, shown as a score + hover only. The legend notes the Treasury basis-trade caveat for leveraged-fund shorts.
- `refresh_cot.R` (Saturday) writes `output/cot_history.csv` (git-ignored; ~86k rows, 38 markets × 5 groups, weekly long/short/net + rolling 3y index) and now loads 8 archive years so the 3y index covers the 5-year chart. Lumber has the shortest history.
- The 4-week window matches the ~40-day trade horizon ([[feedback_size_analytics_to_trade_horizon]]); 1-week figures are in the tooltip.
- Report size about 2.5 MB (was 1.0 MB), mostly SVG and hover data.

**Why:** the user reads positioning to see the fast-moving non-commercial money and whether a net move came from new longs or short covering.

**How to apply:** further COT display changes go in cot_render.R / template.html. The latest-week CSV (`Reports/cot_positioning_latest.csv`) still drives the cells; the history file drives the charts. Related: [[feedback_macro_context_layout]], [[feedback_colourblind_palette]].
