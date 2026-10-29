---
name: reference-browser-pane-local-html
description: "Built-in browser pane refuses file:// report HTML; serve NewTrading/reports with conda python http.server on 127.0.0.1:8765, stop it after"
metadata:
  node_type: memory
  type: reference
  originSessionId: dd8f0bea-ff17-47d3-80cd-59333ebf1b0c
  modified: 2026-10-27T11:43:02.133Z
---

The desktop app's built-in browser refuses `file:///C:/Users/aldoh/Documents/NewTrading/reports/*.html` ("couldn't open file"). What works (2026-10-27): run `C:/Users/aldoh/miniconda3/python.exe -m http.server 8765 --bind 127.0.0.1` from `NewTrading/reports` in the background, then navigate to `http://127.0.0.1:8765/macro_context_YYYYMMDD.html`. Stop the server (TaskStop) when done.

Screenshots of the long macro_context page often time out on the first try; retry, or read values with javascript_tool. Viewport emulation above the pane width returns scaled-down, unreadable screenshots; use the desktop preset. Sections are collapsed by default (only 00 open), so open them via JS `details.open = true` before scrolling.
