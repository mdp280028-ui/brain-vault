# AITEAM Restore Procedure

## When to restore
- Database corruption (PRAGMA integrity_check returns errors)
- Catastrophic agent misbehavior (deleted or corrupted records)
- Hardware swap to a new Mac Mini

## Steps

### 1. Stop everything
Edit `~/agents/config/.env`:
```
SYSTEM_PAUSED=true
```
Save. Within 2 seconds, all agents that source `check_kill_switches.sh` will halt.

Confirm no agent processes are running:
```bash
ps aux | grep -E 'ai-(do|think|cheap)|orchestrator|librarian|editor' | grep -v grep
```
If any are running, kill them.

### 2. Identify backup to restore
```bash
ls -lt ~/.claudeclaw-backups/aiteam_*.db | head -10
```
Pick the most recent backup BEFORE the issue started.

### 3. Restore
```bash
cp ~/.claudeclaw-backups/aiteam_<TIMESTAMP>.db ~/store/aiteam.db
```

### 4. Verify integrity
```bash
sqlite3 ~/store/aiteam.db "PRAGMA integrity_check;"
# Should print: ok
```

### 5. Resume
Edit `~/agents/config/.env`:
```
SYSTEM_PAUSED=false
```

### 6. Log the incident
Append to `~/brain/projects/aiteam/diary/$(date +%Y-%m-%d).md`:
```
## Incident: DB restore
Restored from <TIMESTAMP> due to <REASON>.
Audit log shows last good action at <TIMESTAMP>.
Affected: <WHAT>.
```

## DO NOT
- Never `cp` a live WAL-mode SQLite file. Always use `.backup` for new snapshots.
- Never restore over a live DB while agents are running.
- Never delete the most recent N backups before verifying the restore succeeded.

## Restore from external SSD (catastrophic case)
If `~/.claudeclaw-backups` is also lost:
```bash
cp /Volumes/AgentSSD/backups/db/aiteam_<TIMESTAMP>.db ~/store/aiteam.db
```
Then verify and resume as above.

## Restore from WD HDD (full disaster)
If both internal and external SSD are gone:
```bash
ls /Volumes/WDArchive/aiteam_weekly_*
# Pick most recent
rsync -a /Volumes/WDArchive/aiteam_weekly_<DATE>/db/ ~/.claudeclaw-backups/
rsync -a /Volumes/WDArchive/aiteam_weekly_<DATE>/brain/ ~/brain/
rsync -a /Volumes/WDArchive/aiteam_weekly_<DATE>/agents/ ~/agents/
```
Then restore the DB as in step 3.
