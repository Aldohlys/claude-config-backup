---
name: V5 scanner implementation lessons
description: Pattern lessons from the swing_scanner v5 build that apply to future R/Tdata + scanner work
type: feedback
originSessionId: 53dab338-46f2-4aba-b466-d9cda0aa1007
---
Lessons from building swing_scanner v5 (2026-04-27) that apply beyond this single feature:

**Don't fabricate identifiers when a system stores real ones.** Anywhere code generates a key (expiry, strike, sector code, ticker) that has to match persisted data, the writer's source-of-truth must be the same as the reader's. Phase D originally computed target expiry as `Sys.Date()+N` — never a real Friday — and got zero chain matches against `option_chain_oi_history` rows that used `getExpirationDates()` outputs. Always query the persistence layer for available keys instead of synthesizing.

**Why:** Round-trip failures from key-mismatch look identical to "no data" downstream. The bug is invisible until you log both sides.

**How to apply:** When designing scanner phases that read DB-persisted option/IV/chain data, the phase's lookup keys must come from the same Tdata helper that the daily_option_fetch task used to write — not from R `Sys.Date()` arithmetic.

---

**Don't fail-closed silently in early pipeline phases.** Phase A's first version returned `passes_gate=FALSE` when TWS wasn't connected, which silently zeroed the universe. Fix: permissive default with a `reason` string ("no expiry data — passing by default"). Downstream phases will cull anyway.

**Why:** Conservative defaults at every layer compound into "nothing works." A scanner that fails A→B→C→D→E with all gates closed produces zero output and zero diagnostic value.

**How to apply:** Each phase should fail-open with a reason logged, not fail-closed silently. Use NOT NULL constraints only on columns the writer fully controls (cache_date, sym), never on values sourced from external APIs.

---

**R library promotion is multi-target.** `/build` deploys to renv apps under `RApplication/<app>/renv/library/...` but does not touch `C:/Users/aldoh/Documents/RLibrary/Tdata` (the user-level system library). Plain `Rscript` outside a renv context resolves `library(Tdata)` from RLibrary, so a "successful" /build deploy can still leave standalone scripts on the old version. Use `R CMD INSTALL --library=C:/Users/aldoh/Documents/RLibrary <pkg>` to force-update the system library when standalone scripts depend on the new version.

**Why:** "Deployed to 6 apps" can hide that the runtime path of a specific script is one of the *non*-targets.

**How to apply:** When testing a Tdata change with a standalone Rscript, verify with `find.package("Tdata")` and `packageVersion("Tdata")` BEFORE assuming /build wired the new code into reach.

---

**Prefer observed outcomes over modeled estimates when calibrating.** First R:R_min calibration tried Yahoo + BS-forward at structural target — three external dependencies. The actual answer was a 30-line SQL query against the Trades table (realized exit prices). When you have the system's own observed outcomes, use them.

**Why:** Models import their own assumptions; the realized track record imports nothing. The user explicitly flagged this with "why are you calling Yahoo? Trades table has the data."

**How to apply:** For any calibration of thresholds tied to historical performance (R:R_min, Cheap_Score cutoff, Pull_Score cutoff), look first at the Trades / closed-position data. Reserve modeling for states the system never observed (e.g., Phase D's BS-forward at structural target, which has no historical counterpart per trade).

---

**Output buffering can make slow loops look frozen.** Long Rscript runs with no TTY buffer stdout indefinitely. The 30-min Yahoo-retry run had output sitting in buffer the entire time — looked hung. `flush.console()` after meaningful log lines, or running with stderr (which buffers less aggressively in practice), surfaces progress.

**How to apply:** When an Rscript loop is expected to run >2 minutes, add `flush.console()` after each iteration's `message()` call.
