---
name: /analyze command — always run all phases
description: /analyze must execute every phase (A→B→C→D→E) regardless of SKIPs, no gated stop
type: feedback
originSessionId: 791d7ea0-3b98-46b6-b585-5209bd9337c8
---
When running `/analyze`, run every phase (A, B, C, D, E + vol funnel) for the requested ticker. Do NOT stop at the first SKIP. Treat the call as if `--force` were always set.

**Why:** /analyze is a single-ticker deep-dive tool, not a universe filter. The scanner CSV already does the gating to produce the SKIP/WATCH/TOP-PICK list. When the user asks /analyze on a specific ticker, they want the full evidence picture — including phases the scanner skipped — to make their own judgment. Reporting only "SKIP at Phase B" wastes the deep-dive and forces the user to re-ask with --force.

**How to apply:** Always compute Phase C cheap_score components (IVP, VRP, term, skew, sector x-rank) and the directional vol funnel (C.2.a–C.2.e) from raw Tdata / DB pulls when the v5 CSV has NA. Always compute Phase D (vehicle, structural target sources, chain OI cap, R:R, entry interval) from raw chain data. Always produce Phase E classification + preferred structures. Mark each phase result as PASS / WATCH / SKIP, but do not stop the pipeline. The `--no-html` and `--no-vol-funnel` flags still apply; the implicit `--force` only removes the early-stop behavior.
