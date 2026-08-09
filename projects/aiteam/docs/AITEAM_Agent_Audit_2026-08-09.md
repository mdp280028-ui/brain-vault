# AITEAM Agent Fleet Audit — 2026-08-09

**Mode:** read-only. Zero files modified except this one.
**Auditor:** Claude Code (Opus 5), session 41a5a146.
**Ground truth:** crontab, launchctl, filesystem, git log, `~/store/aiteam.db`.
Docs were read only to detect drift, never to establish state.

---

## SUMMARY

### 1. Findings, ranked by severity

| ID | SEV | Ph | AGENT / FILE | FINDING | CLASS | FIX | HRS |
|---|---|---|---|---|---|---|---|
| **F01** | **CRITICAL** | 1 | `ssg-content/content/run-batch.sh` | SSG pipeline has **no source-verification gate**; asbestos has one at `:1499-1533` and it fires. Fabricated figures with real citations can ship to the live SSG site unchecked. | NEVER-WORKED | Port `verify_sources.py` + the 34-line gate block from asbestos | 3 |
| **F02** | HIGH | 8 | `watchdog/watchdog.sh:190-194` | **254 of 349 incidents ever raised are false.** Probe folds "query failed" into "agent never ran" (`2>/dev/null` then `[ -z "$row" ]`). Alerts/recoveries pair exactly (11/11, 8/8) → ~16–22 meaningless Telegram messages a day. Masked the real 2026-08-07 diary failure. | UNKNOWN (≥84 d) | Separate query-failure into an `unknown` state; never alert on it | 1.5 |
| **F03** | HIGH | 1 | `market/scribe/poll.sh:73` | YouTube API failure, quota exhaustion and "no new videos" are **indistinguishable**. No HTTP status check; `.items[]?` suppresses the error. The whole market chain can go blind for weeks and still deliver a brief. | NEVER-WORKED | Capture `%{http_code}`; treat non-200 as a hard error | 1 |
| **F04** | HIGH | 6 | `config/.env` + `lib/ai-do.sh` | **`WRITER_ENABLED` is decorative.** Its only two readers merely print it; the writers pass `asbestos-writer`/`ssg-writer` straight to `ai-do.sh`, which never consults it. Setting it `false` stops nothing. Same for `EDITOR_ENABLED`. | NEVER-WORKED | 3 lines in `ai-do.sh`'s existing `*-writer\|*-auditor` case, or delete the flags | 0.5 |
| **F05** | HIGH | 5 | crontab `0 0 * * *` | Daily cost-cap reset **missed 11 of the last 30 days** (machine asleep; cron does not catch up). A tripped cap would silently persist for a day or more. | NEVER-WORKED | Move to a launchd `StartCalendarInterval` plist (5 already exist) | 0.5 |
| **F06** | HIGH | 4 | 192 `sqlite3` call sites | **`busy_timeout=0` everywhere except `dashboard/server.js`.** `log_to_audit.sh` runs `set -e`, and 43 callers wrap it in `\|\| true` — a dropped audit row is invisible, and audit_log is the input to every detector. | NEVER-WORKED | `PRAGMA busy_timeout=5000` in the audit/token writers | 3 |
| **F07** | HIGH | 1,10 | `research-opportunity/` | `pains: 0` every week for ≥6 weeks while scanning hundreds of posts. `triage.sh` and `extract.sh` — the stages that produce pains — are **invoked by nothing**. Fleet's busiest actor (6,699 rows/60 d) and 1,824 open issue rows. | NEVER-WORKED | Kill the hourly poll, or wire `extract.sh` | 1 / 6 |
| **F08** | HIGH | 8 | `watchdog/expected_schedule.yaml` | **~20 of 32 agents have no failure detector at all** — incl. watchdog itself, issues-capture, keyword-registry, librarian, idea-agent, domain-hunter, dashboard, the 4 autocommit jobs, and both agents shipped today. | NEVER-WORKED | Add entries; start with `link-monitor` + `gsc-submit` | 0.25+ |
| **F09** | HIGH | 10 | `editor/score.sh` | Editor gate has written **no verdict since 2026-07-19** (21 days) while geo-v1 kept scoring to 07-31. Silently out of the live path; nothing detects it. | UNKNOWN | Investigate the `run-batch` call path | 1 |
| **F10** | MEDIUM | 5,10 | `domain-hunter/` | ~39 Haiku calls/day across **three separate per-domain loops**; 1,305 rows in `domain_candidates` with **zero downstream readers**. Fleet's largest capacity sink with no consumer. | NEVER-WORKED | Kill; or batch like `backlink-prospector/filter.sh:94` | 1 / 4 |
| **F11** | MEDIUM | 1 | `lib/check_kill_switches.sh:19-36` | Only exports switches when `.env` mtime changed — verified equal right now, so it currently exports **nothing**. Harmless only because all 11 callers `source .env` on the line above. Its stated mid-run refresh purpose never worked. | NEVER-WORKED | Always export, or delete the file | 0.25 |
| **F12** | MEDIUM | 1,3 | `lib/ai-think.sh` | Opus wrapper lacks usage-cap detection (75), parse-fail classification (65) and **any audit row on failure** — a missing `result` key dies with a Python traceback, invisible to issues-capture. Zero agents route there today. | NEVER-WORKED | Port ~40 lines from `ai-cheap.sh:56-100` | 1.5 |
| **F13** | MEDIUM | 6 | both `run-batch.sh` | **14 unattended `--dangerously-skip-permissions` invocations** reachable from the 09:00 drafter cron. The boundary is a prompt instruction, not a mechanism. | NEVER-WORKED | Add `AI_DO_TOOLS=…` — `ai-do.sh:73-78` already supports it | 2 |
| **F14** | MEDIUM | 1,3 | `internal-link/propose_backlinks.sh:144`, `apply_approval.sh:54` | `${VAR//\'/\'\'}` produces `O\'\'Brien` (verified on bash 3.2.57) — invalid SQLite escaping. Latent only because slugs are `[a-z0-9-]+`. `propose_backlinks.sh` is on the deploy path. | NEVER-WORKED | Use the `APOS="'"` idiom from the 8 files that have it | 0.25 |
| **F15** | MEDIUM | 1 | `asbestos/run-batch.sh:1552` | GEO gate: `geo_exit -eq 1` (scorer could not parse) → `geo_pass_through=true`. FAIL-path and CANNOT-CHECK-path converge. Documented as deliberate at `:1545`; audit-logged. | REGRESSION-by-design | Split D110: close the parse half, keep this half open | — |
| **F16** | MEDIUM | 1 | both `run-batch.sh` | **No `set` line at all** in 1,801 and 1,509 lines that write to a public site with permissions bypassed. Every unchecked command continues on failure. | NEVER-WORKED | Not a cheap fix — see Phase 11 NOT-WORTH-IT | — |
| **F17** | MEDIUM | 6 | `scripts/backup-rsync.sh` | Backups run nightly and are well-alarmed (last OK 2026-08-08 23:30, 8 sources, 231 MB), but **a restore has never been tested**. No restore script exists anywhere. | NEVER-WORKED | One manual restore + written procedure | 2 |
| **F18** | MEDIUM | 7 | `agent_issues` | 2,911 open / 16 ever resolved / 93 wontfix. 91% from two already-understood causes. The dashboard's one diagnostic panel is unreadable. (= open D114) | REGRESSION | Auto-resolve + per-source suppression | 3 |
| **F19** | MEDIUM | 9 | 6 doc sites | Doc drift: backlink-prospector says Mon 09:00 (really **Sun 08:00**); idea-agent says weekly (really **daily**); `ai-do.sh` header says 1200 s (code **1800**); `log_to_audit.sh` claims 90-day retention (**no pruning exists**); 4 `agent.yaml` files carry invalid `tier:` values; `content-loop/agent.yaml` says `enabled: true` for a dead agent. | mixed | Correct the 6 claims | 1 |
| **F20** | LOW | 7 | `lib/log_to_audit.sh:3` | Documented 90-day `audit_log` retention **does not exist** — the only `DELETE FROM` in the fleet targets `link_monitor_refs`. Oldest row is 87 days old. | NEVER-WORKED | Fix the comment, not the data | 0.1 |
| **F21** | LOW | 7 | `hive_mind`, `warroom_transcript`, `source_verifications`, `audit_link_failures`, `outbound_link_prospects` | Five **write-only** tables (0 readers). `hive_mind` is written today but read by nobody. `warroom_transcript` frozen at 2026-05-14. `memories`/`sessions`/`mission_tasks` have 0 rows; `mission_tasks` has **5 readers and 0 writers**. | NEVER-WORKED | Drop 3 dead tables; leave `source_verifications` as archive | 0.5 |
| **F22** | LOW | 6 | `scout/.claude/settings.json` | `Read(/Users/mmm2/agents/scout/**)` — **single** leading slash, which the project's own rule says never matches. The bob-kit export already fixed this; the live file did not. In `allow`, so it costs an auto-approval, not security. | NEVER-WORKED | Double the slash | 0.1 |
| **F23** | LOW | 9 | `DEFERRED.md` | **D027 appears twice**, verbatim, at lines 102 and 532. `D427` in the prescribed grep is a false positive (a UUID fragment at line 474). Gaps at 29–32, 34–38, 49, 59, 61–62, 92 are documented bookkeeping. | — | Delete one D027; ignore D427 | 0.1 |
| **F24** | LOW | 9 | `D110`, `D118` | Two OPEN items are actually closed: D110's "single-shot parse" half was fixed in `f466673` (`score.sh:100` now retries); D118's exact symptom (`composite_score=0.0 AND passed=1`) returns **0 rows**. | — | Close/split them | 0.25 |
| **F25** | LOW | 7 | `gsc_submission_queue` | **70 rows, 0 submitted, 70 never submitted** — unchanged. No GSC credentials exist on this machine. Correctly labelled by the agent; 70 published URLs have never been announced to Google. | NEVER-WORKED (by design) | Operator action, not code | — |
| **F26** | LOW | 10 | `content-loop/`, `config-synthesizer/` | Dead 74 days. `content-loop` invoked by nothing; `config-synthesizer`'s only caller is `content-loop`. Plus 12 stale `fix-runner/state/run_*` dirs with 34 generated scripts polluting every repo-wide grep. | — | Delete | 0.5 |

### 2. The three I would fix first

**1 — F01, SSG source verification (3 h).** It is the only CRITICAL. Every other
finding costs noise, capacity, or confidence; this one puts a fabricated,
authoritatively-cited number on a live page. The asbestos gate exists, is proven
(it fired 2026-08-08), and is genuinely fail-closed — this is a port, not a
build. Half the published output is currently unprotected against the exact
failure the other half is defended from.

**2 — F02, the watchdog's false alerts (1.5 h).** 254 false incidents and ~20
meaningless messages a day is not a cosmetic problem: it is the mechanism by
which the *next* real failure gets ignored. It already happened once — the
genuine 2026-08-07 diary failure arrived as one message among eight false ones
the same day. Every other detector in the fleet is worth less until this is
fixed, and it is the cheapest high-value change in the document.

**3 — F04 + F05 together (1 h).** Both are safety mechanisms that look present
and are not. `WRITER_ENABLED` is what an operator would reach for to stop the
writers in an emergency, and it does nothing. The cap reset silently skips 37% of
days, so the one enforcement path that *does* work can leave the fleet paused
with no explanation. An hour buys back the two controls you would want on the
worst day.

I would not start with F06 (busy_timeout) despite its HIGH rating, because I
could not prove any row has actually been lost — see §3.

### 3. What I could NOT verify, and what is needed

| # | Unverified | Why | What would settle it |
|---|---|---|---|
| U1 | **Root cause of the watchdog false alerts (F02)** | The defect (failure-path == no-rows-path) is confirmed from code and 254 artifacts. *Why* the query intermittently returns empty is not. WAL readers do not block, so `SQLITE_BUSY` is a weak explanation; empty `in_list` → `action IN ()` syntax error is a better one. | Run `watchdog.sh --dry-run` (documented as side-effect-free) in a loop with `2>/dev/null` removed from `:193`, logging `in_list` each iteration. **I did not run it — this audit executes no agent scripts.** |
| U2 | **Whether any `audit_log` row has ever been lost to `SQLITE_BUSY` (F06)** | Every call site is `2>&1` into `/dev/null` plus `\|\| true`. `grep -rli 'database is locked'` across all logs returns nothing, but that is exactly what a swallowed error looks like. | Remove `2>/dev/null` from one high-collision caller (e.g. `research-opportunity/poll.sh`) for a week, or add the pragma and compare :00-minute row counts before/after. |
| U3 | **Node/TCC grant path (Phase 3i)** | `com.aiteam.dashboard.plist` pins `/opt/homebrew/bin/node` → `Cellar/node/26.0.0`, which moves on a brew upgrade. The path the TCC grant was issued against is not readable: `TCC.db` is SIP-protected and this session has no Full Disk Access. | `sqlite3 '/Library/Application Support/com.apple.TCC/TCC.db'` under FDA, or the operator's recollection of when the grant was last re-issued. Note the staleness alert is **not armed** — the dashboard has no watchdog entry. |
| U4 | **Whether the operator reads tg-monitor digests, or has ever acted on a backlink prospect / domain candidate** | These determine KEEP vs KILL for three agents (Phase 10). Nothing on the machine records human consumption — no `outreach_status` transition, no read receipt, no purchase record. | Two questions to the operator. Their answers decide ~$12/30 d of notional capacity and 3 agent directories. |
| U5 | **Whether `run-batch.sh`'s writers still function under `--tools` (F13)** | The fix is proposed, not tested. `ai-do.sh:73-78` supports the pass-through and scout/majordomo prove it works, but the writer needs Write/Edit and I did not exercise it. | A single guide run with `AI_DO_TOOLS` set, compared against a control. Requires executing the pipeline. |
| U6 | **Whether the backups restore (F17)** | No restore has ever been performed or scripted. Writes are verified; reads are not. | Restore `store/aiteam-backup.db` to a scratch path, diff schema + row counts against live. |
| U7 | **Whether D115 is closed** | `majordomo/daily_brief.sh:54` still has the `MAX(ts) < strftime(...)` shape D115 names, but a `CAST(... AS INTEGER)` has been added, which addresses the affinity bug. | Operator judgement on whether the CAST discharges the item. |

### 4. Files modified

**One file was written by this audit: this report,
`~/brain/projects/aiteam/docs/AITEAM_Agent_Audit_2026-08-09.md`.**

No other file in `~/agents/`, `~/brain/`, `~/projects/*`, `~/store/`, the
crontab, or `~/Library/LaunchAgents/` was created, edited, deleted, moved, or
chmod'ed. No `git add`, `commit`, `push`, `checkout`, or `stash` was run in any
repo. All database access was `SELECT`/`PRAGMA` only. No agent script was
executed, and no model call was made — `ai-do.sh`, `ai-cheap.sh` and
`ai-think.sh` were read, never invoked.

Two scratch files were written under this session's scratchpad
(`/private/tmp/claude-501/.../scratchpad/`): a `crontab -l` dump used for the
`lib/cron.txt` diff, and a list of D-numbers. Neither is inside any audited tree.

`git status` in `~/agents` was **clean at audit start and clean at audit end**.

**Concurrent-session note.** Another CC session was active in
`~/projects/asbestoshq-site` during this audit: `WAVE2_LANEA_PROGRESS.md`
appeared at 11:32 and `src/data/guides/asbestos-drywall-guide.json` was modified
at 11:35, and an `auditor_verdicts` row was written at 11:27. That repo's
untracked count therefore went 1 → 3 between the start and end of this audit
**for reasons unrelated to it**. The Phase 9.1 table records the state as of
audit start. I made no write of any kind in that tree.

---

## PHASE 0 — INVENTORY

### 0.1 Derivation sources

| Source | Command | Result |
|---|---|---|
| Directories | `find ~/agents -maxdepth 2 -type d` | 31 agent-bearing dirs + `_template`, `skills`, `exports` |
| crontab | `crontab -l` | 30 active job lines, 1 commented-out |
| launchd | `ls ~/Library/LaunchAgents` + `launchctl list` | 5 `com.aiteam.*` plists, all loaded |
| audit_log | `SELECT actor_id … ts > now-60d` | 37 distinct actor_ids |
| Executables | `find -perm -u+x` | 148 non-venv `.sh`/`.py` |

`ts` in `audit_log` is INTEGER unix-epoch, not a datetime string. A query written as
`ts > datetime('now','-60 days')` returns **zero rows silently** — noted here because
that is the exact shape of failure this audit is looking for, and it bit the audit itself.

### 0.2 Fleet inventory

LAST RUN is from `audit_log` where the agent logs there, otherwise log-file mtime.
All timestamps local (PDT). "LAST COMMIT" = last commit touching that subtree.

| AGENT | PATH | INVOKED BY | LAST RUN | LAST COMMIT | ORPHAN? |
|---|---|---|---|---|---|
| tg-monitor reader | `tg-monitor/reader.py` | cron `*/5 * * * *` | 2026-08-09 11:05 (log mtime) | bea6c87 07-31 | no |
| tg-monitor analyzer | `tg-monitor/analyzer.py` | cron `0 7 * * *` | 2026-08-09 07:01 | bea6c87 07-31 | no |
| watchdog | `watchdog/watchdog.sh` | cron `*/15 * * * *` | 2026-08-09 11:00 (audit_log) | 22ccc86 07-31 | no |
| watchdog digest | `watchdog/digest.sh` | cron `0 6 * * *` | 2026-08-09 06:00 | 22ccc86 07-31 | no |
| research-opportunity poll | `research-opportunity/poll.sh` | cron `0 * * * *` | 2026-08-09 11:00 (audit_log) | 617ae52 06-21 | no |
| research-opportunity digest | `…/digest.sh` | cron `0 9 * * 1` | 2026-07-27 09:00 | 617ae52 06-21 | no |
| research-opportunity discover_youtube | `…/discover_youtube.sh` | cron `0 9 1 * *` | 2026-08-01 09:00 | 617ae52 06-21 | no |
| ship-to-site digest | `ship-to-site/lib/digest.sh` | cron `0 7 * * *` | 2026-08-09 07:00 (audit_log) | ca2adb2 08-09 | no |
| ship-to-site ship.sh | `ship-to-site/ship.sh` | **cron line COMMENTED OUT** since 2026-05-17 | — | ca2adb2 08-09 | **Class A** |
| ship-to-site preview_ping | `ship-to-site/preview_ping.sh` | cron `0 20 * * *` | log mtime — see Phase 2 | ca2adb2 08-09 | no |
| ship-to-site deploy_batch | `ship-to-site/deploy_batch.sh` | cron `0 23 * * *` | 2026-08-02 23:03 (log mtime) | ca2adb2 08-09 | no |
| assignment-drafter | `assignment-drafter/drafter.sh` | cron `0 9 * * *` | 2026-08-09 09:15 (audit_log) | 2abe15c 06-11 | no |
| idea-agent | `idea-agent/run_weekly.sh` | cron `10 7 * * *` | 2026-08-09 07:15 (audit_log) | 5776114 07-31 | no |
| notes-agent sort | `notes-agent/sort.sh` | cron `0 10 * * 0` | 2026-08-09 10:00 (audit_log) | 2011c1a 05-24 | no |
| notes-agent capture | `notes-agent/capture.sh` | telegram bot.js | 2026-05-24 22:30 (audit_log `bot`) | 2011c1a 05-24 | no |
| market scribe poll | `market/scribe/poll.sh` | cron `0 7 * * *` | 2026-08-09 07:00 (audit_log) | 1292132 06-11 | no |
| market scribe process_queue | `market/scribe/process_queue.sh` | cron `15 7 * * *` | 2026-08-09 07:15 | 1292132 06-11 | no |
| market curator | `market/curator/curate.sh` | cron `15 8 * * *` | 2026-08-09 08:18 (audit_log) | 1292132 06-11 | no |
| market briefer | `market/briefer/brief.sh` | cron `45 8 * * *` | 2026-08-09 08:45 (audit_log) | 1292132 06-11 | no |
| market analyst | `market/analyst/analyze.sh` | called by scribe chain | 2026-08-09 07:15 (audit_log) | 1292132 06-11 | no |
| domain-hunter | `domain-hunter/scan.sh` | cron `30 6 * * *` | 2026-08-09 06:38 (audit_log) | f8e3a84 05-29 | no |
| issues-capture capture | `issues-capture/capture.sh` | cron `*/5 * * * *` | 2026-08-09 11:05 (log mtime) | d33e4e6 05-30 | no |
| issues-capture notify | `issues-capture/notify_telegram.sh` | cron `*/5 * * * *` | 2026-08-09 11:05 | d33e4e6 05-30 | no |
| issues-capture digest | `issues-capture/digest.sh` | cron `5 6 * * *` | 2026-08-09 06:05 | d33e4e6 05-30 | no |
| keyword-registry | `keyword-registry/lib/backfill_registry.sh` | cron `0 3 * * *` | 2026-08-09 03:00 (audit_log) | 08d5663 05-17 | no |
| librarian | `librarian/run_librarian.sh` | cron `0 2 * * *` | 2026-08-09 02:00 (audit_log) | f8962cf 06-09 | no |
| backlink-prospector | `backlink-prospector/run.sh` | cron `0 8 * * 0` | 2026-08-09 08:17 (audit_log) | a0d8202 06-11 | no |
| gsc-submit digest | `gsc-submit/digest.sh` | cron `0 8 * * *` | 2026-08-09 08:00 (audit_log) | ca2adb2 08-09 | no |
| gsc-submit reconcile | `gsc-submit/reconcile.sh` | **manual only** | 2026-08-09 01:17 (log mtime) | ca2adb2 08-09 | Class C (by design) |
| link-monitor | `link-monitor/check.sh` | cron `0 6 * * 1` | 2026-08-09 01:41 (manual smoke) | cd2b2f1 08-09 | no |
| brain-autocommit | `scripts/brain-autocommit.sh` | cron `55 23 * * *` | 2026-08-08 23:55 | — | no |
| agents-autocommit | `scripts/agents-autocommit.sh` | cron `50 23 * * *` | 2026-08-08 23:50 | 41aa86b 07-31 | no |
| content-autocommit ×2 | `scripts/content-autocommit.sh` | cron `35/40 23 * * *` | 2026-08-08 23:40 | 41aa86b 07-31 | no |
| orchestrator write_diary | `orchestrator/write_diary.sh` | cron `45 23 * * *` | 2026-08-08 23:45 (audit_log) | 98f9107 07-31 | no |
| cost-cap reset | inline crontab `0 0 * * *` | cron | 2026-08-09 00:00 (audit_log `cron`) | — | no |
| backup-rsync | `scripts/backup-rsync.sh` | launchd `com.aiteam.backup` 23:30 | 2026-08-08 23:30 (audit_log), exit 0 | — | no |
| dashboard | `dashboard/server.js` | launchd `com.aiteam.dashboard`, RunAtLoad+KeepAlive | PID 80769 live, HTTP 200 on :3141 | f8962cf 06-09 | no |
| grammy bot | `telegram/bot.js` via `bot-launch.sh` | launchd `com.aiteam.grammy-bot`, RunAtLoad+KeepAlive | PID 88856, uptime 8d18h | 98f9107 07-31 | no |
| majordomo check | `majordomo/check.sh` | launchd `com.aiteam.majordomo-check` hourly | 2026-08-09 11:00 (log mtime), exit 0 | 7790257 08-06 | no |
| majordomo daily_brief | `majordomo/daily_brief.sh` | launchd `com.aiteam.jeff-brief` 07:30 | 2026-08-09 07:30 (audit_log) | 7790257 08-06 | no |
| majordomo run.sh | `majordomo/run.sh` | telegram bot.js (`majordomo_invoked`) | 2026-07-31 15:01 | 7790257 08-06 | Class C-adjacent |
| scout | `scout/run.sh` | majordomo / manual | 2026-07-31 18:02 (audit_log) | aa7b60a 07-31 | **Class C** |
| orchestrator run.sh | `orchestrator/run.sh` | telegram bot.js | 2026-08-06 21:08 (`bot` text_responded) | 98f9107 07-31 | no |
| auto-refresh evaluate | `auto-refresh/evaluate.sh` | **nothing** | 2026-08-09 01:45 (manual) | 0db99f1 08-09 | **Class C** (deliberate) |
| auto-refresh fetch_gsc | `auto-refresh/fetch_gsc.sh` | **nothing — stub** | never | 0db99f1 08-09 | **Class C** (deliberate) |
| internal-link propose_backlinks | `internal-link/propose_backlinks.sh` | `ship-to-site/deploy_batch.sh:140` | 2026-05-17 17:11 (audit_log) | ff63975 05-24 | see Phase 1 |
| internal-link audit_links | `internal-link/audit_links.sh` | run-batch pipeline | 2026-07-19 09:14 | ff63975 05-24 | no |
| internal-link build_inventory | `internal-link/build_inventory.sh` | **manual only** | 2026-05-24 19:22 | ff63975 05-24 | **Class C** |
| geo-optimizer | `geo-optimizer/score.sh` | run-batch pipeline | 2026-07-31 14:09 (audit_log `geo-v1`) | f466673 07-31 | no |
| editor | `editor/score.sh` | run-batch pipeline | 2026-07-19 23:00 (audit_log `editor-v1`) | dd8c867 05-27 | no |
| config-synthesizer | `config-synthesizer/synth.sh` | `content-loop/lib/run_one.sh` only | 2026-05-27 23:48 (log mtime) | f8962cf 06-09 | **Class C** (its only caller is itself orphaned) |
| content-loop | `content-loop/loop_driver.sh` | **nothing** | 2026-05-27 23:48 (log mtime) | f8962cf 06-09 | **Class C** |
| fix-runner | `fix-runner/run_fixes.sh` | **manual only, by design** | 2026-06-11 15:29 (audit_log) | c46faf1 06-11 | Class C (documented) |
| content-loop pick/run_one | `content-loop/lib/*.sh` | loop_driver only | 2026-05-27 | f8962cf 06-09 | **Class C** |
| research-opportunity triage/extract | `…/triage.sh`, `…/extract.sh` | **nothing** | see Phase 7 | 617ae52 06-21 | **Class C** |
| telegram transcribe | `telegram/transcribe.sh` | `bot.js:59` | on demand | 98f9107 07-31 | no |
| lib/* helpers | `lib/*.sh` | sourced by many | continuous | 0db99f1 08-09 | no |
| lib/backup_daily / backup_weekly / backup_db | `lib/backup_*.sh` | **nothing** (superseded by `scripts/backup-rsync.sh`) | — | — | **Class C** |
| lib/convert_dropzone, rebuild_command_center, write_to_dropzone | `lib/*.sh` | **nothing** | — | — | **Class C** |
| majordomo playbooks | `majordomo/playbooks/*.sh` | bot.js playbook flow | 2026-08-06 20:58 `playbook_proposed` | 7790257 08-06 | no |
| `majordomo/playbooks/deploy-now.sh` | same dir | **zero inbound references** | never | 7790257 08-06 | **Class C** |
| `scripts/install-git-hooks.sh` | | **zero inbound references** | never | 41aa86b 07-31 | Class C (one-shot installer) |
| `editor/run_tier_test.sh` | | manual test harness | — | dd8c867 05-27 | Class C |
| `market/curator/seed_taxonomy.sh` | | one-shot seed | — | 1292132 06-11 | Class C |
| `telegram/restart.sh` | | manual | — | 41aa86b 07-31 | no |
| `exports/bob-kit/**` | `exports/` | **export snapshot, not live** | — | — | dead copy of `lib/` |

### 0.3 Class flags

**Class A — scheduled but not executing (30d):**
- `ship-to-site/ship.sh` — cron line present but **commented out** since 2026-05-17
  (`crontab -l` line 12: `# */15 * * * * …ship.sh`). The disabling comment says
  "throttled to 23:00 daily", and `deploy_batch.sh` does run at 23:00, so the
  *function* survived; the *line* is dead. Not a silent failure, but the crontab
  carries a misleading disabled line.
- No other cron/launchd entry showed zero execution evidence in 30 days.
  `research-opportunity/digest.sh` last ran 2026-07-27 and is weekly-Monday —
  2026-08-03 and 2026-08-10 are the expected next fires; **2026-08-03 Monday is
  missing from the log.** Flagged for Phase 2.

**Class B — runs but nothing consumes the output:** deferred to Phase 10 (needs
consumer analysis). Preliminary: `domain-hunter`, `research-opportunity`,
`tg-monitor/analyzer`, `idea-agent`, `keyword-registry`.

**Class C — on disk, invoked by nothing:**
`content-loop/` (whole subtree), `config-synthesizer/` (only caller is content-loop),
`auto-refresh/` (deliberate — see its own commit message), `lib/backup_daily.sh`,
`lib/backup_weekly.sh`, `lib/backup_db.sh`, `lib/convert_dropzone.sh`,
`lib/rebuild_command_center.sh`, `lib/write_to_dropzone.sh`,
`majordomo/playbooks/deploy-now.sh`, `internal-link/build_inventory.sh`,
`research-opportunity/triage.sh`, `research-opportunity/extract.sh`,
`scout/run.sh` (last invoked 2026-07-31, 9 days), `exports/bob-kit/**`.

### 0.4 Immediate Phase-0 findings

- **lib/cron.txt lags the live crontab by one nightly cycle — by design.**
  `diff` shows 10 lines live and absent from the mirror (the `gsc-submit` and
  `link-monitor` blocks, added today). `scripts/agents-autocommit.sh:7`
  regenerates it at 23:50 nightly, so tonight it syncs. Not a defect; see
  Phase 2.3.
- **`exports/bob-kit/lib/*` is a byte-level fork of `lib/*`** (notify.sh,
  run_agent.sh, check_cost_caps.sh, log_to_audit.sh…). It is not on any
  execution path, but it is a second copy of security-relevant code that will
  drift. Recorded, not a live defect.


---

## PHASE 1 — ERROR-FOLDING SWEEP

Ranked by worst realistic consequence, not by count.

### 1.1 CRITICAL — SSG pipeline has no source-verification gate at all

**Evidence.** `~/projects/asbestos-contractors/content/run-batch.sh:1499-1533` runs
`python3 scripts/verify_sources.py` between the auditor and approval and fails
closed. The same stage does not exist in `~/projects/ssg-content/content/run-batch.sh`
— `grep -c verify_sources ssg-content/content/run-batch.sh` = 0, and
`ls ~/projects/ssg-content/scripts/` contains no `verify_sources.py`,
`source_fetch.py`, or `safety_checks.py`.

Both pipelines have identical writer prompts in structure and both write to a
public site. The asbestos file's own comment states the reason the gate exists:

```
# it exists because the writer fabricates figures and attaches real agency
# names to them, which every upstream check passes
```

That failure mode is not asbestos-specific. SSG guides ship on
`audit_guide.py` alone, which — per its own code — checks that a claim *carries*
a citation, never that the cited source *contains* the claim.

**Consequence:** fabricated figures with real-looking citations ship to
SmartSourceGuide with no mechanical check. SSG shipped 108 writer and 108
auditor calls in the last 30 days (`token_usage`).

**Classification: NEVER-WORKED.** `verify_sources.py` was added 2026-08-08
(asbestos only, per the `# ── SOURCE VERIFICATION GATE (2026-08-08) ──` marker).
It was never ported.

### 1.2 CRITICAL — `market/scribe/poll.sh` cannot distinguish "API failed" from "no new videos"

`market/scribe/poll.sh:73`:
```bash
VIDEOS_JSON=$(curl -sS "https://www.googleapis.com/youtube/v3/playlistItems?...&key=${YOUTUBE_API_KEY}" | strip_ctl)
```
then line 94:
```bash
done < <(echo "$VIDEOS_JSON" | jq -r '.items[]? | [...] | @tsv')
```

No `-w %{http_code}`, no `$?` check, and `.items[]?` explicitly suppresses the
error when `.items` is absent. A quota-exceeded 403, an expired key, or a
transport failure all produce an error JSON, yield zero rows, and the run
reports `COUNT=0` — byte-identical to a genuinely quiet day. The whole market
chain (`process_queue` → `curator` → `briefer`) then runs on nothing and the
briefer still delivers a brief. The `UPLOADS_PL` fetch one line above (line 66)
at least prints a WARN and `continue`s, but only to a log file with no reader.

**Consequence:** the market pipeline can silently go blind for weeks, and the
morning brief would keep arriving. **UNKNOWN** whether this has already
happened — no HTTP status is recorded anywhere to check.

### 1.3 HIGH — `${VAR//\'/\'\'}` SQL escaping is still live in two files and is broken

Verified on this machine (`/bin/bash` 3.2.57):
```
$ V="O'Brien"; echo "${V//\'/\'\'}"
O\'\'Brien
```
It emits a **backslash-escaped** doubled quote, which SQLite does not accept —
not the doubled quote SQL needs.

Live sites:
- `internal-link/propose_backlinks.sh:144` — `WHERE slug='${SOURCE_SLUG//\'/\'\'}'`
- `internal-link/apply_approval.sh:54` — `WHERE slug='${NEW_SLUG//\'/\'\'}'`

The correct `APOS="'"` idiom is present in 8 other files
(`gsc-submit/reconcile.sh:54`, `link-monitor/check.sh:39`,
`geo-optimizer/score.sh:149`, `ship-to-site/preview_ping.sh:63`,
`lib/log_to_audit.sh:37`, `lib/resolve_model.sh:21`,
`auto-refresh/evaluate.sh:62`, `editor/score.sh:110`). These two were missed.

**Consequence limited in practice:** slugs are `[a-z0-9-]+`, so no apostrophe
reaches these lines today. It is a latent injection/corruption path, not a live
break. `propose_backlinks.sh` IS on the live deploy path
(`ship-to-site/deploy_batch.sh:140`).
**Classification: NEVER-WORKED** (the substitution never produced valid SQL).

Separately, `research-opportunity/triage.sh:46` and
`research-opportunity/poll_youtube.sh:68` use `${VAR//\'/}` — *deleting*
apostrophes. That is safe against injection but silently corrupts data
containing one.

### 1.4 HIGH — `check_kill_switches.sh` is decorative

`lib/check_kill_switches.sh:19-36` only exports the seven kill-switch variables
when `.env`'s mtime differs from `/tmp/.aiteam_env_mtime`. Verified live:

```
$ cat /tmp/.aiteam_env_mtime      → 1786258800
$ stat -f %m ~/agents/config/.env → 1786258800
```

They are equal right now, so the file currently exports **nothing**. Every one
of its 11 callers happens to `source ~/agents/config/.env` on the line
immediately above (verified for `lib/run_agent.sh:7`,
`librarian/run_librarian.sh:7`, `geo-optimizer/score.sh:12`,
`editor/score.sh:12`, `orchestrator/write_diary.sh:7`,
`domain-hunter/scan.sh:21`, `backlink-prospector/search.sh:10`), so the values
are present anyway and nothing is broken today.

But the file's stated purpose — "re-read kill-switch values from .env if it
changed … from any long-running script before sensitive ops" — is not achieved:
every caller sources it once at the top and never again, so no mid-run refresh
ever happens. It is a no-op wrapper that reads as a safety mechanism.
**Classification: NEVER-WORKED as a refresh; harmless as it stands.**
The live hazard is a *future* caller that sources only this file and not `.env`:
it would get an empty `SYSTEM_PAUSED`, which `[ "$SYSTEM_PAUSED" = "true" ]`
reads as **not paused**. Failure and cannot-check produce the same result.

### 1.5 HIGH — `ai-think.sh` (Opus tier) is missing every hardening the other two wrappers have

Compare `lib/ai-think.sh:44` with `lib/ai-do.sh:141-190` and
`lib/ai-cheap.sh:56-100`:

| Guard | ai-do | ai-cheap | ai-think |
|---|---|---|---|
| SIGALRM 142 handler + audit row | ✓ | ✓ | ✓ |
| usage/rate-cap detection → exit 75 | ✓ | ✓ | **✗** |
| `claude_call_failed` audit row on nonzero | ✓ | ✓ | **✗** |
| rc=0-but-unusable payload → exit 65 | ✓ | ✓ | **✗** |
| MAX_THINKING_TOKENS=0 | ✓ | n/a | **✗** |

`ai-think.sh:44` is the whole non-timeout error path:
```bash
[ "$RC" -ne 0 ] && exit "$RC"   # preserve prior set -e behavior on other claude errors
```
and line 46 is:
```bash
python3 -c "import json; print(json.load(open('$TMPFILE'))['result'])"
```
A rc=0 response with no `result` key raises `KeyError`, the script dies with a
Python traceback, and **no audit row is written at all** — the failure is
invisible to `issues-capture` and to the watchdog.

**Live blast radius today is small**: `grep` shows no agent currently routes
through `ai-think.sh` (`majordomo/run.sh` and `orchestrator/run.sh` both
deliberately call `claude` directly to pass `--tools`; `market/analyst/analyze.sh:22`
defines `AI_THINK` but line 110 shows it is behind a non-default override).
`lib/run_agent.sh:23` still routes `tier: think` here, and
`orchestrator/agent.yaml` + `majordomo/agent.yaml` both declare `tier: think`.
**Classification: NEVER-WORKED** (the hardening was added to `ai-do`/`ai-cheap`
on 2026-06-09 and never back-ported).

### 1.6 MEDIUM — `idea-agent/google_search.sh` folds every Serper outcome into `{}`

`idea-agent/google_search.sh:99-103`:
```bash
RESPONSE=$(curl -sS -X POST "https://google.serper.dev/search" \
  ... --data "$Q_PAYLOAD" 2>/dev/null || echo '{}')
```
A 401 (bad key), a 429 (quota), and a DNS failure all become `{}` → zero
results → the search phase reports success with nothing found. The same file
folds three separate `ai-cheap.sh` outcomes to `'[]'` (lines 70, 84, 149, 163).
Given `idea-agent-search-filter` bills 6 calls/day every day, the agent is
running; whether it is *finding* anything is unobservable from the outside.

### 1.7 MEDIUM — `research-opportunity` produces `pains: 0` every week and nothing notices

`audit_log` `digest_sent` payloads, every Monday:
```
2026-08-03 09:00:01 {"pains":0,"posts_scanned":120,...}
2026-07-27 09:00:13 {"pains":0,"posts_scanned":9,...}
2026-07-20 09:00:13 {"pains":0,"posts_scanned":28,...}
2026-07-13 09:00:13 {"pains":0,"posts_scanned":164,...}
2026-07-06 09:00:13 {"pains":0,"posts_scanned":164,...}
2026-06-29 09:00:13 {"pains":0,"posts_scanned":36,...}
```
The agent polls hourly (6,699 audit rows in 60 days — the fleet's busiest
actor), scans hundreds of posts, and extracts zero pain points six weeks
running. `research-opportunity/triage.sh` and `extract.sh` — the two scripts
that would turn posts into pains — are invoked by **nothing**
(`grep -rn extract.sh` returns only self-references). The digest reports `pains:0`
truthfully; the pipeline that fills that number was never wired up.
**Classification: NEVER-WORKED.** See Phase 10.

### 1.8 MEDIUM — GEO gate treats a scorer hard error as a pass

`~/projects/asbestos-contractors/content/run-batch.sh:1552-1556`:
```bash
if [ "$geo_exit" -eq 1 ]; then
    ...
    geo_pass_through=true
```
`score.sh` exits 1 when the model's TSV output could not be parsed after two
attempts (`geo-optimizer/score.sh:119-131`). So "the GEO scorer could not
evaluate this page" and "the page passed GEO" produce the same downstream
result. This is documented as deliberate at line 1545 ("a scoring hiccup should
not block good content") and the failure IS audit-logged as `geo_error`, so it
is a knowing trade-off rather than an accident. Recording it because it is the
one remaining gate in the fleet where FAIL-path and CANNOT-CHECK-path converge.
`audit_log` shows `geo_parse_failed` last fired 2026-07-31 14:09.

### 1.9 MEDIUM — 106 shell scripts have no `set -euo pipefail`; the two pipelines have no `set` at all

`grep -n '^set ' ~/projects/asbestos-contractors/content/run-batch.sh` → **no
match**. Same for `ssg-content/content/run-batch.sh`. These are 1,801 and 1,509
lines respectively, they invoke the model with `--dangerously-skip-permissions`,
and they publish to a live site. Every unchecked command inside them continues
on failure.

Across `~/agents/`, 106 `.sh` files lack the full `set -euo pipefail`. Most have
`set -uo pipefail` (deliberate: `-e` is genuinely wrong for a script that must
audit-log its own failures). The ones with **no** `set` line at all and live
pipelines are:
`internal-link/audit_links.sh` (15 pipes), `gsc-submit/reconcile.sh` (16),
`gsc-submit/digest.sh` (8), `config-synthesizer/synth.sh` (24),
`majordomo/run.sh` (11), `majordomo/runner.sh` (13), `majordomo/daily_brief.sh` (8),
`scout/run.sh` (9), `orchestrator/run.sh` (11), `issues-capture/notify_telegram.sh` (5),
`telegram/smoke_test.sh` (9), `lib/agent_context.sh` (5),
`domain-hunter/deliver/telegram_digest.sh` (12), `gsc-submit/lib/canonical_url.sh` (2),
`content-loop/lib/*` (3 files), `majordomo/playbooks/{approve-slug,restart-grammy,restart-launchd-service}.sh`.

`bash -n` under bash 3.2 parses **all** of them cleanly (0 syntax errors across
the whole tree, verified) — this is a runtime-robustness finding, not a
syntax one.

### 1.10 LOW — 100 `VAR=$(… 2>/dev/null)` sites; the load-bearing ones

`grep -c` gives 100 command substitutions that discard stderr. Most are benign
`date`/`stat` fallbacks. The ones where a swallowed error changes a decision:

- `domain-hunter/scan.sh:53` and `:170` —
  `SPEND=$(sqlite3 … 2>/dev/null || echo "0")`. A broken DB read reports **zero
  spend**, so the agent's own cost gate passes. Fails open on the expensive side.
- `watchdog/watchdog.sh:391,393` — `check_cost_logging()` folds a sqlite error
  to `0` for both `evidence` and `tokens`. `evidence=0` takes the `else`
  branch → "ok". A broken DB makes the cost-logging watchdog report healthy.
- `assignment-drafter/drafter.sh:262` —
  `pending="$(… parse_queue.sh … | grep -c . || true)"`. A crashed
  `parse_queue.sh` yields `0`, which is also the value for a genuinely empty
  queue; both fire the "queue low" operator notification, so the operator sees
  a plausible message either way.

### 1.11 Things that are correct — recorded so they stop being re-audited

- **`verify_sources.py` (asbestos) is genuinely fail-closed.** Every path
  inspected returns FAIL on error: `:426` `except FetchError`, `:530` adjudication
  disabled → False, `:558-561` timeout/OSError → False, `:568` unparseable
  verdict → False, `:574-577` YES-with-no-verbatim-quote → False, `:587` draft
  unreadable → False, `:767` bare `except Exception` → fail closed. Confirmed
  live: `audit_log` `run-batch|srcver_fail` 2026-08-08 23:47:41.
- **`geo-optimizer/score.sh`** exits 1 (not a default score) when the model's
  output cannot be parsed, after one retry — `:119-131`.
- **`ai-do.sh` / `ai-cheap.sh`** distinguish four outcomes with distinct exit
  codes and distinct audit rows: 142 hang, 75 usage cap, 65 rc=0-but-unusable,
  otherwise claude's own rc.
- **`lib/notify.sh`** distinguishes three HTTP outcomes that were once conflated
  (`:120-135`): real non-200 (no retry), `000` transport (backoff+retry),
  empty (curl never ran → `curl_did_not_run_rc<n>`, fail fast).


---

## PHASE 2 — SCHEDULING, LIVENESS, SILENT DEATH

### 2.1 crontab target existence

All 29 distinct cron target paths exist and are executable. Verified by iterating
`crontab -l` and testing `[ -x "$t" ]` on each:
`29 EXEC, 0 EXISTS-NOT-EXEC, 0 MISSING.`

Schedule-vs-doc drift is in Phase 9.

### 2.2 PATH under cron

The crontab PATH header is present at line 1:
```
PATH=/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

Every external binary the scheduled scripts invoke resolves under exactly that
PATH (tested with `PATH=<cron path> command -v`):

| binary | resolves to |
|---|---|
| jq | /usr/bin/jq |
| sqlite3 | /usr/bin/sqlite3 |
| python3 | /opt/homebrew/bin/python3 |
| node | /opt/homebrew/bin/node |
| curl | /usr/bin/curl |
| shlock | /usr/bin/shlock |
| rsync | /usr/bin/rsync |
| git | /opt/homebrew/bin/git |
| perl | /usr/bin/perl |
| awk, sed, uuidgen, bc | /usr/bin/… |
| diskutil | /usr/sbin/diskutil |
| yt-dlp | /opt/homebrew/bin/yt-dlp |
| pdftotext | /opt/homebrew/bin/pdftotext |
| claude | /opt/homebrew/bin/claude |

**PASS.** Four of these (`python3`, `node`, `git`, `yt-dlp`, `pdftotext`,
`claude`) live under `/opt/homebrew` and therefore move on a brew upgrade —
relevant to the TCC finding in Phase 3i, not to PATH.

### 2.3 `lib/cron.txt` vs live crontab

```
$ diff <(crontab -l) ~/agents/lib/cron.txt
66,75d65
<  # GSC submission queue — daily digest …
<  0 8 * * * /Users/mmm2/agents/gsc-submit/digest.sh …
<  # Dead-link monitor — weekly sweep …
<  0 6 * * 1 /Users/mmm2/agents/link-monitor/check.sh …
```
**10 lines live, absent from the mirror. 0 lines in the mirror, absent from
live.** The mirror lags by exactly the two agents added on 2026-08-09
(`gsc-submit`, `link-monitor`).

**This is expected behavior, not a defect.** `scripts/agents-autocommit.sh:7`
regenerates the mirror every night at 23:50:
```bash
crontab -l > ~/agents/lib/cron.txt
```
with a comment at `:4-6` saying it was added precisely because the
hand-maintained mirror had drifted (audit F6). The two missing blocks were added
to the live crontab **today**; tonight's 23:50 run will sync them.

**Not a finding.** The only residual is that the mirror is by construction up to
24 h stale, so it must never be used as ground truth for "what is scheduled" —
which is why this audit read `crontab -l` directly.

### 2.4 launchd

| Label | Program | RunAtLoad | KeepAlive | Schedule | Loaded | Last exit | PID |
|---|---|---|---|---|---|---|---|
| com.aiteam.backup | /bin/bash scripts/backup-rsync.sh | false | false | Cal 23:30 | yes | **0** | — |
| com.aiteam.dashboard | /opt/homebrew/bin/node dashboard/server.js | true | `{SuccessfulExit:false}` | daemon | yes | **15** | 80769 |
| com.aiteam.grammy-bot | telegram/bot-launch.sh | true | `{SuccessfulExit:false}` | daemon | yes | **0** | 88856 |
| com.aiteam.jeff-brief | /bin/bash majordomo/daily_brief.sh | false | false | Cal 07:30 | yes | **0** | — |
| com.aiteam.majordomo-check | /bin/bash majordomo/check.sh | false | false | Cal min=0 (hourly) | yes | **0** | — |

- **No plist on disk is unloaded.** All five appear in `launchctl list`.
- `com.aiteam.dashboard` shows `LastExitStatus = 15` (SIGTERM) *and* a live PID.
  That is a prior restart, not a current failure — the process is up and
  answering (see 2.7). Not a finding.
- Three jobs correctly name `/bin/bash` explicitly rather than a Homebrew path;
  the dashboard is the one that pins `/opt/homebrew/bin/node` — see Phase 3i.

### 2.5 Agents that have historically lost their cron entries

Confirmed present in the live crontab, by name:
- **librarian** — `0 2 * * * /Users/mmm2/agents/librarian/run_librarian.sh`.
  Live: `audit_log actor_id='librarian'` 74 rows, last **2026-08-09 02:00:09**. ✅
- **backlink-prospector** — `0 8 * * 0 /Users/mmm2/agents/backlink-prospector/run.sh asbestos`.
  Live: 36 rows, last **2026-08-09 08:17:02** (today is Sunday). ✅

Both are scheduled and both ran today. No regression.

### 2.6 Log files, sizes, rotation

**Nothing in the fleet rotates any log.** `grep -rn 'logrotate\|newsyslog\|truncate\|> *\$LOG'`
finds no rotation mechanism, and there is no `/etc/newsyslog.d` entry for these
paths.

Logs over 1 MB (all append-only, all unbounded):

| Log | Size | mtime | Growth driver |
|---|---|---|---|
| `tg-monitor/reader.log` | **17.9 MB** | 2026-08-09 11:05 | cron every 5 min |
| `assignment-drafter/state/pipeline_runs.log` | **11.3 MB** | 2026-08-09 09:15 | full run-batch stdout per fire |
| `watchdog/watchdog.log` | **9.5 MB** | 2026-08-09 11:00 | cron every 15 min |
| `issues-capture/log/capture.log` | 1.3 MB | 2026-08-09 11:05 | cron every 5 min |
| `issues-capture/log/notify_telegram.log` | 1.0 MB | 2026-08-09 11:05 | cron every 5 min |

**None exceeds 50 MB today.** `reader.log` is the fastest grower; at its current
rate it reaches 50 MB in roughly six months. Recorded as a known trajectory,
not a current defect.

Two logs are notable for being *frozen*:
- `ship-to-site/preview_ping.log` — **0 bytes, mtime 2026-05-17 20:00**. The job
  IS running: `audit_log` has `ship-to-site|preview_ping_sent` daily through
  2026-08-08 20:00:10. The script simply emits nothing to stdout unless a TTY is
  attached (`preview_ping.sh:22` `[ -t 1 ] && echo …`). Log mtime is a
  **misleading liveness signal** for this agent — worth knowing before anyone
  uses mtime to judge it dead.
- `ship-to-site/deploy_batch.log` — mtime 2026-08-02 23:03 while `audit_log`
  shows `deploy_batch_empty` at 2026-08-08 23:00:09. Same cause.

### 2.7 Long-running processes

- **grammy Telegram bot: UP.** `launchctl list com.aiteam.grammy-bot` → PID
  88856, LastExitStatus 0. `ps -o etime -p 88856` → **8 days 18h 23m** uptime.
  `KeepAlive = {SuccessfulExit: false}` — respawns on crash, does not respawn on
  a clean exit. That is the correct setting for a bot that can be deliberately
  stopped. ✅
- **dashboard: UP.** PID 80769, `curl -s -o /dev/null -w %{http_code}
  http://127.0.0.1:3141/` → **200**. Same KeepAlive policy. ✅

### 2.8 Class A re-check — scheduled but not executing

Every cron and launchd job showed execution evidence within its own period.
One exception was chased and cleared:

`research-opportunity/digest.sh` (`0 9 * * 1`, weekly Monday) — its log's last
entry is 2026-07-27, which looked like a missed 2026-08-03 fire. `audit_log`
disproves it: `research-opportunity|digest_sent` at **2026-08-03 09:00:01** with
`"delivery":"telegram_bot"`. The log only receives a line on *failure*
(`digest.sh:192` writes a WARN only in the notify-failure branch), so the log
going quiet meant delivery started working, not that the job stopped.
The prior six Mondays all read `"delivery":"outbox_fallback"` — see 2.9.

### 2.9 The notify blackout has a second, later episode

`audit_log` `actor_id='notify'`, by day:

```
2026-07-23  notify_failed 18
2026-07-24  notify_failed 35
2026-07-25  notify_failed 28 | notify_sent  1
2026-07-26  notify_failed 28 | notify_sent  1
2026-07-27  notify_failed 23 | notify_sent  1
2026-07-28  notify_failed 24 | notify_sent  1
2026-07-29  notify_failed 24 | notify_sent  1
2026-07-30  notify_failed 28 | notify_sent  1
2026-07-31  notify_failed 20 | notify_sent 20   <- fixed mid-day
2026-08-01 … 2026-08-09  notify_sent only, 0 failures
```

So there was a **second delivery outage from 2026-07-23 to 2026-07-31**,
distinct from the earlier five-week silent drop. It was recorded (the
`notify_failed` rows exist — the audit trail worked this time) and it is
**resolved**: nine consecutive clean days. Not a current defect; recorded so the
"155 notifications over five weeks" incident is not confused with this one.


---

## PHASE 3 — PORTABILITY AND KNOWN-BUG REGRESSION CHECKS

Machine bash: `GNU bash, version 3.2.57(1)-release (arm64-apple-darwin25)` at
`/bin/bash`. Every claim below was tested against that binary, not reasoned about.

### (a) notify.sh empty-array expansion — **STILL FIXED** ✅

`lib/notify.sh:110`, the actual current line:
```bash
      "${PM_ARGS[@]+"${PM_ARGS[@]}"}" 2>/dev/null); rc=$?
```
The bash-3.2-safe `+alternate` form is present. `PM_ARGS` is declared empty at
`:64` and populated only for non-plain parse modes at `:65`, so this line is
exercised on every plain-mode send — which is most of them.

Confirmed the underlying hazard is real on this machine:
```
$ /bin/bash -c 'set -u; A=(); for x in "${A[@]}"; do :; done'
/bin/bash: A[@]: unbound variable
```

### (b) Other unsafe array expansions under `set -u` — **NONE FOUND** ✅

Ten `"${ARR[@]}"` expansions exist across `~/agents/` and the two content repos.
Each was checked for (i) `set -u` in effect and (ii) whether the array can be
empty at that point. All ten are safe:

| Site | set -u? | Guard |
|---|---|---|
| `notes-agent/sort.sh:47,138` | yes | `:37` `if [ "${#INBOX_FILES[@]}" -eq 0 ]; then … exit 0; fi` |
| `orchestrator/commands/weekly_review.sh:32` | yes | `:24` `N=${#DIARY_FILES[@]}; if [ "$N" -lt 2 ]; then … exit 0` |
| `editor/run_tier_test.sh:96` | yes | `:36` `if [ "${#ARTICLES[@]}" -lt 5 ]; then … exit 1` |
| `assignment-drafter/lib/validate.sh:77` | yes | `EXPECTED_LABELS` is a static literal, never empty |
| `idea-agent/run_weekly.sh:58` | yes | `PHASES` is a static literal |
| `librarian/run_librarian.sh:34` | yes | `:20` `files=("$INBOX"/*.md)` — glob, nullglob off ⇒ ≥1 element |
| `market/scribe/process_queue.sh:87` | yes | `:82` same glob idiom ⇒ ≥1 element |
| `domain-hunter/scan.sh:116` | **no `set`** | `DIGEST_ARGS` always gets `--top $TOP` at `:113` |
| `lib/ai-do.sh:136` | yes | uses the `+alternate` form already |
| `run-batch.sh` (both) `${PAGES[@]}` | **no `set` at all** | `set -u` not in effect ⇒ cannot trip |

Verified separately that `${#A[@]}` on an empty array does **not** trip `set -u`
in bash 3.2, so every one of those guards actually works:
```
$ /bin/bash -c 'set -u; A=(); if [ "${#A[@]}" -eq 0 ]; then echo GUARD_OK; fi'
GUARD_OK
```

### (c) bash-4-only syntax — **NONE FOUND** ✅

`grep -rnE '\$\{[A-Za-z_]+,,|\$\{[A-Za-z_]+\^\^|declare -A|mapfile|readarray|wait -n'`
across `~/agents/` and both content repos returns **four hits, all of them
comments saying the construct is deliberately avoided**:
- `fix-runner/run_fixes.sh:14` — "bash 3.2 compatible: no associative arrays, no ${var,,}"
- `majordomo/check.sh:17` — "no associative arrays, no ${var^^}, no mapfile"
- `scout/run.sh:54` — "tr/sed only — no bash-4 ${var,,}"
- `scripts/ctx.sh:8` — "Bash 3.2 compatible (macOS default)"

Independent confirmation: `/bin/bash -n` on all 148 shell files in `~/agents/`
plus both `run-batch.sh` files → **zero syntax errors**. A bash-4 construct
would fail to parse under 3.2.

### (d) GNU-vs-BSD flag assumptions — **CLEAN** ✅

- `sed -i` without a backup argument: **zero hits** across `~/agents/` and both
  content repos. Every occurrence is the BSD-correct `sed -i ''` form (e.g.
  `lib/check_cost_caps.sh:111`, and the `0 0 * * *` crontab line).
- `date -d`: 4 hits, all as the *fallback* half of a BSD-first pair, e.g.
  `orchestrator/write_diary.sh:26`
  `DAY_START=$(date -j -f '%Y-%m-%d %H:%M:%S' "$TODAY 00:00:00" '+%s' 2>/dev/null || date -d "$TODAY 00:00:00" '+%s')`
- `stat -c`: 7 hits, all as the fallback half of `stat -f … || stat -c …`, e.g.
  `lib/check_kill_switches.sh:19`, `scripts/auto-harvest-to-library.sh:48`.
- `readlink -f`: **zero hits.**
- `grep -P`: **zero hits.**
- `xargs -r`: **zero hits.**

### (e) SQL apostrophe escaping — **2 BROKEN SITES REMAIN**

Detailed in Phase 1.3. Current lines:
```
internal-link/propose_backlinks.sh:144:  EXISTS=$(sqlite3 "$DB" "SELECT COUNT(*) FROM internal_link_inventory WHERE slug='${SOURCE_SLUG//\'/\'\'}' AND site='$SITE';")
internal-link/apply_approval.sh:54:NEW_TITLE=$(sqlite3 "$DB" "SELECT COALESCE(NULLIF(h1,''), slug) FROM internal_link_inventory WHERE slug='${NEW_SLUG//\'/\'\'}';")
```
Correct `APOS="'"` form present in 8 files (listed in Phase 1.3). A third,
also-correct idiom `sed "s/'/''/g"` is used in 12 more files.

### (f) ai-do.sh thinking budget and hang reporting — **BOTH CORRECT** ✅

`lib/ai-do.sh:126`, actual line:
```bash
MAX_THINKING_TOKENS="${MAX_THINKING_TOKENS:-0}"
```
Defaults to 0, overridable by an exporting caller. ✅

`lib/ai-do.sh:141-148`, the SIGALRM path:
```bash
if [ "$RC" -eq 142 ]; then
  PAYLOAD=$(python3 -c '… "error":"claude -p exceeded "+sys.argv[1]+"s wall-clock timeout (SIGALRM)" …')
  bash ~/agents/lib/log_to_audit.sh agent "${AI_CALLER:-unknown}" timeout_kill ai-do "$PAYLOAD" …
  echo "ERROR: ai-do.sh claude HANG: exceeded ${AI_TIMEOUT_SEC}s wall-clock, killed by SIGALRM. This is a generation/thinking hang, NOT a usage cap." >&2
  exit 142
fi
```
Distinct exit code (142), distinct audit action (`timeout_kill`), and the
stderr string explicitly negates the cap reading. The usage-cap path is a
*separate* branch at `:151` returning **75**. Never conflated. ✅

One doc-vs-code drift inside the file itself: the header comment says
"AI_TIMEOUT_SEC default 1200s (20 min)" but `:132` sets
`AI_TIMEOUT_SEC="${AI_TIMEOUT_SEC:-1800}"`. The code is 1800; the comment is
stale. (`ai-cheap.sh` and `ai-think.sh` genuinely are 1200.)

### (g) Writer/auditor serialization — **COMPLETE, BOTH PIPELINES** ✅

The lock is acquired in exactly one place, `lib/ai-do.sh:103-117`, gated on the
agent_id:
```bash
case "$AGENT_ID" in
  *-writer|*-auditor)
    WRITER_LOCK="${WRITER_LOCK_FILE:-$HOME/store/flags/writer.lock}"
    …
    until /usr/bin/shlock -p $$ -f "$WRITER_LOCK"; do …
```

Every writer/auditor call site in both pipelines, with the agent_id it passes:

| File:line | agent_id | matches `*-writer\|*-auditor`? | takes lock |
|---|---|---|---|
| asbestos run-batch:505, 539, 569, 796, 822 | `asbestos-writer` | yes | ✅ |
| asbestos run-batch:1090, 1124, 1275 | `asbestos-auditor` | yes | ✅ |
| ssg run-batch:472, 507, 540, 698, 724 | `ssg-writer` | yes | ✅ |
| ssg run-batch:967, 1002, 1101 | `ssg-auditor` | yes | ✅ |

**14 of 14 call sites take the lock.** No unlocked path exists. Confirmed there
is no second writer entry point: `grep -rn 'ai-think.sh\|ai-cheap.sh'` in both
`run-batch.sh` files returns nothing — writers and auditors go through `ai-do.sh`
exclusively.

Release: `lib/ai-do.sh:88` single EXIT trap
`trap 'rm -f "$TMPFILE"; [ -n "$HELD_LOCK" ] && rm -f "$HELD_LOCK"' EXIT`.
On SIGKILL the trap never fires, but `shlock` is PID-aware and the next caller
reclaims a lock whose holder PID is gone — verified by the presence of
`run-batch|lock_stale` in `audit_log` (2026-05-17 11:53:16), i.e. the reclaim
path has actually executed. Fail-closed on wait exceeded (`:108-112`, exit 1
with a `writer_lock_timeout` audit row).

`~/store/flags/` currently contains only `last_backup_ok` — **no stale lock is
held right now.**

### (h) Drafter rc=1 skip path — **STILL FIXED** ✅

`assignment-drafter/drafter.sh:361-395`. The rc=1 branch marks ✓ **and** routes
through `maybe_fire_pipeline`:
```bash
      if [ "${DRY_RUN}" -ne 1 ] && [ -f "${output_dir}/assignment-batch-${slug}.md" ]; then
        bash "${LIB}/mark_complete.sh" "${queue_file}" "${slug}" "✓" >/dev/null 2>&1 || true
        if [ -f "${output_dir}/approved/guides/${slug}.json" ]; then
          echo "[drafter] ${slug}: brief + approved JSON already exist — ✓ without fire"
        else
          fire_out="$(maybe_fire_pipeline "${cfg}" "${slug}")"
```
Both guards described in the fix comment are present: DRY_RUN never mutates, and
an already-approved JSON is ✓'d without re-firing. ✅

### (i) Node TCC version pinning — **EXPOSED, and the detector does not cover it**

Current state:
```
$ readlink /opt/homebrew/bin/node   → ../Cellar/node/26.0.0/bin/node
$ node --version                    → v26.0.0
```
`com.aiteam.dashboard.plist` hard-codes `/opt/homebrew/bin/node`. That is the
symlink, so a brew upgrade to node 27 silently repoints it at a **different
binary**, and any TCC grant issued against the old binary is revoked.

The other four plists deliberately avoid this by naming `/bin/bash` — the
backup, jeff-brief and majordomo-check plists all carry an explicit comment
saying so. The dashboard plist has no such note and no such protection.

**The path the grant was issued against is NOT discoverable from this machine**
— TCC's database is SIP-protected and unreadable without Full Disk Access for
this session. **UNVERIFIED — needs `sqlite3 /Library/Application Support/com.apple.TCC/TCC.db`
under FDA, or the operator's memory of when the grant was last re-issued.**

**The staleness alert is NOT armed for this.** `watchdog/expected_schedule.yaml`
contains 12 agent entries; none of them is the dashboard, and there is no
`signal_type` that probes a binary path or a TCC grant. The nearest thing is the
`grammy-bot` entry using `signal_type: launchctl_label` — the dashboard has no
equivalent entry. If the dashboard died on a brew upgrade, **nothing would say so**;
it would simply stop answering on :3141 and the only detector is a human
opening the page.


---

## PHASE 4 — CONCURRENCY, LOCKS, IDEMPOTENCY

### 4.1 Lock inventory

Every lock in the fleet uses `/usr/bin/shlock` (BSD, PID-aware, atomic create
plus automatic reclaim when the recorded PID is gone). No `flock`, no `mkdir`
locks, no lockfile written by hand.

| Lock file | Acquired at | Released at | Trap covers | Stale-lock recovery |
|---|---|---|---|---|
| `assignment-drafter/state/assignment-drafter.lock` | `drafter.sh:37` | `:44` | `EXIT INT TERM` | shlock PID reclaim |
| `fix-runner/state/fix-runner.lock` | `run_fixes.sh:75` | `:104` (`cleanup`) | `EXIT INT TERM` | shlock PID reclaim |
| `idea-agent/…/idea-agent.lock` | `run_weekly.sh:38` | `:45` | `EXIT INT TERM` | shlock PID reclaim |
| `…/curator.lock` | `market/curator/curate.sh:25` | `:32` | `EXIT INT TERM` | shlock PID reclaim |
| `…/scribe.lock` | `market/scribe/process_queue.sh:36` | `:43` | `EXIT INT TERM` | shlock PID reclaim |
| `…/briefer.lock` | `market/briefer/brief.sh:35` | `:42` | `EXIT INT TERM` | shlock PID reclaim |
| `~/store/flags/writer.lock` | `lib/ai-do.sh:107` | `:88` EXIT trap | `EXIT` only | shlock PID reclaim |

**Release on abnormal exit is guaranteed for SIGINT/SIGTERM everywhere except
`writer.lock`, which traps `EXIT` only.** In practice that is still safe: `EXIT`
fires for SIGINT/SIGTERM under bash unless the trap is overridden, and SIGKILL
defeats every trap regardless. The real backstop is shlock's PID reclaim, and
that path has demonstrably executed — `audit_log` contains
`run-batch|lock_stale` at 2026-05-17 11:53:16.

**Nothing wedges silently on reboot.** shlock re-checks the recorded PID on each
attempt; after a reboot no PID matches, so the next caller steals the lock.
`~/store/flags/` right now contains only `last_backup_ok` — no lock is held.

### 4.2 Overlapping schedules touching the same resource

`audit_log` rows per clock-minute over the last 7 days show where jobs collide:

| Minute | Distinct actors | Rows | Colliding jobs |
|---|---|---|---|
| 07:00 | 5 | 58 | tg-monitor analyzer, ship-to-site digest, market scribe poll, research-opportunity poll (hourly), majordomo-check (hourly) |
| 07:15 | 5 | 63 | scribe process_queue, market analyst |
| 08:00 | 5 | 43 | gsc-submit digest, backlink-prospector (Sun), research poll, majordomo-check |
| 23:00 | 4 | **454** | ship-to-site deploy_batch, research poll, majordomo-check, watchdog |
| 20:00 | 4 | **466** | ship-to-site preview_ping, research poll, majordomo-check, watchdog |

Every one of these writes `~/store/aiteam.db`. The pairs that touch the *same
table* concurrently:

1. **`research-opportunity/poll.sh` (`0 * * * *`) vs everything else on the
   hour** — both write `audit_log`. This is the highest-frequency collision in
   the fleet; research-opportunity is the busiest actor (6,699 rows/60d).
2. **`issues-capture/capture.sh` and `issues-capture/notify_telegram.sh`, both
   `*/5 * * * *`, same minute** — both write `agent_issues`. `capture.sh`
   inserts, `notify_telegram.sh` updates `status`. There is **no lock between
   them.**
3. **`backup-rsync.sh` (launchd 23:30) vs `agents-autocommit.sh` (23:50) and
   `deploy_batch.sh` (23:00)** — backup takes a `sqlite3 .backup` snapshot
   (`backup-rsync.sh:54`), which is itself lock-correct, and it deliberately
   excludes the live `aiteam.db`/`-wal`/`-shm` from the rsync
   (`backup-rsync.sh:9,60`). Correctly handled.
4. **`watchdog.sh` (`*/15`) vs `issues-capture/capture.sh` (`*/5`)** — collide
   every 15 min but watchdog only *reads* `audit_log`. Benign.

### 4.3 SQLite concurrency — **192 writers, zero busy timeouts** (HIGH)

```
$ sqlite3 ~/store/aiteam.db "PRAGMA journal_mode;"  → wal
$ sqlite3 ~/store/aiteam.db "PRAGMA busy_timeout;"  → 0
```

WAL is on ✅. But WAL still permits only **one writer at a time**, and the
`sqlite3` CLI's default `busy_timeout` is **0** — it returns `SQLITE_BUSY`
immediately rather than waiting.

`grep -rn 'sqlite3 ' --include='*.sh'` counts **192 CLI invocations** across
`~/agents/`. `grep -rn 'busy_timeout'` finds it set in exactly **one** place:
`dashboard/server.js:35-36` (`PRAGMA busy_timeout = 5000` on both handles).
Python callers get Python's own 5-second default from `sqlite3.connect()`
(12 sites, all in `domain-hunter/` and `tg-monitor/`), so they are fine.

The exposed hot path is `lib/log_to_audit.sh` — every agent's every audit row
goes through it, it runs `set -euo pipefail`, and on `SQLITE_BUSY` it exits
nonzero. Nearly every caller invokes it as:
```bash
bash "$AUDIT" agent <id> <action> <target> "$PAYLOAD" >/dev/null 2>&1 || true
```
(43 such call sites counted in Phase 1's `|| true` sweep). So a lost audit row
produces **no error, no log line, and no trace**.

**Consequence:** a dropped heartbeat row makes `watchdog.sh` believe an agent
missed its run and fire a false alert; a dropped failure row makes
`issues-capture` miss a real failure. Both directions are wrong and neither is
detectable after the fact.

**Evidence of it actually happening: NOT FOUND, and not findable.**
`grep -rli 'database is locked'` across every `.log` in `~/agents/` returns
nothing — but that is expected, because `2>&1` sends the message to
`/dev/null` at every call site. **UNVERIFIED whether rows have been lost.**
To verify: temporarily run `log_to_audit.sh` without the `2>/dev/null` at one
high-collision caller (e.g. `research-opportunity/poll.sh`) for a week, or add
`PRAGMA busy_timeout` and compare row counts at :00 minutes before/after.

### 4.4 Idempotency — what happens on a double run

| Agent | Safe to re-run? | Why |
|---|---|---|
| assignment-drafter | ✅ | shlock at `:37`; second tick logs `lock_contention` and exits |
| idea-agent | ✅ | shlock at `:38`; comment at `:32` says it exists to stop double-insert into `idea_proposals` |
| market curator / scribe / briefer | ✅ | shlock each; briefer's comment at `:29` names double-send as the risk |
| fix-runner | ✅ | shlock at `:75` |
| writer/auditor (`ai-do.sh`) | ✅ | `writer.lock` serializes all 14 call sites |
| market scribe **poll** | ✅ | `INSERT OR IGNORE` on `seen_videos` + `SELECT changes()` (`poll.sh:81`) — dedup is in the DB, not the lock |
| **`issues-capture/capture.sh`** | ⚠️ **no lock** | `*/5` cron. Dedup relies on `audit_log_id` uniqueness; if two ticks overlap on a slow DB, duplicate `agent_issues` rows are possible. 3,020 rows currently. |
| **`issues-capture/notify_telegram.sh`** | ⚠️ **no lock** | `*/5` cron, same minute as `capture.sh`. Reads unnotified issues and sends Telegram. Two overlapping ticks could **double-send an alert**. |
| **`watchdog/watchdog.sh`** | ⚠️ **no lock** | `*/15`. Dedup is a once-per-day date-stamp file (`cost_log_alerted.date`, `backup_stale_alerted.date`, `notify_health_alerted.date`) — read-then-write with no atomicity. Two overlapping runs could both read the old date and both alert. In practice a 15-min period vs a sub-minute runtime makes this near-impossible. |
| **`link-monitor/check.sh`** | ⚠️ **no lock** | weekly, low risk |
| **`gsc-submit/digest.sh`** | ⚠️ **no lock** | daily, read-only over `gsc_submission_queue`, low risk |
| **autocommit scripts** (`agents`, `brain`, `content` ×2) | ⚠️ **no lock** | Each does `git add -A && git commit && git push`. Two overlapping runs on the same repo would produce a duplicate/empty commit or a push race. Staggered 23:35/23:40/23:50/23:55 so overlap requires a >5-min hang. |
| ship-to-site deploy_batch | ✅ | writes to `shipped.log` and checks the site whitelist before shipping (`lib/validate.sh:66` `slug_already_shipped`) |

**Not safe to re-run, ranked:** `issues-capture/notify_telegram.sh` (duplicate
operator alerts), `issues-capture/capture.sh` (duplicate rows), the four
autocommit scripts (git race). The rest are locked or DB-deduped.


---

## PHASE 5 — COST AND CAPACITY

Real money spent is **$0** — everything runs inside the Max subscription. The
`estimated_cost_usd` column in `token_usage` is a **notional capacity proxy**,
not an invoice. Every figure below is read that way: "$X" means "the Max
capacity a metered API would have priced at $X".

### 5.1 Tier call-sites and daily volume

Measured from `token_usage`, last 30 days:

| agent_id | tier | calls/30d | notional | **per call** | calls/day at current schedule |
|---|---|---|---|---|---|
| asbestos-writer | ai-do (Sonnet) | 110 | $62.82 | **$0.571** | ~3.7 |
| asbestos-auditor | ai-do (Sonnet) | 110 | $62.02 | **$0.564** | ~3.7 |
| ssg-auditor | ai-do (Sonnet) | 108 | $60.82 | **$0.563** | ~3.6 |
| ssg-writer | ai-do (Sonnet) | 108 | $38.99 | **$0.361** | ~3.6 |
| asbestos-source-verifier | ai-cheap (Haiku) | 582 | $16.30 | $0.028 | ~19 |
| curator_cowen | ai-do | 151 | $11.41 | $0.076 | ~5 |
| majordomo | claude --model opus (direct) | 67 | $9.79 | $0.146 | on demand |
| shell | mixed | 187 | $7.58 | $0.041 | ad hoc |
| assignment-drafter-ssg | ai-do | 58 | $5.48 | $0.095 | ~2 |
| idea-agent-search-filter | ai-cheap | 173 | $4.35 | $0.025 | **6/day, fixed** |
| idea-agent-synth | ai-do | 28 | $4.18 | $0.149 | 2/day |
| backlink-prospector-classify | ai-cheap | 38 | $4.01 | $0.105 | batched, weekly |
| tg-monitor-analyzer | ai-cheap | 132 | $3.97 | $0.030 | ~4/day |
| analyst_cowen | ai-do | 56 | $3.65 | $0.065 | ~2/day |
| **domain-hunter-topical** | ai-cheap | **270** | $3.62 | $0.013 | **7–15/day** |
| **domain-hunter-tag** | ai-cheap | **270** | $3.41 | $0.013 | **7–15/day** |
| idea-agent-scan | ai-cheap | 116 | $3.04 | $0.026 | 4/day |
| analyst_casper | ai-do | 18 | $2.14 | $0.119 | ~0.6/day |
| briefer | ai-do | 50 | $1.93 | $0.039 | ~1.7/day |
| domain-hunter-wayback | ai-cheap | 112 | $1.57 | $0.014 | 0–13/day |
| orchestrator | claude --model opus (direct) | 22 | $1.57 | $0.071 | on demand |
| idea-agent-search-gen | ai-cheap | 29 | $1.03 | $0.035 | 1/day |
| scout | claude --model sonnet (direct) | 12 | $0.84 | $0.070 | on demand |
| backlink-prospector-rank | ai-cheap | 10 | $0.59 | $0.059 | weekly |
| telegram_bot_orchestrator | ai-do | 8 | $0.54 | $0.068 | on demand |
| assignment-drafter | ai-do | 4 | $0.26 | $0.065 | rare |

**30-day total: 2,833 calls, $316.00 notional.** Roughly $10.50/day.

**`ai-think.sh` (Opus) is invoked by nothing.** `grep -rn 'ai-think.sh'` finds
only the `lib/run_agent.sh:23` routing line, a conditional override in
`market/analyst/analyze.sh:110`, and comments in `majordomo/run.sh:8` /
`orchestrator/run.sh:11` explaining why they bypass it. Both Opus consumers
(`majordomo`, `orchestrator`) call the `claude` CLI directly with `--model opus`
because they need `--tools`. **No Opus call was found where Sonnet would plainly
do**: majordomo is the operator's diagnostic tool and orchestrator is the
operator's chat agent — both are human-in-the-loop and low-volume (67 + 22 calls
in 30 days, $11.36 combined). Nothing to flag here.

### 5.2 The small-call-in-a-loop pattern

The documented `ai-cheap.sh` fixed floor of "$0.018 per call from system-prompt
cache overhead" **does not match measurement.** The cheapest observed
per-call costs are `domain-hunter-tag` $0.0126 and `domain-hunter-topical`
$0.0134 — about **$0.013**, not $0.018. The doc figure is 35% high. It is still
the right *shape* of concern: a floor exists and it dominates small calls.

Agents making many small `ai-cheap.sh` calls in a loop, by calls-per-run:

| Agent | calls/run | driver | notional/run | Verdict |
|---|---|---|---|---|
| **domain-hunter** | **~26–45** (13 topical + 13 tag + 13 wayback on 2026-08-09) | `score/topical_fit.py` and the tag/wayback scorers: one `ai-cheap.sh` per candidate domain | ~$0.35–0.55 | **Worst offender.** Three separate per-domain loops that could be one batched call. It is also the agent whose output nobody acts on — see Phase 10. |
| **idea-agent** | 13/day fixed (4 scan + 6 search-filter + 1 search-gen + 2 synth) | `scan.sh:52`, `google_search.sh:149` — per-item loops | ~$0.63/day | Second worst. `search-filter` at 6/day every single day is a fixed cost. |
| asbestos-source-verifier | ~19/day, but **per ambiguous claim** | `verify_sources.py:541` adjudicator | $0.53/day | Legitimate: each call needs a different source passage in context. Cannot be batched without losing the whole point of the gate. **Do not "optimize" this one.** |
| tg-monitor-analyzer | ~4/day | per-chat digest | $0.13/day | Fine |
| backlink-prospector-classify | batched **25 URLs per call** | `filter.sh:94` | $0.10/run | **Already fixed** — `filter.sh:52` documents the I2 change from one-call-per-URL to 25-per-call on 2026-06-09. This is the model the other two should copy. |

**The single most likely capacity sink is domain-hunter**: ~39 Haiku calls per
daily run, three separate per-domain loops, ~$8.60/30d notional, feeding a table
(`domain_candidates`, 1,305 rows) that no downstream agent reads.

### 5.3 The cost caps — which ones actually stop anything

Four cap values exist in code today:

| Cap | Value | Defined at | Read by | **ENFORCING or DECORATIVE** |
|---|---|---|---|---|
| `COST_CAP_SOFT_USD` | **25** | `config/.env:31` | `lib/check_cost_caps.sh:26` | **ENFORCING** — `:141` `touch "${SOFT_FLAG}"` (`~/store/flags/DRAFTER_PAUSED`), which `assignment-drafter/drafter.sh:85-90` reads and exits on. Verified it has fired for real: `audit_log cost_cap_soft_tripped` 2026-06-08 23:29:56, `{"spend_usd":"25.8103"}`. |
| `COST_CAP_HARD_USD` | **50** | `config/.env:32` | `check_cost_caps.sh:27` | **ENFORCING** — `:126` rewrites `.env` `SYSTEM_PAUSED=true` *before* touching the flag, and `SYSTEM_PAUSED` is checked at the top of `ai-do.sh:17`, `ai-cheap.sh:19`, `ai-think.sh:19`. That halts every LLM call in the fleet. Last fired 2026-05-17 during a deliberate test (`hard_cap_usd:"0.5"`). |
| `MAJORDOMO_COST_CAP_USD` | **20** | `config/.env` | `check_cost_caps.sh:75` | **ENFORCING** — `:80` touches `MAJORDOMO_PAUSED`, read by `orchestrator/run.sh:56`. Never tripped (majordomo peaked at $9.79 over 30 days). |
| `MONTHLY_API_BUDGET_USD` | **200** | `config/.env:33` | **NOTHING** | **DECORATIVE.** `grep -rn 'MONTHLY_API_BUDGET_USD'` across `~/agents/` and both content repos returns **zero** hits outside `.env` itself. It is a number in a file. Say it plainly: there is no monthly cap. |

The enforcement chain is real: `lib/log_token_usage.sh:89` calls
`check_cost_caps.sh` after every `token_usage` INSERT
(`bash "$HOME/agents/lib/check_cost_caps.sh" &>/dev/null || true`).

**But the soft cap is close to observability-only in practice.** It tripped at
**23:29:56** on 2026-06-08 — by which time the drafter's 09:00 tick and the
whole day's pipeline had already run. `DRAFTER_PAUSED` only stops the *next*
09:00 tick, and the 00:00 cron clears it before then. The soft cap can pause the
drafter only if it trips between 00:00 and 09:00, which given the schedule
(writers fire from the 09:00 drafter) is the least likely window.

### 5.4 The daily cap reset misses ~37% of days (HIGH)

The `0 0 * * *` crontab line clears `SYSTEM_PAUSED`, `DRAFTER_PAUSED` and
`MAJORDOMO_PAUSED`. It logged `caps_reset` on only **19 of the last 30 days**.
Missing:

```
2026-07-11, 07-12, 07-13, 07-16, 07-19, 07-20,
2026-07-26, 07-28, 07-31, 2026-08-02, 08-06
```

The cause is almost certainly the machine being asleep at 00:00 — macOS `cron`
does not catch up missed jobs (unlike `launchd`'s `StartCalendarInterval`, which
fires on wake). Every other launchd job in this fleet uses
`StartCalendarInterval` for exactly this reason; this one reset is the sole
cron-based scheduled job at a time the machine is likely asleep.

**Consequence:** if a cap trips on day N, the pause flags survive into day N+1
and possibly longer. The drafter goes silent and the only signal is its absence.
`token_usage` shows this has not yet coincided with a trip — the one real soft
trip (2026-06-08) was followed by a successful reset — so this is a **latent**
failure, not a realized one. **Classification: NEVER-WORKED reliably** (cron has
always had this property).

### 5.5 token_usage coverage — who is invisible

`token_usage` has 6,343 rows total, 2,833 in the last 30 days, newest today.
**No agent that makes LLM calls is missing from it.** Cross-checking the 26
`agent_id`s in `token_usage` against every `ai-*.sh` / direct-`claude` call site
found in Phase 5.1 leaves no gap: `majordomo`, `scout` and `orchestrator` call
`claude` directly but each still calls `log_token_usage.sh` explicitly.

Two `agent_id`s are junk-drawer buckets worth naming:
- **`shell`** — 187 calls, $7.58/30d. This is `ai-do.sh:24`'s default
  (`AGENT_ID="${2:-shell}"`) for any caller that forgot to pass an agent id.
  $7.58 of capacity is attributed to nobody. Not invisible, but unattributable.
- **`unknown`** — appears in `audit_log` (45 rows) as `AI_CALLER:-unknown`, not
  in `token_usage`. Same class of problem on the audit side.

### 5.6 Unbounded retry / polling loops

Checked every loop that can repeat an LLM or network call:

| Site | Bound | Safe? |
|---|---|---|
| `lib/ai-do.sh:107` writer-lock `until` loop | `MAX_WAIT=2000`s then exit 1 + audit row | ✅ bounded |
| `lib/notify.sh:95` `for a in 1 2 3` | 3 attempts, backoff `sleep $((a*2))` | ✅ bounded |
| `geo-optimizer/score.sh:100` `for ATTEMPT in 1 2` | 2 attempts then exit 1 | ✅ bounded |
| `run-batch.sh` writer↔auditor revision loop | `max_rounds` (`MAX_ROUNDS` var), escalates to Telegram at the limit | ✅ bounded |
| `research-opportunity/poll.sh` hourly cron | no internal retry | ✅ |
| `link-monitor/check.sh` per-URL fetch | `--max-time 25 --connect-timeout 10`, single retry for the meta-refresh case | ✅ bounded |
| `source_fetch.py:188` `for label, ua, extra in UA_CHAIN` | fixed-length UA chain | ✅ bounded |

**No unbounded retry or polling loop found.** The one thing that *could* burn
capacity on a persistent failure is the run-batch revision loop — a page that
fails the auditor every round costs `max_rounds × ($0.57 writer + $0.56
auditor)` — but it is bounded and it escalates.


---

## PHASE 6 — SAFETY, SECRETS, KILL SWITCHES

### 6.1 Every kill switch, and whether the read gates anything

| Flag | Value today | Defined | Read by | **VERDICT** | Enforcing line |
|---|---|---|---|---|---|
| `SYSTEM_PAUSED` | false | `.env` | `ai-do.sh:17`, `ai-cheap.sh:19`, `ai-think.sh:19`, `drafter.sh:81`, `deploy_batch.sh:21`, `ship.sh`, `preview_ping.sh:22`, `domain-hunter/scan.sh:35`, `backlink-prospector/search.sh:13`, `orchestrator/run.sh:48` | **ENFORCING** | `ai-do.sh:17-20` `if [ "${SYSTEM_PAUSED}" = "true" ] … exit 1` — halts every model call in the fleet |
| `LLM_SPAWN_ENABLED` | true | `.env` | same three wrappers + `drafter.sh:81`, `majordomo/run.sh:67`, `orchestrator/run.sh:52` | **ENFORCING** | `ai-cheap.sh:19-22` |
| `PUBLISHER_DEPLOY_ENABLED` | true | `.env` | `ship.sh:69`, `deploy_batch.sh:70`, `fix-runner/run_fixes.sh:293` | **ENFORCING** | `deploy_batch.sh:70-76` — blocks the deploy, writes an audit row, sends one Telegram alert per batch. **This is the switch that was once decorative; it is now genuinely wired.** |
| `TELEGRAM_OUTBOUND_ENABLED` | true | `.env` | `lib/notify.sh:39` | **ENFORCING** | `notify.sh:39-45` — stub mode, prints + audit-logs `notify_stub`, no send |
| `LIBRARIAN_ENABLED` | true | `.env` | `librarian/run_librarian.sh:10` | **ENFORCING** | `:10-12` early exit |
| `RESEARCH_OPPORTUNITY_ENABLED` | true | `.env` | `poll.sh`, `triage.sh:35`, `extract.sh:52`, `digest.sh` | **ENFORCING** | `triage.sh:35-37` |
| `REDDIT_ENABLED` | false | `.env` | `poll.sh:72` `run_source reddit REDDIT_ENABLED …` | **ENFORCING** | `poll.sh:72` |
| `STACKEXCHANGE_ENABLED` / `YOUTUBE_ENABLED` | true | `.env` | `poll.sh` same dispatcher | **ENFORCING** | `poll.sh:72-74` |
| `MAJORDOMO_ENABLED` | true | `.env` | `majordomo/run.sh:44` | **ENFORCING** | `:44-46` |
| `MAJORDOMO_SCOUT_ENABLED` | true | `.env` | `scout/run.sh:38`, `majordomo/run.sh:177` | **ENFORCING** | `scout/run.sh:38-40` |
| `BOB_ENABLED` | true | `.env` | `orchestrator/run.sh:36` | **ENFORCING** | `:36-38`. `telegram/bot.js:268` comments that it deliberately does NOT duplicate the gate — single source of truth ✅ |
| **`WRITER_ENABLED`** | true | `.env` | `lib/rebuild_command_center.sh:21`, `orchestrator/commands/morning_brief.sh:53` | **🔴 DECORATIVE** | **None.** Both readers only *print* the value. The actual writers pass `agent_id=asbestos-writer` / `ssg-writer` straight to `ai-do.sh`, which never consults it. `lib/run_agent.sh:29-33` builds `<AGENT_ID>_ENABLED` dynamically, but `run_agent.sh` is only ever invoked with `orchestrator`/`majordomo` (from `telegram/bot.js:43` and `dashboard/server.js:26`). Setting `WRITER_ENABLED=false` stops nothing. |
| **`EDITOR_ENABLED`** | true | `.env` | `lib/rebuild_command_center.sh:22`, `morning_brief.sh:54` | **🔴 DECORATIVE** | **None.** Same as above. `editor/score.sh` sources `.env` but never tests the variable. |
| **`MONTHLY_API_BUDGET_USD`** | 200 | `.env` | **nothing** | **🔴 DECORATIVE** | **None** — zero grep hits outside `.env`. See Phase 5.3. |

**Three decorative flags.** `WRITER_ENABLED` is the dangerous one: it reads like
the master switch for the thing that spends the most capacity and writes to a
public site, and it does nothing. `lib/rebuild_command_center.sh` — one of its
two readers — is itself invoked by nothing (Phase 0, Class C), so the operator's
"command center" view of these switches is generated by a dead script.

### 6.2 Secrets

- `~/agents/config/.env` — **`-rw-------` (600)** ✅, owner `mmm2`.
- `~/agents/telegram/.env` — **`-rw-------` (600)** ✅.
- `~/.claude_oauth_token` — **`-rw-------` (600)** ✅, held outside the repo
  deliberately (`.env` sources it at the bottom rather than inlining the value).
- Both `.env` files are gitignored and **never tracked**:
  `git log --all -- config/.env telegram/.env` → **no commits**.
  `git check-ignore -v` confirms the matching rule is `.gitignore:60 **/.env`.

**Full git-history scan.** `git log -p --all` piped through pattern searches:

| Pattern | Hits |
|---|---|
| `BOT_TOKEN=[0-9]` | 0 |
| `AIza[0-9A-Za-z_-]{30}` (Google API key shape) | 0 |
| `sk-[A-Za-z0-9]{20}` | 0 |
| `private_key` | 0 |
| `CLAUDE_CODE_OAUTH_TOKEN=sk` | 0 |
| `service_account` | 1 — a prose mention in `gsc-submit/CLAUDE.md` explaining that **no** service-account JSON exists |
| `oauth_token` | 27 — all references to the *path* `~/.claude_oauth_token`, never a value |

**Live-value scan.** Extracted the actual `YOUTUBE_API_KEY` (39 chars),
`SERPER_API_KEY` (40) and `DASHBOARD_TOKEN` (32) values and grepped for each
across `~/agents`, `~/brain`, `~/projects` and the full git history:
**zero hits outside `config/.env` itself.** No key has leaked into a log, a
brain note, a committed file, or history.

**Secret-in-log risk paths (latent, not realized):**
- `research-opportunity/poll_youtube.sh:60` and `discover_youtube.sh:60,88`
  build URLs containing `key=${YOUTUBE_API_KEY}` and call `curl -sS`. `-S`
  prints transport errors to stderr, and cron redirects `2>&1` into
  `logs/poll.log`. A curl error message that echoes the URL would write the key
  to disk. **It has not happened** (the live-value scan above covers those logs),
  but the path is open.
- `lib/dashboard.sh:38,54,64` prints `?token=${DASHBOARD_TOKEN}` to stdout. It is
  an interactive helper and is not in cron, so this is acceptable — but it must
  never be added to a scheduled job.
- **No script sends a secret into a Telegram message on any error path.** Every
  FATAL message names the *variable* (`"SERPER_API_KEY not set in .env"`), never
  the value. Checked all 14 such sites.

### 6.3 `--dangerously-skip-permissions` — 14 unattended invocations

The flag is passed only via `lib/ai-do.sh:41`:
```bash
[ "${AI_DO_SKIP_PERMISSIONS:-0}" = "1" ] && PERM_FLAG="--dangerously-skip-permissions"
```
Opt-in; the default is the safe behavior.

**Unattended (cron-reachable) call sites — 14, all in the content pipelines:**

| File | Lines | Role |
|---|---|---|
| `asbestos-contractors/content/run-batch.sh` | 505, 539, 569, 796, 822 | writer + revise |
| `asbestos-contractors/content/run-batch.sh` | 1090, 1124, 1275 | auditor |
| `ssg-content/content/run-batch.sh` | 472, 507, 540, 698, 724 | writer + revise |
| `ssg-content/content/run-batch.sh` | 967, 1002, 1101 | auditor |

These reach cron via `assignment-drafter/drafter.sh` (09:00) →
`lib/fire_pipeline.sh` → `run-batch.sh`. **No human is present.** The model runs
with permissions fully bypassed, in a repo it can write to, on a machine with the
operator's `~` accessible. The justification in the code comment
(`ai-do.sh:37-40`) is accurate — a headless writer genuinely cannot answer a
permission prompt — but the mitigation is a prompt instruction, not a boundary.
Contrast with `scout` and `majordomo`, which solve the identical problem with
`--tools` and get a real boundary.

**Interactive use** (operator running `claude` at a terminal) is a separate,
accepted trade-off and is not counted here.

### 6.4 Sandbox restrictions — `--tools` vs `permissions.allow`

**No agent relies on `permissions.allow` as a restriction.** Verified by reading
all five `settings.json` files. Each one states the correct model explicitly:

- `scout/.claude/settings.json` `_comment`: *"`deny` is load-bearing. `allow` is
  NOT: verified by live probe 2026-07-31, permissions.allow is an additive
  auto-approve list and does NOT deny what it omits."*
- `majordomo/.claude/settings.json` carries the same warning plus the
  doubled-leading-slash rule (`Read(//Users/...)`).

Fail-closed boundaries confirmed in code:
- `scout/run.sh:85` — `--tools WebSearch WebFetch Read` (web, no shell, no write)
- `majordomo/run.sh:134` — `--tools Read Grep Glob` (no Bash, no web, no write)
- `orchestrator/run.sh:117` — `--tools Bash Read Write Grep Glob` (shell, **no web**)

The separation is deliberate and correct: the component with a shell has no
network reach; the component with network reach has no shell.

**One defect, low severity.** `scout/.claude/settings.json` allow-list contains:
```json
"Read(/Users/mmm2/agents/scout/**)"
```
— a **single** leading slash, which per the project's own documented rule is
treated as project-root-relative and silently never matches. The exported copy at
`exports/bob-kit/scout/.claude/settings.json` explicitly notes this and uses the
doubled form. It is in `allow`, so the only consequence is a missing
auto-approval, not a security hole — but it is a known-broken rule left live in
the one place that matters.

### 6.5 Agents that can write to a public surface

Complete enumeration of every path that pushes, deploys, or sends outward:

| Surface | Agent | Gate |
|---|---|---|
| **asbestoshq-site git push** (live site) | `ship-to-site/deploy_batch.sh` → `lib/git_ops.sh:31` | `SYSTEM_PAUSED` (`:21`), `halt.flag` (`:26`), `PUBLISHER_DEPLOY_ENABLED` (`:70`), per-slug `lib/validate.sh` eligibility (pipeline JSON exists, site repo clean, slug not already shipped), `lib/build_verify.sh`, `lib/rollback.sh` on failure. **Well gated.** |
| asbestoshq-site git push | `ship-to-site/ship.sh` | Same gates — but its cron line is commented out, so unreachable on a schedule |
| `agents-vault` git push | `scripts/agents-autocommit.sh:12` | **None.** `git add -A && git commit && git push origin main`, nightly 23:50, unconditional. Private repo. |
| `brain-vault` git push | `scripts/brain-autocommit.sh:27` | **None.** Nightly 23:55. Private repo. |
| asbestos-contractors + ssg-content git push | `scripts/content-autocommit.sh:34` ×2 | **None.** Nightly 23:35 / 23:40. Private repos. |
| repo pushes from the fix-runner | `fix-runner/run_fixes.sh:293` | `PUBLISHER_DEPLOY_ENABLED`; manual-invocation only, no cron |
| **Telegram to the operator** | `lib/notify.sh` (20 callers) + `telegram/bot.js` | `TELEGRAM_OUTBOUND_ENABLED`; allowlist on `ALLOWLIST_USER_IDS` |

The only *public* surface is the asbestoshq site, and it is the one with four
independent gates. The four unconditional `git push` jobs all target **private**
GitHub repos (`agents-vault`, `brain-vault`, and the two content repos), so an
unwanted commit is a hygiene problem, not disclosure. Worth stating plainly
because "agent can git push with no gate" sounds worse than it is here.

### 6.6 Backups

- **Job:** launchd `com.aiteam.backup`, `/bin/bash scripts/backup-rsync.sh`,
  StartCalendarInterval 23:30, `LastExitStatus = 0`.
- **Last successful run: 2026-08-08 23:30:10**, `audit_log backup-rsync|backup_ok`
  `{"sources":8,"total":"231M","duration_s":7}`. Sentinel
  `~/store/flags/last_backup_ok` mtime matches.
- **Coverage:** 8 source trees → `/Volumes/Backups/mini-backup/`. `/Volumes/Backups`
  is mounted. The live `aiteam.db`/`-wal`/`-shm` are deliberately **excluded**
  from the rsync; instead `:54` takes a consistent `sqlite3 .backup` snapshot and
  `:100` rsyncs that to `store/aiteam-backup.db`. Correct approach.
- **Failure detection:** three distinct Telegram alerts — drive missing (`:39`),
  destination unwritable/TCC (`:46`), snapshot failed (`:54`), rsync failed
  (`:106`) — plus the watchdog's independent 48-hour sentinel-staleness check
  (`watchdog.sh:427-441`). One real failure is on record and was caught:
  2026-08-05 23:30:09 `backup_failed {"failed":"agents(rc=23)"}`.
- **Has a restore ever been tested? NO.** There is no restore script anywhere in
  the repo (`find`/`grep` for `restore` returns only
  `majordomo/playbooks/remount-backups.sh`, which remounts the volume and does
  not restore). No `audit_log` action mentions a restore. **These are UNVERIFIED
  backups.** They are known to be *written*; nothing has ever demonstrated they
  can be *read back into a working system*.


---

## PHASE 7 — STATE AND DATA HYGIENE

DB: `~/store/aiteam.db`, **25.8 MB** + a **4.1 MB** WAL. `journal_mode = wal`.
Three stale `.bak`/`.backup` copies also sit in `~/store/` (11 MB, 2.2 MB,
1.6 MB) from May and July migrations — 15 MB of dead weight, harmless.

### 7.1 Every table: size, range, growth, pruning

| Table | Rows | Oldest | Newest | Pruned? | Note |
|---|---|---|---|---|---|
| `audit_log` | 30,510 | 2026-05-14 19:56 | 2026-08-09 11:24 | **NO** | ~350/day. See 7.2. |
| **`source_verifications`** | **17,793** | **2026-08-08 23:44** | **2026-08-09 00:47** | **NO** | **The whole table was written in 63 minutes.** See 7.3. |
| `token_usage` | 6,343 | — | today | **NO** | ~95/day |
| `agent_issues` | 3,020 | 2026-05-25 | 2026-08-08 | **NO** | 2,911 still `open`. See 7.5. |
| `domain_candidates` | 1,305 | 2026-05-24 | 2026-08-09 | **NO** | ~15/day, nothing reads it downstream |
| `link_monitor_refs` | 264 | — | — | **YES** | `link-monitor/check.sh:278` `DELETE FROM link_monitor_refs;` — rebuilt each run. The only pruning statement in the fleet. |
| `auditor_verdicts` | 223 | — | — | NO | bounded by content volume |
| `conversation_log` | 158 | 2026-05-14 | 2026-08-06 | NO | bounded |
| `outbound_link_prospects` | 146 | 2026-05-17 | 2026-08-09 | NO | weekly |
| `link_monitor_history` | 63 | — | 2026-08-09 | NO | |
| `link_monitor_urls` | 60 | — | — | NO | |
| `gsc_submission_queue` | 70 | — | — | NO | See 7.4 |
| `internal_link_inventory` | 46 | — | — | NO | |
| `hive_mind` | 45 | 2026-05-14 | **2026-08-09 02:00** | NO | See 7.3 |
| `majordomo_runs` | 43 | — | — | NO | |
| `idea_proposals` | 39 | — | — | NO | |
| `zombie_resurface_log` | 36 | — | — | NO | |
| `warroom_transcript` | 26 | 2026-05-14 20:45 | **2026-05-14 23:44** | NO | See 7.3 |
| `idea_calibration` | 24 | — | — | NO | |
| `source_verifications`, `audit_link_failures` | 12 | 2026-05-24 | **2026-06-26** | NO | See 7.3 |
| `content_loop_state` | 8 | — | — | NO | frozen with content-loop |
| `backlink_proposals` | 5 | — | — | NO | |
| `pending_forwards` | 3 | — | — | NO | |
| `model_overrides` | 1 | — | — | NO | |
| **`memories`, `mission_tasks`, `sessions`, `scheduled_tasks`, `page_performance`, `page_refresh_flags`** | **0** | — | — | — | Empty. See 7.3. |

**Unbounded growth to flag:** `audit_log`, `source_verifications`, `token_usage`,
`agent_issues`, `domain_candidates`. None is near a size that hurts today; the
DB would need to reach hundreds of MB before SQLite cares.

### 7.2 `audit_log` claims a 90-day retention that does not exist

`lib/log_to_audit.sh:3`:
```
# Purpose: append a row to audit_log (90-day retention, append-only).
```
`grep -rn 'DELETE FROM'` across `~/agents/` and both content repos returns
**exactly one** statement, and it is `link_monitor_refs`. Nothing prunes
`audit_log`. Oldest row is 2026-05-14 — **87 days old**. In three days the first
row crosses the documented 90-day boundary and nothing will remove it.

**Classification: NEVER-WORKED.** The retention was written into a comment and
never into code. Low severity (growth is ~350 rows/day) but it is a documented
guarantee that is false.

### 7.3 Dead tables and write-only tables

**Written by nobody (dead):**
- `memories` — 0 rows, 0 writers, 0 readers. Fully dead.
- `sessions` — 0 rows, 0 writers, 0 readers. Fully dead.
- `mission_tasks` — 0 rows, **0 writers, 5 readers** (`scripts/ctx.sh:80`,
  dashboard, majordomo state dumps). Every reader queries a table that nothing
  ever fills — they all silently return empty. A reader with no writer is the
  quiet twin of a writer with no reader.
- `scheduled_tasks` — 0 rows, 0 writers, 1 reader.
- `page_performance` / `page_refresh_flags` — 0 rows. **Deliberate**:
  `auto-refresh/fetch_gsc.sh` is an explicit stub, and `evaluate.sh:82` prints
  "has ever been ingested. The ingest side (fetch_gsc.sh) is a stub". This is the
  one case where an empty table is honest — the agent refuses to emit verdicts
  rather than emit fake ones. Correct design; leaving it recorded so it is not
  mistaken for rot.
- `warroom_transcript` — 26 rows, **all from 2026-05-14**, 1 writer, **0
  readers**. Dead since day one of the project.

**Written but read by nobody (write-only):**
- **`source_verifications` — 17,793 rows, 1 writer, 0 readers.** This is the
  most extreme case in the DB. `verify_sources.py` writes a row per claim per
  run; nothing ever queries the table. The gate's *decision* flows through the
  process exit code, so the table is pure archive. Useful forensically, but it
  grew by 17,793 rows in 63 minutes on 2026-08-08 and nobody has read one.
- **`hive_mind` — 45 rows, 2 writers, 0 readers.** It was dead once and it is
  **dead again in the read direction**: it is still being written (newest row
  2026-08-09 02:00:09, from librarian) but no script or dashboard panel selects
  `FROM hive_mind`. Verified: `grep -rl "FROM hive_mind"` → 0 files.
- **`audit_link_failures` — 12 rows, 1 writer, 0 readers**, newest
  **2026-06-26**. The internal-link advisory check writes findings nobody reads.
  It has not written anything in 44 days.
- **`outbound_link_prospects` — 146 rows, 1 writer, 0 readers**, newest today.
  backlink-prospector's whole output. See Phase 10.

### 7.4 `gsc_submission_queue`

```
site      total  submitted_at NOT NULL
asbestos    38          0
ssg         32          0
TOTAL       70          0
```

**As of today: 70 rows, 0 submitted, 70 never submitted.** The historical
statement — that every row has a NULL `submitted_at` and submission has never
once completed — **remains true.**

**Credential state: none exist.** `gsc-submit/CLAUDE.md:30` states "No GSC
credentials exist on this machine. Checked: no service-account…", and
`digest.sh:11` repeats it. There is no `GSC_SERVICE_ACCOUNT_JSON` in `.env` and
no JSON key anywhere. `reconcile.sh:20` is explicit that it *cannot* set
`submitted_at` and never does.

This is honest, not broken: the agent knows it cannot submit and says so rather
than marking rows done. The queue is a to-do list for a human. What it means in
practice is that **70 published URLs have never been announced to Google**, and
the daily 08:00 digest exists to keep reminding the operator of that.
**Classification: NEVER-WORKED, by design and correctly labelled.**

### 7.5 `agent_issues` — 2,911 open, and the count is the signal

```
open      2911
wontfix     93
resolved    16
```
Top open by agent:
```
research-opportunity  1824
notify                 830
idea-agent              106
unknown                 66
assignment-drafter      39
```

Two agents account for **91%** of the open backlog. Both are explained:
- `notify` 830 — the 2026-07-23→07-31 delivery outage (Phase 2.9). **Resolved in
  reality, still `open` in the table.**
- `research-opportunity` 1,824 — the hourly poller with a disabled Reddit source
  (`REDDIT_ENABLED=false`, blocked since 2026-05-28) generating a failure row
  every hour.

An issues queue where 96% of entries are open and the top two causes are both
already-understood is not a queue anyone triages. It is a counter. See Phase 8.

### 7.6 State files: owners and corrupt/missing behavior

| File | Owner (writer) | Readers | On corrupt/missing at read |
|---|---|---|---|
| `~/store/flags/last_backup_ok` | `scripts/backup-rsync.sh` | `watchdog.sh:428` | `BACKUP_TS="$(cat … 2>/dev/null \|\| echo 0)"` then `case "$BACKUP_TS" in *[!0-9]*) BACKUP_TS=0 ;; esac` → treated as **stale → alert**. **Fails safe.** ✅ |
| `~/store/flags/{SYSTEM,DRAFTER,MAJORDOMO}_PAUSED` | `check_cost_caps.sh` | `drafter.sh:85`, `orchestrator/run.sh:56` | Existence-only test (`[ -f … ]`). Cannot be corrupt. Missing = not paused, which is the safe default given `.env` carries the authoritative `SYSTEM_PAUSED`. ✅ |
| `~/agents/watchdog/state.json` | `watchdog.sh` (`state_set`) | `watchdog.sh` | `:58` `SIZE="$(stat -f %z … \|\| echo 0)"` — size-guarded before parse. Currently 1,844 bytes, mode 600. Corrupt JSON would make `jq` fail; **not explicitly handled** — the per-agent status read would return empty and the agent would look never-seen → alert. Fails noisy rather than silent. ✅ |
| `~/agents/watchdog/*_alerted.date` | `watchdog.sh` | same | `[ "$(cat … 2>/dev/null)" != "$TODAY" ]` → missing/corrupt ⇒ **alert fires**. Fails safe (worst case: a duplicate alert). ✅ |
| `/tmp/.aiteam_env_mtime` | `check_kill_switches.sh:35` | itself | Missing ⇒ `echo "0"` ⇒ mtime mismatch ⇒ re-export. Fails safe. **But it lives in world-writable `/tmp`**: any local process can write a matching mtime and suppress the refresh permanently. Given the refresh is already a no-op (Phase 1.4), the practical impact is nil. |
| `assignment-drafter/state/{completed,failed}.log`, `pipeline_runs.log` | drafter | operator | append-only, no parse |
| `ship-to-site/state/shipped.log` | deploy_batch | digest | append-only |
| `research-opportunity/halt.flag`, `idea-agent/halt.flag`, `notes-agent/halt.flag`, `keyword-registry/halt.flag`, `ship-to-site/halt.flag` | operator (manual) | each agent | existence-only. **None currently present.** ✅ |
| `keyword-registry/…/keyword_registry.md` | `backfill_registry.sh` | operator + drafter | regenerated nightly from ground truth; drift emits a `keyword_registry_drift` audit row rather than silently overwriting |

**No state file fails open in a dangerous direction.** Every one either
fails-safe (alert/pause) or cannot be corrupted. That is a genuine pass and the
strongest single area of the codebase.


---

## PHASE 8 — OBSERVABILITY

### 8.1 THE HEADLINE — the watchdog cries wolf ~16–22 times a day

`~/brain/projects/aiteam/incidents/` holds **349 incident files**. By actor:

```
254  orchestrator          <-- 73% of every incident ever raised
 17  market-process-queue
 15  tg-monitor-reader
 12  market-curator
 10  ship-to-site-digest
  9  market-briefer
  6  ship-to-site-preview
  6  ship-to-site-deploy
  5  market-scribe-poll
  5  assignment-drafter
  4  grammy-bot
  4  brain-autocommit
  2  notify-delivery
```

**Every one of the 254 orchestrator incidents carries `"last_seen_ts": null`.**
And the diary rows exist:

```
audit_log  orchestrator|diary_written
  1785825915  2026-08-03 23:45:15
  1785912313  2026-08-04 23:45:13
  1785998714  2026-08-05 23:45:14
  1786085115  2026-08-06 23:45:15
  1786257911  2026-08-08 23:45:11
```

Take one incident and check it by hand:
`2026-08-04_090000_orchestrator.json`, `detected_at 2026-08-04T16:00:00Z` =
09:00 PDT. The last `diary_written` was **2026-08-03 23:45:15 — 9 h 15 m
earlier**, against a `check_window_hours: 26`. The row was well inside the
window. **The alert was false.** The same arithmetic condemns the incidents on
2026-08-05 01:00/02:00/03:00, 2026-08-06 03:00/06:00, 2026-08-09 01:00, and
dozens more.

The alert/recovery counts confirm it is flapping, not a sustained outage:

```
2026-08-07  watchdog_alert 8   watchdog_recovery 8
2026-08-06  watchdog_alert 11  watchdog_recovery 11
2026-08-05  watchdog_alert 8   watchdog_recovery 8
2026-08-04  watchdog_alert 3   watchdog_recovery 4
```

**Alerts and recoveries pair exactly.** Each pair is two Telegram messages, so
the operator receives roughly **16–22 messages a day that mean nothing.**
`watchdog_check_complete` is 96/day (every 15 min, as scheduled), so ~6–11% of
runs mis-probe.

**Mechanism — CONFIRMED.** `watchdog/watchdog.sh:190-194`:
```bash
  row="$(sqlite3 -separator '|' "$DB" \
    "SELECT ts, action FROM audit_log
     WHERE actor_id='${actor}' AND action IN (${in_list}) AND ts > ${thresh}
     ORDER BY ts DESC LIMIT 1;" 2>/dev/null)"
  if [ -z "$row" ]; then echo "|"; else echo "$row"; fi
```
`2>/dev/null` discards any error, and an empty `row` is then treated at `:343`
as "agent never ran". **The query-failed path and the no-rows path are the same
path** — the exact class-6 defect this audit was told to hunt. A probe that
cannot reach its source reports the agent dead.

**Root cause of the intermittent empty result — UNVERIFIED.** Two candidates:
(a) the `sqlite3` call erroring (but note WAL readers do not block on writers, so
`SQLITE_BUSY` is a weaker explanation here than it is for the *writers* in Phase
4.3); (b) `in_list` occasionally parsing empty from the awk YAML reader at
`:83-112`, which would make the SQL `action IN ()` — a syntax error — and return
empty. Orchestrator being the **first** record in `expected_schedule.yaml` and
also the overwhelming majority of incidents points at (b).
**To verify:** run `watchdog/watchdog.sh --dry-run` (documented at `:9` as
"report only; no Telegram, no incidents, no state writes, no audit rows") in a
loop with the `2>/dev/null` removed from `:193`, and log `in_list` per iteration.
I did not run it — this audit does not execute agent scripts.

**Severity: HIGH.** Not because the watchdog is broken in the "misses failures"
direction, but because 254 false alarms are how a *real* alert gets ignored. It
also masked a genuine miss: the diary really did fail to run on **2026-08-07**
(no `diary_written` row between Aug 6 23:45 and Aug 8 23:45), and that true
alert was one message among the day's eight false ones.
**Classification: REGRESSION or NEVER-WORKED — UNKNOWN.** Incident files go back
to `2026-05-17_111500_orchestrator.json`, so the behavior is at least 84 days
old; whether it was ever clean is not determinable from the artifacts on disk.

### 8.2 Per-agent silent-failure detection

"If this agent failed silently right now, what would tell you?"

| Agent | Detector | |
|---|---|---|
| orchestrator (diary) | watchdog `expected_schedule.yaml` | ✅ (but see 8.1) |
| ship-to-site digest / preview / deploy | watchdog, 3 separate entries | ✅ |
| assignment-drafter | watchdog (`drafter_tick_started`) | ✅ |
| brain-autocommit | watchdog (logfile mtime) | ✅ |
| tg-monitor reader | watchdog (`sqlite_row` on `poll_log`) | ✅ |
| grammy bot | watchdog (`launchctl_label` probe) | ✅ |
| market scribe / process_queue / curator / briefer | watchdog, 4 entries | ✅ |
| backup-rsync | watchdog 48 h sentinel staleness (`:427`) + 4 in-script Telegram alerts | ✅ **best-covered agent in the fleet** |
| notify.sh itself | watchdog notify-health (`:494-530`), routed off notify.sh via audit_log + incident JSON + stdout | ✅ **and proven**: fired 2026-07-31 12:17 (82% failure) and 2026-08-01 00:00 (48%) |
| token_usage / cost caps | watchdog `check_cost_logging` (`:382`) | ✅ |
| gsc queue staleness | watchdog `gsc_queue_stale` (fires daily) | ✅ |
| **watchdog itself** | **NONE** | 🔴 |
| **watchdog digest (06:00)** | **NONE** | 🔴 |
| **tg-monitor analyzer** | **NONE** | 🔴 |
| **research-opportunity** (poll / digest / discover) | **NONE** | 🔴 |
| **keyword-registry** | **NONE** | 🔴 |
| **idea-agent** | **NONE** | 🔴 |
| **notes-agent** | **NONE** | 🔴 |
| **domain-hunter** | **NONE** | 🔴 |
| **issues-capture** (capture / notify / digest) | **NONE** | 🔴 |
| **librarian** | **NONE** | 🔴 |
| **backlink-prospector** | **NONE** | 🔴 |
| **gsc-submit digest** | **NONE** (the queue-staleness check watches the *data*, not the job) | 🔴 |
| **link-monitor** | **NONE** | 🔴 |
| **agents/content autocommit** (3 jobs) | **NONE** | 🔴 |
| **cost-cap 00:00 reset** | **NONE** — and it misses 37% of days (Phase 5.4) | 🔴 |
| **dashboard** | **NONE** | 🔴 |
| **majordomo check / daily_brief** | **NONE** | 🔴 |

**12 agents covered, ~20 uncovered.** The uncovered set includes every agent
added since `expected_schedule.yaml` was last extended. The pattern is clear:
watchdog coverage was written once and new agents have not been added to it —
`link-monitor` and `gsc-submit` shipped today with no entry.

### 8.3 Watchdog thresholds and last-fire

| Check | Threshold | Last fired |
|---|---|---|
| Per-agent liveness | per-entry `check_window_hours` (26 h for dailies, 1 h for `*/5` jobs) + `grace_minutes` | continuously — see 8.1 |
| Cost-logging blind | LLM audit evidence >0 AND `token_usage` rows =0, with a 30-min grace | never (`cost_logging_broken` absent from `audit_log`) |
| Backup stale | sentinel missing or >48 h | never (`backup_stale` absent) |
| Notify blackout | 0 delivered of >5 attempted in 24 h | never (the softer `degraded` branch fired instead) |
| Notify degraded | >30% failure in 24 h | 2026-07-31 12:17, 2026-08-01 00:00 |
| GSC queue stale | pending rows unmoved | daily, currently every day |

Alert dedup is a per-check date-stamp file
(`cost_log_alerted.date`, `backup_stale_alerted.date`,
`notify_health_alerted.date`) — once per calendar day per check. Note the
**per-agent liveness alerts have no such dedup**, which is why they can fire 11
times in one day.

### 8.4 Notification paths

**31 files call `lib/notify.sh`.** Parse mode: **11 explicit `plain`, 0
explicit `HTML`** among shell callers; the rest pass no second argument and
therefore default to `plain` (`notify.sh:32`). So essentially the entire fleet
is on the plain path — which is precisely the path the empty-array bug silenced
for five weeks, and precisely the path still protected by the `+alternate` form
at `notify.sh:110`. HTML is used only from `telegram/bot.js`, which does not go
through `notify.sh`.

**Failures of notify.sh itself ARE detected**, and this is the strongest fix in
the codebase. `watchdog.sh:485-492` documents the design explicitly:

```
#   1. audit_log action ending in _failed -> issues-capture sweeps it into
#      agent_issues -> visible on the dashboard :3141 /issues panel
#   2. an incident JSON in ~/brain/projects/aiteam/incidents/
#   3. stdout, which cron appends to watchdog/watchdog.log
# None of the three touch the Telegram path, so this check still fires — and
# stays visible — precisely when outbound notification is dead.
```

Three independent channels, none of which is Telegram. Verified working: it
caught the 2026-07-23→07-31 outage on 07-31 at 12:17, the same day the detector
was committed (22ccc86). Nothing watches the watcher of the watcher, but three
parallel channels is a reasonable stopping point.

### 8.5 The dashboard on :3141

**Up.** PID 80769, `curl -w %{http_code} http://127.0.0.1:3141/` → **200**.

It reads exactly four tables:

| Route | Table | Written today? | Verdict |
|---|---|---|---|
| `/api/audit-log`, `/api/agents`, `/api/health`, `/` | `audit_log` | ✅ 11:24 today | live |
| `/issues`, `/issues/data` | `agent_issues` | ✅ 2026-08-08 23:50 | live, but 2,911 of 3,020 rows are `open` and 91% come from two known causes — the panel is a wall of stale noise (Phase 7.5) |
| `/api/tokens` | `token_usage` | ✅ today | live |
| `/api/gsc-queue` | `gsc_submission_queue` | ✅ (rows exist) | live data, but **every row reads "not submitted" and always will** — no credentials exist |
| `/api/diary` | filesystem (`~/brain/.../diary/`) | last write 2026-08-08 | live |

**No panel reads a dead table.** `hive_mind`, `warroom_transcript`,
`source_verifications`, `audit_link_failures` and `outbound_link_prospects` —
the five write-only tables from Phase 7.3 — are not surfaced anywhere, which is
consistent but means the data has no consumer at all, not even a human one.

**The `/issues` panel is the one showing effectively dead data**: not stale
timestamps, but 2,911 open items nobody will ever work through.

`server.js` is the only DB client in the fleet that sets
`PRAGMA busy_timeout = 5000` (`:35-36`). It is also unchanged since 2026-06-09
(commit f8962cf) while five new agents have shipped since — none of them has a
panel.


---

## PHASE 9 — GIT AND DOC DRIFT

### 9.1 Working-tree state across all five repos

**Nothing was committed, staged, stashed, or reverted by this audit.**

| Repo | Dirty | Untracked | Production code at risk? |
|---|---|---|---|
| `~/agents` | **0 files — clean** | — | ✅ none |
| `~/projects/asbestos-contractors` | 0 modified | `scripts/LANE1_PROGRESS.md` | no — a progress note |
| `~/projects/ssg-content` | `M content/ssg/drafter_queue.txt` | `content/ssg/assignment-batch-bill-com-vs-ramp.md` | no — both are pipeline *state*, and `drafter_queue.txt` is deliberately gitignored per DEFERRED D066's closure note. The untracked assignment brief is drafter output awaiting the nightly content-autocommit at 23:40. |
| `~/projects/asbestoshq-site` | 0 modified | `LANE3_PROGRESS.md` | no |
| `~/brain` | 9 modified | 7 untracked | no — all agent-generated content (cowen expertise notes, keyword_registry, today's ideas file, an orchestrator incident JSON). Nightly brain-autocommit at 23:55 picks them up. |

**No untracked production code found in any repo.** This is a change from the
project's history — the F6 sweep (2026-06-09) plus the four nightly autocommit
jobs appear to have closed it. Every untracked file today is either a
progress-note markdown or agent output that the nightly job will commit.

### 9.2 Unpushed commits

```
~/agents                        git log @{u}..HEAD  → 0
~/projects/asbestos-contractors                     → 0
~/projects/ssg-content                              → 0
~/projects/asbestoshq-site                          → 0
~/brain                                             → 0
```

**Zero unpushed commits in all five repos.** Every repo's HEAD is at or behind
its upstream. Latest commits: agents `0db99f1` (today), asbestos `9bd9841`
(today), asbestoshq-site `40a18bd` (today), ssg-content `5ee51ab` (2026-08-06),
brain `743abcf` (2026-08-08).

### 9.3 Doc-drift ledger — claims that contradict Phases 0–8

| # | Doc claim | Location | Verified reality | Severity |
|---|---|---|---|---|
| 1 | "Weekly **Monday 09:00 PT**" | `backlink-prospector/CLAUDE.md:100` | crontab: `0 8 * * 0` = **Sunday 08:00**. Wrong day AND wrong hour. | MEDIUM |
| 2 | "Entry sits in `~/agents/lib/cron.txt`. Operator activates manually via `crontab -e` — **NOT auto-installed**" | `backlink-prospector/CLAUDE.md:100` | It **is** installed and running (last run 2026-08-09 08:17). Doc describes a pre-activation state that ended 2026-06-09. | LOW |
| 3 | "Entry sits in `~/agents/lib/cron.txt`. Operator activates…" | `domain-hunter/CLAUDE.md:220` | Live in crontab at `30 6 * * *` and running daily. Same stale pre-activation wording. Schedule itself (06:30) is correct. | LOW |
| 4 | "**Weekly** self-improvement loop" | `idea-agent/CLAUDE.md:3` | crontab `10 7 * * *` = **daily**, and the crontab's own comment says "DAILY 07:10 local (was weekly Sun 09:00; switched 2026-05-24)". `token_usage` confirms 13 calls every single day. The agent dir's own doc is 77 days stale. | MEDIUM |
| 5 | "AI_TIMEOUT_SEC default **1200s** (20 min)" | `lib/ai-do.sh:6` (header) | `lib/ai-do.sh:132`: `AI_TIMEOUT_SEC="${AI_TIMEOUT_SEC:-1800}"`. Code is **1800**. (`ai-cheap.sh`/`ai-think.sh` genuinely are 1200 — only `ai-do` drifted.) | LOW |
| 6 | "append a row to audit_log (**90-day retention**, append-only)" | `lib/log_to_audit.sh:3` | No pruning code exists anywhere (Phase 7.2). Oldest row is 87 days old and will not be deleted. | MEDIUM |
| 7 | `tier: sonnet` | `config-synthesizer/agent.yaml` | Not a valid tier. `lib/run_agent.sh:21-26` accepts only `do\|think\|cheap` and exits 1 on anything else. Harmless *today* only because nothing routes this agent through `run_agent.sh`. | LOW |
| 8 | `tier: ai-do.sh` | `market/briefer/agent.yaml` | Same — not a tier name. | LOW |
| 9 | `tier: script` | `fix-runner/agent.yaml`, `ship-to-site/agent.yaml` | Same. These two are genuinely script-only agents, so the *intent* is clear, but `run_agent.sh` would reject the value. | LOW |
| 10 | `enabled: true` | `content-loop/agent.yaml` | content-loop is invoked by **nothing** (Phase 0, Class C) and last ran 2026-05-27. The flag asserts an agent is on when it is unreachable. | MEDIUM |
| 11 | "`ai-cheap.sh` … fixed floor of roughly **$0.018** per call" | project lore / `domain-hunter/CLAUDE.md` cost note | Measured floor is **~$0.013** (`domain-hunter-tag` $0.0126, `domain-hunter-topical` $0.0134 over 270 calls each). Doc is 35% high. | LOW |
| 12 | `scout/.claude/settings.json` `Read(/Users/mmm2/agents/scout/**)` | live file | Single leading slash — per the project's own documented rule (repeated in `majordomo/.claude/settings.json` and the bob-kit export) this silently never matches. The export copy uses `//`; the live one does not. | LOW |
| 13 | `~/agents/CLAUDE.md:24` "Daily auto-commit+push runs at **23:55**" | root CLAUDE.md | crontab: `55 23 * * *` ✅ **correct** | — |
| 14 | `research-opportunity/CLAUDE.md:78` cron wiring `0 * * * *` / `0 9 * * 1` / `0 9 1 * *` | | crontab matches exactly ✅ **correct** | — |

**Ten drifts, none of them dangerous on its own.** The pattern worth naming: the
per-agent `CLAUDE.md` files describe the state at the moment the agent was
*built*, and are not updated when the schedule changes. Three of the four
schedule claims checked are wrong.

### 9.4 DEFERRED.md — collisions, gaps, and status-vs-reality

```
$ grep -oE "\bD[0-9]+\b" ~/brain/projects/aiteam/DEFERRED.md | sort -u -V
D025 D026 D027 D028 D033 D039 … D120  (88 unique)   plus  D427
```

- **`D427` is a false positive**, not a real item. It is a fragment of a UUID in
  a closure note at line 474: `9A3A7ABD-D427-40C1-9C94-074769567CA2`. Anyone
  running the prescribed grep will see it and should ignore it. Worth fixing the
  grep, not the file.
- Restricting to **heading lines** (`^### D<n>`) gives the true item set:
  **82 items, D025–D120.**

**Duplicate: exactly one.**
```
$ grep -nE '^#{2,4} D027' DEFERRED.md
102:### D027 — Add `*.bak` to `~/brain/.gitignore`
532:### D027 — Add `*.bak` to `~/brain/.gitignore`
```
The same item appears twice, verbatim, at lines 102 and 532. (The other
apparent duplicates — D054, D069, D075, D097, D107 — are cross-references in
prose, not second headings.)

**Gaps:** `29, 30, 31, 32, 34, 35, 36, 37, 38, 49, 59, 61, 62, 92`.
Line 1132 explains part of this ("D049 is unused… D093 now assigned"), so the
gaps are known bookkeeping, not lost items.

**Status tally on headings:** 22 OPEN, 18 CLOSED, 1 deferred. (The remaining ~41
headings carry their status elsewhere in the entry body.)

**Open items my audit shows are actually closed:**

| Item | Claim | Reality |
|---|---|---|
| **D110** — "GEO scorer: **single-shot parse**, and ship.sh folds 'scorer broke' into a benign skip" — 🟡 OPEN | half of it | The single-shot half is **FIXED**: `geo-optimizer/score.sh:100` is now `for ATTEMPT in 1 2` with a retry prompt, landed in `f466673` (2026-07-31) "retry once on a rejected score line". The *second* half — the fold — is still real: `run-batch.sh:1552` `geo_exit -eq 1 → geo_pass_through=true` (my Phase 1.8). **Should be split: close the parse half, keep the fold half open.** |
| **D118** — "auditor_verdicts rows with composite_score **0.00** AND passed=1" — 🟡 OPEN | symptom | `SELECT COUNT(*) … WHERE composite_score=0.0 AND passed=1` → **0 rows**. The exact stated symptom no longer exists. There are 59 rows with `composite_score IS NULL AND passed=1`, but all 59 are `scorer='audit_guide'`, and NULL there is **documented as correct** (`ship-to-site/preview_ping.sh:95`: "audit_guide writes composite_score=NULL (mechanical only)"). **Closeable as stated.** |

**Open items my audit confirms are still genuinely broken:**

| Item | Confirmation |
|---|---|
| **D106** — "SSG has never written a verdict row" | `SELECT … FROM auditor_verdicts GROUP BY site` → asbestos 227 rows, **ssg 0 rows**. Still true. |
| **D114** — "agent_issues has re-accumulated to ~2,900 open rows" | **2,911 open** today. Exactly as described, and still growing. |
| **D115** — "`MAX(ts) < strftime(...)` silently always-true (SQLite affinity)" | Still live at `majordomo/daily_brief.sh:54`: `HAVING MAX(ts) < CAST(strftime('%s','now','-36 hours') AS INTEGER)`. The `CAST` was added, which fixes the affinity mismatch — but the pattern is still the one D115 names. **Needs the operator's judgement whether the CAST closes it.** |
| **D107** — "audit every `<agent>/.claude/settings.json` for phantom allow-list enforcement" | Done in Phase 6.4: no agent relies on `allow` as a restriction; one dead single-slash rule found in `scout`. **This audit largely discharges D107.** |
| **D102** — "Nightly agents-vault auto-commit sweeps unknown runtime files blindly" | Confirmed: `scripts/agents-autocommit.sh:9` is `git add -A` with no filter. |

**No item marked Closed was found to be still broken.**


---

## PHASE 10 — VALUE AND DEAD WEIGHT

No diplomacy. For each: what it makes, who eats it, when a human last looked,
what breaks if it dies tomorrow.

**assignment-drafter** — Produces assignment briefs and fires the content
pipeline. Consumed by `run-batch.sh`, which produces the guides that are the
entire point of the project. A human sees its output every day via the 20:00
preview ping. Delete it and the content machine stops. **KEEP. This is the
spine.**

**ship-to-site (digest / preview_ping / deploy_batch)** — Produces the actual
deploys to asbestoshq.com plus two daily Telegram touchpoints. Consumed by the
public internet and the operator. Looked at daily. Delete it and nothing ships.
**KEEP.**

**verify_sources (asbestos)** — Produces per-claim FAIL verdicts that block
fabricated figures from shipping. Consumed by `run-batch.sh`'s exit-code check.
It fired for real on 2026-08-08. 17,793 archive rows nobody reads, but the
*decision* is load-bearing. Delete it and fabricated agency-attributed numbers go
live. **KEEP — and clone it into SSG (Phase 1.1).**

**watchdog** — Produces liveness alerts. Consumed by the operator's phone. But
254 of 349 incidents it has ever raised are false, and it pairs 8–11 alerts with
8–11 recoveries daily. Its cost-logging, backup-staleness and notify-blackout
checks are genuinely good and have caught real failures. Its per-agent liveness
check is actively harmful. **FIX (not kill): the three system-level checks earn
their keep; the per-agent probe needs the conflation removed.**

**issues-capture** — Produces `agent_issues` rows. Consumed by the dashboard
`/issues` panel. 2,911 open, 16 ever resolved, 91% from two known causes. A human
last acted on this... 16 times, total, in 76 days. It is a counter that nobody
triages, and it costs three cron jobs at `*/5`. **FIX or PAUSE: either add
auto-resolution and source suppression, or stop pretending it is a queue.**

**notify / lib** — Infrastructure, not an agent. **KEEP.**

**market pipeline (scribe → analyst → curator → briefer)** — Produces a daily
08:46 market brief to Telegram and Cowen/Casper expertise notes in `~/brain`.
The operator reads the brief on the phone; `~/brain/expertise/cowen/**` had
uncommitted edits today, so a human is actively working with the output. Costs
$17/30d notional. Not connected to revenue at all. **KEEP but be honest: this is
a personal-interest feed, not mission work.** If the $200/mo bar is the goal, it
is the largest discretionary spend in the fleet.

**domain-hunter** — Produces 1,305 rows in `domain_candidates` and a Telegram
digest. Consumed by: **nothing downstream**. No agent reads `domain_candidates`
except domain-hunter's own scorers. A human last acted on a candidate…
`surfaced=1` rows exist, but no `domain_candidates` row has ever become a
purchase, a redirect, or a backlink — there is no table or file recording one.
It burns ~39 Haiku calls/day in three separate per-domain loops. **KILL or
PAUSE.** It is the fleet's clearest example of an agent that runs beautifully and
produces nothing anyone uses.

**research-opportunity** — Produces `pains: 0` every week for at least six weeks
while scanning hundreds of posts, and 1,824 open `agent_issues` rows from its
disabled Reddit source. Its extraction stage (`triage.sh`, `extract.sh`) is wired
to nothing. It is the fleet's **busiest actor** (6,699 audit rows/60d) and its
output is a weekly Telegram message that says zero. **KILL the hourly poll, or
finish wiring `extract.sh`.** Running an hourly job for six weeks to produce zero
is not "no signal yet" — the pipeline that would produce signal was never
connected.

**idea-agent** — Produces `idea_proposals` (39 rows) and a weekly ideas file in
`~/brain/projects/aiteam/ideas/`. A `weekly_2026-08-09.md` is untracked in brain
right now, so it wrote one today. Consumed by the operator when reviewing
DEFERRED. 13 Haiku/Sonnet calls every day, $0.63/day, of which `search-filter`
(6/day) folds every error into `[]`. **PAUSE the daily cadence — make it weekly
again.** It was weekly until 2026-05-24; nothing suggests daily produced 7× the
ideas. Its own CLAUDE.md still says "weekly".

**backlink-prospector** — Produces 146 `outbound_link_prospects` rows and a
weekly top-10 Telegram digest. `outbound_link_prospects` has **0 readers**. The
digest goes to a human for manual outreach. Has any outreach happened? No column
records it (`outreach_status` exists; every row's value is untested by any
script). **PAUSE until the operator answers "have I ever acted on one of these?"**
If yes, keep — it is cheap ($4.60/30d) and correctly batched. If no, kill.

**librarian** — Sorts a brain inbox nightly. Writes `hive_mind` (0 readers).
Cheap, quiet, ran successfully today. Delete it and brain inbox files pile up.
**KEEP — low cost, real filing work.**

**keyword-registry** — Regenerates `keyword_registry.md` from ground truth
nightly and emits a drift audit row. The file is modified in `~/brain` right now,
so it is doing work. Consumed by the drafter and the operator. **KEEP.**

**notes-agent** — Capture path is wired into the Telegram bot; sort runs weekly
and ran today. Last `bot`-side capture in `audit_log` was **2026-05-24**, so the
capture half has been idle for 77 days while the sort half keeps running on an
empty-ish inbox. **KEEP (it costs almost nothing) but the capture path is
effectively unused.**

**tg-monitor (reader + analyzer)** — Reader polls every 5 min into its own DB;
analyzer produces daily digests. 17.9 MB log, $3.97/30d. Consumed by the
operator via digest artifacts (`tg-monitor/digest_outbox/`, last committed
2026-07-31). **KEEP if the operator reads the digests; otherwise the highest
log-volume agent in the fleet is producing unread summaries.** I could not
determine consumption from the machine — **UNVERIFIED, needs the operator.**

**gsc-submit** — Produces a daily digest naming 70 URLs Google has never been
told about. Consumed by a human who must submit them by hand. Honest about its
own impotence. Delete it and 70 unannounced URLs become 100 unannounced URLs with
nobody counting. **KEEP — it is a correct nag.**

**link-monitor** — Shipped today. Weekly dead-link sweep over `authorityLinks`,
alerts only on status change. Nothing to judge yet. **KEEP, revisit in a month.**

**auto-refresh** — Produces nothing; `fetch_gsc.sh` is an explicit stub and
`evaluate.sh` refuses to emit verdicts without data. **KEEP AS-IS.** This is the
single best-behaved thing in the repo: an agent that knows it cannot do its job
and says so instead of inventing output. It is also invoked by nothing, which is
correct.

**geo-optimizer / editor** — Gates inside `run-batch.sh`. geo last scored
2026-07-31, editor **2026-07-19** (21 days). Editor appears to have fallen out of
the live path. **INVESTIGATE editor: it has 12 verdict rows and none since July
19.**

**internal-link** — `audit_links.sh` runs advisory-only inside the pipeline and
writes `audit_link_failures`, which has **0 readers** and no new row since
2026-06-26. `propose_backlinks.sh` last produced a row 2026-05-17.
`build_inventory.sh` is manual-only, last run 2026-05-24. **PAUSE — the whole
subsystem writes to tables nobody reads.**

**content-loop + config-synthesizer** — Last activity 2026-05-27. `content-loop`
is invoked by nothing; `config-synthesizer`'s only caller is `content-loop`.
Together: two agent directories, 8 executables, an `agent.yaml` still claiming
`enabled: true`. **KILL.** Dead for 74 days.

**fix-runner** — Manual-only by design, last run 2026-06-11. Leaves 12 stale
`state/run_*/` directories with 34 generated `.sh` files that pollute every
repo-wide grep in this audit. **KEEP the tool, DELETE the stale state dirs.**

**scout / majordomo / orchestrator** — Operator-facing diagnostic and chat
agents. Last used 2026-07-31 (scout), 2026-07-31 (majordomo run), 2026-08-06
(orchestrator). Correctly sandboxed, correctly cost-capped in their own lane.
**KEEP.**

**dashboard** — Up, 200 OK, four live panels. Unchanged since 2026-06-09 while
five agents shipped. **KEEP, but it is drifting into irrelevance.**

**exports/bob-kit** — A byte-level fork of `lib/` and the sandbox configs, on no
execution path. It is already *ahead* of the live tree in one place (it fixed the
single-slash `Read()` rule that `scout/.claude/settings.json` still has). **KEEP
as an export artifact but stop treating it as a copy — it has already diverged.**

### The list

**KILL:** `content-loop`, `config-synthesizer`, `domain-hunter`,
`research-opportunity`'s hourly poll, `fix-runner/state/run_*` directories,
`warroom_transcript` + `memories` + `sessions` tables.

**PAUSE (pending one operator answer each):** `backlink-prospector` ("have you
ever acted on a prospect?"), `internal-link` advisory path, `tg-monitor` ("do you
read the digests?"), `idea-agent` daily→weekly.

**FIX:** SSG source verification (Phase 1.1), watchdog per-agent probe
(Phase 8.1), `issues-capture` triage model, `editor` gate (silent since 07-19),
`ai-think.sh` hardening, the two `${VAR//\'/\'\'}` sites.

Killing domain-hunter and the research-opportunity poll removes the fleet's #1
capacity sink and its #1 audit-noise source, and loses nothing any downstream
consumer reads. That is a real improvement, not a retreat.


---

## PHASE 11 — IMPROVEMENTS

Every item traces to an observation in Phases 0–10. Effort in hours. Worth is
judged against the actual bar: **$200/mo revenue target, current revenue $0.**
That bar makes "does this get a correct page in front of a buyer" the only test
that matters, and makes everything else overhead.

### NOW — cheap, fixes something currently broken

| # | Observation | Change | Hours | Worth |
|---|---|---|---|---|
| N1 | Phase 1.1 — SSG ships guides with no source-verification gate; asbestos has one and it fires | Copy `verify_sources.py`, `source_fetch.py`, `safety_checks.py` into `ssg-content/scripts/`, and port the 34-line gate block from `asbestos/run-batch.sh:1499-1533` into `ssg/run-batch.sh` after its `audit_guide.py` stage | **3** | Highest in the document. SSG is half the published output and currently has zero protection against the exact failure mode the asbestos gate was built for. A fabricated statistic on a live page is the one thing that can destroy the site's value before it earns a dollar. |
| N2 | Phase 8.1 — 254 false watchdog alerts, ~16–22 noise messages/day | In `watchdog.sh:190-194`, capture the sqlite exit code separately from the output; emit a third state `unknown` when the query fails, and never alert on `unknown` — log it instead. Also guard `in_list` being empty before building the SQL | **1.5** | Alert fatigue is why the 2026-08-07 real diary failure was invisible among eight false ones. Fixing this restores the meaning of every future alert. Cheapest high-value fix here. |
| N3 | Phase 5.4 — the 00:00 cap-reset cron missed 11 of 30 days (machine asleep) | Move the reset out of crontab into a launchd plist with `StartCalendarInterval {Hour:0, Minute:0}` — launchd fires missed calendar jobs on wake, cron does not | **0.5** | Today it is latent. The day a cap trips and the reset is skipped, the drafter goes silent for a day or more with no signal. Five other jobs already use this pattern; this is copying an existing plist. |
| N4 | Phase 6.1 — `WRITER_ENABLED` and `EDITOR_ENABLED` are decorative | Either wire them (add a check in `ai-do.sh`'s `*-writer\|*-auditor` case block, 3 lines) or delete them from `.env` and from `morning_brief.sh` | **0.5** | A safety switch that does nothing is worse than no switch, because it will be reached for in an emergency. Wiring is 3 lines in a file that already has the case block. |
| N5 | Phase 1.3 — two live `${VAR//\'/\'\'}` SQL escapes that produce `O\'\'Brien` | Replace with the `APOS="'"` idiom already used in 8 other files, at `internal-link/propose_backlinks.sh:144` and `apply_approval.sh:54` | **0.25** | Latent, not live (slugs are `[a-z0-9-]+`). But `propose_backlinks.sh` is on the deploy path and this is a 2-line copy-paste from a working file. |
| N6 | Phase 9.3 — 10 doc drifts, 3 of 4 schedule claims wrong | Correct the 6 factual claims: backlink-prospector day/hour, idea-agent weekly→daily, `ai-do.sh` 1200→1800, `log_to_audit.sh` retention, the 4 invalid `tier:` values, `content-loop/agent.yaml enabled:` | **1** | Doc-vs-reality drift is this project's stated top failure mode. These are the specific instances found. |
| N7 | Phase 10 — `content-loop` + `config-synthesizer` dead 74 days; 12 stale `fix-runner/state/run_*` dirs | Delete both agent dirs and the stale state dirs | **0.5** | Removes 2 agent directories, ~40 files, and stops them polluting every future repo-wide grep — which cost real time in this audit. |
| N8 | Phase 8.2 — `link-monitor` and `gsc-submit` shipped today with no watchdog entry | Add two entries to `watchdog/expected_schedule.yaml` | **0.25** | Do it while the agents are fresh, or they join the 20 uncovered ones. |

**NOW total: ~7.5 hours.**

### SOON — real value, more than 2 hours

| # | Observation | Change | Hours | Worth |
|---|---|---|---|---|
| S1 | Phase 4.3 — 192 sqlite CLI writers, `busy_timeout=0`, `log_to_audit.sh` failures swallowed by `\|\| true` at 43 sites | Add `PRAGMA busy_timeout=5000;` to `lib/log_to_audit.sh`, `log_token_usage.sh`, and the sqlite writers in the top-10 busiest agents | **3** | The audit trail is the input to every detector in the fleet. A silently dropped row corrupts watchdog and issues-capture in both directions. `dashboard/server.js` already does this — it is a known-good pattern in-repo. Note: I could **not** prove rows have been lost (Phase 4.3); this is insurance, priced accordingly. |
| S2 | Phase 10 — domain-hunter: ~39 Haiku calls/day, 3 per-domain loops, 1,305 rows, 0 downstream readers | Kill the agent. If the operator wants it kept, batch its three scorers the way `backlink-prospector/filter.sh:94` already batches 25 URLs per call | **1 (kill) / 4 (batch)** | Killing is the better trade at a $0-revenue bar. It is the fleet's largest capacity sink with no consumer. Batching preserves an agent nobody uses — pay 4 hours only if the operator says the digest has ever changed a decision. |
| S3 | Phase 1.7 / 10 — research-opportunity: 6,699 audit rows/60d, `pains:0` for six weeks, 1,824 open issues, `extract.sh` wired to nothing | Either drop the poll to daily and finish wiring `triage.sh`→`extract.sh`, or delete the hourly cron line | **1 (kill) / 6 (finish)** | Six weeks of zero is the answer. Finishing it is only worth 6 hours if pain-mining is actually on the roadmap; otherwise the hourly cron is pure noise generation. |
| S4 | Phase 1.5 — `ai-think.sh` lacks cap detection, parse-fail classification, and any audit row on failure | Port the ~40 lines from `ai-cheap.sh:56-100` | **1.5** | Zero agents route through it **today**, so this is pre-emptive. Worth doing because `lib/run_agent.sh:23` still routes `tier: think` there and two `agent.yaml` files declare it — the next agent that uses the tier inherits a silent-failure path. |
| S5 | Phase 6.6 — backups written nightly, restore **never tested** | Restore `store/aiteam-backup.db` to a scratch path, run the schema + a row-count check against live, write the procedure down | **2** | "Untested backup" and "no backup" differ only in how confident you are while losing the data. Two hours buys the difference. |
| S6 | Phase 7.5 / D114 — 2,911 open issues, 16 ever resolved, 91% from two known causes | Add auto-resolution (close issues whose agent has since succeeded) and per-source suppression for known-disabled sources like Reddit | **3** | The `/issues` panel is currently unreadable, which means the dashboard's one diagnostic surface is dead weight. |
| S7 | Phase 6.3 — 14 unattended `--dangerously-skip-permissions` writer/auditor calls | Add `AI_DO_TOOLS="Read Write Edit Glob Grep"` to the writer/auditor call sites — `ai-do.sh:73-78` already supports the array-safe pass-through | **2** | Converts a prompt-instruction boundary into a real one, using a mechanism the file already has and that scout/majordomo already prove works. Needs testing that the writer can still save drafts. |

**SOON total: ~13.5 hours** (taking the kill options for S2 and S3).

### NOT WORTH IT — named so they stop being reconsidered

- **Adding `set -euo pipefail` to the 106 scripts that lack it.** Most
  deliberately use `set -uo pipefail` because `-e` would abort before their own
  audit-logging runs. Retrofitting `-e` fleet-wide would break error reporting to
  buy theoretical robustness. The two `run-batch.sh` files with *no* `set` at all
  are a genuine gap, but they are 1,800 and 1,500 lines with no tests — adding
  `-e` there is a rewrite, not a fix.
- **Log rotation.** The largest log is 17.9 MB and nothing exceeds 50 MB. At
  current growth `reader.log` hits 50 MB in ~6 months. Revisit then.
- **Pruning `audit_log` to the documented 90 days.** 30,510 rows in a 25 MB DB.
  SQLite will not notice for years, and the history is the only forensic record
  this audit had. Fix the *comment* (N6), not the data.
- **Batching `asbestos-source-verifier`'s 582 Haiku calls.** Each adjudication
  needs a different source passage in context. Batching them would destroy the
  gate's correctness to save $16/30d of notional capacity. **Do not touch this.**
- **Building a restore *automation*.** S5 is a one-time manual test and a written
  procedure. Automating restore for a single-machine hobby fleet is
  architecture-astronautics.
- **Fixing the `hive_mind` / `warroom_transcript` / `source_verifications`
  write-only tables by adding readers.** They have no readers because nothing
  needs them. Drop `warroom_transcript`, `memories`, `sessions` (N7-adjacent);
  leave `source_verifications` as the forensic archive it already is.
- **A monthly cost cap to enforce `MONTHLY_API_BUDGET_USD=200`.** Real money
  spent is $0. Capacity is the constraint, and the daily soft/hard caps already
  address it. Either delete the variable (N4) or leave it; do not build
  enforcement for a dollar figure that is not being charged.
- **Migrating the remaining cron jobs to launchd.** Only the 00:00 reset (N3)
  suffers from the sleep problem; every other cron job runs at a time the machine
  is awake and shows a full execution record. Wholesale migration is churn.


---

## CLOSURE NOTE — FIX LANE, 2026-08-09 (appended by the implementing session)

Written by the session that acted on this audit. Findings below either confirm,
correct, or narrow what is above. The audit body is left exactly as written.

### F09 — the editor gate. The audit's framing was wrong; the conclusion is (a)+(d).

`editor/score.sh` has **one caller in the entire fleet**: `ship-to-site/ship.sh:295`.
`geo-optimizer/score.sh` has **two**: `ship.sh:245` *and*
`asbestos-contractors/content/run-batch.sh:1549`. Comparing "editor last scored
07-19" against "geo last scored 07-31" compared two different call sites.

Worse, the 07-31 geo rows are not pipeline traffic at all. They are at 13:25 and
14:07–14:09 on 2026-07-31, on already-scored slugs, interleaved with
`geo_parse_failed` retries — manual verification runs of the `f466673` retry fix
committed that same afternoon. The asymmetry the audit spotted is an artifact of
manual runs and pipeline runs being indistinguishable in `audit_log`.

What actually happened, from `audit_log` + `auditor_verdicts`:

| when | what | gate |
|---|---|---|
| 07-19 09:14 | run-batch geo-scores `asbestos-abatement-process` 3.50 pass | geo ran |
| 07-19 20:00 | preview_ping fires editor; `editor_parse_failed` | editor ran |
| 07-19 23:00 | deploy_batch fires editor; 3.60 **pass** | editor ran |
| 07-19 23:03 | `asbestos-abatement-process` ships | **fully gated** |
| 07-20 → 07-28 | run-batch runs daily; every guide fails `audit_guide` | nothing reaches ship |
| 07-28 23:03 | `atera-alternatives` (**ssg**) ships | both gates skipped |
| 07-29 → 08-07 | no run-batch content rows at all | — |
| 08-02 23:03 | `helpscout-alternatives` (**ssg**) ships | both gates skipped |
| 08-08 23:47 | run-batch, `srcver_fail` on `deliberately-failing-draft` | test, not content |

So: **(a) for asbestos** — the gate is idle because nothing asbestos has cleared
`audit_guide` into `approved/` since 07-19, and idle is correct. **(d) for SSG** —
`ship-to-site/config/ssg.yaml` carries `skip_geo_gate: true` and
`skip_editor_gate: true`, landed in `dd8c867` (2026-05-27) with a stated reason:
the asbestos-tuned rubrics punish SSG rich-schema content and score it
non-deterministically. Not (b), not (c). It is a config flag, and it is
deliberate and documented.

Independently corroborated by D110's own 2026-07-31 scope correction, which
already said geo went quiet "because `ship.sh` only invokes it for slugs reaching
`approved/guides/`, and nothing new arrived after 07-19". That correction applies
to the editor identically and was available before this audit was written.

The flag was **not** flipped. Doing so would block SSG shipping outright;
retargeting the rubrics is the actual remedy and belongs to whoever owns the SSG
rubrics.

**What was fixed instead**, because it is the real defect: a config-skipped gate
produced one line on stdout and nothing durable — no audit row, no field on the
ship row. A slug that shipped ungated was byte-identical, in `audit_log`, to one
that passed both gates. That is the same failure-equals-success fold this audit
is about, and it is why a 21-day silence needed a human to notice and still got
read wrong. `ship.sh` now writes `geo_gate_skipped` / `editor_gate_skipped` rows
and stamps `gates:{geo,editor}` on every `ship_to_site` success row.

**Ungated remediation queue** — all SSG, zero `geo-v1`/`editor-v1` verdicts, all
after the flags landed. Listed, not remediated:

```
voicenation-vs-patlive   2026-06-11
ramp-alternatives        2026-06-26
front-alternatives       2026-07-02
ramp-vs-expensify        2026-07-19
atera-alternatives       2026-07-28
helpscout-alternatives   2026-08-02
```

### F02 / U1 — the loop the audit could not run was run. Neither candidate reproduced.

25 consecutive `watchdog.sh --dry-run` passes with per-probe tracing: **275
`audit_log` probes, every one rc=0, every `in_list` well-formed and singular,
zero empty**. All 25 summaries identical (`healthy=12 missing=0 unknown=0`), and
`state.json` md5 unchanged throughout — which also confirms `--dry-run` is
side-effect-free as documented.

So candidate **(b)**, empty `in_list` → `action IN ()`, which §8.1 called the
better explanation, is **not supported on this machine**. Candidate (a),
`SQLITE_BUSY`, did not appear either. **The live trigger for the 254 orchestrator
incidents remains unidentified.**

An A/B truth table against the old and new probe bodies is what settles the
defect regardless:

| case | OLD | NEW |
|---|---|---|
| healthy DB, real actor | healthy | healthy |
| agent genuinely never ran | **missing → ALERT** | **missing → ALERT** |
| empty `expected_actions` | **missing → ALERT** | unknown → no alert |
| corrupt DB file | **missing → ALERT** | unknown → no alert |
| DB under EXCLUSIVE lock | **missing → ALERT** | unknown → no alert |

Caveat stated plainly: the lock case used a rollback-journal copy. The live DB is
WAL, where readers do not block on writers, so rc=5 demonstrates the mechanism
rather than proving it is the live trigger. Whatever the third candidate turns
out to be, the fix makes it harmless in the same way.

### F26 / N7 — HALTED on the two agent directories. Reported, not actioned.

The 12 stale `fix-runner/state/run_*` dirs were in fact **5** dirs / 165 files /
56 generated `.sh` (not 12 / 34 as recorded). All gitignored and untracked, no
inbound references, removed; repo-wide greps are clean.

`content-loop/` and `config-synthesizer/` were **not** deleted. The lane's rule
was to halt on any inbound reference and report, and there is one:

```
issues-capture/capture.sh:64
  CORE_PIPELINE = _env_set("CORE_PIPELINE",
    "writer,auditor,ship-to-site,assignment-drafter,config-synthesizer,orchestrator,content-loop")
```

Assessment, for whoever decides: this is **not a caller**. `CORE_PIPELINE` is a
severity-classification token list — `is_core()` substring-matches `actor_id` to
promote a `*_failed` row from soft to hard. Both tokens already match nothing,
since neither agent has emitted an `audit_log` row since 2026-05-27. `HARD_ACTIONS`
carries a third dead token, `config_synth_failed`. Deleting the directories cannot
break `capture.sh`; it would only leave three names in a filter that can never
fire. If the deletion is approved, those three tokens should go in the same change
or it re-creates exactly the doc-drift F19 just cleared.

### F10 / Phase 10 / U4 — domain-hunter. The KILL verdict was wrong.

The cron line was paused on the strength of "1,305 rows in `domain_candidates`
with **zero downstream readers**", and **reverted the same day** on operator
instruction. The agent is active, daily 06:30, unchanged.

The finding is wrong in a way worth recording, because the same mistake is
available about several other agents in this fleet. The digest **is** consumed —
by the operator, over Telegram. That consumption leaves no trace anywhere on this
machine: no `outreach_status` transition, no read receipt, no row written back.
A code-level reader search therefore returns zero readers whether the digest is
read every morning or never opened. The two cases are indistinguishable from the
filesystem and the database, and absence of evidence was reported as evidence of
absence.

This audit already knew that. §3/U4 lists exactly this question as UNVERIFIED and
says it needs "two questions to the operator", whose answers "decide ~$12/30 d of
notional capacity and 3 agent directories". Phase 10 then ruled KILL on the
reader count regardless. **The caveat and the verdict contradicted each other and
the verdict won** — that is the process defect here, not the arithmetic.

Generalisation: "zero downstream readers" is a valid finding about a *table* and
an invalid finding about a *digest*. For any agent whose output terminates in a
Telegram message, reader count is not evidence of value and the operator is the
only source of truth. The same caution applies to the other two agents U4 covers,
`tg-monitor` and `backlink-prospector`, whose PAUSE recommendations rest on the
identical unmeasurable.

The ~39 Haiku calls/day are real and not disputed. That is a batching question,
not a keep-or-kill question. Batching was deliberately not done: each scorer
needs a different per-domain context, so it changes what the model sees and needs
its own before/after digest comparison rather than being bolted onto this lane.

### Corrections to the audit's own numbers

- **F26**: 5 stale `run_*` dirs, not 12; 56 generated `.sh`, not 34.
- **F19 #5**: `fix-runner` and `ship-to-site` declaring `tier: script` was
  recorded as an invalid value. The declaration is honest — those agents call no
  model. `lib/run_agent.sh` was taught to recognise and refuse `script` rather
  than the yaml being changed to a tier that would have been a lie.
- **F10 / Phase 10**: the domain-hunter KILL verdict is withdrawn — see above.

### Out of scope this lane, untouched and still open

F01 (SSG source-verification gate — sister lane), F06 (`busy_timeout`), F07
(research-opportunity), F13 (`AI_DO_TOOLS`), F17 (restore test), F18 (issues
auto-resolve). F15 is now tracked as D110 half B.
