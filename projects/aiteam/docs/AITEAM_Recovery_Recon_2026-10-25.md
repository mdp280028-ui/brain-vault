# AITEAM Recovery Recon — 2026-10-25

**Mode:** READ-ONLY. Writes performed: this report, one Telegram test message, one model auth probe. Nothing else.
**Auditor:** Claude Code (Opus 5), recon session, 2026-10-25 ~20:00 PDT.
**Ground truth:** `last reboot`, `pmset -g log`, crontab, launchctl, filesystem, git log, `~/store/aiteam.db`, live HTTP.
Docs were read only to detect drift.

## SECTION 8 — VERDICT AND FIX LIST

### 8.1 One line

**The fleet is ALIVE and already running — but it has produced nothing since
2026-08-13, and the outage is only half the reason.**

### 8.2 The thing to understand before anything else

The brief's premise is off by 25 days. **The box was not unplugged on 2026-08-09.
It ran until 2026-09-03 19:06 and was off for 52 days.** More importantly, it
spent the 21 days from **2026-08-14 to 2026-09-03 totally broken while fully
powered on**, and nobody knew, because:

1. Every outbound socket failed with `Can't assign requested address` — curl,
   ssh, python, and the `claude` CLI, all four verified (§2.0);
2. `notify.sh` *is* the only alerting path for all 20 callers, so it failed too
   — **`notify_sent 0/day, notify_failed 17/day` for 21 straight days**;
3. The fleet therefore logged ~69 failed model calls a day into a silent box.

`token_usage` is the single cleanest artifact in this report:
```
2026-10-25 20:10:31 | recon | claude-haiku-5-5            <- this recon's probe
2026-08-14 07:05:39 | shell | claude-haiku-4-5-20251001   <- the one before it
```
**Both halves are now repaired** (CLI reinstalled today 20:09; network verified
live on four stacks), and the fleet self-started cleanly at 19:44.

### 8.3 Must-fix before tomorrow 09:00

## **Nothing.**

That is a finding, not a shrug. Everything that fires in the next 24 hours is
either safe or a proven no-op, and I checked each one rather than assuming:

- **23:00 deploy ships nothing.** The 20:00 preview ping already ran the exact
  same query: 58 candidates, 58 skipped, zero `[would-ship]`. It will log
  `deploy_batch_empty` and stay silent (§2.6).
- **No bad content can ship.** Three independent gates; two active right now
  (§2.7). Answer to the fork question: **NO**.
- **No data is at risk.** Backups fire at 23:30 to a mounted volume with 9.1 TiB
  free; the 20 unpushed commits in each of `~/agents` and `~/brain` push
  themselves at 23:50/23:55 now that ssh works.
- **No issue-queue spam.** `notify_telegram` high-water is already caught up at
  audit id 39773 and reporting `sent 0/0`.

**Three things worth eyes, none blocking:**

| | What | When | Why |
|---|---|---|---|
| W1 | `assignment-drafter/state/failed.log` | before 09:00 | `coupa-alternatives` has failed **21 consecutive days**; nothing de-prioritises it, so it will burn the run again if it fails for any new reason |
| W2 | 07:00 `market/scribe/poll.sh` | 07:00 | First real test of the network fix — 147 consecutive `URLError [Errno 49]` failures |
| W3 | 11:00 `research-opportunity/poll.sh` | 11:00 | **842 untriaged rows** backed up; 60 Haiku + Sonnet extract. Largest single spend of the first day back |

### 8.4 FIX LIST

Ranked: traffic/revenue first, then data safety, then reliability.
Minutes are implementation only.

| # | Problem | Class | Evidence | Fix in one line | Who | Min | Repo |
|---|---|---|---|---|---|---|---|
| **1** | **Asbestos drafter queue is empty — 0 pending of 67 lines.** Asbestos cannot produce a guide no matter what is fixed | NEVER-WORKED (undocumented) | §5.1; `parse_queue.sh` → 0; 45 ✓, 2 ✗, 19 `#`, 1 blank | Add new `slug\|primary_keyword` rows to `asbestos-contractors/content/asbestos/drafter_queue.txt` | **MARK** | 30 | asbestos-contractors |
| **2** | **70 live URLs never submitted to Google**; oldest 201 days | NEVER-WORKED (by design) | §5.5; `gsc_submission_queue` 70 pending / 0 submitted | Create GSC credentials and wire `gsc-submit`; or submit by hand in Search Console | **MARK** | 45 | gsc-submit |
| **3** | **SSG drafter stuck on `coupa-alternatives` for 21 runs** — queue head never advances | REGRESSION | §2.8 #29; `drafter.log` same 4-line block ×21; `assignment_draft_failed {"reason":"sonnet"}` | Retry once now that the network is up; if it fails again, skip-list the slug so the other 47 can run | CC | 20 | agents |
| **4** | **SSG source gate has NEVER returned PASS** — `srcver_passed` has zero rows ever, on either site; 58 invocations, 58 failures | UNKNOWN | §6 #10; `ssg-verify-sources srcver_fail ×58`, `srcver_error ×1`, `run-batch-ssg srcver_escalated ×3` | Run one known-good approved guide through the gate and determine whether PASS is reachable at all | CC | 60 | ssg-content |
| **5** | **asbestos `INTENT1` still literal-match; SSG's stem-tolerant fix never ported** — guides fail the mechanical gate on phrasing, not substance | NEVER-WORKED | §6 #11; asbestos `audit_guide.py` v3.4 **:215** vs ssg v4.4 **:306-309**; `[FAIL] INTENT1 0/4` across 6+ feedback files | Port the `_stem()` branch from ssg `audit_guide.py:306-309` into asbestos `:215` | CC | 20 | asbestos-contractors |
| **6** | **3 stray untracked `.md` files block all 34 asbestos deploy candidates** via `site_repo_dirty` | REGRESSION | §2.7, §3.2; `?? LANE3_PROGRESS.md`, `?? WAVE2_LANEA_PROGRESS.md`, `?? WAVE2_LANEB_PROGRESS.md` | Commit or delete them — **but read §2.7 first**, they are currently a load-bearing accident | CC | 5 | asbestoshq-site |
| **7** | **D121 — `ship.sh --dry-run` is not dry**: fires both Sonnet gates and writes `auditor_verdicts` rows | REGRESSION | `DEFERRED.md:1867`; gates `ship.sh:239-296`/`:297-348` sit **above** the dry-run exit at `:349` | Move the `DRY_RUN` short-circuit above the gate blocks | CC | 30 | agents |
| **8** | **Pipeline/site fork: 24 of 41 approved guides differ** — the pipeline still holds the pre-safety-repair text | REGRESSION | §5.7; byte-diff identical=14 / DIFFERENT=24 / not-on-site=3; matches Lanes A+B exactly | Back-port the 24 repaired site JSONs into `content/asbestos/approved/guides/` | CC | 40 | asbestos-contractors |
| **9** | **40 unpushed commits (20 `~/agents` + 20 `~/brain`)**; remote still at `1f2eb16` (2026-08-09) | REGRESSION | §3.1; `git ls-remote` vs `git log @{u}..` | **Self-resolving tonight at 23:50/23:55** — just verify tomorrow | CC | 2 | agents, brain |
| **10** | **Backup restore has still never been tested** | NEVER-WORKED | §4.3; audit F17 unchanged; no restore script exists | Restore `mini-backup/store/aiteam-backup.db` to scratch, diff schema + row counts, write the procedure | CC | 45 | agents |
| **11** | **`agent_issues` at 5,189 open** (120 hard + 5,069 soft), up from D114's 2,894 | REGRESSION | §6 #12; `DEFERRED.md:1708` | Bulk-resolve the 08-14→09-03 storm rows as one root cause; add auto-aging for soft | CC | 40 | agents |
| **12** | **The fleet's only alert path has no out-of-band backstop.** `notify.sh` failed 17×/day for 21 days and nothing else could say so — this is *why* the 21-day blackout was invisible | NEVER-WORKED | §2.0, §8.2; `notify_failed 17/day`, `notify_sent 0/day`, 2026-08-14→09-03 | Add a second channel (email/SMS/file+launchd) that fires when `notify_failed` outnumbers `notify_sent` over a window | CC | 60 | agents |
| **13** | **Network `EADDRNOTAVAIL` root cause unknown** and nothing would capture it if it recurs | UNKNOWN | §2.0; 4 stacks, same errno 49, no interface state recorded | On N consecutive `notify_failed`, dump `ifconfig -a`, `netstat -an \| wc -l`, `scutil --nwi`, `sysctl net.inet.ip.portrange.*` to a file | CC | 30 | agents |
| **14** | **`claude` CLI can self-destruct via a partial update** ("native binary not installed") and no agent detects it | REGRESSION | §2.5b; `librarian/run.log:299`; `claude_call_failed` with empty `payload_excerpt`, 2026-08-13 02:00:00 | Add a daily `claude --version` + one-token probe to `majordomo/check.sh` | CC | 20 | agents |
| **15** | **market-briefer delivers a brief even when the whole upstream chain is dead** — it "succeeded" every day for 21 days on no input | NEVER-WORKED | §4 row 21; audit **F03 still open**; `briefer_run ×102` to 09-03 while `scribe_poll_channel` stopped 08-13 | Make `briefer` refuse to send when zero new analyst rows exist in the window | CC | 30 | agents |
| **16** | **librarian + orchestrator-diary silent since 08-12 / 08-13** — explained, but unverified post-fix | REGRESSION | §2.5b, §4 rows 14 & 23 | Watch 23:45 (diary) and 02:00 (librarian) tonight; both are low blast radius | CC | 10 | agents |
| **17** | **editor + geo gates dark since 2026-08-09** — no verdict in 77 days | REGRESSION | §6 #7/#8 | No fix needed — they are dark because `run-batch` is stopped. **Supersede audit F09**, which blamed the editor | CC | 5 | agents |
| **18** | **`majordomo/check.out.log` is 1.3 MB, append-only, never rotated** | NEVER-WORKED | §2.2; `ls -la majordomo/check.out.log` | Truncate on run, or add a size cap | CC | 10 | agents |
| **19** | **`auto-refresh` still unwired** — no cron line, no caller | NEVER-WORKED (deliberate) | §6 #5; commit `0db99f1`; depends on GSC data that does not exist | Leave it. Blocked behind fix #2 | CC | 0 | agents |
| **20** | **`/Volumes/ActiveSSD` is empty** (1.0 MiB of 3.6 TiB, scaffolding dirs dated May 13); **no volume named `AgentSSD` or `WDArchive` exists** | UNKNOWN | §1.3; `diskutil list`; `grep -r WDArchive ~/agents` → no hits | Confirm whether ActiveSSD is meant to hold anything; nothing currently breaks | **MARK** | 5 | — |
| **21** | **`HANDOFF.md` is 148 days stale** and wrong on 6 specific claims; `PRIME.md` 161 days | REGRESSION | §7.1, §7.2 | Rewrite `HANDOFF.md` from this report | CC | 30 | brain |
| **22** | **`lib/ai-cheap.sh:5` header still says `AI_TIMEOUT_SEC` default 1200**; code is 1800 | NEVER-WORKED | §7.2 #8; audit F19 fixed the same drift in `ai-do.sh` via `4ca6d79` | One-line comment fix | CC | 2 | agents |

### 8.5 MARK-ONLY list

| | Task | Why only Mark can do it |
|---|---|---|
| M1 | **Refill the asbestos drafter queue** | Keyword and topic selection is a business judgement, not a lookup (fix #1) |
| M2 | **Create Google Search Console credentials** | Requires a Google login (fix #2) |
| M3 | **Decide what `/Volumes/ActiveSSD` is for** — and whether `AgentSSD`/`WDArchive` are drives that should be plugged in | Physical hardware + intent (fix #20) |
| M4 | **Decide whether Wave 2 Lanes C and D ever existed** | No artifact on the machine names them; only recollection can settle it (§5.6) |
| M5 | **Judge whether the SSG source gate is correct-but-strict or broken** | Needs an editorial call on whether the content really is unsourced (fix #4) |
| M6 | **Confirm the market / domain-hunter / backlink agents are still wanted** | `domain-hunter` has sent 0 useful digests since 2026-05-24 but spends Haiku daily; the bot has had no conversation in 80 days. Audit U4, still unanswered |

### 8.6 Agents safe to resume as-is vs needing a fix first

**SAFE AS-IS — already running or will recover unaided (24):**
`dashboard` · `grammy-bot` · `watchdog` (+digest, +heartbeat) · `issues-capture`
(+notify, +digest) · `tg-monitor` (reader + analyzer) · `ship-to-site`
(digest / preview / deploy) · `majordomo` (brief + check) · `keyword-registry` ·
`backup-rsync` · `cost-cap-reset` · `backlink-prospector` · `notes-agent` ·
`link-monitor` · `gsc-submit` (reports only) · `internal-link` · `scout` ·
`agents/brain/content autocommit`

**SAFE, WITH A COST OR QUALITY CAVEAT (4):**
- `research-opportunity` — 842-row backlog, biggest first-day spend
- `domain-hunter` — runs clean, has produced nothing since 2026-05-24
- `idea-agent` — 4 non-LLM phases healthy; only synth was failing
- `market/curator`, `market/scribe`, `market/analyst` — pure network casualties

**NEEDS A FIX FIRST (3):**
- **`assignment-drafter`** — stuck on `coupa-alternatives`, 21 consecutive failures (fix #3)
- **`market/briefer`** — will confidently brief on nothing (fix #15)
- **`run-batch` / `run-batch-ssg`** (and with them `editor` + `geo-optimizer`) — no path to run until the drafter fires again; the SSG source gate has never passed (fix #4)

**NO PATH TO RUN / DELIBERATELY OFF (3):**
`auto-refresh` (unwired by design) · `fix-runner` (vestigial) · `editor` +
`geo-optimizer` (only `run-batch` calls them)

### 8.7 Lane-collision note

A second CC session (**pid 8644**) is live and writing
`~/projects/ssg-content/docs/AITEAM_SSG_Content_Lane_2026-10-25.md` (mtime
20:21). Its header names three parallel lanes: **SSG-CONTENT-1** (sole editor of
`ssg-content`), **RECOVERY-RECON-2** (this report, read-only), and
**FLEET-AUDIT-55** (declared *"sole editor of `~/agents`, `~/brain/projects/aiteam`"*).

**This report lives in `~/brain/projects/aiteam/docs/`, which FLEET-AUDIT-55
claims as its own.** The operator assigned that path, so it was written there —
but any FLEET-AUDIT-55 session should treat this file as read-only input.
Fix-list repo assignments above are deliberately split by repo so the lanes do
not collide: **#1/#5/#8 → asbestos-contractors · #4 → ssg-content · #6 →
asbestoshq-site · #21 → brain · the rest → agents.**

---

## SECTION 1 — Box health

### 1.1 The actual outage window (premise correction)

**The brief says "unplugged from about 2026-08-09". That is wrong by 25 days.
The box ran normally until 2026-09-03 19:06 and was off for 52 days.**

| Event | Evidence |
|---|---|
| Power-off | `last reboot` → `shutdown time   Thu Sep  3 19:06` |
| Last audit_log row before gap | `2026-09-03 19:06:30 \| bot \| shutdown` |
| Power-on | `last reboot` → `reboot time   Sun Oct 25 19:43`; `sysctl -n kern.boottime` → `Sun Oct 25 19:44:03 2026` |
| First audit_log row after gap | `2026-10-25 19:45:04 \| bot \| startup` |
| Gap length | 52 days 0 h 39 min |

Daily `audit_log` row counts confirm a clean stop, not a decay:

```
2026-10-25|124      <- today, since 19:45
2026-09-03|230      <- partial day, stopped 19:06
2026-09-02|367
2026-09-01|368
2026-08-31|367
2026-08-30|373
2026-08-29|367
2026-08-28|366
```

~367 rows/day right up to the shutdown. **Nothing was degrading before power-off.**
The "about 2026-08-09" date in the brief is the date of the last big audit doc
(`AITEAM_Agent_Audit_2026-08-09.md`), not the date the box went down.

**Consequence for the whole report:** every "stale since August" claim must be
re-based against 2026-09-03, not 2026-08-09. An agent last seen 2026-08-20 was
already silently dead for **14 live days** before the box went off — that is a
real bug, not outage. An agent last seen 2026-09-02/03 is outage-only.

### 1.2 The fleet is ALREADY RUNNING

`uptime` → `20:09 up 25 mins`. In those 25 minutes the fleet self-started and
has written 124 audit rows, including a completed 20:00 `ship-to-site`
preview-ping cycle:

```
2026-10-25 20:00:10|ship-to-site|preview_ping_sent
2026-10-25 20:00:10|notify|notify_sent
2026-10-25 20:00:10|ship-to-site|ship_to_site_skipped   (x many)
2026-10-25 19:45:04|bot|startup
2026-10-25 19:45:04|watchdog|watchdog_alert
```

This is not a dormant box awaiting a decision. **Scheduled work is firing right
now and the 23:00 deploy is ~3 h away.** See Section 2.

### 1.3 Disk and volumes

```
/dev/disk3s1s1   460Gi   11Gi used   336Gi avail    4%   /
/dev/disk3s5     460Gi  103Gi used   336Gi avail   24%   /System/Volumes/Data
/dev/disk7s1     3.6Ti  1.0Mi used   3.6Ti avail    1%   /Volumes/ActiveSSD
/dev/disk5s1     9.1Ti  366Mi used   9.1Ti avail    1%   /Volumes/Backups
/dev/disk8s2     3.6Ti   70Gi used   3.6Ti avail    2%   /Volumes/TimeMachine
```

336 GiB free on `/` (73%). **No disk pressure. Stop-condition not met.**

| Volume named in brief | Present? | Evidence |
|---|---|---|
| **AgentSSD** | **NOT PRESENT under that name** | `diskutil list` has no `AgentSSD`. The 3.6 Ti external is named **ActiveSSD** (`disk7s1`). Likely the same physical disk under the name the operator misremembered. |
| **WDArchive** | **NOT PRESENT** — UNKNOWN whether it should be | `diskutil list` shows no `WDArchive`. No script in `~/agents` references it (`grep -r WDArchive ~/agents` → no hits). Not referenced by the backup path, so nothing is currently broken by its absence. |
| **Backups** | **MOUNTED AND HEALTHY** | `/Volumes/Backups/mini-backup`, 298 MB, 8 source dirs. This is the one the nightly backup needs (`scripts/backup-rsync.sh:16` `BACKUP_VOL="${BACKUP_VOL:-/Volumes/Backups}"`). |
| TimeMachine | Mounted, 70 GiB | Not referenced by any agent script. |

**ActiveSSD is effectively empty** — 1.0 MiB used of 3.6 TiB, containing only
`cache/ logs/ platforms/ templates/` all dated May 13 12:18 and all empty. No
agent script writes there. Flagged as UNKNOWN-intent in Section 8, not as a
failure: per operator memory, models live on the internal SSD and the external
is for bulk rsync archives that may simply never have been populated.

**Backups last success: 2026-09-02 23:30** (`/Volumes/Backups/mini-backup/backup.log`):
```
2026-09-01 23:30:29 | sources=8 | total=296M | duration=24s
2026-09-02 23:30:11 | sources=8 | total=298M | duration=6s
```
No 2026-09-03 line — the box went down at 19:06, before the 23:30 job. No
failure line anywhere in the tail. The backup was healthy at power-off and the
destination volume is mounted now, so tonight's 23:30 run should work.
`~/store/flags/last_backup_ok` is dated Sep 2 23:30, consistent.

### 1.4 Memory and processes

```
System-wide memory free percentage: 72%
Pageouts: 11625
```
No memory pressure.

Top RSS: `WebKit.WebContent 1.5 GB`, `claude 583 MB` (this session, pid 8532),
`WebKit 725 MB`, `claude 216 MB` (pid 8644), `Terminal 412 MB`. Nothing
abnormal; the heavy processes are Safari/WebKit, not agents.

**Other Claude Code sessions: 1** (excluding this one).
```
8532   07:12  claude --dangerously-skip-permissions                       <- THIS session
8644   01:35  claude --dangerously-skip-permissions --model claude-opus-5-5  <- OTHER session
```
A second CC window is live and started ~1.5 min before this recon. **Anything
this report says about repo state can be changed underneath it by pid 8644.**
Treat Section 3 as a snapshot.

### 1.5 Versions — TCC grant intact

| Thing | Value | Changed since 2026-09-03? |
|---|---|---|
| `claude --version` | `2.1.310 (Claude Code)` | **YES — and it self-updated during this session.** `/opt/homebrew/bin/claude` symlink + `claude.exe` both mtime `Oct 25 20:09`. |
| `which claude` | `/opt/homebrew/bin/claude` → `../lib/node_modules/@anthropic-ai/claude-code/bin/claude.exe` | path unchanged |
| `node --version` | `v26.0.0` | **NO** |
| `readlink -f node` | `/opt/homebrew/Cellar/node/26.0.0/bin/node` | **NO** |
| `brew list --versions node` | `node 26.0.0` | **NO** — Cellar dir mtime `May 13 18:51`, single version installed |

**The node TCC grant is SAFE.** It is pinned to `/opt/homebrew/bin/node` →
`Cellar/node/26.0.0`, and node has not moved since May 13. Only `claude`
updated, and the dashboard plist pins node, not claude (per the Aug-09 audit
U3). No action needed on TCC. (Remains UNKNOWN whether the grant itself is
still present — `TCC.db` is SIP-protected and this session has no Full Disk
Access. Settle it by watching the dashboard serve a page, or by reading TCC.db
under FDA.)

### 1.6 Model auth — ALIVE

```
$ cd ~/agents && AI_CALLER=recon bash lib/ai-cheap.sh "reply OK" recon
RC=0
OK
```
Exit 0, clean "OK", empty stderr. **Haiku tier authenticates and returns.**
Kill switches permit it: `SYSTEM_PAUSED=false`, `LLM_SPAWN_ENABLED=true`.
Every agent's model path is therefore live, not dead.

### 1.7 Telegram — ALIVE

```
$ cd ~/agents && NOTIFY_DEBUG=1 bash lib/notify.sh "[RECON TEST] 2026-10-25 state check, no reply needed"
RC=0
[notify] attempt=1 rc=0 http='200' ok=true message_id=1196 err=''
```
**rc 0, HTTP 200, message_id 1196.** Delivery path is fully healthy on the
first attempt. `TELEGRAM_OUTBOUND_ENABLED=true`.

### 1.8 Kill-switch state at recon time (`config/.env`)

```
SYSTEM_PAUSED=false            LLM_SPAWN_ENABLED=true
PUBLISHER_DEPLOY_ENABLED=true  <-- 23:00 deploy is ARMED
WRITER_ENABLED=true            EDITOR_ENABLED=true        LIBRARIAN_ENABLED=true
TELEGRAM_OUTBOUND_ENABLED=true
COST_CAP_SOFT_USD=25           COST_CAP_HARD_USD=50
RESEARCH_OPPORTUNITY_ENABLED=true
REDDIT_ENABLED=false  (disabled 2026-06-09, F5 — 403s)
STACKEXCHANGE_ENABLED=true     YOUTUBE_ENABLED=true
MAJORDOMO_ENABLED=true         MAJORDOMO_SCOUT_ENABLED=true   MAJORDOMO_COST_CAP_USD=20
MAJORDOMO_PLAYBOOKS_ENABLED=true   BOB_ENABLED=true
```
No pause flags set: `~/store/flags/` contains only `last_backup_ok`.
**Nothing is holding the fleet back. It is running wide open.**

---

## SECTION 2 — Schedulers and the next 24 hours

### 2.0 THE HEADLINE: the box went network-dead on 2026-08-14, 21 days BEFORE power-off

This is the single most important finding in the report and it is not an
outage artifact. **There are two overlapping root causes — a broken `claude`
CLI install (§2.5b) and a total outbound-network failure (below). Both are
repaired as of today; neither was ever diagnosed at the time, because the
alerting channel died with them.**

Every outbound socket on the machine started failing with
**`Can't assign requested address` (EADDRNOTAVAIL / errno 49)** in the early
hours of 2026-08-14 and never recovered before shutdown.

Four independent subsystems, four different network stacks, same error, same day:

| Subsystem | Evidence | First failure |
|---|---|---|
| **curl** (Telegram delivery) | `audit_log notify_failed` → `{"http_code":"000","error":"transport_failed_after_3_tries"}` | 2026-08-14 03:15:14 |
| **python urllib** (YouTube polls) | `scribe_poll_channel_failed` → `{"status":"api_error","reason":"URLError: <urlopen error [Errno 49] Can't assign requested address>"}` | 2026-08-14 07:00:01 |
| **ssh** (git push) | `scripts/agents-autocommit.log` → `ssh: connect to host github.com port 22: Can't assign requested address` | 2026-08-14 23:50 |
| **claude CLI** (all model calls) | `claude_call_failed` → `{"is_error":true,"duration_api_ms":0,...,"input_tokens":0,"output_tokens":0}` — zero API time, zero tokens: it never reached the API | 2026-08-14 |

**The exact boundary:**
```
2026-08-14 00:00:04  notify_sent    http_code 200   <- LAST SUCCESSFUL OUTBOUND
2026-08-14 03:15:14  notify_failed  http_code 000   <- FIRST FAILURE
   ... 100% failure, every day, for 21 days ...
2026-09-03 19:06:30  bot shutdown
```

**Daily totals through the blackout (every day identical):**
```
notify:           notify_sent 0/day, notify_failed 17/day  (Aug 14 -> Sep 3)
claude calls:     claude_call_failed ~69/day               (Aug 13: 2 -> Aug 14: 70)
youtube polls:    scribe_poll_channel_failed 7/day         (all 7 channels)
git push:         "commit ok, push failed (offline?)" nightly
```

**`token_usage` is the cleanest proof of all:**
```
2026-10-25 20:10:31 | recon | claude-haiku-5-5            <- this recon's probe
2026-08-14 07:05:39 | shell | claude-haiku-4-5-20251001   <- the one before it
```
**Not one successful model call happened between 2026-08-14 07:05 and this
recon.** Daily spend went `Aug 13: $12.11 / 146 calls` → `Aug 14: $0.04 / 1 call`
→ `$0.00/day` for 20 days.

**Why nobody noticed:** the alerting channel died in the same instant as
everything else. `notify.sh` is the fleet's single delivery path for all 20
callers, and it went to 100% `http_code 000`. **The operator received zero
Telegram messages from the fleet between 2026-08-14 00:00 and 2026-09-03.**
The fleet screamed 17 times a day for three weeks into a dead socket.

**Is it fixed now?** Yes, on all four stacks, verified live this session:

| Stack | Probe | Result |
|---|---|---|
| curl | `lib/notify.sh` | **rc 0, http 200, message_id 1196** |
| claude CLI | `lib/ai-cheap.sh "reply OK"` | **rc 0, "OK"** |
| ssh | `git ls-remote --heads origin` (read-only) | **rc 0** — returned `1f2eb16 refs/heads/main` |
| https (sites) | `majordomo/check.sh` 20:00 run | **asbestoshq.com 200, smartsourceguide.com 200** |

Root cause of the EADDRNOTAVAIL itself is **UNKNOWN**. The error means the
kernel could not assign a local address/port for the outbound socket — a
machine-local network-stack condition (ephemeral-port exhaustion, a lost/
unconfigured interface address, a VPN or filter-driver leftover), not a DNS or
remote-host failure. Nothing on the box recorded the interface state at the
time. It cleared across the power cycle. **What would settle it:** if it
recurs, capture `ifconfig -a`, `netstat -an | wc -l`, `sysctl net.inet.ip.portrange.*`
and `scutil --nwi` at the moment of failure. Until then, treat it as a
one-off that is *not* currently reproducing but has no known fix.

### 2.1 crontab vs `lib/cron.txt` — NO DRIFT

```
$ crontab -l > /tmp/recon_crontab.txt && diff /tmp/recon_crontab.txt ~/agents/lib/cron.txt
(no output — files are byte-identical)
```
32 active entries. `lib/cron.txt` mtime `Sep 2 23:50` (swept by the nightly
autocommit). **HANDOFF.md's claim that `lib/cron.txt` is "NOT a full crontab
mirror (drifted)" is WRONG as of today** — see Section 7.

### 2.2 launchctl

```
$ launchctl list | grep -i aiteam
1510	0	com.aiteam.grammy-bot          RUNNING, last exit 0
1503	0	com.aiteam.dashboard           RUNNING, last exit 0
-	0	com.aiteam.watchdog-heartbeat  idle,  last exit 0
-	2	com.aiteam.majordomo-check     idle,  LAST EXIT 2
-	0	com.aiteam.cost-cap-reset      idle,  last exit 0
-	0	com.aiteam.backup              idle,  last exit 0
-	0	com.aiteam.jeff-brief          idle,  last exit 0
```
All 7 plists present in `~/Library/LaunchAgents/` and all 7 loaded.

**`com.aiteam.majordomo-check` exit 2 is NOT a malfunction.** It ran at 19:4x
and again at 20:00; `check.sh` exits non-zero when any check FAILs, and both
runs had FAILs. The 19:4x run (before network recovery finished) had 2 FAILs:
```
FAIL site https://www.asbestoshq.com — no response (transport failure)
FAIL site https://www.smartsourceguide.com — no response (transport failure)
== SUMMARY: 28 checks — 26 OK, 0 WARN, 2 FAIL
```
The 20:00 run had 1 FAIL and both sites back to 200:
```
OK   site https://www.asbestoshq.com — 200
OK   site https://www.smartsourceguide.com — 200
FAIL backup sentinel — 1268h stale (last 2026-09-02 23:30, threshold 48h)
== SUMMARY: 28 checks — 27 OK, 0 WARN, 1 FAIL
```
The one remaining FAIL is the backup sentinel, which is **correct and expected**
(the box was off; next backup is tonight 23:30). The exit-2 clears itself at
the 23:30 backup. Note this is a working detector doing its job, not a bug.

**`majordomo/check.out.log` is 1.3 MB and append-only with no rotation** — minor,
logged in the fix list.

### 2.3 `expected_schedule.yaml` coverage vs crontab + launchctl

41 detector entries; every one maps to a real crontab line or launchd plist.
**No detector is orphaned and no scheduled job is uncovered.** The F08
coverage pass (2026-08-09) closed the gap the Aug audit found.

Current watchdog verdicts (`watchdog/state.json`, written 20:00): **6 healthy,
35 missing, 0 unknown.**

```
HEALTHY (6):  tg-monitor-reader, grammy-bot, watchdog-self,
              issues-capture, issues-capture-notify, dashboard
MISSING (35): everything else
```

**Nearly all 35 "missing" are outage artifacts and will self-clear** as each
job hits its next scheduled firing — the probes look back 26 h and the box was
off. Five are NOT outage artifacts and matter (see 2.5).

### 2.4 What fired since power-on (19:44)

launchd replays missed *interval* work on wake but **did not replay the missed
00:00 `cost-cap-reset` calendar firing across a cold boot** — `scripts/reset-caps.out.log`
mtime is still `Sep 3 00:00`, and there is no `caps_reset` row today. That is a
boot-vs-wake distinction worth knowing: **cron does not catch up, and launchd
calendar jobs do not catch up across a power-off either.** Only `StartInterval`
jobs do.

| Job | Fired? | Evidence | Result |
|---|---|---|---|
| `com.aiteam.grammy-bot` | YES, at load | `bot\|startup` 19:45:04; pid 1510 | OK |
| `com.aiteam.dashboard` | YES, at load | pid 1503; `curl localhost:3141` → **200** | OK |
| `com.aiteam.majordomo-check` | YES ×2 | `check.out.log` 19:4x + 20:00 | exit 2, see 2.2 |
| `watchdog.sh` (cron */15) | YES ×2 | `watchdog_check_complete` 19:45:30, 20:00:05 | OK — but fired **26 `watchdog_alert`** on first run |
| `issues-capture` (*/5) | YES | `capture.log` `scanned id>39773: 0 matched, 0 new` | OK |
| `issues-capture/notify_telegram` (*/5) | YES | `notify-tg sent 0/0, last_audit_id=39773` | OK — **high-water is caught up, no backlog spam** |
| `tg-monitor/reader.py` (*/5) | YES | `tg-monitor/reader.log` touched 19:4x+ | OK |
| `ship-to-site/preview_ping.sh` (20:00) | YES | `preview_ping_sent` 20:00:10 + 58 `ship_to_site_skipped` | OK — see 2.6 |
| `com.aiteam.watchdog-heartbeat` | **NOT YET** | `heartbeat.out.log` mtime still `Sep 3 18:35` | `StartInterval 3600` → first fire ~20:44 |
| `com.aiteam.cost-cap-reset` | **NO** | `reset-caps.out.log` mtime `Sep 3 00:00`; no `caps_reset` row | Missed firing not replayed; next 00:00 |
| `com.aiteam.backup` | NO (not due) | `backup.out.log` empty since Jun 21 | Next 23:30 |
| `com.aiteam.jeff-brief` | NO (not due) | — | Next 07:30 |

**26 Telegram alerts hit the operator's phone at 19:45** from the watchdog's
first post-boot pass (26 `watchdog_alert` → 26 `notify_sent`), plus
`backup_stale` and `gsc_queue_stale`. That is expected post-outage noise, not a
new fault, and it decays as jobs re-fire. 32 `notify_sent` total since boot,
**0 `notify_failed`**.

### 2.5 The five "missing" that are REAL bugs, not outage

These agents went silent **while the box was up and running**, and three of
them died *before* the 08-14 network event — so the network is not their excuse.

| Agent | Last real signal | Dead for (live days before power-off) | Class |
|---|---|---|---|
| **librarian** | 2026-08-12 02:00:08 (`sort_completed`) | 22 live days | REGRESSION — **fully explained, see 2.5b** |
| **orchestrator diary** | 2026-08-13 23:45:12 (`diary_written`) | **21 live days** | REGRESSION — died exactly at the network break |
| **market chain (analyst/process_queue)** | 2026-08-13 07:17:36 (`analyst_batch_run`) | **21 live days** | REGRESSION — network |
| **market YouTube polls** | 2026-08-13 07:00:09 (`scribe_poll_channel`) | **21 live days**, then 147 consecutive `_failed` | REGRESSION — network |
| **assignment-drafter output** | 2026-08-13 09:00:34 (`assignment_drafted`) | **21 live days** of `assignment_draft_failed` | REGRESSION — network |

#### 2.5b librarian is NOT a separate death — and it preserved the only direct evidence of the second root cause

librarian's last `sort_completed` is 2026-08-12, which looks like it pre-dates
the network break. It does not. `librarian/run.log` has an mtime of
**2026-09-03 02:03** — the agent kept running nightly right up to power-off; it
just never got far enough to write a success row. And its log is the **only
place in the fleet** that captured what actually killed the model calls:

```
librarian/run.log:299
Error: claude native binary not installed.

Either postinstall did not run (--ignore-scripts, some pnpm configs)
or the platform-native optional dependency was not downloaded
(--omit=optional).

Run the postinstall manually (adjust path for local vs global install):
  node node_modules/@anthropic-ai/claude-code/install.cjs
```

That is an **npm packaging failure, not a network failure** — the `claude` CLI
had no native binary to execute. It matches the very first model-call failure
in the whole sequence:

```
2026-08-13 02:00:00 | unknown | claude_call_failed | payload_excerpt: ""   <- EMPTY, not JSON
```
The excerpt is empty because `claude` wrote that error to stderr and nothing to
stdout. **librarian's 02:00 run on 2026-08-13 caught the claude CLI mid-upgrade,
with its files swapped out.** That one run died; the rest of 08-13 was healthy
(146 successful model calls, $12.11 spent). So this is a transient
update race, not the main event — but it is why librarian's last success reads
08-12 rather than 08-13.

**The permanent break is 24 hours later and has a different signature.** From
`2026-08-14 02:02:59` onward every failure carries `duration_api_ms: 0`,
`input_tokens: 0` — the CLI exists and runs but cannot reach the API — and
curl/ssh/python start returning EADDRNOTAVAIL within the hour.

**Class: REGRESSION, root cause now understood, both halves already repaired.**
The CLI half repaired itself today at 20:09 when the package reinstalled
(`/opt/homebrew/lib/node_modules/@anthropic-ai/claude-code/` — every file has
mtime `Oct 25 20:09`), and the probe in §1.6 proves it works. The network half
is verified on four stacks in §2.0.

**Note the dependency, because it matters for next time:** the CLI could only
repair itself *because the network came back first*. Had the operator powered
on with the network still broken, nothing would have recovered on its own, and
the fleet would have logged another 69 failures/day in silence.

### 2.6 What tonight's 23:00 `deploy_batch` would ship: **NOTHING**

Not a prediction — tonight's 20:00 `preview_ping` already ran the exact same
query (`ship.sh --dry-run`, `deploy_batch.sh:31`) 15 minutes before this recon.
Its 58 skip rows, by reason:

```sql
SELECT json_extract(payload_json,'$.site'), json_extract(payload_json,'$.reason'), count(*)
  FROM audit_log WHERE action='ship_to_site_skipped' AND ts >= <boot>;

asbestos | in_needs_review_queue  | 7
asbestos | site_repo_dirty        | 34
ssg      | slug_already_shipped   | 17
```

**41 asbestos + 17 ssg = 58 candidates, 58 skipped, zero `[would-ship]` lines.**
`deploy_batch.sh:61-66` will therefore take the `count -eq 0` branch, write
`deploy_batch_empty`, send no Telegram, and exit 0.

The refresh path is also inert: `ship-to-site/state/updated_slugs_queue.txt` is
**0 bytes**.

### 2.7 Can any ship path copy a pre-safety-repair guide over a repaired one?

## **NO.** Three independent guards, two of them active right now.

**The fork is real and large.** Byte-comparing the pipeline fork against live
site data:
```
~/projects/asbestos-contractors/content/asbestos/approved/guides/*.json   (41 files)
  vs ~/projects/asbestoshq-site/src/data/guides/<slug>.json

identical = 14      DIFFERENT = 24      not on site = 3
```
24 approved guides in the pipeline repo differ from what is live. The site repo
holds the Wave-2 safety repairs (last commit `fd27d3c 2026-08-09
"content(asbestos-remediation-cost): fact-verify, add FAQ and table, source all
claims"`); the pipeline repo still holds the pre-repair originals.

**Code path traced:** cron `0 23` → `deploy_batch.sh:31` `ship.sh --dry-run` →
`ship.sh:459 process_site()` → `ship.sh:465` globs
`content/asbestos/approved/guides/*.json` → `ship.sh:491 ship_one_slug()` →
three gates before any write:

| # | Gate | Location | Status tonight |
|---|---|---|---|
| 1 | `in_needs_review_queue` | `ship.sh:198-209` — checked **before** validate | **ACTIVE**: blocks 7 asbestos slugs |
| 2 | `site_repo_dirty` | `lib/validate.sh:36-40` — any uncommitted change in `~/projects/asbestoshq-site` skips **every** slug for that site | **ACTIVE**: blocks the other 34 |
| 3 | `slug_already_shipped` | `lib/validate.sh:65-68` — slug present in `APPROVED_GUIDE_SLUGS` (`src/components/GuideArticle.tsx`) → skip | **Would block all 24 differing slugs** |

**Gate 3 is the one that answers the question, and it is airtight here.**
Enumerating which approved slugs are absent from the whitelist:
```
approved guides: 41      whitelist entries: 38
approved AND NOT in whitelist:
  asbestos-insulation-identification
  asbestos-management-plan
  asbestos-roof-tiles
```
Those three are **exactly** the three that are not on the site at all
(`not on site = 3`) — so **every one of the 24 differing slugs is in the
whitelist and hits `slug_already_shipped`.** And all three unwhitelisted slugs
are themselves in `needs_review_queue.txt`, so Gate 1 stops them independently
of Gates 2 and 3.

**Net: there is no reachable path from the pre-repair fork onto the live site.**
A repaired guide can only be overwritten by removing it from
`APPROVED_GUIDE_SLUGS` first, which nothing automated does.

**One caveat worth stating plainly.** Gate 2 (`site_repo_dirty`) is currently
doing a lot of work, and it is an accident: the asbestos site is blocked only
because of three stray untracked files —
```
?? LANE3_PROGRESS.md
?? WAVE2_LANEA_PROGRESS.md
?? WAVE2_LANEB_PROGRESS.md
```
Clean those up and 34 slugs get re-evaluated. **They would then all hit Gate 3
and still not ship** — so the answer stays NO — but anyone tidying that repo
should know they are removing a safety layer they may not have known was load-
bearing. It is also why a 23:00 deploy has not been attempted since 2026-08-09.

### 2.8 Every job due in the next 24 hours (now 2026-10-25 ~20:15 PDT, Sunday)

| # | When | Job | Verdict | Why |
|---|---|---|---|---|
| 1 | every 5 min | `issues-capture/capture.sh` + `notify_telegram.sh` | **SAFE** | Already running clean; high-water at 39773, `sent 0/0` — no backlog blast |
| 2 | every 5 min | `tg-monitor/reader.py` | **SAFE** | watchdog `healthy`, last poll 19:55 |
| 3 | every 15 min | `watchdog/watchdog.sh` | **SAFE (noisy)** | Works; 26 alerts already sent, decaying as jobs re-fire |
| 4 | ~20:44 | launchd `watchdog-heartbeat` | **SAFE** | `StartInterval 3600` from load; will report watchdog fresh |
| 5 | 21:00, hourly | launchd `majordomo-check` | **SAFE** | Ran twice; exits 2 only because the backup sentinel is legitimately stale |
| 6 | 23:00 | `ship-to-site/deploy_batch.sh` | **SAFE** | Proven no-op by the 20:00 dry run — `deploy_batch_empty`, 0 slugs, silent |
| 7 | 23:30 | launchd `com.aiteam.backup` | **SAFE** | `/Volumes/Backups` mounted and writable, 9.1 TiB free; clears the one majordomo FAIL |
| 8 | 23:35 | `content-autocommit asbestos-contractors` | **SAFE** | Network restored — first successful push in 72 days |
| 9 | 23:40 | `content-autocommit ssg-content` | **SAFE** | Same |
| 10 | 23:45 | `orchestrator/write_diary.sh` | **UNKNOWN** | Silent since 2026-08-13 23:45. Needs a model call, which now works — but the failure was never diagnosed. Harmless if it fails again (writes a diary file, touches no site) |
| 11 | 23:50 | `scripts/agents-autocommit.sh` | **SAFE** | Will commit 2 untracked files and push the **20-commit backlog** (remote is still at `1f2eb16`, 2026-08-09) |
| 12 | 23:55 | `scripts/brain-autocommit.sh` | **SAFE** | Same; brain has its own unpushed backlog |
| 13 | 00:00 | launchd `cost-cap-reset` | **SAFE** | Did not replay on boot; fires normally tonight |
| 14 | 02:00 | `librarian/run_librarian.sh` | **NEEDS FIX FIRST** | Silent-dead since 2026-08-12 — **2 days before the network break**, so the network fix probably does NOT fix it. Only writes to `~/brain` inbox; low blast radius but will stay dead |
| 15 | 03:00 | `keyword-registry/lib/backfill_registry.sh` | **SAFE** | Ran clean to 2026-09-03 03:00; no model calls |
| 16 | 06:00 Mon | `link-monitor/check.sh` | **SAFE** | Tomorrow IS Monday. Network restored; report-only, never edits a guide |
| 17 | 06:00 | `watchdog/digest.sh` | **SAFE** | — |
| 18 | 06:05 | `issues-capture/digest.sh` | **SAFE (loud)** | Will summarise the soft pile; **5,069 open soft issues** — expect an ugly number, not a failure |
| 19 | 06:30 | `domain-hunter/scan.sh` | **UNKNOWN** | ~39 Haiku calls/day; model path now works, but it has not run in 72 days. Cost, not safety |
| 20 | 07:00 | `market/scribe/poll.sh` | **LIKELY FIXED — verify** | 147 consecutive failures were all EADDRNOTAVAIL. First real test of the network fix |
| 21 | 07:00 | `tg-monitor/analyzer.py` | **SAFE** | — |
| 22 | 07:00 | `ship-to-site/lib/digest.sh` | **SAFE** | Read-only digest |
| 23 | 07:10 | `idea-agent/run_weekly.sh` | **UNKNOWN** | Heavy model user; was failing on every call since 08-14 |
| 24 | 07:15 | `market/scribe/process_queue.sh` | **UNKNOWN** | Depends on #20 queueing something |
| 25 | 07:30 | launchd `jeff-brief` (majordomo brief) | **SAFE** | — |
| 26 | 08:00 | `gsc-submit/digest.sh` | **SAFE** | No credentials by design; reports the queue |
| 27 | 08:15 | `market/curator/curate.sh` | **UNKNOWN** | Model-dependent |
| 28 | 08:45 | `market/briefer/brief.sh` | **UNKNOWN** | Will deliver a brief even with no new input (audit F03 — still unfixed) |
| 29 | **09:00** | **`assignment-drafter/drafter.sh`** | **NEEDS FIX FIRST** | See below |
| 30 | 11:00 | `research-opportunity/poll.sh` (full mine cycle) | **UNKNOWN (cost risk)** | 60 Haiku triage calls + Sonnet extract per run. All 60 failed every day since 08-14; with the network back this is the fleet's largest spend and it has **842 untriaged rows** backed up |
| 31 | 12:00 | `research-opportunity/digest.sh` | **SAFE** | — |
| 32 | 20:00 | `ship-to-site/preview_ping.sh` | **SAFE** | — |

Not due in the next 24 h: `backlink-prospector` (Sun 08:00 — already passed
before boot, next 2026-11-01), `notes-agent/sort.sh` (Sun 10:00 — same),
`research-opportunity/discover_youtube.sh` (monthly, 1st).

**#29, the 09:00 drafter, is the one to look at before morning.** It is stuck
in a retry loop on a single slug:
```
[draft] coupa-alternatives: generating...
[generate_draft] Sonnet call 1 failed, sleeping 30s + retry
[generate_draft] FATAL: Sonnet failed twice
[fail] coupa-alternatives: Sonnet call failed; see state/failed.log
```
That identical 4-line block repeats for every run from 08-14 to 09-03 —
**the same slug, 21 times.** The underlying Sonnet failure was the network, which
is fixed, so tomorrow's 09:00 may simply succeed. But the queue head has not
moved in 21 attempts and nothing in the drafter de-prioritises a slug that has
failed 21 times, so if `coupa-alternatives` fails for any *other* reason it will
burn the run again. 30 s of eyeballs on `assignment-drafter/state/failed.log`
before 09:00 is cheap.

---

## SECTION 3 — Git state across the six repos

Snapshot taken ~20:20. **Another CC session (pid 8644) is live and may change
these repos underneath this report.**

| Repo | Branch | Last commit | Uncommitted | Unpushed | Worktrees |
|---|---|---|---|---|---|
| `~/agents` | main | `a6be4c4` 2026-09-02 "agents: auto-commit 2026-09-02" | **2** | **20** | 1 (clean) |
| `~/brain` | main | `3515544` 2026-09-02 "brain: auto-commit 2026-09-02" | **31** | **20** | 1 (clean) |
| `~/projects/asbestos-contractors` | main | `09021c6` 2026-08-09 "asbestos: auto-commit 2026-08-09" | 0 | 0 | 1 (clean) |
| `~/projects/asbestoshq-site` | main | `fd27d3c` 2026-08-09 "content(asbestos-remediation-cost): fact-verify, add FAQ and table, source all claims" | **3** | 0 | 1 (clean) |
| `~/projects/ssg-content` | main | `3aca08e` 2026-08-13 "ssg: auto-commit 2026-08-13" | 0 | 0 | 1 (clean) |
| `~/projects/smartsourceguide` | main | `43097c4` 2026-08-02 "Ship: helpscout-alternatives" | 0 | 0 | 1 (clean) |

**No stale worktrees in any repo.** Every repo is on `main` with a valid
`origin/main` upstream.

### 3.1 The 20-commit push backlog is the 08-14 network break, exactly

`git ls-remote origin` returns `1f2eb16 refs/heads/main` — the **2026-08-09**
commit. Local `main` is 20 commits ahead: one nightly auto-commit per day from
2026-08-14 through 2026-09-02. `~/brain` is ahead by the same 20.

This is not data loss — every commit exists locally and the remotes are
reachable again (`git ls-remote` rc 0). **Tonight's 23:50 / 23:55 auto-commit
jobs will push the whole backlog automatically.** No manual action needed; just
confirm it worked tomorrow.

Evidence the break is the cause, from `scripts/agents-autocommit.log`:
```
[main a6be4c4] agents: auto-commit 2026-09-02
 1 file changed, 21 insertions(+)
ssh: connect to host github.com port 22: Can't assign requested address
fatal: Could not read from remote repository.
```
and `logs/brain-autocommit.log`:
```
Wed Sep  2 23:55:00 PDT 2026: commit ok, push failed (offline?)
```
The script handled it correctly — committed locally, logged the push failure,
did not lose work. That is the designed behaviour working.

### 3.2 Uncommitted files

**`~/agents` (2)** — both benign agent state, will be swept at 23:50:
```
?? tg-monitor/digest_outbox/2026-09-03.txt
?? watchdog/probe_unknown_logged.date
```

**`~/brain` (31)** — 1 modified + 30 untracked, all machine-generated today:
```
 M projects/aiteam/keyword_registry/keyword_registry.md
?? projects/aiteam/docs/AITEAM_Recovery_Recon_2026-10-25.md   <- this report
?? projects/aiteam/incidents/2026-10-25_1945*.json            <- 26 watchdog incidents from the boot alert storm
?? projects/aiteam/incidents/2026-09-03_110003_*.json
?? projects/aiteam/research-opportunity/digests/2026-09-03.md
```
The 26 incident JSONs are the file-side record of the 26 boot `watchdog_alert`
rows from §2.4. Expected, not a fault.

**`~/projects/asbestoshq-site` (3)** — three stray progress markdowns:
```
?? LANE3_PROGRESS.md
?? WAVE2_LANEA_PROGRESS.md
?? WAVE2_LANEB_PROGRESS.md
```
These are session scratch notes from the 2026-08-09 Wave-2 work. **They are
currently the gate that blocks all 34 asbestos deploy candidates** (§2.7).
Deleting or committing them is safe for the live site — Gate 3 still holds —
but do it deliberately, not by reflex, and read §2.7 first.

### 3.3 Production scripts running untracked: NONE

Every `.sh`/`.py` referenced by a crontab line exists on disk and is tracked by
git:
```
for p in $(crontab -l | grep -oE "/Users/mmm2/agents/[...]\.(sh|py)" | sort -u); do
  git ls-files --error-unmatch "${p#/Users/mmm2/agents/}" || echo "UNTRACKED: $p"
done
→ (no output)
```
No untracked executables anywhere in `~/agents`. **Clean.**

### 3.4 Why the two content repos show zero backlog

`asbestos-contractors` (last commit 08-09) and `ssg-content` (last commit 08-13)
have nothing unpushed because they had nothing new to commit after the content
factory stopped producing on 08-14. Their silence is a *symptom* of §2.0, not
independent evidence of health.

---

## SECTION 4 — Agent roster

32 directories under `~/agents`. Nine are not agents (`_template`, `config`,
`exports`, `lib`, `logs`, `scripts`, `skills` — plus `dashboard` and `telegram`,
which are services, listed anyway because they have liveness state).

Read the "last success" column against **2026-09-03 19:06** (power-off), not
against today. "Last success" = the newest audit row for that agent's *success*
action, not merely its newest row — several agents kept writing rows while
producing nothing.

| # | Agent | Sched / on-demand | Last SUCCESS | Last FAILURE + reason | 48 h pre-power-off | Since power-on | SAFE TO RESUME |
|---|---|---|---|---|---|---|---|
| 1 | **assignment-drafter** | cron `0 9` | `assignment_drafted` **2026-08-13 09:00:34** | `assignment_draft_failed` 2026-09-03 09:06:54 `{"slug":"coupa-alternatives","reason":"sonnet"}` | 2 draft_failed + 4 claude_call_failed | not due | **NO** — 21 consecutive failures on the same slug; see §2.8 #29 |
| 2 | auto-refresh | **unwired by design** | `evaluate_run` 2026-08-09 01:45:46 | none | — | — | n/a — ingest deliberately not wired (`0db99f1`) |
| 3 | backlink-prospector | cron Sun `0 8` | `blp_run_done` **2026-08-30 08:00:25** | none | n/a (weekly) | not due until 2026-11-01 | **YES** — last Sunday before power-off ran clean |
| 4 | dashboard | launchd always-on | HTTP 200 on :3141 **now** | none | — | pid 1503, serving | **YES** — already up |
| 5 | domain-hunter | cron `30 6` | `daily_scan` + `dh_digest_empty` **2026-09-03 06:30:15** | none | none | not due | **YES** (cost caveat) — ran to the end, but `dh_digest_sent` last fired **2026-05-24**: 94 consecutive empty digests. Spends Haiku, produces nothing |
| 6 | **editor** (`editor-v1`) | invoked by `run-batch` only | `editor_scored` **2026-08-09 15:32:50** | `editor_parse_failed` 2026-07-19 | — | — | **NO PATH TO RUN** — its only caller, `run-batch`, has not executed since 2026-08-09 |
| 7 | fix-runner | on-demand | 2026-06-11 15:29:09 | — | — | — | vestigial; audit F26 recommends deletion |
| 8 | **geo-optimizer** (`geo-v1`) | invoked by `run-batch` only | `geo_scored` **2026-08-09 15:32:44** | `geo_parse_failed` 2026-07-31 | — | — | **NO PATH TO RUN** — same as editor |
| 9 | **gsc-submit** | cron `0 8` | `digest_sent` **2026-08-13 08:00:02** | `digest_failed` ×21, last 2026-09-03 08:00:13 | 2 digest_failed | not due | **YES, now** — the 21 failures are the dead notify path; fixed |
| 10 | **idea-agent** | cron `10 7` | partial: `google_search_complete` 2026-09-03 07:16:07 | `weekly_run_failed` ×68, `claude_call_failed` ×121, last 2026-09-03 07:19:09 | 6 claude_call_failed | not due | **YES, probably** — the 4 non-LLM phases ran clean to the end; only the synth died, on the CLI/network fault |
| 11 | internal-link | invoked by `deploy_batch` on new asbestos slug | 2026-07-19 09:14:05 | — | — | — | **YES** — no deploys since, so no invocations |
| 12 | issues-capture (×3) | cron `*/5`, `*/5`, `5 6` | `capture.log` 20:15 today | — | — | running clean, `sent 0/0` | **YES** — already running |
| 13 | keyword-registry | cron `0 3` | `keyword_registry_rebuilt` **2026-09-03 03:00:04** | none | none | not due | **YES** — no model calls, ran clean to power-off |
| 14 | **librarian** | cron `0 2` | `sort_completed` **2026-08-12 02:00:08** | CLI "native binary not installed" then 21 d of Haiku failures | n/a (writes no failure row) | not due | **YES** — fully explained by §2.5b, both causes repaired |
| 15 | link-monitor | cron Mon `0 6` | `check_run` **2026-08-31 06:01:31** | none | n/a (weekly) | **due 06:00 tomorrow** | **YES** — report-only, never edits a guide |
| 16 | majordomo (`jeff-brief`) | launchd `07:30` | `daily_brief_sent` **2026-09-03 07:30:20** | none | none | not due | **YES** |
| 17 | majordomo-check | launchd hourly | ran 19:4x + 20:00 today | exit 2 (1 legitimate FAIL) | — | ran ×2 | **YES** — see §2.2 |
| 18 | **market / scribe (poll)** | cron `0 7` | `scribe_poll_channel` **2026-08-13 07:00:09** | `scribe_poll_channel_failed` ×147 (7/day × 21 d), last 2026-09-03 07:00:04, all `URLError [Errno 49]` | 14 failures | not due | **YES, now** — pure network fault; tomorrow 07:00 is the first real test |
| 19 | **market / analyst (process_queue)** | cron `15 7` | `analyst_batch_run` **2026-08-13 07:17:36** | — (stopped emitting entirely) | none | not due | **YES** — nothing to process while the poll was dead |
| 20 | market / curator | cron `15 8` | `curator_run` **2026-09-03 08:37:19** | `claude_call_failed` ×63, 1 `usage_cap` | 5 claude_call_failed | not due | **YES** — but note it writes `curator_run` even when the LLM call failed |
| 21 | **market / briefer** | cron `45 8` | `briefer_run` **2026-09-03 08:45:14** | only 3 ever, last 2026-07-09 | none | not due | **YES mechanically / NO editorially** — it "succeeded" every day for 21 days on a chain that had been dead since 08-13. Audit **F03 is still open and this is it happening** |
| 22 | notes-agent | cron Sun `0 10` | `notes_weekly_sort_complete` **2026-08-30 10:00:01** | none | n/a | missed today's 10:00 (box off) | **YES** |
| 23 | **orchestrator (diary)** | cron `45 23` | `diary_written` **2026-08-13 23:45:12** | `diary_write_failed` once, 2026-07-04 | none | **due 23:45 tonight** | **UNKNOWN** — stopped exactly at the break; needs a model call, which now works. Low blast radius (writes a brain file) |
| 24 | research-opportunity | cron `0 11` + `0 12` | `mine_cycle_completed` + `digest_sent` **2026-09-03** | `triage_batch_failed`/`mine_cycle_failed` ×21 each | 2 + 2 + fetch failures | not due | **YES, with a cost caveat** — **842 untriaged rows** backed up; 60 Haiku + Sonnet extract on the first real run |
| 25 | scout | on-demand (majordomo) | 2026-07-31 18:02:58 | — | — | — | **YES** |
| 26 | ship-to-site | cron `0 7`, `0 20`, `0 23` | `preview_ping_sent` **2026-10-25 20:00:10** | none | none | ran clean tonight | **YES** — already running |
| 27 | telegram / grammy-bot | launchd always-on | `bot\|startup` 2026-10-25 19:45:04 | none | none | pid 1510 | **YES** — already up |
| 28 | tg-monitor (reader + analyzer) | cron `*/5`, `0 7` | reader poll 19:55 today | none | — | reader running | **YES** |
| 29 | watchdog (+ digest + heartbeat) | cron `*/15`, `0 6`; launchd hourly | `watchdog_check_complete` **2026-10-25 20:00:05** | none | none | 2 runs, 26 alerts | **YES** — already running |
| 30 | **content pipeline (`run-batch` / `run-batch-ssg`)** | invoked by drafter's `pipeline_fired` | `run-batch-ssg` 2026-08-09 15:07:59; `run-batch` 2026-08-08 23:47:41 | `srcver_fail` | — | — | **NO** — last `pipeline_fired` was **2026-08-13 09:14:02**. See below |

### 4.1 The content factory has produced nothing since 2026-08-13

This is the business-level consequence of §2.0 and deserves its own line:

```
assignment_drafted   last 2026-08-13 09:00:34
pipeline_fired       last 2026-08-13 09:14:02
run-batch-ssg        last 2026-08-09 15:07:59
run-batch            last 2026-08-08 23:47:41
audit_guide          last 2026-08-09 13:17:33
editor_scored        last 2026-08-09 15:32:50
geo_scored           last 2026-08-09 15:32:44
ship-to-site shipped last 2026-08-03 (state/shipped.log)
```
**Zero guides drafted, audited, scored, or shipped in 73 days.** 21 of those
days were a running machine failing silently; 52 were the power-off.

### 4.2 Notify delivery since power-on

```sql
SELECT action, count(*) FROM audit_log
 WHERE actor_id='notify' AND ts >= <boot> GROUP BY action;

notify_sent   | 32
notify_failed |  0
```
**32 sent, 0 failed, 100% success.** Compare the 21 days before power-off:
`notify_sent 0/day, notify_failed 17/day`. The delivery path is fully restored.

### 4.3 Backups

| | |
|---|---|
| Last success | **`backup_ok` 2026-09-02 23:30:12** — `sources=8, total=298M, duration=6s` |
| Last failure | **`backup_failed` 2026-08-10 23:30:31** — `{"failed":"agents(rc=23)","duration_s":24}` |
| Lifetime | 66 × `backup_ok`, 9 × `backup_failed`, 3 × `backup_drive_missing` (last 2026-06-12), 2 × `backup_dest_unwritable` (last 2026-06-10) |
| Destination | `/Volumes/Backups/mini-backup` — **mounted**, 298 MB, 9.1 TiB free |
| Sentinel | `~/store/flags/last_backup_ok` dated Sep 2 23:30 → currently **1268 h stale**, correctly flagged FAIL by majordomo |
| Next run | **tonight 23:30** (launchd `com.aiteam.backup`) |

No backup was lost — the box was off at 23:30 on 09-03. `rc=23` is rsync
"partial transfer due to error", which recurred 9 times over the fleet's life
and always self-cleared the next night. **A restore has still never been
tested** (audit F17, unchanged).

### 4.4 Token usage since power-on

```
2026-10-25 20:10:31 | recon | claude-haiku-5-5 | in 2 | out 4 | cache_read 11111 | cache_write 7667 | $0.0016
```
**One call: this recon's own auth probe. $0.0016.**
*(Max-subscription notional value, not real dollars billed.)*

For contrast, the last normal days before the break:
```
2026-08-13 | 146 calls | $12.107   (Max-sub value, not real $)
2026-08-12 | 145 calls |  $9.186
2026-08-11 | 124 calls |  $5.414
2026-08-14 |   1 call  |  $0.042   <- break
```
Nothing is currently burning capacity. The first full day back (tomorrow) will
be expensive by comparison, with `research-opportunity`'s 842-row backlog the
largest single item.

---

## SECTION 5 — Content and sites

### 5.1 Drafter queues — parsed with the drafter's own parser

Counted with `assignment-drafter/lib/parse_queue.sh` (the same code the drafter
runs), **not** `wc -l`. The parser drops blanks, `#` comments, `✓`-done and
`✗`-failed lines, and rejects anything that is not a valid `slug|primary_kw`.

| Site | File | total lines | `#` comments | blank | ✓ done | ✗ failed | **PENDING** |
|---|---|---|---|---|---|---|---|
| asbestos | `asbestos-contractors/content/asbestos/drafter_queue.txt` | 67 | 19 | 1 | 45 | 2 | **0** |
| ssg | `ssg-content/content/ssg/drafter_queue.txt` | 93 | 10 | 0 | 35 | 0 | **48** |

**The asbestos drafter queue is EXHAUSTED — zero pending assignments.** Even
with every bug fixed and the network back, the asbestos side of the factory
cannot produce a single new guide: it has nothing to write about. This is why
`queue_low` fired 46 times. Refilling it needs keyword decisions —
**operator work, not CC work.**

SSG has 48 pending. The head of the queue is the stuck slug:
```
coupa-alternatives|coupa alternatives     <- failed 21 consecutive days
vendr-alternatives|vendr alternatives
tidio-alternatives|tidio alternatives
divvy-vs-ramp|divvy vs ramp
fleetio-pricing|fleetio pricing
```

### 5.2 needs-review and approved-but-not-deployed

**Two different things share the name "needs review"** — do not conflate them:

| | asbestos | ssg |
|---|---|---|
| JSONs in `content/<site>/needs-review/` | **31** | **29** (incl. 2 `.superseded-*`) |
| Slugs in `ship-to-site/state/needs_review_queue.txt` (what the shipper actually gates on) | **10** | 0 |

`needs_review_queue.txt` (shared file, all 10 are asbestos):
```
asbestos-shingles-guide            asbestos-removal-near-me
how-to-dispose-of-asbestos         asbestos-insulation-identification
asbestos-ceiling-tile-guide        asbestos-roof-tiles
asbestos-management-plan           asbestos-abatement-process
asbestos-disposal-supplies         asbestos-gasket
```

**Approved but not deployed:**

| Site | approved JSONs | whitelist entries | **approved AND NOT live** |
|---|---|---|---|
| asbestos | 41 | 38 | **3** |
| ssg | 17 | 32 | **0** |

The 3 asbestos stragglers are:
```
asbestos-insulation-identification
asbestos-management-plan
asbestos-roof-tiles
```
All three are also in `needs_review_queue.txt`, so they are **deliberately
held, not stuck**. Note Lane A recorded `asbestos-insulation-identification
DOES NOT EXIST on disk` in the *site* repo — consistent: it is approved in the
pipeline and was never shipped.

SSG's whitelist (32) exceeds its approved set (17) because 15 live SSG guides
predate the current pipeline. Nothing is orphaned in the shipping direction.

### 5.3 Last real deploy per site (from the site repo's own git log)

```
asbestoshq-site     99c9b03  2026-07-19  Ship: asbestos-abatement-process
smartsourceguide    43097c4  2026-08-02  Ship: helpscout-alternatives
```
**98 days and 84 days ago respectively.** Corroborated independently by
`ship-to-site/state/shipped.log`, whose last line is
`2026-08-03T06:03:29Z ssg helpscout-alternatives`.

Note `asbestoshq-site`'s newest commit overall is `fd27d3c` (2026-08-09) — but
that is a Wave-2 *repair*, not a deploy. No new page has gone live on either
site in roughly three months.

### 5.4 Live sites — all green

| Check | asbestoshq.com | smartsourceguide.com |
|---|---|---|
| `curl -sI /` | **HTTP/2 200** | **HTTP/2 200** |
| `curl -sI /sitemap.xml` | **HTTP/2 200** | **HTTP/2 200** |
| `<loc>` entries in live sitemap | **48** | **39** |
| guide JSONs in site repo | 38 (`src/data/guides/`) | 32 (`data/guides/**`) |
| whitelist entries | 38 | 32 |

Repo data count and whitelist count match exactly on both sites — **no drift
between what is whitelisted and what exists.** The sitemaps carry more URLs
than guides because they also list non-guide pages (home, about, categories).

**5 random guide URLs per site, live:**
```
200  https://www.asbestoshq.com/guides/does-plaster-have-asbestos/
200  https://www.asbestoshq.com/guides/asbestos-siding-guide/
200  https://www.asbestoshq.com/guides/white-asbestos-vs-blue-asbestos/
200  https://www.asbestoshq.com/guides/asbestos-respirator/
200  https://www.asbestoshq.com/guides/asbestos-pipe-guide/

200  https://www.smartsourceguide.com/it-support/atera-reviews/
200  https://www.smartsourceguide.com/fleet-tracking/field-service-management-software-quickbooks/
200  https://www.smartsourceguide.com/answering-services/best-answering-service-law-firm/
200  https://www.smartsourceguide.com/spend-management/ramp-alternatives/
200  https://www.smartsourceguide.com/it-support/manageengine-alternatives/
```
**10/10 return 200.** Both sites served continuously through the entire
outage — they are static builds on a CDN and never depended on the Mac. **No
traffic was lost to the outage; only growth was.**

### 5.5 `gsc_submission_queue`

| | |
|---|---|
| total | **70** |
| pending (`submitted_at IS NULL`) | **70** — 38 asbestos + 32 ssg |
| submitted | **0** |
| oldest pending | `answering-service-pricing` (ssg), published `2026-04-08`, **201 days old** |

**Zero of 70 published URLs have ever been announced to Google**, unchanged
since the Aug-09 audit (F25). This is NEVER-WORKED-by-design: no Google Search
Console credentials exist on this machine. `gsc-submit/digest.sh` correctly
labels the queue rather than pretending. **Operator action, not a code fix.**

Of all findings in this report, this is the one most directly costing traffic:
70 live pages, the oldest 6½ months old, never submitted for indexing.

### 5.6 Asbestos Wave 2 — Lanes A and B landed; C and D do not exist

| Lane | Status | Evidence |
|---|---|---|
| **Lane 3** (infra) | **DONE** | `LANE3_PROGRESS.md`: "ALL THREE ITEMS DONE"; commits `d8b6e80`, `40a18bd`, `b8301bf` |
| **Lane A** (12 slugs) | **DONE** | `WAVE2_LANEA_PROGRESS.md`: "ALL 12 EXISTING SLUGS DONE"; 12 commits `54dd1eb`…`6eebd14` |
| **Lane B** (12 slugs) | **DONE** | `WAVE2_LANEB_PROGRESS.md`: "ALL 12 LANE B SLUGS DONE"; 12 commits `aebdecf`…`fd27d3c` |
| **Lane C** | **DOES NOT EXIST** | No progress file, no commits, no reference anywhere in `~/projects` or `~/brain` |
| **Lane D** | **DOES NOT EXIST** | Same |

`git log --since=2026-08-08` in `asbestoshq-site` shows **53 commits**, all on
2026-08-08/09, all consistent with Lanes 3 + A + B. Lane A also recorded a true
negative worth keeping: `asbestos-insulation-identification DOES NOT EXIST on
disk (38 guides present). Reported, not substituted.`

**Is "Lanes C and D" a real open loop or a mis-remembered plan?** UNKNOWN. There
is no artifact on this machine that ever named a Lane C or D. **What would
settle it:** the operator's recollection, or the session transcript that planned
Wave 2. Recorded as UNKNOWN rather than folded into "not done".

### 5.7 Pipeline/site fork: 24 of 41 guides differ

```
~/projects/asbestos-contractors/content/asbestos/approved/guides/*.json  (41)
  byte-compared against ~/projects/asbestoshq-site/src/data/guides/<slug>.json

identical   = 14
DIFFERENT   = 24
not on site =  3
```
The 24 differing slugs are **exactly the 24 repaired by Lanes A and B** (12+12,
with `does-plaster-have-asbestos` repaired in the Lane-B block). The site holds
the repaired version; the pipeline still holds the pre-repair original.

```
asbestos-abatement-process        asbestos-respirator
asbestos-air-quality-test         asbestos-roof-removal
asbestos-ceiling-tile-guide       asbestos-siding-guide
asbestos-drywall-guide            asbestos-siding-removal-cost
asbestos-duct-wrap                asbestos-tile-guide
asbestos-encapsulation-vs-removal asbestos-under-carpet
asbestos-floor-tile-removal       asbestos-vs-fiberglass
asbestos-glue                     black-mastic-guide
asbestos-inspection-cost          chrysotile
asbestos-one-time-exposure        does-plaster-have-asbestos
asbestos-pipe-guide               friable-vs-nonfriable-asbestos
asbestos-popcorn-ceiling-removal-cost
asbestos-remediation-cost
```

**The fork is a real data-integrity problem, but not an active danger** — §2.7
proves no automated path can push the stale side onto the live side. The risk is
*human*: anyone who opens the pipeline repo to edit a guide will be editing the
pre-repair text, and any future manual back-port would silently undo 24 safety
repairs. **Fix: back-port the site's repaired JSONs into the pipeline
`approved/` dir so the two agree.** Mechanical, no model calls, but it is a
write — out of scope for this recon.

---

## SECTION 6 — Open loops from the August saves

| # | Loop | Verdict | Evidence |
|---|---|---|---|
| 1 | **YouTube channel loop** (MeetKevin, CryptoCrewUniversity, trvon, CryptoAudiKingg, Crashtrading) in market config | **DONE** | `market/scribe/watchlist.yaml` holds all 7 slugs: `cowen` (:37), `casper` (:61), `meetkevin` (:79), `cryptocrew` (:101), `trvon` (:122), `audiking` (:145), `crashtrading` (:175). All have `display_name` + `expertise_enabled`. Per-channel watchdog detectors exist for all five new ones (`expected_schedule.yaml`). **Config is done; the polls themselves have failed 147 consecutive times on the network fault (§2.0).** |
| 2 | **`MAJORDOMO_TIMEOUT_SEC`** | **DONE** | `majordomo/run.sh:119` `MAJORDOMO_TIMEOUT_SEC="${MAJORDOMO_TIMEOUT_SEC:-600}"`, enforced by `perl alarm` at `:130`, with a `timeout_sec` audit payload at `:147`. Not in `.env` — runs on the 600 s shell default, which is intended |
| 3 | **`SCOUT_TIMEOUT_SEC`** | **DONE** | `scout/run.sh:77` same shape, `perl alarm` at `:81`, audit payload at `:92`. Also exported in `exports/bob-kit/scout/run.sh:77` and documented in `exports/bob-kit/INVENTORY.md:101` ("Default 600s. Below the bot's 660s backstop") |
| 4 | **D115 — `MAX(ts) < strftime` affinity bug in watchdog** | **DONE in watchdog; DONE-with-CAST in majordomo** | **The bug is not in the watchdog at all.** `watchdog/watchdog.sh:175-178 compute_threshold()` does the arithmetic in bash (`NOW - window_hours*3600 - grace*60`) and the probe compares `ts > ${thresh}` — two integers, no `strftime`, no affinity hazard. The only live instance of the D115 shape is `majordomo/daily_brief.sh:54`, which now reads `HAVING MAX(ts) < CAST(strftime('%s','now','-36 hours') AS INTEGER)` — the `CAST` is present and discharges it. `watchdog/digest.sh:77` compares against a bash variable, also safe. **Close D115.** |
| 5 | **auto-refresh wired** | **NOT DONE (deliberate)** | `auto-refresh/` contains `evaluate.sh`, `fetch_gsc.sh`, `CLAUDE.md`, `logs/`, `tests/`. **Not in crontab** and **no script anywhere calls it** (`grep -rln auto-refresh --include=*.sh` outside its own dir → no hits). Last audit row `evaluate_run` 2026-08-09 01:45:46. Commit `0db99f1` states "decay detector, schema and rules only — ingest deliberately unwired". Also note it depends on GSC data, which does not exist (#6) |
| 6 | **gsc-submit credentials** | **NOT DONE** | `gsc-submit/` holds only `CLAUDE.md`, `digest.sh`, `reconcile.sh`, `lib/`, `logs/`. **No credential file of any kind.** Queue: 70 pending / 0 submitted, oldest 201 days (§5.5). **MARK-only — needs a Google login, not code** |
| 7 | **Last GEO verdict** | **2026-08-09 15:32:44** | `SELECT max(ts) FROM audit_log WHERE actor_id='geo-v1' AND action='geo_scored'`. 77 days ago |
| 8 | **Last editor verdict** | **2026-08-09 15:32:50** | `SELECT max(ts) ... actor_id='editor-v1' AND action='editor_scored'`. 77 days ago. Both gates are dark because their only caller (`run-batch`) has not executed since 2026-08-09 — **F09 is superseded: the editor is not "silently out of the live path", the whole live path is stopped** |
| 9 | **GEO out-of-range parse bug fixed?** | **DONE** | `geo-optimizer/score.sh:80` "call LLM + parse, with **one retry** on a rejected line"; `:84-85` explains the failure is a nondeterministic sample so one retry recovers it; `:149-154` writes `geo_parse_failed` with an `attempts` count only after the retry also fails. The all-or-nothing validator is deliberate and documented at `:81` and `:88` ("Deliberately NOT clamping out-of-range values"). This matches audit F24's claim that D110's parse half closed in `f466673`. **Last `geo_parse_failed` was 2026-07-31**, before the fix |
| 10 | **SSG source gate ever returned PASS in a real run?** | **NO — NEVER** | The success action is `srcver_passed` (`ssg-content/content/run-batch.sh:1362`, `asbestos-contractors/content/run-batch.sh:1500`). **`srcver_passed` has zero rows in `audit_log`, ever — on either site.** Everything ever recorded is a failure: `ssg-verify-sources srcver_fail ×58`, `srcver_error ×1`, `run-batch-ssg srcver_fail ×3` + `srcver_escalated ×3`, `run-batch srcver_fail ×1`, `verify_sources parse_fail ×5`. The F01 port shipped 2026-08-09 and **every one of its 58 invocations failed**. Whether that is the gate working (content really is unsourced) or the gate being unpassable is **UNKNOWN**. *What would settle it:* run one known-good guide through `ssg-verify-sources` and see whether a `srcver_passed` row can be produced at all |
| 11 | **asbestos INTENT1 stem-tolerant fix ported?** | **NOT DONE** | SSG `audit_guide.py` **v4.4**, lines 306-309, is stem-tolerant: `… or (len(_stem(c.lower())) >= 4 and any(t.startswith(_stem(c.lower())) for t in lead_tokens))`. Asbestos `audit_guide.py` **v3.4**, line 215, is still the bare literal: `matched = sum(1 for c in intent_concepts if c.lower() in first_150)`. The consequence is visible in the feedback files — `[FAIL] INTENT1 ✗ Intent concepts in first 150 words: 0/4 (min 2)` recurs across `asbestos-rash-r3`, `how-to-choose-asbestos-abatement-company-r3`, `asbestos-inspection-near-me-r2`, `asbestos-testing-kit-home-r1`, `asbestos-board-and-sheet-r2/r3` and more. **A real, cheap, unported fix** |
| 12 | **D114 — issues backlog** | **OPEN AND WORSE** | D114 (`DEFERRED.md:1708`) was logged at 2,894 open. Now: **5,189 open** (120 hard + 5,069 soft), 16 resolved, 93 wontfix. It nearly doubled, almost entirely from the 21-day failure storm. The 06:05 digest will report it tomorrow |
| 13 | **D121** | **OPEN** | `DEFERRED.md:1867` — "`ship.sh --dry-run` is not dry: it fires both Sonnet gates and writes verdict rows". Gates sit at `:239-296` / `:297-348`, **above** the dry-run short-circuit at `:349`. **Did not bite tonight** — every slug was rejected by needs-review / dirty-repo / already-shipped, all of which are checked *before* the gates, so zero `geo_scored` or `editor_scored` rows exist for today. It will bite the first time a slug becomes genuinely eligible |
| 14 | **D122** | **CLOSED-AS-ACCEPTED** | `DEFERRED.md:1920` — "SSG ships with both Sonnet quality gates disabled — 🟢 ACCEPTED, recorded not to be re-litigated". Matches `ship-to-site/config/ssg.yaml` (`skip_geo_gate: true`, `skip_editor_gate: true`). Not an open loop |
| 15 | **Highest D-number** | **D122** | `grep -oE "\bD[0-9]+\b" DEFERRED.md \| sort -u -V \| tail -5` → `D118 D119 D120 D121 D122`, then `D427`. **D427 ignored as instructed** (UUID fragment, audit F23). Next free number is **D123** |

### 6.1 One loop the August saves did not know about

**The asbestos drafter queue is empty** (§5.1 — 0 pending of 67 lines). Nothing
in `HANDOFF.md`, `DEFERRED.md` or the Aug-09 audit records this. It is the
binding constraint on asbestos content production, ahead of every code bug in
this report, and it cannot be fixed by CC.

---

## SECTION 7 — Doc drift

### 7.1 Last-modified dates

| Doc | mtime | Age (days) |
|---|---|---|
| `PRIME.md` | **2026-05-17 11:49** | **161** |
| `HANDOFF.md` | **2026-05-30 22:41** | **148** |
| `DEFERRED.md` | 2026-08-09 12:24 | 77 |
| `LESSONS.md` | 2026-08-09 15:42 | 77 |
| `POLICY.md` | 2026-05-17 00:11 | 161 |
| `KILLED.md` | 2026-08-22 07:10 | 64 |

**`HANDOFF.md` is 148 days stale and describes a different fleet.** It is the
document `PRIME.md` tells the next session to "read first", and following it
would send someone chasing bugs that were fixed in June and miss every bug in
this report. Treat it as a historical artifact until rewritten.

`KILLED.md`'s 2026-08-22 mtime is interesting — it was touched *during* the
blackout, on a day when the fleet could not reach the network. That is a human
edit, the only one in the window.

### 7.2 HANDOFF.md / PRIME.md claims that Sections 1-6 prove wrong

| # | Claim | Where | Reality |
|---|---|---|---|
| 1 | "`lib/cron.txt` — doc-snapshot of issues-capture cron lines. **NOT a full crontab mirror (drifted)**" | HANDOFF "Files to reference" | **FALSE.** `diff <(crontab -l) ~/agents/lib/cron.txt` is **empty**. It is a byte-exact full mirror, kept current by the nightly auto-commit (§2.1) |
| 2 | "Current phase: ... two genuine silent-dead agents (**market-briefer**, orchestrator-diary) still awaiting a fix pass" | HANDOFF "Current phase" | **FALSE for market-briefer.** `briefer_run` has 102 rows and fired every single day through **2026-09-03 08:45:14**. It was never dead. The real briefer problem is the opposite and worse: it kept *succeeding* on an input chain that died 2026-08-13 (§4, row 21 — audit F03) |
| 3 | "market-briefer silently stale since 2026-05-25" | HANDOFF "Known bugs" table | **FALSE**, same evidence. Fixed at some point between May and August and never struck from the doc |
| 4 | "orchestrator diary silent-dead (~6d), last `diary_written` 2026-05-24" | HANDOFF "Known bugs" | **STALE BUT DIRECTIONALLY RIGHT.** `diary_written` has 64 rows and ran until **2026-08-13 23:45:12** — so it recovered after May and died again at the network break. The "~6d" and the 05-24 date are both wrong |
| 5 | "`com.aiteam.dashboard` has no watchdog liveness probe" | HANDOFF "Known bugs" | **FALSE.** `expected_schedule.yaml` has a `dashboard` entry, `signal_type: launchctl_label`, added in the F08 pass with a comment explicitly closing audit U3. Watchdog currently reports it `healthy`; `curl localhost:3141` → **200** |
| 6 | "`agent_issues` ... Backlog 109→0 (clean baseline)" | HANDOFF "Completed" | **LONG SUPERSEDED.** Now **5,189 open**. D114 tracks it |
| 7 | "**Blocked:** none" / "In progress: Watch first real soft-issue digest fire at 06:05 (05-31)" | HANDOFF | Five months obsolete. The digest has fired ~100 times since |
| 8 | "`AI_TIMEOUT_SEC` default 1200" | `lib/ai-cheap.sh:5` header | **STALE in ai-cheap.** HANDOFF records the raise to 1800 and audit F19 flagged the same drift in `ai-do.sh` (fixed by `4ca6d79`); `ai-cheap.sh`'s header comment still says 1200. Cosmetic |
| 9 | "Cost caps enforced by `check_cost_caps.sh` — $25 soft / $50 hard" | PRIME "Hard rules" | **TRUE** — `.env` has `COST_CAP_SOFT_USD=25`, `COST_CAP_HARD_USD=50`. No drift |
| 10 | "Wiki retrieval primary, RAG when document count > 1,500" | PRIME "Decision substrate" | **UNVERIFIED by this recon.** Not checked; out of scope. Recorded as UNKNOWN rather than implied-true |
| 11 | "Phone via grammy Telegram bot" | PRIME | **TRUE** — pid 1510, `bot\|startup` 19:45:04. But the newest `conversation_log` row is **2026-08-06 21:08:13**: the bot is alive and has had no conversation in 80 days |

### 7.3 One claim this recon confirms the docs got right

HANDOFF's gotcha — *"macOS cron does NOT catch up missed firings after sleep/
reboot — check `kern.boottime` FIRST when an agent 'skipped'"* — is exactly the
right instinct and is why §1.1 leads with the boot times. This recon extends it:
**launchd `StartCalendarInterval` jobs also do not catch up across a cold boot**
(the 00:00 `cost-cap-reset` did not replay — §2.4). Only `StartInterval` does.
That belongs in `LESSONS.md`.

---
