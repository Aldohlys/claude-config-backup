---
name: CL futures options multiplier is 1000
description: CL (Light Sweet Crude Oil) NYMEX futures and options have a contract multiplier of 1000, not the equity-default 100. Critical for P&L and risk-cap math.
type: reference
originSessionId: 8a62184d-ede9-4981-9f34-2762b2d293d8
---
## Rule
**CL options multiplier = 1000** (1,000 barrels per contract). This applies to all NYMEX CL futures and options (root LO).

Confirmed in the trading DB:
```
sqlite> SELECT Name, Type, TradingClass, Multiplier FROM Tickers WHERE Name LIKE 'CL%';
CLM6|FUT|USD|LO|1000|NYMEX|NYMEX|YES
```

## Why it matters
- A $0.33 option premium = **$330** per contract, not $33
- A $1.00-wide call spread max payoff = **$1,000** gross, not $100
- The $300 BOT risk cap (per /analyze lot rule) means **a 1-wide CL spread at $0.30 cost is right at the cap**; anything wider blows through it fast

## Common multipliers to remember
| Symbol | Type | Multiplier | Notes |
|---|---|---|---|
| Equity options (SPY, AAPL, etc.) | OPT | 100 | Default |
| **CL** (crude) | FOP | **1000** | 1,000 bbl |
| MCL (micro crude) | FOP | 100 | 1/10 of CL |
| ES (S&P) | FOP | 50 | $50 × index |
| NQ (Nasdaq) | FOP | 20 | $20 × index |
| 6E (EUR) | FOP | 125,000 | EUR notional |
| 6S (CHF) | FOP | 125,000 | CHF notional |
| GC (gold) | FOP | 100 | 100 oz |
| NG (nat gas) | FOP | 10,000 | 10,000 mmBtu |

## How to apply
- For any futures-options sizing or R:R calc, query `Multiplier` from `Tickers` (or use `Tdata::getMultiplier(sym)`) BEFORE multiplying premium × 100 by reflex.
- Especially relevant for /analyze on futures-options trades — the $300 lot cap behaves very differently when 1 contract = 1000-multiplier vs 100.
