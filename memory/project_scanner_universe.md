---
name: Scanner Universe design principles
description: ScannerUniverse table should mirror Tickers (IV=YES, ≤$500); Tickers table sectors are the canonical namespace
type: project
---

ScannerUniverse must include all Tickers table securities with IV=YES and price ≤$500 (the BOT strategy price cap for longs).

**Why:** The swing scanner's Gate 2 (optionality) requires IV data from the Prices table, which is populated via getVolMetrics() — this only works for tickers in the Tickers table. If a ScannerUniverse stock isn't in Tickers, it will always get "NO DATA" and can never reach TRADE status.

**How to apply:** When adding new tickers to either table, ensure they exist in both (with appropriate Role in ScannerUniverse). When creating new sectors, use the Tickers table sector name as canonical (singular form, e.g., "Financial" not "Financials"). Sectors represent groups of highly correlated securities — they should be consistent across both tables for reporting.

Sector cleanup done 2026-03-23: unified naming (singular) across both tables. Two new scanner sectors added: Communications (XLC), Consumer cyclical (XLY).
