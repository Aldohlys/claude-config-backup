---
name: project_claude_api_integration_todo
description: "TODO — add Claude API calls to NewTrading/RApplication scripts (summarize, classify, extract, report)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7314cd86-c7d4-45e3-9431-a260c000ac0f
---

Open work program (added 2026-06-23): integrate Claude API calls into the trading scripts via the official Anthropic Python SDK.

Candidate tasks:
- **Summarize journal entries** from `mydb.db` (Journal table, 1,405 entries; `text` field) — per-symbol or per-theme rollups.
- **Classify trades** — label/categorize trades from the Trades table (e.g. setup type, breakout vs mean-reversion, outcome reason).
- **Extract structured data from PDF research letters** — the Daubasses letters in `Trades/Daubasses/*.pdf` → structured fields (ticker, thesis, valuation, target). Use the document input API (base64 PDF or Files API).
- **Extract from transcripts** — output of the [[reference_transcribe_pipeline_atomic]] / `/transcribe` pipeline → structured summary or signal extraction.
- **Generate report text** — narrative prose for the markdown reports in `Reports/`, `Trades/`, `Discussions/`.

**Why:** these are LLM-shaped tasks (summarize/classify/extract/generate over natural language) currently done manually or not at all; batching across DB rows or PDFs is high-leverage.

**How to apply:** Use the `/claude-api` skill for SDK-correct code. Python via conda ([[reference_python_conda_launch]]) — `pip install anthropic`, default model `claude-opus-4-8`. For many-row jobs (e.g. all journal entries) use the Batches API (50% cost). For PDFs use document content blocks; for forced-shape output use structured outputs (`output_config.format`). Escape `\$` in any generated report text per [[feedback_no_latex_math_blocks]].
