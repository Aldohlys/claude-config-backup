---
name: feedback_schema_change_regression_tests
description: "On a DB schema change the user expects regression suites re-run against a recorded baseline and new regression tests added — exact column-list tests, fixture tables, writers with/without the column, consumers of all columns, preview-vs-write paths"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7b2126a4-19e3-44e8-8acf-3feca1d38bea
  modified: 2026-09-15T10:26:13.579Z
---

When a change alters the DB schema, re-run the regression suites against a recorded baseline and add tests where coverage is missing. The user asked for it explicitly while `Account.Notes` was being added (TODO #81, 2026-09-15): *"Be mindful of any regression due to DB schema change re-run regression tests, add some more regression tests if necessary"*.

**Why:** a new column breaks things far from the change, and green unit tests on the new code say nothing about them. Seen on the Notes change:
- Tdata `test-account.R` asserts the **exact ordered column list** of `readAccount()` for DU5221795 and Gonet — both failed until `Notes` was appended.
- Fixture tables copied from the real one (`TestAccount`, read through `TestAccountWithConversionRate`) need the column too, or the mocked-view tests break.
- The Cash Flow modal previewed rows built WITH the note, but its OK handler called `record_cash_flow()` without `notes` — shipped, then caught only by reading the whole module (fixed Tuser `54692e1`).

**How to apply:**
- Before migrating, write down the baseline tallies (Tdata full suite with `get_tdata_py()` called first; Tuser and RReporting standalone scripts). After, compare counts and explain every difference, including extra skips.
- Migration script: backup first, idempotent, migrate fixture tables alongside the real one, verify views expose the column.
- Add tests for: the reader returns the column with the right type; writers append **with and without** the column (in-memory DB built from the live `sqlite_master` CREATE statement); consumers still work on real rows (read-only); every UI path that previews a value also writes it.
- Scripts that load installed packages see the OLD code until `/build` deploys — a failing "new column" check before the deploy is expected; re-run it after.

Related: [[reference_account_cashflow_conventions]], [[feedback_named_vector_to_json]], [[reference_testserver_shiny_wiring]].
