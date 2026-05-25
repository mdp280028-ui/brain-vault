# AITEAM Context Save — 2026-05-24_0940
**Generated:** 2026-05-24T09:40:18-0700
**Since last save:** 2026-05-24 09:39:13
**Session topic:** Fleet health-check after 2-day Mac-off period → diagnosed `claude -p` rc=1 silent failure from cron (OAuth not available in non-interactive context) → switched all cron LLM calls to long-lived `CLAUDE_CODE_OAUTH_TOKEN` via `claude setup-token` → fixed 2 env-leak paths (check_kill_switches.sh `set -a`, analyzer.py dotenv→os.environ) → restored grammy bot under launchd KeepAlive + wrapper → wrote missing `process_queue.sh` batch wrapper for market pipeline → activated D033 (4-stage market cron) → extended watchdog with `launchctl_label` probe + 4 market entries. Session window: 2026-05-23 21:55 PDT – 2026-05-24 09:40 PDT.

Note: this is the SECOND ctx fire within a 3-min window (sister chat claimed `_0937.md` for idea-agent/notes-agent work). Per LESSONS.md convention, each chat keeps its own narrative file — this file is for the fleet-health/cron-auth/market-activation session. The `_0937.md` mechanical record happens to be the canonical roll-up for the whole night since neither save's window covered actual session work.

---

## Mechanical record

### Git activity since last save
```
(no commits since last save)
```

### Files changed
```
(no file changes)
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
(no audit rows since last save)

### Token usage
(no token usage since last save)

### Open mission tasks
(no open mission tasks)

---

## Mechanical record (actual — ctx.sh window was empty)

### Commits this session, agents repo (my work)
```
da88d09  fix(cron-claude-auth): wire CLAUDE_CODE_OAUTH_TOKEN + 3 env-handling fixes
07925f8  feat(grammy): restore bot + launchd KeepAlive supervision
b11a439  feat(watchdog): launchctl_label probe + grammy-bot entry
23ccd52  feat(market): activate cron pipeline (D033) + watchdog coverage
3631871  agents: auto-commit 2026-05-23   ← auto-commit swept up my process_queue.sh under generic subject
```

### Commits this session, brain repo (my work)
```
4c5d30c  DEFERRED: close D033 — Market pipeline cron activation
```

### Files changed (my work only — sister-session changes excluded)
```
agents/lib/check_kill_switches.sh           rewrite — explicit grep, no set -a leak
agents/tg-monitor/analyzer.py               env.pop ANTHROPIC_API_KEY before subprocess.run
agents/config/.env                          LOCAL ONLY (gitignored): removed dead `ANTHROPIC_API_KEY=  # required` line,
                                            added `[ -r "$HOME/.claude_oauth_token" ] && . "$HOME/.claude_oauth_token"`
agents/telegram/bot-launch.sh               NEW launchd wrapper (sources OAuth token, cd to script dir, exec node)
agents/watchdog/watchdog.sh                 new launchctl_label probe + parser/dispatch threading
agents/watchdog/expected_schedule.yaml      grammy-bot entry + 4 market entries
agents/market/scribe/process_queue.sh       NEW wrapper — drains .job queue, 3-attempt retry, workspace/failed/ abandon
~/Library/LaunchAgents/com.aiteam.grammy-bot.plist   NEW outside repo — KeepAlive SuccessfulExit:false, ThrottleInterval=30
~/.claude_oauth_token                       NEW outside repo, perms 600 — `export CLAUDE_CODE_OAUTH_TOKEN=sk-ant-oat01-...`
brain/projects/aiteam/DEFERRED.md           D033 marked CLOSED 2026-05-23 + closure entry in top priority table
crontab                                     user-scope (not in repo) — 4 new lines (07:00 / 07:15 / 08:15 / 08:45 PT)
```

### Audit_log rows this session (selected, with UUIDs for traceability)
```
auth_method_change      target=cron-claude-p     uuid=E42D5DE8-B46F-4B20-A931-D49F7BFD429D
agent_restored          target=grammy-bot         uuid=8B6EEFA6-64DF-43F4-97AE-D497A5EBF67E
cron_activated          target=market-pipeline    uuid=AB6F026B-5C0C-4E20-B848-918969BCAF31
analyst_batch_run × 2   18 + 10 briefs (smoke + cron-equiv)
curator_run + briefer_run from cron-equiv smoke — telegram_message_sent=true
```

### Cost this session (LLM only)
```
$0.14   tg-monitor analyzer (first successful run since May 16)
$4.62   curator one-time backlog catch-up (15 cowen + 13 casper videos through cowen synth)
~$0.90  process_queue: 18 + 10 = 28 briefs at ~$0.03 each
─────
~$5.66  total
```

---

## Decisions made this session

1. **Auth method for cron — Option A (`CLAUDE_CODE_OAUTH_TOKEN`), not Option B (raw API key).**
   Stays on Pro/Max billing; no separate API charges. Token from `claude setup-token` stored at `~/.claude_oauth_token` (perms 600, outside repo). `config/.env` *sources* the external file rather than inlining the value — keeps the token out of git, out of auto-commit transcripts, and out of any future ctx-save mechanical record that greps .env.

2. **Removed `ANTHROPIC_API_KEY=` from config/.env entirely, not just emptied.**
   The empty-with-inline-comment pattern (`KEY=  # required`) is exactly the bash trap that caused the silent rc=1. Removing the line + leaving a comment explaining why eliminates fat-finger re-instate risk.

3. **`check_kill_switches.sh` rewrite — explicit grep of 7 known keys, NOT `set -a; source $ENV_FILE; set +a`.**
   The bulk-export pattern was leaking DASHBOARD_TOKEN, SERPER_API_KEY, YOUTUBE_API_KEY, and (until #2) empty ANTHROPIC_API_KEY into every child process of every caller. Scope-limit to just kill-switch keys preserves intent without leaking secrets.

4. **Grammy bot supervised via launchd plist with `KeepAlive { SuccessfulExit: false }`.**
   Operator can manually stop without launchd fighting them; crashes (non-zero exit) trigger auto-restart with ThrottleInterval=30. Trade-off accepted: clean SIGTERM (e.g., OS sleep, manual stop) requires explicit restart. Watchdog catches the "stayed down" case within ~15 min.

5. **New watchdog signal type `launchctl_label` instead of `logfile_mtime` for grammy.**
   bot.log is only written on startup + message events. Idle long-poll bot looks identical to dead bot via mtime. `launchctl list <label>` PID extraction is the authoritative liveness signal.

6. **ONE wrapper at `market/scribe/process_queue.sh`, NOT two split fetch/analyze.**
   A `.job` file is the atomic "this video needs end-to-end processing" unit. Splitting fetch and analyze across two cron firings creates a fragile transcript-fetched-but-brief-not-yet-generated handoff state.

7. **Market pipeline cron times pushed later than operator's spec (07:00/07:15/08:15/08:45 vs spec's 07:30/08:15/08:30).**
   Curator's observed wall-clock runtime is ~20 min per the May 15 audit_log spread. Operator's tighter schedule would have given curator only 15 min slack.

8. **Retry tracking via inline `attempts=N` line on the .job file, NOT a sidecar.**
   Single file = single thing to ls / mv to failed/ / parse. Same regex parser handles fresh jobs (no attempts line) and retried jobs.

## Lessons learned

See LESSONS.md for canonical bullets — appended under "Fleet health-check + OAuth-from-cron fix + market pipeline activation (2026-05-23 21:55 – 2026-05-24 09:30 window)".

## Operator corrections

Light session — operator approved most proposals as written. Two explicit nudges captured for future-self:

- **"Don't bundle under a misleading subject."** When I proposed wrapping the watchdog extension into the grammy-restart commit, operator stopped me. I split into separate commits (`07925f8` grammy, `b11a439` watchdog) with subjects matching actual diff scope. Pattern: if the diff covers two distinct features, two commits — even if both are small.

- **`bot-launch.sh` modified mid-session** to add `cd "$(dirname "$0")"` so `dotenv` in bot.js finds `telegram/.env` regardless of caller CWD. I'd missed that `import "dotenv/config"` reads from CWD, not from script-relative path. System reminder flagged the change as intentional; I left it in place. **Lesson:** when wrapping a Node script under launchd (or any non-shell supervisor), ALWAYS `cd` to the script's directory first — `dotenv` defaults to CWD, not __dirname.

## What's next

**Immediate (within ~5 hours of session close):**
- Market pipeline first production fire: 07:00 PT (poll) → 07:15 PT (process_queue) → 08:15 PT (curator) → 08:45 PT (briefer). Brief should land on operator's phone ~08:46 PT. If nothing by 09:00 PT, investigation trail is in commit `23ccd52`'s message body.

**Within ~15 min after session close:**
- Watchdog cron will fire `✅ WATCHDOG: grammy-bot firing again` recovery message to operator's phone (missing→healthy transition queued in state.json after the live kickstart). Expected, not noise.

**Out of scope this session — deferred:**
- analyzer-with-delivery watchdog coverage (currently watches that analyzer fires, NOT that delivery succeeded)
- diary content quality (write_diary.sh captures LLM stdout verbatim, including hallucinated preambles like "The diary directory is outside my session's write scope. Here is the diary to paste or save:")
- Other watchdog blind spots
- Market pipeline cost-cap (D033 closes *activation* only; no cost-ceiling on the new chain yet)
- Grammy bot's `SuccessfulExit: false` trade-off: if operator restarts the Mac, bot stays down unless launchd-stopped state is overridden (operator workflow choice)

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-24_0937.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-17 22:00 (Keyword Registry live + brain-tracked pain log + digest archive · 4 production hooks · nightly rebuild cron firing) **Last session summary:** Shipped Keyword Registry agent (`~/agents/keyword-registry/` + `~/brain/projects/aiteam/keyword_registry/`). 162 canonical keywords tracked across 32 live asbestoshq.com slugs, alias-collapsed (chrysotile/white-asbestos/amosite/crocidolite etc — 14 seed rules). Three lib scripts: rebuild_registry.py (full regen, 0.5s), update_registry.py (write-through helper, verb dispatch), backfill_registry.sh (cron wrapper). Four production hooks wired best-effort: deploy_batch.sh new+refresh branches, internal-link/apply_approval.sh, research-opportunity/extract.sh pre-Sonnet shortcut (multi-word title match, 31 of 162 keywords qualify). Nightly cron at 0 3 * * * installed. Brain commit `6c15f3f`, agents commit `6047fe0`. Earlier this wall-clock window: brain-tracked pain_points_log + digest archive (commit 3f7145e) + Gap-1 fix for digest_log.posts_scanned + D078/D079/D080 deferred. Sister chat shipped Backlink Prospector v0.1 (b873996) in parallel — not inspected here. **Prior session summary (preserved):** Session 2 of Research/Opportunity Agent live (SE + YouTube + weekly digest + cron + master switch ON, commit bc7504b). Session 1 (Reddit vertical slice) at commit 2413a94. D075 Opportunity Scout deferred at brain 83cb3ed. Prior: D068 watchdog hygiene + D067 SSG plan recovery + D056 editor production runner + GEO Optimizer Phases 1 & 2 + Internal Link Agent v1.  --- 
