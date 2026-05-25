# AITEAM Context Save — Fleet Restoration + OAuth Cron Fix + D033 Activation
**Date:** May 23-24, 2026 (~21:55 PDT through 01:30 PDT)
**Session topic:** Full Telegram-messaging agent health check after Mac was off several days → discovered cron-claude OAuth auth issue (root cause for 3 agents) → fixed it + 3 env-handling side bugs → restored grammy bot with launchd KeepAlive → activated market pipeline cron (D033, deferred since 2026-05-15)
**Project:** AITEAM (separate from PayrollInsider/UST/SMART)

---

## TL;DR

Operator noticed the daily YouTube/market brief notification had stopped arriving. Ran a full health check on every Telegram-messaging agent (grammy bot, TG monitor analyzer, market briefer, ship-to-site, diary writer, assignment drafter). Discovered 4 separate failures, all traced to two root causes:

1. **Cron-claude OAuth incompatibility** (silent root cause since 2026-05-17). `claude -p` can't authenticate from a non-interactive cron environment — it depends on Claude Code-injected env vars (CLAUDE_CODE_ENTRYPOINT, CLAUDECODE=1, etc.) that don't exist in cron. The May 16 PATH fix surfaced this latent issue. Affected analyzer (C), diary (E), drafter (G) identically.
2. **Grammy bot died May 16 from clean SIGTERM** (operator closed a terminal). No launchd, no supervisor, no auto-restart. 7 days dead before health check found it.
3. **Market pipeline cron was never activated** (D033 deferred since 2026-05-15 close-out, then forgotten across sessions).
4. **3 env-handling side bugs** discovered during OAuth diagnosis: empty `ANTHROPIC_API_KEY=` in `.env` due to inline `#` comment, analyzer.py leaking that empty value to subprocess via `load_dotenv`, and `check_kill_switches.sh` leaking ALL .env vars (including secrets like DASHBOARD_TOKEN, SERPER_API_KEY) to every child process via `set -a`.

**All four problems fixed in one session, in dependency order:**
- OAuth fix via `claude setup-token` → `CLAUDE_CODE_OAUTH_TOKEN` in `~/.claude_oauth_token` (chmod 600)
- 3 env-handling side fixes in single commit
- Grammy bot restored + launchd KeepAlive plist + bot-launch.sh wrapper (token sourced before exec, not embedded in plist) + watchdog coverage
- Market pipeline cron activated — new `scribe/process_queue.sh` wrapper (~120 lines, .job retry/abandon semantics), 4 cron entries, 4 watchdog entries

**End state: 12 watchdog signals healthy / 0 missing.** Cleanest the fleet has been.

---

## DECISIONS MADE THIS SESSION

### Decision 1 — Option A (claude setup-token) over Option B (real API key)

CC presented three auth-fix options. Option A keeps the project on Max sub (real cost = $0). Option B (real `sk-ant-...` API key) would have 2-5x'd costs on the highest-volume cron load (analyzer + drafter + diary). Option C (long-running interactive shell) was theatrical.

**Locked: Option A.** Aligns with project's "$0 real money" architecture. Option B becomes the right answer only if/when project moves off Max sub (D013 territory, deferred until that decision is made).

### Decision 2 — Token storage in `~/.claude_oauth_token` (chmod 600), sourced by .env, NOT pasted in chat

Two options were offered: (a) paste token into the Claude.ai chat, (b) save to a local file and reference it. Operator chose (b). Token never appeared in any transcript, never in any commit, never in any future grep.

This is now the project pattern for any long-lived OAuth token: file at `~/.claude_oauth_token`, chmod 600, sourced by the configs that need it.

### Decision 3 — launchd `SuccessfulExit: false` for grammy bot

KeepAlive auto-respawns on crashes, but a clean SIGTERM (operator manually stops the bot) is honored — launchd won't fight the operator. Caught this in practice tonight when the bot stopped during the session and launchd correctly held it down. Manual `launchctl kickstart -k` restarts it.

### Decision 4 — Single batch wrapper (`process_queue.sh`), not split fetcher/analyst wrappers

The .job file is the atomic unit ("this video needs end-to-end processing"). Splitting fetch_transcript and analyze into two wrappers would have introduced a fragile state handoff between cron firings. Single wrapper owns the full lifecycle per job, retry/abandon logic is local.

### Decision 5 — `attempts=N` line appended to .job files for retry tracking

Simplest possible state tracking. Same parser handles both shapes. After 3 attempts, .job moves to `workspace/failed/` and audits as `analyst_batch_job_abandoned`.

### Decision 6 — Distinguish transient vs permanent vs fatal failure rc codes

- **Permanent** (e.g., transcript unavailable on YouTube side): remove .job, audit, continue
- **Transient** (network, Sonnet rc=5, missing transcript): bump attempts, leave .job, retry next batch
- **Fatal class** (rc=2/3 template files missing): move to `workspace/failed/`, audit, continue — operator action needed
- **Abandoned** (attempts > 3): same as fatal — preserve forensics, don't infinite-loop

### Decision 7 — Cron schedule: 07:00 / 07:15 / 08:15 / 08:45 PT (conservative buffers)

Trade-off was between earlier brief delivery (~08:00 PT possible) vs robustness against worst-case curator runtime (~25 min observed). Picked the conservative version. 55-min curator slack absorbs a heavy backlog. Brief lands ~08:46 PT.

### Decision 8 — Process the 7 stale May 15 .job files, don't clear them

First-run on stale jobs validates the wrapper's error handling on real-world failure modes (deleted/privated videos, expired transcripts). If 7 errors land cleanly, robustness is proven. Result: 18/18 succeeded (the 7 stale + 11 new), 2 idempotent skips, 0 failures. Best possible outcome.

### Decision 9 — Push commits after fleet was verified clean, not before each fix

Single push at session end after watchdog dry-run confirmed 12/12 healthy. Cleaner than pushing piecemeal across an in-flight investigation.

---

## WHAT WAS BUILT / FIXED

### Code/commits landed

| Commit | Repo | What |
|---|---|---|
| `da88d09` | ~/agents | OAuth fix (CLAUDE_CODE_OAUTH_TOKEN) + empty-ANTHROPIC_API_KEY fix + analyzer.py env leak fix + check_kill_switches.sh set-a leak fix |
| `07925f8` | ~/agents | grammy bot restore + launchd KeepAlive plist + bot-launch.sh wrapper |
| `b11a439` | ~/agents | watchdog: launchctl_label probe + grammy-bot entry |
| `23ccd52` | ~/agents | market: activate cron pipeline (D033) + process_queue wrapper + watchdog coverage |
| `4c5d30c` | ~/brain | DEFERRED.md: D033 marked resolved 2026-05-23 |

Both `~/agents` and `~/brain` pushed to origin/main at session close.

### New files

- `~/.claude_oauth_token` (chmod 600, gitignored by design — lives outside any repo)
- `~/agents/telegram/bot-launch.sh` (sources OAuth token then `exec node bot.js`)
- `~/Library/LaunchAgents/com.aiteam.grammy-bot.plist` (RunAtLoad, KeepAlive with SuccessfulExit=false, ThrottleInterval=30, separate stdout/stderr logs)
- `~/agents/market/scribe/process_queue.sh` (~120 lines, .job state machine, retry/abandon, audit logging)

### Modified files

- `~/agents/config/.env` — added `export CLAUDE_CODE_OAUTH_TOKEN=...` (sources from `~/.claude_oauth_token`); fixed empty `ANTHROPIC_API_KEY=` (inline `#` comment bug — moved comment to its own line)
- `~/agents/market/tg-monitor/analyzer.py` — no longer leaks empty ANTHROPIC_API_KEY to subprocess
- `~/agents/lib/check_kill_switches.sh` — replaced `set -a; source .env; set +a` with explicit reads of only the 7 kill-switch keys
- `~/agents/watchdog/expected_schedule.yaml` — added grammy-bot + 4 market pipeline entries
- `crontab` — added 4 market pipeline entries (07:00 poll, 07:15 process_queue, 08:15 curator, 08:45 briefer)
- `~/brain/projects/aiteam/DEFERRED.md` — D033 marked resolved

### Audit log entries

- `auth_method_change` target=cron-claude-p
- `agent_restored` target=grammy-bot (launchd_keepalive_added=true)
- `cron_activated` target=market-pipeline ({"D033":"resolved","stages":4})
- `analyst_batch_run` payload={"processed":20,"briefs_generated":18,"skipped":2,"failed":0,"abandoned":0}

---

## TECHNICAL DETAILS WORTH PRESERVING

### Why claude -p fails from cron (the actual mechanism)

The `claude` CLI authenticates via OAuth when invoked from inside Claude Code (interactive context). It detects "trusted interactive parent" via env vars the parent injects: `CLAUDE_CODE_ENTRYPOINT`, `CLAUDECODE=1`, `CLAUDE_CODE_SESSION_ID`. In that context, it loads cached OAuth credentials from macOS Keychain (stored as `Claude Code-credentials`).

In a clean non-interactive environment (cron, `env -i`, no controlling TTY, stdin /dev/null), none of those env vars exist. `claude -p` exits rc=1 with JSON on stdout: `{"is_error":true,"result":"Not logged in · Please run /login",...}`.

**Fix:** `claude setup-token` (interactive one-time) produces a long-lived OAuth token. Export as `CLAUDE_CODE_OAUTH_TOKEN` in any env that needs non-interactive claude access. Token format: `sk-ant-oat01-...`. Sourced from `~/.claude_oauth_token` (chmod 600) so it stays out of any repo.

### Why this didn't show up until May 17

May 16 failure was `claude: command not found` (PATH bug — cron PATH didn't include `/opt/homebrew/bin`). Fixed via commit `0be9d1f`. After fix, cron successfully invoked claude for the first time — which then immediately exited rc=1 due to OAuth-unavailable-in-cron. Each PATH layer surfaced the next.

**Lesson:** when fixing a "command not found" cron error, immediately follow up with a cron-equivalent rc check, not just a "the binary exists now" check. The next failure layer is often right behind.

### Cron-equivalent reproduction recipe (golden)

```bash
env -i HOME="$HOME" PATH=/opt/homebrew/bin:/usr/bin:/bin bash -c \
  'source ~/.claude_oauth_token && <script_to_test>' 2>&1
echo "exit=$?"
```

This is now the project's pre-deploy test for any cron-bound script that calls claude. Always run this before adding the cron entry. Catches OAuth bugs, PATH bugs, missing-env bugs, all in one.

### process_queue.sh per-job state machine

| Stage | Exit | Action |
|---|---|---|
| fetch_transcript rc=0 + transcript file present | success | proceed to analyze |
| fetch_transcript rc=0 + transcript_unavailable audited | permanent | remove .job, continue |
| fetch_transcript rc=2 (metadata fail — deleted/privated/network) | transient | bump attempts, leave .job, retry next batch |
| analyze.sh rc=5 (Sonnet failure) | transient | bump attempts, leave .job + transcript, retry next batch |
| analyze.sh rc=4 (transcript file missing) | transient | bump attempts, leave, retry |
| analyze.sh rc=2/3 (template files missing) | fatal class | move to workspace/failed/, audit, continue |
| Any stage, attempts > 3 | abandoned | move to workspace/failed/, audit `analyst_batch_job_abandoned`, continue |

Transcript files are never deleted by the wrapper — once fetched, reusable. Only the .job (work-pending marker) is touched.

### Observed runtimes (for future cron planning)

| Stage | Typical | Worst case |
|---|---|---|
| scribe/poll.sh | ~10s | ~30s |
| scribe/process_queue.sh | 0s empty, ~90s/new video | ~5 min for 3-4 overnight videos |
| curator/curate.sh | ~20 min | ~25 min |
| briefer/brief.sh | ~1 min | ~3 min |

First batch on 20 stale jobs took ~30 min total (yt-dlp + Sonnet are wall clock).

### Grammy bot launchd architecture

- **Plist:** `~/Library/LaunchAgents/com.aiteam.grammy-bot.plist`
- **Wrapper:** `~/agents/telegram/bot-launch.sh` — sources `~/.claude_oauth_token` then `exec node bot.js`
- **Why wrapper:** plists are world-readable by default. Secrets don't belong there. Wrapper keeps token in chmod-600 file.
- **KeepAlive: SuccessfulExit=false** — auto-respawn on crashes, but honor clean SIGTERM (operator can manually stop without launchd fighting back).
- **ThrottleInterval: 30** — don't thrash-restart on rapid crashes.
- **Recovery from clean stop:** `launchctl kickstart -k gui/$(id -u)/com.aiteam.grammy-bot`
- **Real env-var names** (caught by CC pre-flight): `BOT_TOKEN` and `ALLOWLIST_USER_IDS` (not `TELEGRAM_BOT_TOKEN`/`ALLOWED_CHAT_ID` as project instructions assumed). Loaded by bot from `~/agents/telegram/.env` via `dotenv/config`.

---

## OPEN ITEMS

### Tomorrow morning's verification signals (no action required tonight)

1. **07:00 PT — analyzer 7am digest** fires on cron. Should produce real content (not the silent failure of May 17-23). Proves OAuth fix survives real cron context, not just `env -i` simulation.
2. **08:46 PT — first market brief** lands on Telegram. Empty-day brief expected if no overnight videos (smoke test already produced one cleanly).
3. **Recovery ping for grammy-bot** — watchdog will send a "✅ WATCHDOG: grammy-bot firing again" alert. Expected, not noise.

If any of these don't arrive, investigation trail:
```bash
tail ~/agents/market/{scribe,curator,briefer}/*.log
sqlite3 ~/store/aiteam.db "SELECT ts, actor_id, action FROM audit_log WHERE ts > strftime('%s','now')-7200 ORDER BY ts DESC LIMIT 20"
~/agents/watchdog/watchdog.sh --dry-run
```

### Deferred this session (intentional, not bugs)

| ID | Item | Trigger |
|---|---|---|
| Diary preamble bug | write_diary.sh captures LLM's hallucinated "here is the diary to paste" preamble verbatim. Cosmetic, ~10 min fix (strip preamble or tighten prompt). | Next quality-pass session |
| Analyzer-with-delivery watchdog | Current watchdog only checks "tick fired", not "actually delivered". Same blind spot that hid May 17-23 silent failure. | Next watchdog hardening session |
| Other watchdog blind spots | Diary content quality, other agents | Same as above |
| D045 still open | Rewire run-batch.sh from `claude -p` (Opus) to `ai-do.sh` (Sonnet). ~75% cost reduction. | Soon — highest-$ optimization available |
| 7 operator-policy questions | Still unanswered. Q1, Q4, Q7 block D056 (editor verdict persistence). Q2 blocks cost-cap reconciliation. | Focused operator session |

---

## LESSONS LEARNED

1. **PATH fixes often surface auth/env layer issues underneath.** When fixing "command not found" in cron, always immediately run a cron-equivalent rc check to catch the next failure layer. The May 16 PATH fix unveiled the May 17-23 OAuth failure that hid for 7 days under "rc=1 stderr=empty."

2. **rc=1 with empty stderr is a tell.** It means the binary exits cleanly with structured output on stdout (JSON error in this case), and the wrapper's `set -euo pipefail` aborts before any downstream parsing. Always check stdout when stderr is empty on a non-zero exit.

3. **Health check found 4 separate problems with 2 shared causes.** The pattern: when multiple agents fail simultaneously, look for shared infrastructure (auth, env, PATH) before diagnosing each separately. This session's 4 outages collapsed to 2 root causes once OAuth was understood.

4. **Stale-job first-run is a free real-world robustness test.** The wrapper's error handling was validated on 20 stale jobs with 0 failures. Better signal than synthetic test data.

5. **Project instruction memory can drift from real code.** I'd specced `TELEGRAM_BOT_TOKEN` and `ALLOWED_CHAT_ID` from the project instructions; CC found the actual code uses `BOT_TOKEN` and `ALLOWLIST_USER_IDS`. CC reading actual code beats Claude.ai reading project files. Pattern: always have CC pre-flight env-var names against actual entry-point code before writing config touches.

6. **`SuccessfulExit: false` is the right launchd default for operator-controlled services.** Auto-restart on crashes, honor manual stops. Caught in practice when the bot was manually stopped mid-session and launchd correctly held it down — `launchctl kickstart -k` is the recovery command.

7. **Single-wrapper > split-wrapper for atomic units of work.** The .job file is one unit; splitting fetch and analyze across two cron firings would have introduced fragile state handoff. Same principle for any future "discover → process → output" pipeline.

8. **18/18 success rate on a first batch run is a signal to be slightly suspicious.** Real-world failure modes (deleted videos, expired transcripts, network blips) should produce *some* errors. The 100% success rate likely means the test corpus was too clean — operator should expect the first real-world failure within ~1-2 weeks of daily operation, and the wrapper's retry/abandon logic will get its first real exercise then. Not a bug, just calibration of expectations.

9. **Token-secret hygiene matters even on a personal Mac Mini.** Pasting the OAuth token into chat would have put it in this conversation history, any context save, any future grep. Saving to chmod-600 file outside any repo is the right pattern and only costs 30 seconds.

10. **Watchdog signal coverage = silent-rot insurance.** Grammy bot rotted for 7 days because watchdog didn't track it. Tonight's watchdog now covers it + 4 market stages. Pattern: every new agent gets a watchdog entry as part of build sign-off, not a follow-up step.

---

## OPERATOR CORRECTIONS APPLIED

1. **"wtf is g?"** — caught me using the `g` shorthand at the end of my own responses, which is wrong. `g` is the operator's signal *to me* to keep it short, not a tag I put on my own messages. Stopped immediately. Worth flagging in project instructions: the rule is one-directional.

2. **"first should I commit an. push the things cc just did?"** — operator caught that commits were local-only and asked whether to push before continuing. Right instinct. Pushed at session close instead.

3. **Operator chose Option (b) for token storage without prompting.** Good security instinct — keeping the token out of chat history is the right call even though Option (a) was faster.

---

## STATE OF AITEAM AT SESSION CLOSE

### What's live and healthy
- Mac Mini M4 Pro — running, ethernet, UPS
- **3 original agents:** orchestrator, librarian, editor (calibration only)
- **TG monitor pipeline:** reader (every 5 min, Telethon, 5 monitored groups), analyzer (07:00 daily digest — will fire tomorrow on real cron for first time post-fix)
- **Market pipeline:** scribe → process_queue → curator → briefer (07:00/07:15/08:15/08:45 PT, D033 resolved tonight, first cron fire tomorrow 07:00 PT)
- **Asbestos pipeline:** drafter (*/30 tick), ship-to-site deploy throttle (20:00 preview, 23:00 batch)
- **Diary writer:** 23:45 PT cron (fixed tonight as part of OAuth fix)
- **Grammy bot:** launchd-supervised, watchdog-monitored, survives crashes + reboots
- **Watchdog:** 12 healthy signals, 0 missing

### Mission bar
- **$200/mo to break even on Claude Max:** $0 revenue, $0 toward bar
- Tonight's work was fleet hardening, not revenue. Mission bar unmoved.

### Recent commits
- `~/agents/`: da88d09, 07925f8, b11a439, 23ccd52 (all pushed)
- `~/brain/`: 4c5d30c (pushed)

---

## FIRST MOVES FOR NEXT SESSION

### Morning (passive — just check phone)
1. Did the 07:00 PT analyzer digest arrive on Telegram?
2. Did the 08:46 PT market brief arrive?
3. Did the grammy-bot watchdog recovery ping arrive (expected, not noise)?

If yes to all three: fleet restoration is fully verified end-to-end.

### Active session work (recommend in order)
1. **D045 — rewire run-batch.sh from `claude -p` (Opus) to `ai-do.sh` (Sonnet).** ~75% cost reduction on the highest-volume pipeline. Single biggest cost-reduction lever in the project.
2. **The 7 operator-policy questions** from cross-agent failure modes audit §7. Q1, Q4, Q7 block D056 (editor verdict persistence). Focused operator session, not parallel work.
3. **Diary preamble fix** — 10-min cosmetic cleanup, do it when in the neighborhood.
4. **Analyzer-with-delivery watchdog coverage** — silent-rot insurance for the agent that just rotted for 7 days. Same blind spot still exists.
5. **SSG site rewrite + first article** — the actual revenue track. Still gated on operator keyword research + Ahrefs sessions + persona string cleanup in run-batch.sh.

### Out of scope reminder
- Other watchdog blind spots beyond analyzer-with-delivery
- Market pipeline tuning (let it run a week first)
- Cost-cap reconciliation (F17/F18, gated on Q2 of policy questions)

---

## DEFERRED ITEMS UPDATE

**Resolved this session:**
- **D033** — Market pipeline cron activation. RESOLVED 2026-05-23, commit 23ccd52.

**New deferreds:**
- **Diary preamble cosmetic bug** — write_diary.sh captures LLM's hallucinated "here is the diary to paste" preamble. Fix: strip preamble in script OR tighten prompt. ~10 min CC.
- **Analyzer-with-delivery watchdog** — current watchdog signal only checks "tick fired", not "actually delivered." Silent-rot blind spot.

**Carrying forward unchanged:**
- D013, D014, D025, D028, D045, D052-D070, F17/F18, 7 operator-policy questions, watchdog blind spots beyond analyzer

Sync to `~/brain/projects/aiteam/DEFERRED.md` via CC at next session open.
