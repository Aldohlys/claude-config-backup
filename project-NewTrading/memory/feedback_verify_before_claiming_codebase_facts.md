---
name: Verify codebase facts with a wide sweep before claiming, not after pushback
description: When stating "X is defined as Y" or "the codebase does Z" — especially formulas, schemas, version numbers, conventions — do the cross-file sweep BEFORE the answer, not after the user pushes back. Three avoidable errors this session traced to single-file confirmation when the actual truth was distributed across files.
type: feedback
originSessionId: 9cbc474c-d67a-496b-92d1-fb42a1346715
---
## The rule

When making a factual claim about the codebase that will end up in memory, in a slash-command spec, or in a commit/PR — do the comprehensive sweep first. Single-file confirmation is not enough when the same concept is referenced in multiple places (formulas, version constants, configuration conventions).

## Why

Three corrections from a single session (2026-04-27) all had the same root cause — I cited the first file I found and the user had to push back to surface the wider truth:

1. **VRP formula**: I claimed "VRP = log(iv30/rv30)·100, full stop" after reading `volatility.R`. User asked "have you looked into the whole codebase?" — sweep revealed `.claude/commands/analyze.md` described it as vol-points and the Trades DB Remarques used vol-points by convention. Two co-existing forms, only one of which I had documented. Memory entry was wrong until rewritten.

2. **Version number**: I bumped the CHANGELOG to `[5.11.0]` based on "this is a feature, deserves minor bump". User caught it from the parallel `/build Tdata auto` output: actual version path was 5.10.8 → 5.10.9 (auto = patch). Would have shipped wrong without the catch.

3. **Cache architecture**: I proposed an in-memory dict for option-quote caching. User said "we already have parquet files that store historical option prices data — look". Sweep found `ParquetChainsStorage`/`ParquetStrikesStorage`/`ParquetMaintenanceManager` — three layers of an existing cache pattern I should have extended instead of duplicating.

In each case the wide search was 30 seconds of grep that I should have done up front.

## How to apply

Before answering questions of the form "what does the code do" / "where is X defined" / "how does Y work":

- **Grep across all relevant repos**, not just the file the user mentions or the first hit. Tdata, RStudies, RApplication scripts, NewTrading slash commands, CHANGELOGs — these are interconnected and each can have its own definition that drifts from the others.
- **Check for stale comments and parallel definitions** — the `vol_profile.R` legacy comment "VRP context: iv30 - rv30" is exactly the kind of artifact that survives a refactor and creates confusion years later.
- **Check version-tagged conventions** (CHANGELOG, DESCRIPTION) before proposing version-numbered work. The build pipeline has its own opinions.
- **Check existing infrastructure before designing new** — caching, persistence, logging, config — Tdata already has battle-tested patterns for most of these.

## Trigger

When about to write something to memory, edit a spec/changelog, or assert "this is how it works" with conviction — pause and ask: "have I checked every place this concept lives?" If the answer is one file, do the sweep before answering.
