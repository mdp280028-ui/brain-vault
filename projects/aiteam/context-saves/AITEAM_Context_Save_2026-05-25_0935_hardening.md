# AITEAM Context Save — 2026-05-25_0935 (hardening lane)

**Generated:** 2026-05-25T09:35:00-0700
**Since last save:** 2026-05-24 17:59:20
**Session topic:** Hybrid-agent hardening lane — **D092** (issues-capture nested-agent log-path resolution + Telegram pager), **D026** (dashboard → launchd auto-restart), **D094** (lockfile + wall-clock timeout for cron hybrids). Each was preceded by a read-only "stop and report" recon gate.

**Note on the filename suffix:** the bare `_0935.md` slot was claimed by a sister chat (verification/SSG-fy/memory/dropzone lane) in the same minute — same-minute `ctx.sh` collision, the documented gotcha. This file is the hardening lane's save; `_0935.md` is the sister's. The mechanical record below is reconstructed from my own `ctx.sh` skeleton before it was overwritten (it scans all of `~/agents`, so it spans both lanes' commits).

---

## Mechanical record

### Git activity since last save (all of ~/agents — both lanes)
```
74c1cde agents: auto-commit 2026-05-24
1627643 feat(telegram): typing-indicator pulse during orchestrator runs        [sister lane]
dc1dfc4 feat(orchestrator): wire conversation_log replay into run_agent.sh     [sister lane]
9f7c5c4 feat(lib): add build_conversation_context.sh + agents_with_memory.txt  [sister lane]
633f6dc feat(agents): lockfile + wall-clock timeout hardening for hybrid agents (D094)   [THIS lane]
ff63975 fix(internal-link): D091 audit upgrade — body links + render-path mirroring      [sister lane]
934dc76 feat(internal-link): SSG-fy v2 (items 1-5) — multi-site schema + path handling   [sister lane]
fb33435 feat(dashboard): migrate to launchd auto-restart (D026)                [THIS lane]
eaa4fe9 fix(dashboard): cd to correct dir for nested agents in CC prompt        [THIS lane]
24b7cd7 feat(issues-capture): Telegram pager for new issues (notify_telegram.sh) [THIS lane]
20611fd fix(issues-capture): resolve nested-agent log paths in capture.sh        [THIS lane]
```
**This lane's 5 commits:** `20611fd`, `24b7cd7`, `eaa4fe9` (D092), `fb33435` (D026), `633f6dc` (D094). Brain: `1d89771` (D092 close), `adfe1e5` (D026 close + D094 added), `904992f` (D094 close).

### Files changed (this lane)
```
D092:  M issues-capture/capture.sh          (nested log-path resolution)
       A issues-capture/notify_telegram.sh  (Telegram pager)
       M lib/cron.txt                        (notify_telegram */5 line, staged)
       M dashboard/server.js                 (generate-prompt cd fix)
D026:  A dashboard/com.aiteam.dashboard.plist
       A dashboard/CLAUDE.md
       A dashboard/log/.gitkeep
       M lib/dashboard.sh                     (rewrite: launchctl wrap)
       M dashboard/server.js                  (cd fix — same file as D092)
D094:  M lib/ai-do.sh  M lib/ai-think.sh  M lib/ai-cheap.sh   (perl-alarm timeout)
       M assignment-drafter/drafter.sh
       M market/scribe/process_queue.sh  M market/curator/curate.sh  M market/briefer/brief.sh
       M idea-agent/run_weekly.sh             (shlock lock + AI_CALLER)
```
(The full `ctx.sh` file list also shows sister-lane files: `internal-link/*`, `lib/build_conversation_context.sh`, `lib/agents_with_memory.txt`, `lib/run_agent.sh`, `orchestrator/*`, `telegram/bot.js`, `lib/write_to_dropzone.sh`, plus `domain-hunter/score/__pycache__/*.pyc`.)

### Token usage (period, both lanes)
| model | in_tok | out_tok | cost_usd |
|---|---|---|---|
| claude-sonnet-4-6 | 44 | 43828 | 1.4014 |
| claude-opus-4-7 | 11059 | 18790 | 1.2386 |
| claude-haiku-4-5-20251001 | 151360 | 43772 | 0.9428 |
(This lane's spend was tiny — three ~2-token wrapper smoke-test calls; the volume is sister-lane + production cron.)

### Agent activity (audit_log) — health evidence relevant to this lane
- **assignment-drafter** `drafter_tick_started` fires cleanly every 30 min (07:00/07:30/08:00/08:30/09:00/09:30) — the new D094 shlock lock did **not** break the `*/30` cadence.
- **market chain** ran end-to-end today: `transcript_fetched` → `analyst brief_generated` → `scribe analyst_batch_run` (07:15–07:19) — the new locks on `process_queue`/`curate`/`brief` did **not** block the daily run.
- **idea-agent** full chain completed (`weekly_run_complete` 07:20) with the new lock — and notably proposed `stale-lockfile-pid-guard` + `diagnose-cost-per-call-delta-fix` as ideas (07:20), independently echoing this lane's theme.
- `watchdog_recovery | market-process-queue` (07:30) — a benign missing→healthy transition tick, not a lock-induced failure (the chain's success rows precede it).

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

**1. D092 — nested log paths + Telegram pager (recon inverted the brief).** Recon found the brief's premise was backwards: nested agents log to `audit_log` under their **bare child name** (`scribe`, not `market/scribe`) and write `*.log` **directly in their agent dir**, not a `log/` subdir. So the fix is `resolve_agent_dir` = try `~/agents/<id>` → glob one level deep `~/agents/*/<id>` → explicit `a/b` form; unknown → `NULL` (prompt reads "no log found" rather than a wrong path). Telegram pager (`notify_telegram.sh`): operator chose **no dashboard link** in the message (avoids token-in-Telegram; opens `/issues` manually); high-water mark on `audit_log_id` (monotonic, rotation-proof) over byte-offset; rate-limited 10/tick; reuses `lib/notify.sh`. Same root-cause `cd` bug fixed in the dashboard generate-prompt route.

**2. D026 — dashboard to launchd (dashboard-only scope).** The dashboard was the only always-on process not under launchd (nohup + `/tmp` pidfile). Cloned the existing `com.aiteam.grammy-bot` plist verbatim → `com.aiteam.dashboard` (`KeepAlive{SuccessfulExit=false}`, `ThrottleInterval=30`, `RunAtLoad`, `ProcessType=Background`). Rewrote `lib/dashboard.sh` to wrap `launchctl bootstrap/bootout/print` — no more nohup/pidfile (single source of truth); kill the live nohup PID **before** bootstrap to avoid the `:3141` collision; logs relocated to `dashboard/log/dashboard.{out,err}.log`. No crontab change (it was never in cron); no other agent touched. D026's original scope (the Telegram bot) was already satisfied by grammy-bot — closed covering both always-on processes.

**3. D094 — lockfile + timeout (zero new dependencies).** Recon found `gtimeout`/`flock`/coreutils all **absent** on this Mac → used `shlock` (built-in, PID-aware) for locks + `perl alarm+exec` for the wall-clock timeout (`claude -p` has no timeout flag). **Timeout centralized in the 3 `ai-*.sh` wrappers** (not per-agent) so one change covers every agent + future ones and fixes the infinite-hang mode globally. `AI_TIMEOUT_SEC` default **1200s** — raised from the brief's 600 after recon proved `idea-agent-synth` legitimately hits ~680s on a 34K-token call (600 would have killed it). **Lock per entry-point** via `shlock` on `~/agents/<agent>/state/<agent>.lock` (drafter + market chain + idea-agent); contention → `lock_contention` audit + `exit 0`. `timeout_kill` carries an `"error"` payload key so issues-capture surfaces it on `/issues`; `lock_contention` is audit-only (expected, not a failure). Filed as **D094** because D093 was taken by a sister chat (domain-hunter v0.2) mid-session.

## Lessons learned

- **Recon can invert a brief's core premise — always verify the data shape before coding.** The D092 brief assumed nested actor_ids look like `market/scribe`; `audit_log` actually stores the **bare child name** (`scribe`), and nested agents write `*.log` directly in their dir (no `log/` subdir). The correct resolver is bare-name → glob `~/agents/*/<id>` one level deep, then prefer `log/*.log` else `*.log`, else NULL.
- **Verify an approved constant against real data before shipping it.** The operator-approved 600s AI timeout would have killed `idea-agent-synth`'s legitimate ~680s (34K-token) call. `token_usage` has no duration column — measure single-call wall-clock via the gap between consecutive rows of the same `correlation_id` (`LAG(ts) OVER (PARTITION BY correlation_id ORDER BY ts)`). Surfacing this got the default raised to 1200s.
- **macOS ships neither `gtimeout` nor `flock`; coreutils isn't installed.** The zero-dependency primitives are `shlock(1)` (built-in, PID-aware: auto-steals a stale lock from a dead holder) for locking, and `perl -e 'alarm shift @ARGV; exec @ARGV' N cmd…` for a wall-clock timeout (the alarm timer survives `exec`; SIGALRM terminates the call → exit 142).
- **`claude -p` has no wall-clock timeout flag** — `--max-turns` bounds loop iterations and `--max-budget-usd` bounds cost, but a network-stalled call hangs forever. An external timeout is mandatory for any cron-invoked claude call.
- **Centralize a cross-cutting guard in the shared wrapper, not per-caller.** Every agent's model call funnels through `ai-do.sh`/`ai-think.sh`/`ai-cheap.sh`, so putting the timeout there (env-tunable `AI_TIMEOUT_SEC`) covered all current + future agents in one change and fixed the infinite-hang mode globally — far better than wrapping each agent's call site.
- **launchd `KeepAlive` fights a manual stop.** Once a server runs under `KeepAlive{SuccessfulExit=false}`, a plain `kill` triggers an immediate respawn and a hand-started second `node` collides on the port. The management script must wrap `launchctl bootstrap/bootout`, and the pre-existing nohup PID must be killed *before* bootstrap or the port is already taken.
- **Whether a new audit action surfaces on `/issues` is a payload-shape decision.** `issues-capture` filters on `action LIKE '%_failed'/'%error%'` OR `payload_json LIKE '%"error"%'/'%"failed"%'/'%fatal%'/'%traceback%'`. A clean action name like `timeout_kill` won't match unless its payload carries an `"error"` key — include one to surface it (timeout_kill), omit it to keep it audit-only (lock_contention).
- **Committing a shared brain file sweeps a parallel session's uncommitted hunks under your commit.** When I `git add projects/aiteam/DEFERRED.md`, a sister chat's in-flight edits to the same file rode along under my message (nothing lost, but attribution blurred). Re-grep the file immediately before editing; stage exact paths; expect the sweep on hot shared files like DEFERRED.md.
- **D-number race, again: a number free at brief-writing time can be taken by the time you write DEFERRED.md.** D093 was free when the task was specified but claimed by a sister chat (domain-hunter v0.2) mid-session; I filed the hybrid item as D094 and documented the renumber in the entry. Always `grep -oE 'D0[0-9]{2}'` immediately before writing.

## Operator corrections

No "you got it wrong" corrections — the recon-gate format meant the operator's input came as directional calls at decision points:
- **D092 dashboard link:** chose Option 3 (no link in the TG message; open `/issues` manually) over adding a `DASHBOARD_PUBLIC_URL` env — explicitly to avoid putting the dashboard token in Telegram.
- **D026:** confirmed dashboard-only scope, the `dashboard.sh`→launchctl rewrite, killing the nohup PID before bootstrap, and relocating logs to `dashboard/log/`.
- **D094 timeout default:** after I flagged that the approved 600s would clip `idea-agent-synth`'s legitimate ~680s call, the operator raised it to **1200s** and added a spec to document the default + worst-observed numbers in each wrapper header.
- **D094 the rest:** approved the zero-dep `shlock`+`perl` tooling, centralizing the timeout in the wrappers, lock scope = drafter + market chain + idea-agent, and `timeout_kill`→/issues vs `lock_contention`→audit-only.

## What's next

**Immediate / verify-on-next-use (this lane):**
1. **First real `timeout_kill` / `lock_contention` in production** — both verified synthetically (drafter lock-held → exit 0; `AI_TIMEOUT_SEC=10` sleep-stub → SIGALRM at 10s → rc 142 → `/issues` row). Watch for the first organic one; `timeout_kill` will appear on `/issues`.
2. **D026 reboot persistence** — verified via `kill -9` respawn (~1s) but NOT via an actual reboot. Confirm `com.aiteam.dashboard` comes up on `:3141` after the next real reboot (`RunAtLoad`).
3. **`notify_telegram.sh` cron** is staged in `lib/cron.txt` and **installed live** (`*/5`, paste confirmed last session) — watch the first organic new-issue page lands on the phone (currently `new_issues.log` clean, high-water reset).

**Deferred / follow-ups:**
- **D094 follow-up:** the out-of-scope pipeline writer/auditor (`run-batch.sh`, routes through `ai-do.sh`) inherits the 1200s default; its single-call duration is unmeasured. If a long writer call ever `timeout_kill`s, raise `AI_TIMEOUT_SEC` for that path (run-batch is out of `~/agents` scope, can't edit here).
- **Watchdog could monitor the dashboard:** `watchdog.sh` already has a `probe_launchctl_label` liveness probe; `com.aiteam.dashboard` is not yet in its monitored set (noted in `dashboard/CLAUDE.md`).
- **Light daily/weekly hybrids left unlocked by design** (domain-hunter, backlink-prospector, research-opportunity) — cadence makes overrun-overlap impossible; revisit only if a hang is observed.

**Carried (other lanes / prior, unchanged):** operator live-smoke of the three new agents (/model, notes, idea-agent proposals); market pipeline production fires; sister lane's D083 remainder + forwarded-photo dropzone verification.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-24_1758.md
**Sister chat's save (same minute, different lane):** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-25_0935.md
**Active HANDOFF.md:** overwritten this save — see `~/brain/projects/aiteam/HANDOFF.md`.
