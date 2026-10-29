---
name: Trades table schema (mydb.db)
description: Column layout and conventions for the Trades table in data/mydb.db (English column names since 2026; Symbol is the ticker)
metadata:
  type: reference
---
DB at `Sys.getenv("R_DB_PATH")` = `C:/Users/aldoh/Documents/RApplication/data/mydb.db`. Columns (checked 2026-10-26):

TradeNr, Account, TradeDate (INTEGER YYYYMMDD), DateTime, TimeZoneSource, Strategy, Instrument, **Symbol**, Pos, **Price**, **Commission**, Total, Exp.Date, Risk, Reward, PnL, **Status**, Currency, **Notes**, Strike, Right, EventType, Return, **Underlying**, Stop, UnderlyingPrice.

- The old French names are gone: `Ssjacent`→`Symbol`, `Prix`→`Price`, `Comm.`→`Commission`, `Statut`→`Status`, `Remarques`→`Notes`. Any code still using them errors ("no such column").
- **Key per-name joins to `Tickers.Name` on `Symbol`, not `Underlying`.** `Underlying` is NULL on all but 4 BOT rows and holds the futures contract there (GCM5 while Symbol = GC).
- `Strategy = 'BOT'` is the "long bought" strategy, not an action column. There is no `Action` column.
- `PnL` is populated only on the closing leg. `Status = 'Fermé'` marks closed trades.

Closed-BOT record per name (pattern used by bot_monthly's trade-record override):
```sql
SELECT name, COUNT(*) n, SUM(pnl) pnl FROM (
  SELECT TradeNr, Symbol AS name, SUM(PnL) pnl FROM Trades
   WHERE Strategy = 'BOT' GROUP BY TradeNr, Symbol HAVING COUNT(PnL) > 0)
GROUP BY name
```
