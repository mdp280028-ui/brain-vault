# AITEAM Context Save — Sister Chat: D026 + D092 + D094 Lane Closeout

**Date:** 2026-05-24
**Session topic:** Sister-chat "Agent infrastructure lane" — launchd auto-restart (D026), Issues Queue v1.1 (D092), Hybrid agent lockfile + timeout hardening (D094)
**Companion chat:** Sister chat working internal-link / auto-refresh / brain-bookkeeping lane. No file collisions this session.

---

## TL;DR

Three back-to-back ships, all with real end-to-end verification, all autonomous on cron/launchd:

- ✅ **D092 — Issues Queue v1.1** — nested-agent log path resolution + Telegram pipe for new_issues.log live on `*/5` cron
- ✅ **D026 — launchd auto-restart** — dashboard migrated to launchd with KeepAlive (grammy-bot already done; D026 effectively closed for both always-on processes)
- ✅ **D094 — Hybrid agent lockfile + timeout hardening** — shlock per entry-point + perl-alarm timeout (1200s default) centralized in 3 ai-*.sh wrappers

Net: full failure-response loop is now closed across D092 + D094. Hang → 1200s timeout-kill → audit_log → capture daemon → TG message → `/issues` page with copy-paste CC prompt pointed at correct log file.

---

## Session arc

1. **Lane scoped.** Sister chat took 2-item lane: D026 launchd + D092 Issues Queue v1.1. D-number D092 claimed up front to avoid parallel-chat collision.
2. **D092 first** (faster, lower risk). Pre-flight recon found the brief's premise inverted — nested agents log as bare names (`scribe`, `analyst`), not `market/scribe`. Fix was glob-one-level-deep, not split-on-slash. Shipped clean. TG pipe live.
3. **D026 second.** Pre-flight recon revealed scope collapse: only the dashboard is a true launchd candidate. grammy-bot already migrated. Everything else is cron-native batch or on-demand. Shipped 1-agent migration with real verify (kill -9 → respawned in ~1s).
4. **D094 unplanned addition.** CC's D026 recon surfaced the real cron failure modes (hang → double-spawn) live in hybrid batch agents, not in the always-on processes launchd was migrating. Logged as D093 originally, renumbered D094 after parallel-chat collision.
5. **D094 recon corrected the brief on four points.** ship-to-site is pure batch (drop from scope), `gtimeout`/`flock` absent on this Mac, claude -p has no timeout flag, all AI calls funnel through 3 wrappers → centralize timeout there.
6. **D094 timeout default escalated mid-build.** CC caught that 600s would kill idea-agent-synth's legitimate ~680s call. Approved 1200s default with documented rationale in each wrapper header.
7. **Lane closed clean.** Five items shipped, zero regressions, real verification on each.

---

## What got built / committed

### D092 — Issues Queue v1.1

**~/agents commits:**
| SHA | What |
|---|---|
| `20611fd` | `capture.sh` — `resolve_agent_dir` (top-level → glob one level deep `~/agents/*/<id>` → explicit a/b) + `find_log_in_dir` (prefer `log/*.log`, then `*.log`). Unknown → NULL so prompt reads "no log found." |
| `24b7cd7` | `notify_telegram.sh` + cron — drains `new_issues.log` via high-water mark on `audit_log_id`, one TG message per issue via `lib/notify.sh`, rate-limited 10/tick |
| `eaa4fe9` | Bonus fix: dashboard `generate-prompt` route had same root-cause bug (`cd ~/agents/scribe`). Now derives dir from resolved log path → `cd ~/agents/market/scribe` |

**~/brain:**
| SHA | What |
|---|---|
| `1d89771` | D092 closed in DEFERRED.md (also swept parallel-session DEFERRED.md hunks — flagged but not corrupting) |

**Cron line installed manually:**
```
*/5 * * * * /Users/mmm2/agents/issues-capture/notify_telegram.sh >> /Users/mmm2/agents/issues-capture/log/notify_telegram.log 2>&1
```
Crontab: 53 → 54 lines, one line added, nothing else changed.

**E2E verify (real):** synthetic scribe failure → `agent_issues` row with `log_file_path = ~/agents/market/scribe/poll.log` → real TG message landed (HTTP 200) → re-run idempotent (0 sent) → `/issues` feed shows correct path → test row deleted.

### D026 — launchd auto-restart (dashboard)

**~/agents commit `fb33435`** (one commit, four items):
- `dashboard/com.aiteam.dashboard.plist` — cloned verbatim from `com.aiteam.grammy-bot.plist`. `KeepAlive{SuccessfulExit=false}`, `ThrottleInterval=30`, `RunAtLoad=true`, `ProcessType=Background`, `WorkingDirectory=~/agents/dashboard`, `EnvironmentVariables{HOME,PATH}`.
- `lib/dashboard.sh` — rewritten to wrap `launchctl bootstrap`/`bootout`/`print` (start|stop|restart|status|url|install). No more nohup/pidfile. Repo plist re-copied to `~/Library/LaunchAgents` on start.
- `dashboard/log/` (+ `.gitkeep`) — launchd Std{Out,Err} streams; `*.log` gitignored.
- `dashboard/CLAUDE.md` — documents launchd cadence, mgmt commands, new log location.

**~/brain commit `adfe1e5`** — D026 closed, D094 added.

**Verify (real, macOS 26.3, all PASS):**
1. `bootstrap` → `curl :3141` = 200
2. `launchctl print` → state = running, pid 81096
3. `kill -9` → respawned in ~1s, new pid 81124, 200
4. `bootout` → process gone, :3141 unbound, job not loaded
5. Re-bootstrapped → left running

### D094 — Hybrid agent lockfile + timeout hardening

**~/agents commit `633f6dc`** (8 files):

**Timeout** — centralized in 3 wrappers (`lib/ai-do.sh`, `lib/ai-think.sh`, `lib/ai-cheap.sh`):
- `perl -e 'alarm shift; exec @ARGV'` wrapping the single `claude -p` line
- `AI_TIMEOUT_SEC` default **1200s** (20 min), env-overridable per call
- Documented rationale in each wrapper's header comment
- On SIGALRM-kill (rc 142): `audit_log` row `action='timeout_kill'`, payload `{error, model, timeout_sec, command_excerpt, agent_caller}` — the `error` key is what makes it surface on `/issues`

**Lock** — `shlock` per entry-point on `~/agents/<agent>/state/<agent>.lock`:
- `drafter.sh`, `process_queue.sh`, `curate.sh`, `brief.sh`, `run_weekly.sh`
- Contention → `audit_log` row `action='lock_contention'` (audit-only, not surfaced to `/issues`) + `exit 0`
- Trap release on EXIT/INT/TERM
- Each entry script exports `AI_CALLER=<agent>` so timeout_kill rows are attributed correctly
- State dirs `mkdir -p` at runtime (gitignored via `**/state/`)

**~/brain commit `904992f`** — D094 closed in DEFERRED.md (clean, no parallel-session sweep).

**Verify (real):**
1. Lock: held drafter's lock with live PID → `drafter.sh` exited 0, `lock_contention` row with holder PID, zero `run-batch`/`pipeline_fired`
2. Timeout: `ai-cheap.sh` claude line swapped for `sleep 99999`, `AI_TIMEOUT_SEC=10` → SIGALRM kill at exactly 10s, rc=142, `timeout_kill` row with full payload. Reverted; real call → rc=0
3. `/issues` surfacing: `capture.sh` picked up the `timeout_kill` (matched the `error` payload key)
4. Happy-path smoke on all 3 wrappers → rc 0, output intact
5. Test artifacts deleted (0 test issues, 0 open issues, no stray lock files)

---

## Decisions made (and why, briefly)

### D092 — Telegram message contains no dashboard URL
Putting dashboard token in TG history is security drift; Tailscale/LAN URL setup is yak-shave for v1.1. TG message = `agent_id + action + ~200 chars`. Operator opens `/issues` manually when alerted. Defer phone-tappable dashboard to its own item if needed.

### D026 — Dashboard logs relocated to `dashboard/log/`
Matches agents-as-folders convention. Old `~/brain/.../diary/dashboard.log` location stopped getting written; brain file left untouched (don't delete brain files mid-migration).

### D026 — `dashboard.sh` rewritten to wrap launchctl
Required to avoid EADDRINUSE collision (KeepAlive would fight nohup respawn). Single source of truth for dashboard lifecycle.

### D094 — Zero-dep tooling (shlock + perl alarm)
Over `brew install coreutils` (for `flock` + `gtimeout`). Both primitives are macOS built-ins, battle-tested, no install required. Future Mac Mini migration stays simple.

### D094 — Timeout in 3 wrappers, not per-agent
One change, every current + future agent covered. The 3 ai-*.sh wrappers are the single funnel for all `claude -p` calls.

### D094 — 1200s default (not 600s as briefed)
CC caught that 600s would kill idea-agent-synth's legitimate ~680s call (34K-token output). Timeout's job is catching *infinite* hangs (which never return), so generous ceiling is correct — asymmetric cost: a clipped legitimate call costs broken drafts + investigation, an infinite hang at 1200s vs 600s costs ~10 extra minutes. Per-call override remains for known-long calls.

### D094 — `lock_contention` audit-only, `timeout_kill` surfaced to `/issues`
Contention is expected behavior (cron-friendly, exit 0). Timeout is a real failure worth surfacing. Achieved by including `error` key in `timeout_kill` payload (matches `capture.sh` filter).

### D094 — Scope: drafter + market chain + idea-agent (not all hybrids)
Light daily/weekly hybrids (domain-hunter, backlink-prospector, research-opportunity) left unlocked — cadence makes overrun-overlap effectively impossible.

---

## What was left mid-task

Nothing for this lane. All three items closed end-to-end.

**D074 (editor + GEO gate live verify)** — still passive. Triggers when next new approved slug ships through asbestos pipeline. No dedicated session needed.

---

## New DEFERRED items captured this session

(D-number was claimed: D092 for Issues Queue v1.1, D094 for hybrid hardening. D093 was lost to parallel-chat collision on "Domain hunter v0.2" — handled via grep-then-take-next-free.)

No new deferred items beyond D094 itself (now closed). The hybrid-hardening work was itself the deferred item born from D026's recon.

---

## Open decisions (live design space)

No new ones from this session. Carrying forward:

1. **The 7 operator-policy questions** from cross-agent failure modes audit §7 (still blocking D056 editor production runner).
2. **D045** — rewire `run-batch.sh` from `claude -p` (Opus) to `ai-do.sh` (Sonnet) for ~75% cost reduction. With D094 in place, the pipeline writer now inherits the 1200s timeout via the wrapper, removing one prior risk of the migration.

---

## Coordination notes with the other chat

This chat owned: `~/agents/issues-capture/`, `~/agents/dashboard/`, `~/agents/lib/dashboard.sh`, `~/agents/lib/ai-*.sh` (3 wrappers), `~/agents/assignment-drafter/`, `~/agents/market/{scribe,curator,briefer}/`, `~/agents/idea-agent/`, `~/Library/LaunchAgents/`.

Other chat owned: internal-link / auto-refresh / brain-bookkeeping lane.

**No file collisions.** Two DEFERRED.md interactions:
- D092 brain commit swept parallel-session DEFERRED.md hunks (F17/F18 close, D045→D050 renumber, idea-agent cron note) under sister-chat's commit message. Coherent content, no corruption.
- D093 number collision (parallel chat claimed it for "Domain hunter v0.2"). CC caught via grep before commit, took D094.

---

## Lessons learned

### Pre-flight recon reshaped 3 of 3 builds this session
- **D092:** brief assumed `actor_id = market/scribe`; reality is `actor_id = scribe`. Fix changed from "split on slash" to "glob one level deep."
- **D026:** brief assumed fleet migration; reality is dashboard-only (everything else cron-native or already migrated).
- **D094:** brief assumed `gtimeout` + per-agent wrapping; reality is `gtimeout` absent → centralize in 3 wrappers using zero-dep primitives.

Pattern: every recon found a structural correction the build couldn't have proceeded correctly without. "Stop and report after recon" is paying for itself every session.

### Honest deviation flagging beats brief-letter-matching
D094 timeout test #2 specified "edit ai-cheap.sh, invoke notes-agent's sort.sh." But sort.sh calls ai-do.sh, not ai-cheap.sh. CC ran the sleep-stub test on ai-cheap.sh directly (the wrapper actually being tested) and documented why the caller-fail-path is covered by contract. Better than fabricating notes-inbox data to force the named code path.

### Three-piece failure-response loop now exists end-to-end
D092 (nested-path issue capture + TG pipe) + D094 (timeout_kill audit row matching `/issues` filter) compose: hang → timeout_kill → capture → TG alert + dashboard row with correct log path → copy-paste CC prompt → diagnosis. Two unrelated builds in the same lane combined into one capability.

### Asymmetric cost of timeout defaults
A clipped legitimate call costs broken outputs + investigation time. An infinite hang at higher ceiling costs only the difference in wall-clock. When in doubt, raise the ceiling. The timeout's job is preventing the unbounded case, not bounding the legitimate one.

### Parallel-chat DEFERRED.md sweeps are a recurring pattern
Twice in 24 hours (sister chat's D072 close + this lane's D092 close) absorbed concurrent chats' uncommitted DEFERRED.md hunks under one chat's commit message. Coherent content, but audit trail blurs. Worth a LESSONS.md entry: before `git commit` on `~/brain/`, check `git diff --stat`, confirm only intended files. Split or name-bundled-commit otherwise.

### D-number collisions across parallel chats keep happening
D093 was taken before this chat got there. CC's grep-then-take-next-free is the working fallback. The documented sync command:
```
grep -oE "\bD[0-9]+\b" ~/brain/projects/aiteam/DEFERRED.md | sort -u -V
```
remains the right tool.

---

## Cost data points

| Source | Burn (Max-sub value) | Real $ |
|---|---|---|
| D092 recon + build + verify | ~$1.50-2 | $0 |
| D026 recon + build + verify | ~$1 | $0 |
| D094 recon + build + verify | ~$2.50-3 | $0 |
| **Session total** | ~$5-6 (Max-sub value) | $0 |

Real-money cost: $0. Approximately 4-5 hours of CC work across three builds.

---

## Commits this session — consolidated

| Repo | SHA | What |
|---|---|---|
| ~/agents | `20611fd` | fix(issues-capture): nested log path resolution (D092 part 1) |
| ~/agents | `24b7cd7` | feat(issues-capture): notify_telegram.sh + cron line staged (D092 part 2) |
| ~/agents | `eaa4fe9` | fix(dashboard): generate-prompt cd uses resolved path |
| ~/brain | `1d89771` | chore(deferred): close D092 |
| ~/agents | `fb33435` | feat(dashboard): launchd migration (D026) — plist, lib/dashboard.sh rewrite, log dir, CLAUDE.md |
| ~/brain | `adfe1e5` | chore(deferred): close D026, add D094 |
| ~/agents | `633f6dc` | feat(hardening): D094 — shlock locks on 5 entry scripts + perl-alarm timeout in 3 wrappers |
| ~/brain | `904992f` | chore(deferred): close D094 |

Crontab installed manually (1 line): notify_telegram.sh `*/5`.
Launchd installed manually: `launchctl bootstrap` for `com.aiteam.dashboard`.

---

## Next session start point

**Cheapest immediate path:** observe the new infrastructure in the wild. Specifically:
- Any `timeout_kill` rows in audit_log this week → tells you the default 1200s is too tight (or that hangs really are happening)
- Any `lock_contention` rows → tells you cron overlap is real for assignment-drafter
- Issues Queue `/issues` page receives its first real failure → validates the full chain end-to-end
- Dashboard launchd survives a real reboot

**No dedicated session needed for any of the above.** Surfaces organically.

**For new work:** the 7 operator-policy questions remain the highest-leverage unblocker (D056 editor production runner is gated on them). Or D045 rewire (claude -p Opus → ai-do.sh Sonnet) is now safer to attempt with the timeout in place.

---

## Mission-bar nuance reminder

Dashboard "API cost" figures = Max-sub burn, not real $. Tonight's ~4-5 hours of CC work cost $0 real money. Only Max-subscription capacity consumed.
