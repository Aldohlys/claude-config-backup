---
name: project_xlf_breakout_screen_20260608
description: "XLF financials component breakout screen 2026-06-08; C is the only GREEN, sector is a spread-not-long-call sector"
metadata: 
  node_type: memory
  type: project
  originSessionId: 766f90e9-4750-4c67-8300-8b0e485a5e25
---

Ran the Energy/Healthcare sector-screen treatment on **XLF main components** (2026-06-08). Script `Strategies/Breakouts/xlf_screen.py` (mirrors `xlv_screen.py`: BOT S1-S6/BK1-BK4 gates, EMA50, RS vs XLF) folds in the [[reference_atr_move_multiples]] beta filter so low-beta names (sell-OTM-call-won't-pay) are flagged automatically. Report: `Reports/xlf_breakout_screen_20260608.md`.

**Key results (XLF +4.4%/3m, leading):**
- **C (Citigroup) = the only GREEN (S5/BK3)** — RS +18.6 vs an already-leading sector, +6.9% above rising EMA50, fresh trigger. THE pick.
- MS/GS = strongest *trends* (dEMA +10/+9.5, RS +28/+21) but BK1 only = already-moving, no fresh trigger → chasing, not BOT entry.
- BK/BAC = S4/BK2 (live trigger, milder leadership); JPM mature/flat slope.

**Vehicle conclusion (the rule of thumb doing its job):** financials cluster in the MID ATR bucket (~2%, Ppay ~20-32%) → **"spread sector, not long-call sector."** Breakout vehicle = debit call vertical, NOT outright calls. Only **BX (Blackstone, 3.05% ATR, 45% payable)** has the beta for outright long calls — but no setup yet (watch).
**Filtered OUT (low beta, sell-call won't pay):** BRK-B (1.42% ATR/8% payable, canonical reject), plus V/MA/SPGI/CME/CB doubly disqualified (low-MID beta AND no setup).
MMC + FI(Fiserv) = Yahoo 404 at run time (data outage, re-run to capture).

Next if acting on C: `/analyze C` for the debit vertical at ~30 & ~50 DTE. See [[feedback_dont_switch_frameworks_midtrade]] (MS/GS = momentum, not breakout).
