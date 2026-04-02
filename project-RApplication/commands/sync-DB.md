# Sync Database Command

Bidirectional database synchronization between local PC and Google Cloud VM (trading-vm).

**INSTRUCTIONS FOR CLAUDE:**

You MUST follow these steps sequentially. Report progress at each step.

## Step 0: Parse Arguments

Parse optional flags from $ARGUMENTS:
- `--help` or `-h`: Show usage help and examples, then stop
- `--pull`: Force pull direction (VM -> PC), skip analysis
- `--push`: Force push direction (PC -> VM), skip analysis
- `--dry-run`: Show what would happen without making changes
- `--update`: Also sync value-changed rows (not just new rows)
- No arguments: Smart mode (compare, analyze, then act)

### If `--help` or `-h` is present, display this and STOP:

```
/sync-DB - Bidirectional database sync between local PC and Google Cloud VM

USAGE
  /sync-DB [flags]

FLAGS
  (none)       Smart mode: compare -> analyze -> recommend -> execute
  --dry-run    Compare and recommend only, no changes made
  --pull       Force pull VM -> Local (skip analysis)
  --push       Force push Local -> VM (skip analysis)
  --update     Also overwrite value-changed rows (same key, different values)
  --help, -h   Show this help

SMART MODE WORKFLOW
  1. Starts the VM if needed
  2. Downloads VM database snapshot
  3. Compares all tables by content hash
  4. Analyzes differing tables at row level (new rows, changed values)
  5. Decides direction automatically:
     - Only VM has changes   -> Pull (VM -> Local)
     - Only Local has changes -> Push (Local -> VM)
     - Both sides changed     -> Merge (bidirectional)
  6. Asks for confirmation, backs up target DB, executes
  7. Uploads to VM if needed, verifies, cleans up

EXAMPLES
  /sync-DB                 Full smart sync (most common)
  /sync-DB --dry-run       See what differs without changing anything
  /sync-DB --pull          Just pull VM data to local (e.g. after VM ran overnight)
  /sync-DB --push          Just push local data to VM (e.g. before VM scheduled run)
  /sync-DB --update        Smart sync including value-changed rows
  /sync-DB --pull --update Pull VM data including updated values

SAFETY
  - Always backs up target DB before writing
  - Asks for confirmation before executing
  - merge_db.ps1 only inserts new rows by default (--update for overwrites)
  - VM is NOT auto-stopped after sync (reminder shown if started)

SCRIPTS USED
  scripts/merge_db.ps1     Row-level merge (insert new rows, optionally update)
  scripts/diff_db.ps1      Row-level diff (for detailed inspection)
  scripts/sync_db.ps1      Full table replace (Compare/Pull/Push)
```

Do NOT proceed to Step 1. Return immediately after showing help.

## CRITICAL: gcloud Must Run via PowerShell

**gcloud CLI fails in Git Bash** (Python not found). ALL gcloud commands MUST be wrapped:

```bash
powershell -Command "gcloud compute ..."
```

Never run bare `gcloud` in bash.

## CRITICAL: VM Two-User SSH Issue

The VM has two users:
- `aldoh` — gcloud default SSH user
- `aldohlys` — owns the DB file and RProjects directory

**Consequence**: Cannot SCP directly to `/home/aldohlys/...` (permission denied).

**Pattern for uploading files to VM**:
1. SCP to `/tmp/` (any user can write)
2. SSH as `aldohlys` to copy from `/tmp/` to final path
3. Tolerate `/tmp` cleanup failures with `|| true`

```bash
# Upload
powershell -Command "gcloud compute scp 'LOCAL_FILE' 'trading-vm:/tmp/mydb_upload.db' --zone=us-east1-b --quiet"
# Copy as aldohlys
powershell -Command "gcloud compute ssh trading-vm --zone=us-east1-b --quiet --ssh-flag='-l' --ssh-flag='aldohlys' --command='cp /tmp/mydb_upload.db /home/aldohlys/RProjects/RApplication/data/mydb.db; rm -f /tmp/mydb_upload.db || true'"
```

## Step 1: Ensure VM is Running

Check VM status and start it if needed:

```bash
powershell -Command "gcloud compute instances describe trading-vm --zone=us-east1-b --format='get(status)'"
```

If not RUNNING, start it:
```bash
powershell -Command "gcloud compute instances start trading-vm --zone=us-east1-b --quiet"
```

Then wait 20s + retry SSH up to 3 times with 15s intervals (auto-accept host key with `echo 'y' |`):
```bash
sleep 20 && powershell -Command "echo 'y' | gcloud compute ssh trading-vm --zone=us-east1-b --quiet --command='echo ok'"
```

Record whether YOU started the VM (to remind user at the end).

## Step 2: Download VM Database Snapshot

```bash
powershell -Command "gcloud compute scp 'trading-vm:/home/aldohlys/RProjects/RApplication/data/mydb.db' 'C:\Users\aldoh\Documents\RApplication\data\mydb_remote_snapshot.db' --zone=us-east1-b --quiet"
```

Verify the download succeeded before continuing.

## Step 3: Compare Tables

For each table in the local DB, compare content hashes between local and VM snapshot.

Write and execute a temp script:

```bash
cat > /tmp/compare_db.sql << 'EOF'
.mode column
.headers on
SELECT 'Comparing databases...' AS status;
EOF
sqlite3 "C:/Users/aldoh/Documents/RApplication/data/mydb.db" ".tables"
```

For each table, get row counts from both DBs and compare using SHA256 of `.dump` output.

**IMPORTANT**: Some tables may exist only on one side. Get tables from BOTH DBs and union them. For tables missing on one side, mark as `LOCAL-ONLY` or `VM-ONLY` instead of querying counts (which would error).

```bash
powershell -Command "
\$LOCAL_DB = 'C:\Users\aldoh\Documents\RApplication\data\mydb.db'
\$REMOTE_DB = 'C:\Users\aldoh\Documents\RApplication\data\mydb_remote_snapshot.db'
\$localTables = ((sqlite3 \$LOCAL_DB '.tables') -join ' ') -split '\s+' | Where-Object { \$_ -ne '' }
\$remoteTables = ((sqlite3 \$REMOTE_DB '.tables') -join ' ') -split '\s+' | Where-Object { \$_ -ne '' }
\$allTables = (\$localTables + \$remoteTables) | Sort-Object -Unique
foreach (\$t in \$allTables) {
    \$inLocal = \$t -in \$localTables
    \$inRemote = \$t -in \$remoteTables
    if (\$inLocal -and -not \$inRemote) {
        \$lc = (sqlite3 \$LOCAL_DB \"SELECT COUNT(*) FROM \$t;\").Trim()
        Write-Host ('{0,-30} {1,8} {2,8} {3}' -f \$t, \$lc, '-', 'LOCAL-ONLY')
    } elseif (-not \$inLocal -and \$inRemote) {
        \$rc = (sqlite3 \$REMOTE_DB \"SELECT COUNT(*) FROM \$t;\").Trim()
        Write-Host ('{0,-30} {1,8} {2,8} {3}' -f \$t, '-', \$rc, 'VM-ONLY')
    } else {
        \$lc = (sqlite3 \$LOCAL_DB \"SELECT COUNT(*) FROM \$t;\").Trim()
        \$rc = (sqlite3 \$REMOTE_DB \"SELECT COUNT(*) FROM \$t;\").Trim()
        \$ld = (sqlite3 \$LOCAL_DB \".dump \$t\") -join [char]10
        \$rd = (sqlite3 \$REMOTE_DB \".dump \$t\") -join [char]10
        \$sha = [System.Security.Cryptography.SHA256]::Create()
        \$lh = [BitConverter]::ToString(\$sha.ComputeHash([System.Text.Encoding]::UTF8.GetBytes(\$ld))).Replace('-','').Substring(0,16)
        \$rh = [BitConverter]::ToString(\$sha.ComputeHash([System.Text.Encoding]::UTF8.GetBytes(\$rd))).Replace('-','').Substring(0,16)
        \$status = if (\$lh -eq \$rh) { 'equal' } else { 'DIFFERS' }
        Write-Host ('{0,-30} {1,8} {2,8} {3}' -f \$t, \$lc, \$rc, \$status)
    }
}
"
```

Present results as a table:

```
Table                          Local       VM   Status
-----                          -----       --   ------
Account                          120      125   DIFFERS
Trades                           340      340   equal
...
```

**IMPORTANT**: Exclude VIEWS from comparison. `AccountWithConversionRate` is a VIEW — its `.dump` is just the CREATE VIEW statement (always matches), while its row count reflects the underlying Account table. Query `sqlite_master` to identify views:
```sql
SELECT name FROM sqlite_master WHERE type='view';
```
Skip any names returned.

If ALL tables are equal: report "Databases are in sync" and skip to Step 7 (cleanup).

## Step 4: Analyze Differences (Row-Level)

For each table that DIFFERS, determine the nature of the difference using the natural keys defined below:

**Table Keys** (for EXCEPT queries):
| Table | Natural Keys |
|-------|-------------|
| Account | account, date, heure |
| U1804173 | TradeNr, date, symbol, expdate, strike |
| DU5221795 | TradeNr, date, symbol, expdate, strike |
| Gonet | TradeNr, date, symbol |
| Trades | TradeNr, Instrument, Pos, Prix, TradeDate |
| Journal | entryId |
| Prices | datetime, sym |
| ConvertToCHF | date, currency |
| ConvertToUSD | date, currency |
| Tickers | Name |
| Currencies | Name |
| Strategies | Name |
| Param | Name |
| Alerts | id |
| ScannerUniverse | Symbol |
| scanner_history | date, Symbol |
| scanner_transitions | date, Symbol |
| mrbreakouts_cache | date, Symbol |
| macro_regimes | date |
| macro_scenarios | scenario_id |
| macro_breadth_cache | date |
| macro_context_cache | date |
| macro_context_results | date |
| macro_context_mismatches | date, ticker |
| backtest_regime_signals | date |

**For tables not in this list** (or LOCAL-ONLY / VM-ONLY tables without keys defined):
skip row-level EXCEPT analysis. For local-only tables, they'll be pushed as whole tables.
For VM-only tables, they'll be pulled as whole tables.

For each differing table, run EXCEPT queries to count:
- **Local-only rows**: rows in Local but not in VM (by key)
- **VM-only rows**: rows in VM but not in Local (by key)
- **Value-changed rows**: same key exists on both sides but non-key columns differ

Use ATTACH to query both DBs:

```sql
ATTACH 'C:/Users/aldoh/Documents/RApplication/data/mydb_remote_snapshot.db' AS vm;

-- Local-only keys
SELECT <keys> FROM main.<table>
EXCEPT
SELECT <keys> FROM vm.<table>;

-- VM-only keys
SELECT <keys> FROM vm.<table>
EXCEPT
SELECT <keys> FROM main.<table>;
```

Present a summary:

```
=== Difference Analysis ===

Table: Account
  Local-only rows:  0
  VM-only rows:     5    (VM has new data)
  Value-changed:    2

Table: Prices
  Local-only rows:  12   (Local has new data)
  VM-only rows:     0
  Value-changed:    0
```

## Step 5: Decide Action

Based on the analysis, classify the situation and recommend an action:

### Case A: Only VM has new/changed rows (no local-only rows in ANY table)
- **Recommendation**: PULL (VM -> Local)
- **Method**: `merge_db.ps1 -Direction VMToLocal -Apply` (and `-Update` if value-changed rows exist)
- Tell user: "VM has newer data. Pulling VM -> Local."

### Case B: Only Local has new/changed rows (no VM-only rows in ANY table)
- **Recommendation**: PUSH (Local -> VM)
- **Method**: `merge_db.ps1 -Direction LocalToVM -Apply` (and `-Update` if value-changed rows exist)
- Then upload snapshot: `gcloud compute scp` back to VM
- Tell user: "Local has newer data. Pushing Local -> VM."

### Case C: Both sides have new rows (bidirectional changes)
- **Recommendation**: MERGE (bidirectional)
- **Method**: `merge_db.ps1 -Apply` (and `-Update` if value-changed rows exist)
- Then upload snapshot: `gcloud compute scp` back to VM
- Tell user: "Both sides have changes. Merging bidirectionally."

### Case D: Only value-changed rows (same keys, different values)
- **Recommendation**: Ask user which side is authoritative
- Present the tables with value changes and ask: "Which version should win — Local or VM?"

### Local-only / VM-only tables (always applies in addition to above cases)
- **Local-only tables**: Push whole tables to VM snapshot via SQL: `CREATE TABLE IF NOT EXISTS vm.<table> AS SELECT * FROM main.<table>`
- **VM-only tables**: Pull whole tables to local via SQL: `CREATE TABLE IF NOT EXISTS main.<table> AS SELECT * FROM vm.<table>`
- These are handled SEPARATELY from merge_db.ps1 (which only handles common tables)

**If `--dry-run` flag**: Show the recommendation and stop. Do not execute.

**Present the recommendation to the user and ask for confirmation before proceeding.**

## Step 6: Execute Sync

### 6a: Backup First

Always backup the target DB before any writes:

```bash
# For Pull (backup local)
cp "C:/Users/aldoh/Documents/RApplication/data/mydb.db" "C:/Users/aldoh/Documents/RApplication/data/backups/mydb_pull_$(date +%Y%m%d_%H%M%S).db"

# For Push (backup VM snapshot which will be uploaded)
cp "C:/Users/aldoh/Documents/RApplication/data/mydb_remote_snapshot.db" "C:/Users/aldoh/Documents/RApplication/data/backups/mydb_push_$(date +%Y%m%d_%H%M%S).db"
```

### 6b: Execute via merge_db.ps1

Run the merge script from the scripts directory. The `-Apply` flag makes it execute (vs dry-run).

**Pull (VM -> Local):**
```bash
cd C:/Users/aldoh/Documents/RApplication/scripts && powershell -File merge_db.ps1 -Direction VMToLocal -Apply
```

**Push (Local -> VM):**
```bash
cd C:/Users/aldoh/Documents/RApplication/scripts && powershell -File merge_db.ps1 -Direction LocalToVM -Apply
```

**Merge (Both directions):**
```bash
cd C:/Users/aldoh/Documents/RApplication/scripts && powershell -File merge_db.ps1 -Apply
```

Add `-Update` flag if value-changed rows need syncing:
```bash
cd C:/Users/aldoh/Documents/RApplication/scripts && powershell -File merge_db.ps1 -Apply -Update
```

### 6b-2: Sync Local-Only / VM-Only Tables

**IMPORTANT**: merge_db.ps1 only handles tables present in BOTH databases. Tables that exist on only one side must be handled separately via raw SQL.

**Local-only tables → push to VM snapshot:**
```bash
# Build SQL dynamically for each local-only table
cat > /tmp/push_local_tables.sql << 'SQLEOF'
ATTACH 'C:/Users/aldoh/Documents/RApplication/data/mydb_remote_snapshot.db' AS vm;
CREATE TABLE IF NOT EXISTS vm.<table1> AS SELECT * FROM main.<table1>;
CREATE TABLE IF NOT EXISTS vm.<table2> AS SELECT * FROM main.<table2>;
-- ... repeat for each local-only table
DETACH vm;
SQLEOF
sqlite3 "C:/Users/aldoh/Documents/RApplication/data/mydb.db" < /tmp/push_local_tables.sql
```

**VM-only tables → pull to local:**
```bash
cat > /tmp/pull_vm_tables.sql << 'SQLEOF'
ATTACH 'C:/Users/aldoh/Documents/RApplication/data/mydb_remote_snapshot.db' AS vm;
CREATE TABLE IF NOT EXISTS main.<table1> AS SELECT * FROM vm.<table1>;
-- ... repeat for each VM-only table
DETACH vm;
SQLEOF
sqlite3 "C:/Users/aldoh/Documents/RApplication/data/mydb.db" < /tmp/pull_vm_tables.sql
```

### 6c: Upload to VM (Push or Merge only)

After LocalToVM or Both merge, upload the merged snapshot back to the VM.

**CRITICAL**: Must use the two-step upload pattern (see "VM Two-User SSH Issue" above):

```bash
# Step 1: Upload to /tmp (gcloud SSH user = aldoh)
powershell -Command "gcloud compute scp 'C:\Users\aldoh\Documents\RApplication\data\mydb_remote_snapshot.db' 'trading-vm:/tmp/mydb_upload.db' --zone=us-east1-b --quiet"

# Step 2: Copy to final path as aldohlys, tolerate /tmp cleanup failure
powershell -Command "gcloud compute ssh trading-vm --zone=us-east1-b --quiet --ssh-flag='-l' --ssh-flag='aldohlys' --command='cp /tmp/mydb_upload.db /home/aldohlys/RProjects/RApplication/data/mydb.db; rm -f /tmp/mydb_upload.db || true'"
```

Verify on VM (use simple commands — avoid special characters like semicolons in gcloud --command):
```bash
powershell -Command "gcloud compute ssh trading-vm --zone=us-east1-b --quiet --ssh-flag='-l' --ssh-flag='aldohlys' --command='ls -la /home/aldohlys/RProjects/RApplication/data/mydb.db'"
powershell -Command "gcloud compute ssh trading-vm --zone=us-east1-b --quiet --ssh-flag='-l' --ssh-flag='aldohlys' --command='sqlite3 /home/aldohlys/RProjects/RApplication/data/mydb.db .tables'"
```

### 6d: Verify Sync

Re-run the hash comparison from Step 3 (for Pull, compare local DB against snapshot; for Push/Merge, the snapshot was just uploaded so they should match).

Report: "Sync complete. X table(s) synchronized."

## Step 7: Cleanup

Remove the remote snapshot file:

```bash
rm -f "C:/Users/aldoh/Documents/RApplication/data/mydb_remote_snapshot.db"
```

If the VM was started by this command, remind the user:

```
NOTE: VM was started for this sync and is still running.
To stop it: gcloud compute instances stop trading-vm --zone=us-east1-b --quiet
```

## Output Summary

At the end, present a concise summary:

```
=== DB Sync Complete ===
Direction: Pull (VM -> Local)  /  Push (Local -> VM)  /  Merge (bidirectional)
Tables synced: Account, Prices, U1804173
Rows inserted: 17
Rows updated: 3
Backup: data/backups/mydb_pull_20260402_143022.db
VM status: running (started by sync)
```

## Examples

- `/sync-DB` — Smart sync: compare, analyze, recommend, execute
- `/sync-DB --dry-run` — Compare and recommend only, no changes
- `/sync-DB --pull` — Force pull VM -> Local (skip analysis)
- `/sync-DB --push` — Force push Local -> VM (skip analysis)
- `/sync-DB --update` — Also sync value-changed rows (not just new rows)

## Notes

- merge_db.ps1 requires the remote snapshot file to exist (downloaded in Step 2)
- merge_db.ps1 only INSERTS new rows by default; `-Update` also overwrites changed values
- merge_db.ps1 only handles tables that exist in BOTH databases — local-only/VM-only tables need separate SQL
- sync_db.ps1 Pull/Push does full table REPLACE — merge_db.ps1 is safer for mixed scenarios
- Always backup before writing
- The snapshot file acts as the intermediary for Push/Merge operations

## Gotchas (Lessons Learned 2025-04-02)

1. **gcloud in Git Bash**: Always wrap with `powershell -Command "gcloud ..."` — bare gcloud fails (Python not found)
2. **VM two users**: SCP goes to `aldoh`, DB owned by `aldohlys` → upload to `/tmp`, then SSH as aldohlys to `cp`
3. **Cross-user /tmp cleanup**: `rm` fails on files created by other user → always append `|| true`
4. **gcloud --command quoting**: Avoid semicolons and single-quoted SQL in `--command='...'` — gcloud parses them as its own args. Use simple commands or write temp scripts on VM
5. **Local-only tables**: merge_db.ps1 silently ignores them — must use `CREATE TABLE IF NOT EXISTS` via raw SQL
6. **Table list from both sides**: Always union local + VM table lists before comparing — querying a missing table errors out
7. **Views look like tables**: `AccountWithConversionRate` is a VIEW — `.dump` returns identical CREATE VIEW on both sides (hash matches) but row counts differ. Exclude views from comparison by checking `sqlite_master WHERE type='view'`
