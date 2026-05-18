# AITEAM Context Save — 2026-05-17_1823
**Generated:** 2026-05-17T18:23:42-0700
**Since last save:** 2026-05-17 17:39:34
**Session topic:** Research/Opportunity Agent (Agent A, master ranking #10) — Session 1 vertical slice (Reddit-only) + D075 Opportunity Scout deferred entry.

---

## Mechanical record

### Git activity since last save
```
2413a94 feat(research-opportunity): v1 agent — Reddit poller + Haiku triage + Sonnet extract + approval handler
```

### Files changed
```
A	research-opportunity/apply_pain_approval.sh
A	research-opportunity/CLAUDE.md
A	research-opportunity/config.yaml
A	research-opportunity/extract.sh
A	research-opportunity/init_db.py
A	research-opportunity/poll_reddit.sh
A	research-opportunity/poll.sh
A	research-opportunity/triage.sh
M	.gitignore
M	telegram/bot.js
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
|  id  |         ts          |       actor_id       |         action          |             target              |
|------|---------------------|----------------------|-------------------------|---------------------------------|
| 1268 | 2026-05-17 18:20:49 | research-opportunity | pain_rejected           | 2                               |
| 1267 | 2026-05-17 18:20:49 | research-opportunity | pain_approved           | 1                               |
| 1266 | 2026-05-17 18:19:37 | research-opportunity | extract_batch_completed |                                 |
| 1265 | 2026-05-17 18:17:44 | research-opportunity | extract_batch_completed |                                 |
| 1264 | 2026-05-17 18:15:01 | watchdog             | watchdog_check_complete | _summary                        |
| 1263 | 2026-05-17 18:14:47 | research-opportunity | triage_batch_completed  |                                 |
| 1262 | 2026-05-17 18:14:45 | research-opportunity | triage_batch_completed  |                                 |
| 1261 | 2026-05-17 18:13:14 | claude-aiteam        | geo_phase3_closed       | run-batch-sh                    |
| 1260 | 2026-05-17 18:13:06 | research-opportunity | triage_batch_completed  |                                 |
| 1259 | 2026-05-17 18:07:54 | research-opportunity | poll_tick_completed     |                                 |
| 1258 | 2026-05-17 18:07:54 | research-opportunity | reddit_poll_completed   | reddit                          |
| 1257 | 2026-05-17 18:07:54 | research-opportunity | poll_tick_started       |                                 |
| 1256 | 2026-05-17 18:07:38 | research-opportunity | poll_tick_started       |                                 |
| 1255 | 2026-05-17 18:07:34 | research-opportunity | poll_tick_started       |                                 |
| 1254 | 2026-05-17 18:07:23 | research-opportunity | reddit_poll_completed   | reddit                          |
| 1253 | 2026-05-17 18:07:16 | research-opportunity | reddit_poll_completed   | reddit                          |
| 1252 | 2026-05-17 18:05:34 | geo-v1               | geo_scored              | white-asbestos-vs-blue-asbestos |
| 1251 | 2026-05-17 18:03:44 | deferred-update      | d075_appended           | DEFERRED.md                     |
| 1250 | 2026-05-17 18:00:00 | watchdog             | watchdog_check_complete | _summary                        |
| 1249 | 2026-05-17 18:00:00 | assignment-drafter   | drafter_tick_started    |                                 |
| 1248 | 2026-05-17 17:45:01 | watchdog             | watchdog_check_complete | _summary                        |

### Token usage
|           model           | in_tok | out_tok | cost_usd |
|---------------------------|--------|---------|----------|
| claude-sonnet-4-6         | 19     | 12801   | 0.5979   |
| claude-haiku-4-5-20251001 | 109055 | 30993   | 0.4903   |

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

- **Pre-flight discipline before build.** Operator ran two STOP-AND-REPORT recon passes (Reddit-only, then +SE+YouTube delta) before any code landed. Both surfaced architectural forks (single vs three tables, fixed list vs auto-discovery, etc.) that were pinned before Session 1 began. Result: zero scope drift mid-build.
- **Single polymorphic `source_posts` table with `source` discriminator** (vs three parallel tables). Matches tg-monitor's `messages` table convention; one index covers all sources; simplifies future HN/IndieHackers extensions.
- **Local SQLite at `~/agents/research-opportunity/pain_points.db`**, NOT `aiteam.db`. Raw posts are chatty; isolating them keeps `aiteam.db` lean and backup-cheap. Matches tg-monitor `messages.db` precedent.
- **`/approve_pain <id>` explicit step before drafter_queue.txt append.** Refused to direct-write. Each approved pain gets `# from-pain-point #<id>` provenance comment so we can trace queue lines back to their source post.
- **Sonnet single-call for extract + coverage match** (vs two calls). One JSON object returns pain + category + coverage verdict + best_match_slug + proposed_slug + proposed_kw. ~12KB input × 7 pains = ~$0.04 this smoke run.
- **PyYAML via per-agent venv** (matches tg-monitor) rather than `pip install --break-system-packages`. Externally-managed env (PEP 668) blocked system-wide install.
- **D075 deferred to `~/brain/projects/aiteam/DEFERRED.md`** as a separate Opportunity Scout agent (business-idea discovery, output = briefs not slugs). Operator preempted scope creep by making the cousin agent visible.

## Lessons learned

- **bash 3.2 mis-parses apostrophes inside `$(cat <<EOF ... EOF)` heredocs.** Even with unquoted `EOF`, the outer `$()` parser scans for matching single quotes. Rephrased the triage prompt to remove `don't` / `it's` etc. Future heredoc-inside-`$()` should avoid contractions. Documented in `research-opportunity/CLAUDE.md` Hard Rules section.
- **sqlite3 column values can contain raw `\n`; bash `read -r ... <<<"$ROWS"` then splits at every newline.** Cost: 22 wasted Haiku calls before the fix. Fix: `REPLACE(REPLACE(col, CHAR(10), ' '), CHAR(13), ' ')` in the SELECT, on every text column used for shell-loop iteration. Apply this pattern in every future bash-driven SQL extraction loop.
- **Sonnet sometimes returns JSON arrays even when prompted for a single object** (especially when the post contains multiple distinct pain points). The parser must accept both `{...}` and `[...]` shapes and pick the highest-relevance entry from a list. Built into `extract.sh` `PARSE_RESULT` block.
- **Sonnet sometimes wraps responses in `` ```json ``` `` code fences** despite "no code fence" instructions. Always strip leading/trailing ` ``` ` before JSON.loads.
- **Reddit public JSON works fine with the right User-Agent** at this volume — 14 posts ingested first try, HTTP 200, no rate limit hits. The `User-Agent: aiteam-research-opportunity/0.1 (contact via repo issue)` form is acceptable. Stayed below the 60 req/min unauth ceiling by a factor of ~100.
- **Smoke-state rollback after operator-impact tests matters.** Approval handler smoke wrote a line to `drafter_queue.txt` and flipped two pain rows to queued/rejected. Rolled both back so operator's first real `/approve_pain` lands in clean state. Always snapshot+restore when smoke-testing operator-facing surfaces.

## Operator corrections

- Operator did not surface any in-session corrections during build. Pre-flight forks were pinned upfront in the GO message; build executed against those pins without deviation.

## What's next

- **Session 2 (deferred to next CC session):**
  - `poll_stackexchange.sh` — api.stackexchange.com v2.3, tagged-question pull
  - `poll_youtube.sh` + auto-discovery of seed videos (monthly `0 9 1 * *` cron)
  - Per-source triage prompt tuning (YouTube comments noisier than Reddit/SE)
  - `digest.sh` — weekly Sonnet synthesis + Telegram delivery
  - Cron wiring: `0 * * * *` poll.sh, `0 9 * * 1` digest.sh, `0 9 1 * *` poll_youtube.sh --discovery
  - Flip `RESEARCH_OPPORTUNITY_ENABLED=true` once Session 2 lands clean
- **Operator next-action:** review 7 pain points currently in `pain_points` table (`sqlite3 ~/agents/research-opportunity/pain_points.db "SELECT id, our_coverage, asbestos_relevance_score, proposed_slug, pain_text FROM pain_points;"`). Decide whether any are worth queuing now (via `/approve_pain <id>` once bot restarts, or via direct CLI: `bash ~/agents/research-opportunity/apply_pain_approval.sh <id> approve`). Even one approval = first live end-to-end test of the Reddit → Sonnet → drafter_queue pipeline.
- **Blocked:** none. Master kill switch (`RESEARCH_OPPORTUNITY_ENABLED=false`) is the only thing preventing autonomous operation, and Session 2 cron wiring is the natural moment to flip it.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-17_1735.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-17 17:36 (D068 watchdog hygiene + D067 SSG plan recovery + D056 editor production runner + GEO Optimizer Phases 1 & 2 + Internal Link Agent v1) **Last session summary:** Closed three deferred items and shipped a new verdict-gate agent. D068 watchdog hygiene — 3 refactor commits (compute_threshold helper, drop `_stale` qualifier, single-load state.json cache) all byte-identical except FIX B's narrow intentional stale-path diff; live tick id=1171 clean. D067 SSG data-driven rewrite plan recovered from chat history to `~/brain/projects/aiteam/docs/` (brain commit `b1c4214`). D056 editor production runner shipped end-to-end: `score.sh` + threshold tune 3.8→3.6 per 4-slug burn-in + ship.sh editor gate at step 2.5 (commits `6895dcc`, `dc05c51`, `1323040`); SQL-trace integration test (live blocked by whitelist coverage of all 32 approved slugs); 3 new D-items opened (D072 truncation, D073 rubric redundancy, D074 live-gate verification). GEO Optimizer Phases 1 + 2 shipped: 8-axis rubric + `geo-optimizer/score.sh` (Phase 1, commit `0335838`) + ship.sh GEO gate at step 2.4 BEFORE editor gate at 2.5 (Phase 2, commit `32b3ea3`); smoke test on white-asbestos-vs-blue scored composite 3.88 PASS with predicted floors confirmed (tables=1, quotations=1, other 6 axes 4-5). Stack now: `validate → GEO (2.4) → editor (2.5) → dry-run/stage/build/push`. Sister chat shipped internal-link agent v1 mid-session (`928671a` + `ec3f087`) — not yet inspected. **Prior session summary (preserved):** Watchdog agent built and activated. Drafter heartbeat row (`fb53d04`); watchdog scaffold (`88f8d7a`); cron activated 11:02 PT; first production tick fired 11:15:00 alerting once on orchestrator (real signal — `diary_written` missing in 26h). Four follow-on fixes landed: state-corrupt guard (`1053913`); grace_minutes wired (`38ed525`); drafter check_window 2h→1h (`26d3311`); LESSONS.md docs (`1e90165` brain). SSG site repo located at `~/projects/smartsourceguide/`.  --- 
