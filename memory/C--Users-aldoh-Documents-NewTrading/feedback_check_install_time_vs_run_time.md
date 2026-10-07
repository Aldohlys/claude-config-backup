---
name: feedback_check_install_time_vs_run_time
description: "When a scheduled run misbehaves, compare the installed package's Built: timestamp with the run time before blaming the current source code"
metadata:
  node_type: memory
  type: feedback
  originSessionId: a3517eda-94f8-48a8-b46e-2095ae048c64
  modified: 2026-10-05T06:28:06.088Z
---

Diagnosing a scheduled-run failure from the CURRENT source is wrong when the package was rebuilt after the run. Check `Built:` in `<lib>/Tdata/DESCRIPTION` (or the file mtime) against the log timestamp first.

**Why:** on 2026-10-05 the 06:16 Gonet run sent DSPF/CSBGU0 to real-time data although the source routed them to delayed. 5.20.11 had been installed at 06:29:59, after the run. I had also first stated a wrong mechanism for the missing prices ("symbols absent from output skip the fallback") without reading the run log. The log showed a batch-wide `-1` from getValue.

**How to apply:** for any "why did run X do Y" question, read that run's log lines (`RApplication/logs/t-YYYYMMDD.log`, `daily_portfolio_update.log`) and check the build time of the installed package before proposing a cause or a fix. Related: [[feedback_tdata_rebuild_restart_r]], [[feedback_verify_before_claiming_codebase_facts]], [[reference_tdata_getvalue_batch_all_or_nothing]].
