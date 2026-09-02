---
name: reference_ibkr_symbol_with_space
description: "IBKR stock symbols can contain an internal space (BRK B); the IBKR Name, the option TradingClass and the YahooName can all three differ for the same ticker"
metadata: 
  node_type: memory
  type: reference
  originSessionId: fed3d834-ab45-425c-b8ba-bcfec60d789d
  modified: 2026-08-27T02:18:48.836Z
---

IBKR stock symbols are **not** guaranteed space-free. Berkshire Hathaway class B
is spelled `BRK B` — verified live 2026-08-26 against TWS:

- `reqContractDetails(Stock("BRK B","SMART","USD"))` → `conId=72063691`,
  `localSymbol='BRK B'`, `tradingClass='BRK B'`, `primaryExchange='NYSE'`
- `BRK.B`, `BRK-B`, `BRKB` **all fail** with Error 200 (no security definition)
- `reqSecDefOptParams` on that conId → option `tradingClass='BRKB'` (no space)
- Yahoo spells it `BRK-B`

So one ticker legitimately needs **three different spellings** in its Tickers row:

| Column | Value | Source |
|---|---|---|
| `Name` | `BRK B` | IBKR contract symbol |
| `TradingClass` | `BRKB` | option class from `reqSecDefOptParams` |
| `YahooName` | `BRK-B` | Yahoo |

Other Tickers rows already carry dots for the same reason (`U.UN`, `CA.PA`,
`EUR.USD`) — the punctuation is IBKR's, not a formatting choice.

**`Tickers.Name` is the canonical key.** `getTicker`/`getYahooName` match it
**exactly** — `getYahooName("BRK.B")` returns `NA`, not `BRK-B`. Any user-typed
symbol must be resolved to `Name` before it touches Tdata.

**This is not a BRK-B curiosity — measured 2026-08-26 over Tickers `Type='STK'`:**

| divergence | rows | examples |
|---|---|---|
| `YahooName != Name` | 46 | every `.PA`/`.SW`/`.L`/`.DE`/`.TO` name — ABBN.SW, SIE.DE, U-UN.TO |
| `TradingClass != Name` | 14 | TTE→TOTB, SGO→GOB, UBSG→UBSN, RO→ROG, SAF→SEJ, HOLN→HOLNO |
| `TradingClass` blank/NA | 89 | fall back to the symbol itself |

`BRK B` is the only row where **all three** spellings differ, which is why it
broke things nothing else did.

**tdata_py auto-resolves TradingClass — except in one function.** This asymmetry
is the trap:

- `getExpirationDates`, `getStrikesAuto`, `get_chain_oi` → `tradingClass=None`
  default, resolve internally ("finds best trading class"). Pass `sym` alone.
- `compute_spread_risk_reward(sym, trading_class, ...)` → **required positional,
  no resolution.** Every caller must look up `Tickers.TradingClass` itself.
  Passing the symbol yields `Chain not available for BRK B BRK B` and zero
  strikes. Both callers are now correct (`Tuser/spread/app.R:201`,
  `RStudies/reports/analyze/structures.R`).

**Consequences already handled in code (don't re-break):**
- `Tdata::getInstrument` parses option Instrument strings
  (`"<SYM> <DDMMMYY> <STRIKE> <P|C>"`) from the **end** of the token list, since a
  spaced symbol yields five tokens, not four.
- `Tuser/ticker/app.R::validate_form` rejects only tabs/newlines and *repeated*
  spaces, not a single internal space.
- `/analyze` `main.R::.resolve_ticker()` maps `BRK.B`/`BRK-B`/`BRKB` → `BRK B`
  and **stops** on an unknown symbol; a space-form symbol passed unquoted is
  rejoined from `argv` instead of reading `B` as DIRECTION.
- `RStudies/reports/shared/live_sources.R::.live_rv30` routes through
  `getYahooName` before calling yfinance. The scanner's `fetch.R:12` already did.

**How to apply:** never assume a symbol is one whitespace-free token — parse
option/instrument strings from the right, resolve user input to `Tickers.Name`
before any Tdata call, and use `Name`/`TradingClass`/`YahooName` deliberately per
call site rather than reusing one string for all three. Confirm the exact IBKR
spelling with `reqContractDetails` before adding or renaming a Tickers row (see
[[feedback_test_tws_first]]). For futures the field conventions differ again —
see [[reference_tickers_fut_row_convention]]. When a report comes back all-failed,
see [[feedback_cascade_failures_check_root_input]].
