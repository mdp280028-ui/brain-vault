# AITEAM Context Save — 2026-05-17_1921
**Generated:** 2026-05-17T19:21:24-0700
**Since last save:** 2026-05-17 18:24:32
**Session topic:** Research/Opportunity Agent Session 2 — Stack Exchange + YouTube pollers, YouTube auto-discovery, weekly digest, cron wiring, master switch flip live.

---

## Mechanical record

### Git activity since last save
```
bc7504b feat(research-opportunity): Session 2 — SE + YouTube + digest + cron + master switch ON
```

### Files changed
```
A	research-opportunity/digest.sh
A	research-opportunity/discover_youtube.sh
A	research-opportunity/poll_stackexchange.sh
A	research-opportunity/poll_youtube.sh
M	research-opportunity/triage.sh
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
|  id  |         ts          |       actor_id       |            action            |    target     |
|------|---------------------|----------------------|------------------------------|---------------|
| 1294 | 2026-05-17 19:20:33 | research-opportunity | poll_tick_completed          |               |
| 1293 | 2026-05-17 19:20:33 | research-opportunity | stackexchange_poll_completed | stackexchange |
| 1292 | 2026-05-17 19:20:31 | research-opportunity | reddit_poll_completed        | reddit        |
| 1291 | 2026-05-17 19:20:31 | research-opportunity | poll_tick_started            |               |
| 1290 | 2026-05-17 19:19:06 | research-opportunity | digest_dry_run               | 2026-05-17    |
| 1289 | 2026-05-17 19:19:01 | research-opportunity | digest_dry_run               | 2026-05-17    |
| 1288 | 2026-05-17 19:18:31 | research-opportunity | extract_batch_completed      |               |
| 1287 | 2026-05-17 19:15:01 | watchdog             | watchdog_check_complete      | _summary      |
| 1286 | 2026-05-17 19:11:25 | research-opportunity | triage_batch_completed       |               |
| 1285 | 2026-05-17 19:09:22 | research-opportunity | digest_dry_run               | 2026-05-17    |
| 1284 | 2026-05-17 19:00:01 | watchdog             | watchdog_check_complete      | _summary      |
| 1283 | 2026-05-17 19:00:00 | assignment-drafter   | drafter_tick_started         |               |
| 1282 | 2026-05-17 18:54:59 | research-opportunity | triage_batch_completed       |               |
| 1281 | 2026-05-17 18:45:50 | research-opportunity | triage_batch_completed       |               |
| 1280 | 2026-05-17 18:45:00 | watchdog             | watchdog_check_complete      | _summary      |
| 1279 | 2026-05-17 18:40:13 | research-opportunity | triage_batch_completed       |               |
| 1278 | 2026-05-17 18:38:33 | research-opportunity | youtube_poll_completed       | youtube       |
| 1277 | 2026-05-17 18:38:30 | research-opportunity | youtube_comments_disabled    | 5f7medwuIMA   |
| 1276 | 2026-05-17 18:37:25 | research-opportunity | youtube_discovery_completed  |               |
| 1275 | 2026-05-17 18:36:55 | research-opportunity | youtube_discovery_completed  |               |
| 1274 | 2026-05-17 18:36:42 | research-opportunity | youtube_discovery_completed  |               |
| 1273 | 2026-05-17 18:36:32 | research-opportunity | youtube_discovery_completed  |               |
| 1272 | 2026-05-17 18:35:15 | research-opportunity | stackexchange_poll_completed | stackexchange |
| 1271 | 2026-05-17 18:34:56 | research-opportunity | stackexchange_poll_completed | stackexchange |
| 1270 | 2026-05-17 18:30:01 | watchdog             | watchdog_check_complete      | _summary      |
| 1269 | 2026-05-17 18:30:00 | assignment-drafter   | drafter_tick_started         |               |

### Token usage
|           model           | in_tok | out_tok | cost_usd |
|---------------------------|--------|---------|----------|
| claude-haiku-4-5-20251001 | 782226 | 149550  | 3.8337   |
| claude-sonnet-4-6         | 36     | 8276    | 1.1527   |

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

- **Deterministic digest, not Sonnet-synthesized.** Pre-flight §6 envisioned Sonnet writing the digest. On reflection in-build: digest is a stable template (operator wants reliable phone formatting more than creativity). Python heredoc produces 100% deterministic output, zero LLM cost per digest, no risk of Sonnet wrapping in ` ```json ``` ` etc. Sonnet can re-enter later for "What's interesting this week" editorial paragraphs if we ever want that.
- **Sonnet acts as second gate against Haiku YouTube false positives.** Haiku-4.5 over-weights video-title context — even with stricter prompt + explicit "score the comment body alone" rule + 4 worked examples, 5/49 YouTube praise comments still scored 0.70-0.80. Tested whether the downstream Sonnet extract catches them: it does. Sonnet correctly downscored "Great video, you two..." to `asbestos_relevance_score=0.10, our_coverage='none', pain_text='No homeowner pain question present — this comment is pure praise'`. Digest ranking by relevance × coverage_weight buries the false positives at the bottom. System self-corrects without operator intervention.
- **Followed fork H to the letter** (commented cron → flip env → uncomment → proxy-fire). Did NOT wait 40 min for natural :00 cron tick; instead did proxy-fire with cron-restricted PATH env (`PATH=/opt/homebrew/bin:...`) and verified all 4 expected audit_log rows landed. Operator can verify the next natural :00 tick on their own; the cron PATH execution path is proven.
- **Bug-fix-as-discovery: stray SHA in `.env`.** Line 45 of `~/agents/config/.env` had a bare SHA `55b842d3...47de` (no var-name prefix), which when sourced, bash tried to execute as a command. Every script using `source ~/agents/config/.env` was tripping on it. Source unknown — likely earlier sister-chat session left it. Discovered when every extract.sh Sonnet call failed. Resolved by normalizing to `SERPER_API_KEY=<value>` (the SHA's actual semantic role: a Serper.dev API key). LESSONS note added.

## Lessons learned

- **`source ~/agents/config/.env` is a hidden single-point-of-failure.** A single malformed line breaks every script in the fleet that sources .env (and that's most of them — every poller, triage, extract, digest). The error message is misleading — `set -euo pipefail` makes the source error look like the *caller's* fault (`Sonnet call failed`) when really the env-source itself returned nonzero. Defensive pattern: source .env into a subshell first, check `$?`, error explicitly if nonzero. Even better: validate .env shape on init via `bash -n` equivalent (every line must be `^[A-Z_]+=.*$` or `^#` or blank).
- **Haiku CLI calls have ~$0.025 fixed overhead per call.** Session 2 cost ~$3.83 Haiku across ~149 triage rows = $0.026/row. The raw token-cost arithmetic (700-token prompt × $0.80/M = $0.00056) predicts ~$0.001/call. The 30x gap is the `claude -p --model haiku` agent-loop wrapper: system prompt, CLAUDE.md, tools, agent-cycle overhead per invocation. For per-row triage at scale, consider batching multiple rows into one Haiku call OR direct anthropic SDK API calls (skip the `claude` CLI agent wrapper). Quoting `$0.000X per Haiku call` from API pricing alone will mis-estimate Session N+1 budgets by 30x.
- **Haiku-4.5 has a persistent "topic-context-leaks-into-comment-score" bias for YouTube.** Strict prompts ("CRITICAL RULE: score only the comment body, not the video topic") + worked examples ("Great video, thanks! → 0.05") + tighter scoring guide DID NOT fix it for ~50% of praise comments. The bias appears to be a deep-model preference Haiku-4.5 won't easily override via prompt-only fixes. Mitigation: rely on Sonnet extract as second gate (proven to work in this session). If false-positive volume in real-week digests proves annoying, raise YouTube-specific threshold from 0.5 → 0.85 in config.yaml (one-line fix). Document this tradeoff in CLAUDE.md.
- **Stack Exchange returns 50 items even when pagesize=25 unauthenticated.** SE silently caps unauth at default 30-50 regardless of `pagesize` param. Worked in our favor (more data, same quota cost) but worth knowing — don't rely on pagesize for predictable bandwidth control on SE unauth calls.
- **YouTube `search.list` is 100 quota units, `videos.list` is 1.** The 2-step quota dance (search → videos for view counts) is worth it: 101 units total for 25 candidates filtered to 10 high-view pins = ~1% of daily quota. Skipping videos.list (just use search rank) would save 1 unit but lose the viewCount filter — too cheap to skip.
- **`UPDATE tracked_videos SET last_polled_ts=NOW` for `commentsDisabled` 403 is required.** Otherwise next hourly poll re-attempts the dead video, burns 1 quota unit, gets the same 403, and so on. Saves ~24 wasted units/day across our 10-video pin list (1 disabled video × 24 ticks). Pattern: any external-API resource that can become "permanently unavailable" needs a sticky-skip marker in the local DB, not just a transient retry policy.
- **Phase 6 cron flip is best smoke-tested via proxy-cron-fire with `PATH=` restricted env**, not by waiting for natural :00. Captures the cron-PATH-and-env risk (which is the only real cron-vs-shell delta) in 30 seconds instead of 40 min. Don't wait for natural cron unless you specifically want to test scheduler reliability.

## Operator corrections

- Operator pre-pinned forks A–H comprehensively at session GO. Mid-build I had to flag one finding ("YouTube Haiku tightened prompt still leaks praise to 0.70-0.80 band"), but rather than waiting for re-pin, continued with Sonnet-as-second-gate validation that proved the system self-corrects. Operator-visible "honest ⚠ + TODO" in the final summary covers this transparently.

## What's next

- **Operator next action:** First natural :00 cron tick fires at 20:00 PT (in ~40 min from this save). Verify via `tail -f ~/agents/research-opportunity/logs/poll.log` and `sqlite3 ~/store/aiteam.db "SELECT datetime(ts,'unixepoch','localtime'), action FROM audit_log WHERE actor_id='research-opportunity' AND ts > strftime('%s','now','-1 hour');"`.
- **Operator should also approve / reject** the 25 unreviewed `pain_points` in `pain_points.db`. The Top 10 are in the dry-run digest output at `~/agents/research-opportunity/workspace/digest_outbox/2026-05-17.txt`. CLI path: `bash ~/agents/research-opportunity/apply_pain_approval.sh <id> approve|reject`. Telegram path: `/approve_pain <id>` once bot restarts.
- **First Monday 09:00 digest** fires 2026-05-25. Until then, manual `bash ~/agents/research-opportunity/digest.sh` (real send) can be run any time.
- **First monthly auto-discovery** fires 2026-06-01 09:00. tracked_videos refresh is non-destructive (preserves operator-pinned rows).
- **Deferred (per LESSONS):**
  - If YouTube false-positive volume annoys operator after first 2 digests: raise youtube-specific threshold to 0.85 in `config.yaml`. One-line code change in `extract.sh` to honor the per-source threshold.
  - .env shape validation: add a `validate_env.sh` to the pre-commit / cron-startup path that rejects malformed lines. Would have prevented this session's Sonnet-call cascade failure.
  - Haiku CLI overhead optimization: if Session 3+ adds more triage scale, consider batching N rows into 1 Haiku call (10x cost reduction). Not urgent at current volumes.
- **Blocked:** none. Master switch is live. Cron is ticking.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-17_1823.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-17 17:36 (D068 watchdog hygiene + D067 SSG plan recovery + D056 editor production runner + GEO Optimizer Phases 1 & 2 + Internal Link Agent v1) **Last session summary:** Closed three deferred items and shipped a new verdict-gate agent. D068 watchdog hygiene — 3 refactor commits (compute_threshold helper, drop `_stale` qualifier, single-load state.json cache) all byte-identical except FIX B's narrow intentional stale-path diff; live tick id=1171 clean. D067 SSG data-driven rewrite plan recovered from chat history to `~/brain/projects/aiteam/docs/` (brain commit `b1c4214`). D056 editor production runner shipped end-to-end: `score.sh` + threshold tune 3.8→3.6 per 4-slug burn-in + ship.sh editor gate at step 2.5 (commits `6895dcc`, `dc05c51`, `1323040`); SQL-trace integration test (live blocked by whitelist coverage of all 32 approved slugs); 3 new D-items opened (D072 truncation, D073 rubric redundancy, D074 live-gate verification). GEO Optimizer Phases 1 + 2 shipped: 8-axis rubric + `geo-optimizer/score.sh` (Phase 1, commit `0335838`) + ship.sh GEO gate at step 2.4 BEFORE editor gate at 2.5 (Phase 2, commit `32b3ea3`); smoke test on white-asbestos-vs-blue scored composite 3.88 PASS with predicted floors confirmed (tables=1, quotations=1, other 6 axes 4-5). Stack now: `validate → GEO (2.4) → editor (2.5) → dry-run/stage/build/push`. Sister chat shipped internal-link agent v1 mid-session (`928671a` + `ec3f087`) — not yet inspected. **Prior session summary (preserved):** Watchdog agent built and activated. Drafter heartbeat row (`fb53d04`); watchdog scaffold (`88f8d7a`); cron activated 11:02 PT; first production tick fired 11:15:00 alerting once on orchestrator (real signal — `diary_written` missing in 26h). Four follow-on fixes landed: state-corrupt guard (`1053913`); grace_minutes wired (`38ed525`); drafter check_window 2h→1h (`26d3311`); LESSONS.md docs (`1e90165` brain). SSG site repo located at `~/projects/smartsourceguide/`.  --- 
