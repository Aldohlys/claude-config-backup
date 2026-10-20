---
name: project_macro_context_redesign
description: "macro_context Main/Detail/Legend tab redesign (mockup only, R port pending) + FRED 10Y split and YTD band added 2026-10-16 (RStudies c2117ff); BlackRock house views excluded"
metadata:
  node_type: memory
  type: project
  originSessionId: b2c1dc04-7dd9-4dc9-a48d-437eadf5cf6b
  modified: 2026-10-20T13:07:43.600Z
---

**Tab redesign (requested 2026-10-16, "lots of text and figures").** User asked for a Main tab, a Detail tab and a Legend tab holding definitions and shared information. Methodology is folded into Legend (user's decision). Mockup: scratchpad macro_context_mockup_20261016.html, built by build_mockup.py from the generated HTML; it is not in the R code yet.
- Main: bias bar, banners, 10 summary tiles linking to Detail, scenarios, regime, "Changed since", BOT chips, high/critical events for the next 7 days.
- Detail is grouped by question:
  - 00 movie;
  - 02/01/03/04 grid;
  - 09 assets;
  - 07/06/05 regime;
  - 10/11;
  - 08 calendar.
- Legend: L1 colour code, L2 glossary, L3 indicators (auto-built from tooltips, replaces M5), L4 sources and lags, L5 how to read charts, L6 reading principles, L7 methods.
- Pending user decisions:
  - trim Main to one screen (move the timeline to Detail, shorten "Changed since");
  - whether to keep section numbers.
- Then port to R with one `legend.R` dictionary driving the Legend, the row hovers and the links.
- Mockup issues seen:
  - Detail grid gap when 02 is open beside a collapsed 01;
  - stale label "08 Events — Edit every Monday" (events.R is automated).

**Main/Detail decisions (2026-10-20, prototype only, not in R).** Prototype `NewTrading/reports/macro_context_prototype_main_detail_20261020.html`, built by scratchpad `build_main_detail.py` from the generated HTML.
- Main: bias bar, summary, principles, 00 with only the top scenario card, 08 events, 09, 10, 11.
- Detail: 00b (other scenario cards and the full scenario table), then 01–04.
- Dropped by user decision: 05 Mismatches, 06 Synthesis, 07 regime model, and Methodology M6/M7.
- Daily Bias now comes from the 00 scenarios that are in place (BUILDING/ESTABLISHED/MATURE/FADING), not from the regime model:
  - DEFENSIVE: dash for cash, dollar wrecking ball, bond rout, stagflation, growth scare, EM/China, credit event, leadership unwind.
  - LONG BIAS: reflation, goldilocks, policy pivot, narrow leadership.
  - Debasement: neutral.
  - None in place → NEUTRAL; opposite directions in place → MIXED; EMERGING scenarios are shown but do not set the bias.
- On 10-20 the new rule gave DEFENSIVE (stagflation established), where the regime model gave NEUTRAL.
- The R port must also remove `synthesize()` in scenarios.R and the regime note in the summary.
- Superseded the same day: the user wants daily bias and macro regime as two distinct concepts, both shown. The bar now carries two badges:
  - Regime: the 00 scenarios, using the rule above.
  - Risk: CALM / ELEVATED / STRESS, from VIX and the high-yield spread (20-session z, enter at 1, exit at 0.5; one on = elevated, both = stress). Provisional thresholds.
  - Plus a "Read" line from a regime × risk table (rows long / neutral / defensive, columns calm / elevated). The user approved the 2×2 table.
- Summary layout: three columns (map | regime × risk table + "Changed since" | BOT sector chips), so the right side of the map is no longer empty. The ranked scenario table moved to Detail 00b, and the top-scenario card comes first in section 00.

**Rates additions (2026-10-16, committed RStudies c2117ff).** Source: comparison with the BlackRock weekly commentary.
- `fetch_fred()` reads FRED graph CSVs, no API key: DGS2, DGS10, DFII10, T10YIE, THREEFYTP10, DFF.
- The 10Y change is split into breakeven + term premium + real path, with the driver named.
- 2Y, 2s10s, and policy priced = DGS2 − DFF.
- YTD range band computed on front-month prices: roll-adjusted Brent gave +122.7% against the true +74.1%.
- Methodology fix: curve_inv is 10Y − 3M bill, not 10Y − 2Y.
- When committing, exclude positioning.R (user commits it by hand).

**Why:** user wants the page scannable, with definitions factored out ([[feedback_legend_definitions_not_rationale]]).

**How to apply:**
- Term premium is Kim-Wright (THREEFYTP10), chosen over ACM because the NY Fed file is a 10 MB xls.
- BlackRock house views (opinions) are not imported, the same rule as "Ceresna advisory excluded" ([[reference_ceresna_macro_tenets]]).
- Keep [[feedback_macro_context_layout]] and [[feedback_colourblind_palette]] when porting.
- Related: [[project_intermarket_macro_scenarios]].

**Regime × risk cells (2026-10-20, later).**
- Each cell holds a fixed BOT framework line, the user's rule set (e.g. "Fewer trades, reduced size"). Wording confirmed by the user the same day: long-calm "Normal operation: usual number of trades and size"; long-hot and neutral-hot "Usual selection, reduced size, no entries during spike"; neutral-calm "Usual size, no sector tilt"; defensive-calm "Fewer trades, longs (shorts) only in groups the scenario expects to rise (fall)"; defensive-hot "Limit trades, smallest size".
- The neutral row is kept (user's decision).
- The historical figures (hit vs base, n) are on Methodology M7, which took the slot of the dropped regime-model M7. Main shows only the rules.
- Other decisions the same day:
  - Section 00 sits right under the summary.
  - The BOT line is a table: section-10 verdict rows × scenario "expects rise / expects fall" columns.
  - The chains and yen blocks are on Detail; their lights stay on Main.
- Stagflation ETF lists (decided 2026-10-20, prototype only; `archetypes.R` still unchanged):
  - GDX removed from the long side. Calibration dropped GOLD from the fingerprint. Gold falls with rising real yields and the dollar (4-week r −0.47 / −0.44), and miners' energy costs rise with oil.
  - XLP and XLV added.
  - Open for the R port: check every scenario's `bot$long` / `bot$short` list against its calibrated `fp` (DBA/MOO under stagflation are suspect).
- Legend tab (2026-10-20): the Methodology tab is renamed Legend.
  - L1 Glossary, 69 terms: every term used on Main and Detail.
  - L2: all table footnotes moved off Main and Detail, each labelled with its source section.
  - Then the existing M1–M9.
  - The "Emerging: …" line stays on Main: it is information, not a footnote.
  - For the R port: one `legend.R` dictionary should drive L1.
- Scenario band flicker (2026-10-20): the user wants a scenario to last at least a week.
  - Test: `NewTrading/scripts/macro_forces/scenario_band_dwell.py`, 2004–2026.
  - Current rule (top state ≥ 50%, recomputed daily): 34% of runs last one day, 62% are under 5 sessions.
  - Proposed R4: enter after 3 sessions ≥ 50%, leave after 5 sessions < 50%, a challenger must lead for 5 sessions to take the label.
  - R4 result: 3.4% of runs under 5 sessions, median run 16 sessions. Episodes detected unchanged (21/34); median delay 15 → 22 sessions.
  - Not yet approved by the user, not in R.
- Scenario band, user decision 2026-10-20: no 1-day scenarios; "none in place" is no longer listed (grey gaps stay in the bar).
  - Prototype rule on the daily top labels: a run counts from 3 sessions; gaps of 4 sessions or less between runs of the same scenario are bridged; a run of 1–2 sessions still going today is shown as pending.
  - User decision 10-20: Regime and status follow the same rule. A scenario with fewer than 3 days in place is PENDING n/3 and does not count for Regime. On 10-20 the Regime read NEUTRAL (stagflation pending 2/3), and the framework cell was neutral × elevated.

**Ported to R (2026-10-20):** RStudies branch `claude/macro-context-main-detail`, created from `claude/vibrant-newton-2xvfgc`. Commits 63e4ce7 (port), 00f264e (header), de3f1ad (drivers table), d72d643 (lost-top card); pushed to origin. `main` is not merged yet. The branch is checked out, so the 09:00 RunScanner uses it.
- New files:
  - `regime_risk.R`: Regime/Risk, FRAMEWORK, band_runs, MIN_IN_PLACE = 3.
  - `legend.R`: LEGEND_GLOSSARY, plus `leg_note()`, which replaces every im-legend footnote.
- PENDING status is set in `archetypes.R::scenario_age`.
- High-yield spread: `fetch_hy_spread()`, own cache table `macro_fred_hy_cache`.
- `macro_context_results`: the `bias` column now holds the Regime; a new `risk` column is added. bot_fwd intake reads `bias`.
- `synthesize()`, the regime cards and M6/M7-old are deleted. `run_scenarios` is kept for the COT stale banner. Mismatches are still computed and exported to the DB, but not shown.

- Header since 00f264e: "Run <date time> · data to <last ^GSPC bar> close". A same-day re-run reuses the morning cache, so the run time alone said nothing about data freshness.
- Detail 00b, "What moved the state since <5 sessions ago>": a table under each of the top two cards, plus a "Lost the top place" card for the scenario that was top 5 sessions ago, if it has left the top two.
  - Each row: per-asset weight, range position then/now, contribution change in points; the column sums to the state change.
  - Asked by the user after the 10-16 switch from narrow leadership to stagflation.
  - Explanation of that switch: narrow leadership fell on Nasdaq 100 vs S&P (−9.8) and equal weight catching up (−7.9); stagflation rose on oil at its 1-year top (+11.1).
  - Ad-hoc version for any dates: `NewTrading/scripts/macro_forces/scenario_switch_drivers.R`.
