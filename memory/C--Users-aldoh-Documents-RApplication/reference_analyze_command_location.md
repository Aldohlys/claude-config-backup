---
name: reference_analyze_command_location
description: /analyze command lives in NewTrading/.claude/commands/analyze.md — thin wrapper to RStudies reports/analyze
metadata: 
  node_type: memory
  type: reference
  originSessionId: 672ba0ce-0bbd-4810-bd8c-9c6150cb270e
---

The **`/analyze` slash command is a NewTrading command**, not RApplication: `NewTrading/.claude/commands/analyze.md`. (It's gitignored in NewTrading, so its only versioned copy is `claude-config-backup/project-NewTrading/commands/analyze.md`.) Confused once in the 2026-06-23 session because the backup carried a stale copy misfiled under `project-RApplication/`.

**It is a thin wrapper** — the real computation lives in the RStudies repo at `RApplication/RStudies/reports/analyze/main.R`:
- **Windows:** `Rscript C:/Users/aldoh/Documents/RApplication/RStudies/reports/analyze/main.R <TICKER> <DIRECTION> [flags]`, **CWD must be `RStudies`** (renv + Tdata).
- **VM (trading-vm):** call `/opt/scripts/analyze.sh <TICKER> <DIRECTION>` (pre-checks IB Gateway; exit 2 = gateway down → `sudo systemctl start trading-stack.service`). Source = `RApplication/scripts/analyze.sh`, deployed via `scripts/push-analyze-to-vm.ps1`.
- Config: `RStudies/config.yml` under `default.analyze`. Output HTML: `NewTrading/reports/analyze_<TICKER>_<YYYYMMDD>.html`. Flags: `--no-html`, `--no-vol-funnel`.

**Current version is the NEUTRAL data report** (Phases A–E, PASS/SKIP/NO SIGNAL/STALE) — explicitly NOT a recommender (no GO/NO-GO, no ranking, no conviction). That neutrality is enforced in `reports/analyze/report.R`, not the prompt stub. The old 421-line GO/NO-GO "BOT Candidate" version is retired. See [[project_analyze_degate]], [[feedback_analyze_neutral_stance]], [[project_vm_no_option_cache_consumer]].
