---
name: feedback_sql_aggregation_event_filter
description: "When verifying per-trade invariants (e.g. sum(Risk)=0 after backfill), don't filter the aggregation by EventType — use a subquery for the trade-set, then aggregate over all rows. Filtering-then-aggregating gave a false-alarm rollback signal after a clean Phase 3 commit on 2026-05-18."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4ed7508e-d187-4257-a7e0-5e399d43e3c1
---

When the Trades table stores rows that play different roles in a trade lifecycle (Open / Adjust / Close after [[project_trades_signed_delta_risk]]), verification queries that aggregate per-trade must NOT filter rows by EventType before the aggregation — they need every row of every trade in scope.

**Wrong** (the bug that triggered a false rollback panic post Phase 3):
```sql
SELECT TradeNr, SUM(Risk) AS sum_risk
  FROM Trades
 WHERE Strategy='BOT' AND EventType='Close'
 GROUP BY TradeNr
 HAVING ABS(sum_risk) > 0.01;
```
This only sums the Close rows — which carry the negative balancing values. Every closed BOT trade looks "broken" (sum = negative balancing) when in fact the full-trade sum is zero.

**Right**:
```sql
SELECT TradeNr, SUM(Risk) AS sum_risk
  FROM Trades
 WHERE TradeNr IN (SELECT DISTINCT TradeNr FROM Trades
                    WHERE Strategy='BOT' AND EventType='Close')
 GROUP BY TradeNr
 HAVING ABS(sum_risk) > 0.01;
```
Subquery scopes the trade set; outer SELECT aggregates over the whole trade.

**Why:** Phase 3 backfill (2026-05-18) wrote a balancing negative `Risk` on each Close row to make `sum(Risk over TradeNr) = 0`. A post-commit verification using the first query reported "98 of 101 closed BOTs still broken" when all 101 were actually correct. Almost triggered a destructive rollback of a clean commit.

**How to apply:** Any post-migration sanity check on event-typed tables. Same pattern applies to dplyr: `filter(EventType=="Close") |> group_by(TradeNr) |> summarize(...)` will drop the rows you need. Use `group_by(TradeNr) |> summarize(...)` over the full row set, or scope via `filter(TradeNr %in% closed_set)` first.
