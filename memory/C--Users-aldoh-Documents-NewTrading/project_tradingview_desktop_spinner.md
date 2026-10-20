---
name: project_tradingview_desktop_spinner
description: "TradingView Desktop 3.4.1.8194 shows only a spinner in every tab since 2026-10-15 (after crash + KB5127648); web in Edge works; support ticket filed, answer pending"
metadata:
  node_type: memory
  type: project
  originSessionId: 1fec3f05-77cb-45c8-93b4-dfad172e8166
  modified: 2026-10-20T15:02:53.370Z
---

Since the 2026-10-15 crash reboot (Windows build 26200.9711 / KB5127648 applied), TradingView Desktop (MSIX, 3.4.1.8194) renders nothing: every tab, even an empty New tab, stays on the spinner. Sign-in works and the window title shows live quotes. tradingview.com in Edge works.

Ruled out on 10-15: memory, CPU, disk latency, network/DNS/IPv6, Defender, cache corruption (six cache folders cleared), local state (Reset-AppxPackage, which also wiped all logs and tabs), GPU (`--disable-gpu` still blank; the local /app/new-tab page took 35 s vs 104 s with GPU). App updater reports no update; the winget "latest" MSIX is the same 3.4.1.8194. Launching TradingView.exe directly from WindowsApps with args works but hands off to an existing instance; Stop-Process -Force often leaves the main process alive — use `taskkill /F /T /IM TradingView.exe`.

Ticket filed 10-15 via tradingview.com/support chatbot → Platforms → Desktop → Application performance → "looks/works odd" (image uploads only; log rendered to PNG). Files in `Downloads\TV_support_20261015\`. Workaround: Edge, or Edge ⋯ → Apps → Install TradingView.

**Why:** TradingView is the user's main charting tool; the cause is still unknown (suspect KB5127648 vs the app's old Chromium).
**How to apply:** if the user mentions TradingView problems, ask whether support answered or a new desktop build/Windows update changed things before re-running the diagnostics above. Related: [[reference_transcribe_pipeline_atomic]] (same crash).
