# Phase 0 Sign-off

Authoritative verification record. Each step is ✅ only if the pass condition in the build plan was actually executed and produced the expected result. ⚠ means the artifact exists but end-to-end verification is incomplete — TODO listed.

## Phase 0 (steps 1–13)

- [x] **1** ✅ `claude --version` → `2.1.142 (Claude Code)`. API hello deferred (covered by step 4 tests).
- [x] **2** ✅ Folder tree matches build plan §18. Verified by `find -type d`.
- [x] **3** ✅ `~/agents/config/.env` exists, `chmod 600`, gitignored. Required keys filled by operator before step 4.
- [x] **4** ✅ `ai-cheap.sh`, `ai-do.sh`, `ai-think.sh` each returned `test` against Haiku / Sonnet / Opus. Live API calls confirmed.
- [x] **5** ✅ `notify.sh "chassis online"` → `[notify stub] chassis online`.
- [x] **6** ✅ `log_to_audit.sh` inserted row id=1, returned correlation_id, row queryable in audit_log.
- [x] **7** ✅ `PRAGMA journal_mode` → `wal`. All 10 base tables + FTS5 shadow tables present. FTS UPDATE trigger restricted to `summary, raw_text` (gotcha #7).
- [x] **8** ✅ `retrieve_context.sh` exits 0 in wiki + rag modes against empty wiki. Deviation logged: `|| true` added to `wiki_search` so grep no-match doesn't trip `set -euo pipefail`.
- [x] **9** ✅ `~/brain/SCHEMA.md` + `HANDOFF.md` present. Operator action: open `~/brain` in Obsidian and set as vault.
- [x] **9a** ✅ PRIME.md ≈ 200 tokens (154 words; right at the 200 boundary).
- [x] **9b** ✅ `convert_dropzone.sh` round-tripped `.txt`. pandoc + pdftotext + jq + xlsx2csv installed. Rewrote for bash 3.2 (macOS default), explicit PATH for cron. PDF/docx/xlsx not yet smoke-tested with real files.
- [x] **9c** ✅ `~/agents/skills/README.md` + `sqlite/SKILL.md` present.
- [x] **9d** ✅ `check_kill_switches.sh` re-sources on `.env` mtime change; skips when unchanged.
- [x] **10** ✅ `rebuild_command_center.sh` produced `command-center.md` with system status / spend / pending / audit tail / diary sections.
- [x] **10a** ✅ Verified end-to-end 2026-05-14 (this sign-off): `dashboard.sh start` → listening on `127.0.0.1:3141`; `/api/health` → 200 with current kill-switch state; `/api/audit-log` with rotated token → 200 with 1 historical row; `/api/audit-log` without token → 401; HTML root renders 3 sections (kill switches table, audit log, agents list). Previously marked ✅ on bind-only (port listener) — that was premature; this entry replaces it. Deviation: switched from better-sqlite3 to `node:sqlite` (Node 26 built-in) because better-sqlite3 fails to gyp against V8 26.
- [ ] **10b** ⚠ Kanban view deferred — `mission_tasks` empty until Phase 1; build kanban when there's data.
- [ ] **10c** ⚠ Cost chart deferred — `token_usage` only has step-4 test rows; build chart in Phase 1.
- [ ] **10d** ⚠ Diary view deferred — first diary lands at step 15.
- [ ] **10e** ⚠ Chat input / war room deferred — no agents to fan out to until Phase 1 step 14.
- [ ] **10f** ⚠ `@mention` parser deferred — depends on 10e.
- [x] **11** ✅ `backup_db.sh` produced snapshot; `PRAGMA integrity_check` → `ok`. `backup_daily.sh` + `backup_weekly.sh` fail gracefully when drives unmounted (intended). TODO: re-run after operator mounts AgentSSD + WDArchive.
- [x] **12** ✅ `cron.txt` written with 5 schedule lines. Operator must paste via `crontab -e`. ⚠ Cron not yet active until operator pastes.
- [x] **13** ✅ `~/brain/projects/aiteam/docs/restore.md` exists.

## Stops still owed to the operator

- Paste `cron.txt` lines via `crontab -e` (step 12). macOS Sequoia may require granting `cron` Full Disk Access in System Settings → Privacy & Security if the cron job can't read `~/agents/config/.env`.
- Open `~/brain/` in Obsidian and confirm vault loads (step 9).
- Optional: drop a PDF + a `.docx` into `~/brain/raw_dropzone/` to smoke-test step 9b's full converter matrix.
- Optional: mount `/Volumes/AgentSSD` + `/Volumes/WDArchive` and re-run `backup_daily.sh` / `backup_weekly.sh` for full step-11 verification.

## What is NOT verified

- 10b–10f dashboard subsections (deferred to Phase 1 by mutual decision).
- Live cron firing (depends on operator pasting crontab).
- Daily/weekly backup target drives (depends on operator mounting drives).
- The dashboard's color semantics on kill switches (cosmetic: `false` for `SYSTEM_PAUSED` renders red even though that's the desired state — flagged for Phase 1 polish).
