---
name: Scanner Universe design principles
description: ScannerUniverse origin (top-5-per-sector seed), sync rules (IV=YES, ≤$500), and canonical sector namespace
type: project
originSessionId: 1347083d-d9f2-4281-99ce-26cad155e9e3
---
## Origin (initial seed, Tdata v5.9.5 — 2026-03-17)

ScannerUniverse was seeded with ~170 symbols using these criteria:
- **Top 5 stocks per sector ETF** (by holding weight)
- Market cap > **$10B USD**
- Stock price ≥ **$10**
- **US-only** (explains diversity across sectors but no international coverage)

The seed script is NOT in git — only the `getScannerUniverse` / `addScannerSymbol` / `removeScannerSymbol` functions were committed in v5.9.5. The ~170 seed symbols (147 scanner + 14 macro + 9 ETFs) were inserted in a prior Claude Code session and are not reproducible from code. If universe is lost, macro + ETF rows must be re-seeded by hand.

**Why it matters:** The universe is intentionally diverse (US large-caps across 10 sectors) but capped at $500 price and US-only. Don't assume international tickers or small-caps belong here.

## Ongoing sync rules (Tdata v5.9.11 — 2026-03-25)

ScannerUniverse must include all Tickers table securities with IV=YES and price ≤$500 (the BOT strategy price cap for longs). `syncTickersToScanner()` runs on each scanner invocation via `get_universe()` in `RStudies/reports/shared/universe.R`.

**Why:** The swing scanner's Gate 2 (optionality) requires IV data from the Prices table, which is populated via getVolMetrics() — this only works for tickers in the Tickers table. If a ScannerUniverse stock isn't in Tickers, it will always get "NO DATA" and can never reach TRADE status.

**How to apply:** When adding new tickers to either table, ensure they exist in both (with appropriate Role in ScannerUniverse). When creating new sectors, use the Tickers table sector name as canonical (singular form, e.g., "Financial" not "Financials"). Sync is additive-only — removing a ticker from Tickers does NOT remove it from ScannerUniverse; soft-delete via `removeScannerSymbol()` (sets IsActive=0) instead.

Sector cleanup done 2026-03-23: unified naming (singular) across both tables. Two new scanner sectors added: Communications (XLC), Consumer cyclical (XLY).
