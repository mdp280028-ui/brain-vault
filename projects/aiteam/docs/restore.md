# AITEAM Restore Procedure

> Rewritten 2026-10-26 (FLEET-AUDIT-55). The May-14 version pointed at backup locations that no longer exist (`~/.claudeclaw-backups` holds one file from 2026-05-14; `/Volumes/AgentSSD` and `/Volumes/WDArchive` are not attached), and its restore step `cp`'d a file over the live WAL-mode DB.
> **Tested:** steps 2-3 below were run on 2026-10-26 against the 23:30 nightly backup. All checks matched: SHA-256, `integrity_check` ok, schema identical, `audit_log` content md5 identical for 39,998 rows (OVERNIGHT-AUDIT-1 §1E). Step 4's `.restore` was tested on scratch copies only (a WAL-mode DB was fully replaced: integrity ok, identical `audit_log` md5). **Not tested:** step 4 against the real `~/store/aiteam.db`.

## When to restore
- Database corruption (`PRAGMA integrity_check` returns anything but `ok`)
- Catastrophic agent misbehavior (deleted or corrupted records)
- Hardware swap to a new Mac Mini

## Where the backups are

| Source | What | How fresh |
|---|---|---|
| `/Volumes/Backups/mini-backup/` | nightly `scripts/backup-rsync.sh` (launchd `com.aiteam.backup`, 23:30): store, agents, brain, 4 site/pipeline repos, hobbit-village | last night. **Single generation** for files: each night's `rsync --delete` replaces the last |
| `/Volumes/Backups/mini-backup/store/aiteam-backup.db` | consistent `sqlite3 .backup` snapshot of `aiteam.db` | last night (overwritten nightly) |
| `/Volumes/Backups/mini-backup/db-generations/aiteam-YYYY-MM-DD.db` | dated DB copy, one per night, never pruned (since 2026-10-26, `ad816a1`) | every night since 2026-10-26 |
| Time Machine (`tmutil listbackups`) | whole disk incl. store/agents/brain; 54 snapshots on 2026-10-26 | hourly |
| `~/store/aiteam.db.bak-*` | one-off copies taken before bulk DB edits (e.g. `.bak-d6-…`, `.bak-d7-…` on 2026-10-26) | as named |

The Backups volume is **not encrypted** and holds plaintext `.env` / session files (D129).

## Steps

### 1. Stop everything
Edit `~/agents/config/.env`: `SYSTEM_PAUSED=true`. Agents read it on their next run (via `lib/check_kill_switches.sh` or their own `source`). Then stop the long-running writers:
```bash
launchctl bootout gui/$(id -u)/com.aiteam.grammy-bot
launchctl bootout gui/$(id -u)/com.aiteam.dashboard
ps -axo pid,command | grep -E '[a]i-(do|think|cheap)\.sh|[r]un-batch|[c]laude -p'   # wait until empty
```

### 2. Copy the backup to a temp dir. Never restore in place
```bash
B=/Volumes/Backups/mini-backup        # or a db-generations/aiteam-YYYY-MM-DD.db file, or: tmutil restore <snapshot>/Users/mmm2/store/aiteam.db "$T/"
T=$(mktemp -d)
cp -p "$B/store/aiteam-backup.db" "$T/aiteam.db"
cp -p "$B/agents/lib/notify.sh"   "$T/"          # any single file: same pattern
```

### 3. Verify before trusting it
```bash
shasum -a 256 "$B/store/aiteam-backup.db" "$T/aiteam.db"   # must match
sqlite3 "$T/aiteam.db" "PRAGMA integrity_check;"           # must print: ok
diff <(sqlite3 -readonly ~/store/aiteam.db .schema) <(sqlite3 "$T/aiteam.db" .schema)
M=$(sqlite3 "$T/aiteam.db" "SELECT max(id) FROM audit_log")
for db in ~/store/aiteam.db "$T/aiteam.db"; do sqlite3 -readonly "$db" "SELECT * FROM audit_log WHERE id<=$M ORDER BY id" | md5; done   # must match if the live DB is only newer, not damaged
```
If the live DB is the damaged one, the md5 comparison is expected to differ. Use it only to find where the damage starts.

### 4. Write it into place (with writers stopped)
```bash
sqlite3 ~/store/aiteam.db ".timeout 5000" ".restore '$T/aiteam.db'"
sqlite3 ~/store/aiteam.db "PRAGMA integrity_check;"        # ok
```
`.restore` goes through SQLite, so the live WAL is handled. Don't `cp` over `aiteam.db` while `-wal`/`-shm` exist.

### 5. Resume
`SYSTEM_PAUSED=false` in `.env`, then:
```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.aiteam.grammy-bot.plist
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.aiteam.dashboard.plist
bash ~/agents/majordomo/check.sh | tail -1     # expect 0 FAIL
```

### 6. Log the incident
Append to `~/brain/projects/aiteam/diary/$(date +%Y-%m-%d).md`:
```
## Incident: DB restore
Restored from <source + date> due to <REASON>.
Audit log shows last good action at <TIMESTAMP>.
Affected: <WHAT>.
```

## DO NOT
- Never `cp` a live WAL-mode SQLite file. New snapshots go through `.backup`, restores through `.restore`.
- Never restore over a live DB while agents are running.
- Never delete backups or `db-generations/` files before the restore is verified (pruning is Mark's call).
