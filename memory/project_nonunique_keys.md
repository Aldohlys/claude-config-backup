---
name: project_nonunique_keys
description: Sync/merge scripts hardcode keys that aren't actually unique in these two tables; JOIN-based merges produce Cartesian counts and -Update is unsafe
type: project
originSessionId: 20890e27-e237-4b5b-ad56-028d4f1ab618
---
The `TABLE_KEYS` hashtables in `scripts/diff_db.ps1`, `scripts/merge_db.ps1`, and `scripts/sync_db.ps1` declare row-identity keys for each table. For two tables these keys are NOT actually unique in practice:

- **Account** — key is `(account, date, heure)`. In practice the same sub-account can have multiple snapshots at the same minute (from overlapping daily updates, multi-account batch writes, etc.).
- **Trades** — key is `(TradeNr, Instrument, Pos, Prix, TradeDate)`. `TradeNr` is not row-unique; the same trade can legitimately appear multiple times sharing all five fields.

No `UNIQUE` constraint enforces this at the schema level, so `.schema` doesn't reveal the issue. You only see it when:
- `diff_db.ps1` reports "value-changed" counts that approach the total row count (e.g. 2350 out of 2357 for Trades — Cartesian explosion from JOIN on non-unique keys)
- `merge_db.ps1 -Apply` reports `WARN ... expected +N, got +M` where M > N (duplicate source rows sharing one missing key)

**Why:** Historical data layout and business meaning — these tables record events, not entities. `TradeNr` groups legs of the same trade, not one row. `Account` records snapshots, which can collide at minute granularity.

**How to apply:**
- NEVER use `merge_db.ps1 -Update` on Account or Trades. The DELETE step matches multiple target rows per key group, INSERT re-adds only matching source rows, net result is data loss.
- For wholesale sync of these tables, use `sync_db.ps1 Push -Tables Account,Trades` (REFRESH mode: DELETE + INSERT SELECT * preserves all rows and their multiplicity).
- Inflated "value-changed" counts in diff_db.ps1 output for these two tables are cosmetic noise from the JOIN pattern, not real diffs — verify with `SELECT * EXCEPT` if in doubt.
- If the defined keys ever need tightening (e.g. adding `DateTime` to Trades for true row uniqueness), the scripts' `TABLE_KEYS` must be updated in all three files in sync.
