---
name: Trades table schema (mydb.db)
description: Column layout and conventions for the Trades table in data/mydb.db, used for portfolio-history queries
type: reference
originSessionId: bc7eb8bf-68dc-4f7b-b7cb-a86ad3ffcd75
---
DB at `Sys.getenv("R_DB_PATH")` = `C:/Users/aldoh/Documents/RApplication/data/mydb.db`. Trades table columns:

- `TradeNr` INTEGER — trade ID (often 0 for older imports)
- `Account` TEXT — IBKR account (U1804173, U25343478, DU5221795...) or "Gonet"
- `TradeDate` INTEGER — YYYYMMDD format
- `DateTime` TEXT — ISO timestamp
- `TimeZoneSource` TEXT
- `Strategy` TEXT — values include: `OFI`, `BOT`, `BPT`, `Perso`, `Sharpe2`, `WHEEL`, `LTO`, `CS`, `CAL`, `A14`, `TBILL`, `Gonet`, `Erreur`, `Tactical`, `Money`, `CS SPY`, `FOREX`, `VALUE`, `SPY`, `TB`, `Ryan`, `Daubasses`, `CCT`, `Uranium`, `MC`, `RUT`, `IWM`, `WITT`, `USO`, `MOS`. **`BOT` is literally the strategy name for "long bought" trades** (not an action column).
- `Instrument` TEXT — full description, often includes spreads (e.g. "HOLN Vertical Spread 16.12.2022 40.5/43 1P/-1P")
- `Ssjacent` TEXT — **the underlying ticker** (e.g. "AAPL", "VZ", "HOLN")
- `Pos` INTEGER — signed quantity (>0 long, <0 short)
- `Prix` REAL — entry price
- `Comm.` REAL — commission
- `Total` REAL — net cash flow
- `Exp.Date` TEXT — option expiry, French format DD.MM.YYYY (string, not date)
- `Risk` REAL, `Reward` REAL — planned values
- `PnL` REAL — realized P&L (populated only on close)
- `Statut` TEXT — `'Fermé'` (closed) or other; **filter `Statut = 'Fermé'`** for realized winners
- `Currency` TEXT — USD/EUR/CHF/JPY
- `Remarques` TEXT
- `Strike` TEXT, `Right` TEXT — option-leg fields

There is **no `Action` column** despite IBKR-style nomenclature.

Top realized winners by underlying for closed BOT-strategy trades (snapshot 2026-04-28): AAPL, GOLD, XOM, DOW, JNJ, OXY, URA, WMT, NFLX, TGT, GLD.

Useful query template for "best long winners by underlying":
```sql
SELECT Ssjacent AS sym, ROUND(SUM(PnL),0) AS total_pnl, COUNT(*) AS n_trades
FROM Trades
WHERE Statut = 'Fermé' AND PnL IS NOT NULL AND PnL != 0
  AND Strategy = 'BOT'
  AND Ssjacent IS NOT NULL AND Ssjacent != ''
GROUP BY Ssjacent HAVING total_pnl > 0
ORDER BY total_pnl DESC LIMIT 20
```
