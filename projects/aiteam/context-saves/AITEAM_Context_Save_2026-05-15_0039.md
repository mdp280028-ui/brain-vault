# AITEAM Context Save — 2026-05-15_0039
**Generated:** 2026-05-15T00:39:30-0700
**Since last save:** all time (first save)
**Session topic:** Phase 1 closure + first ctx save (inaugural ctx.sh run, captures all of Phase 1 retrospectively)

---

## Mechanical record

### Git activity since last save
```
8ffd923 Step 17 prep: editor agent (agent.yaml + CLAUDE.md), rubric.md (5-axis 1-5 anchors for ad-revenue content), run_tier_test.sh (dual-tier scoring loop + Pearson correlation report). Awaiting operator corpus + manual scores before firing the live tier test.
893fe97 Step 10d: /api/diary endpoint + Diary section on dashboard. Day selector + content render. Path traversal guard via strict YYYY-MM-DD regex. Fixed local-vs-UTC bug: today default uses local time to match BSD date in write_diary.sh. Verified: today's diary loads, future/bogus days return exists:false, traversal returns 400.
33d839c Step 16 + 10f-live: librarian agent (Haiku tier) with inbox sorter. Fixed bash 3.2 declare -A (same bug class as 9b). Live verified: 4 fixtures classified 100% correctly (2 moved, 2 skipped) with substantive reasoning. Differential @mention test passed: 'hi' fans to both, '@orchestrator' isolates Sonnet, '@librarian' isolates Haiku. All warroom_transcript / hive_mind / audit_log rows present.
491f5e8 Step 15: write_diary.sh (orchestrator daily diary). DAY_START fix (BSD date needs explicit 00:00:00). End-to-end verified: 1455-byte diary written, audit_log id=5. Known gap surfaced: cost line shows $0.00 because token_usage is unwired (will fix at 10c).
230864f Step 10f: @mention parser + descriptive 422 on unmatched mentions. Server returns {unmatched, registered} for clear UX. Parser verified across 7 cases including email-not-mention. Live differential test deferred until step 16 lands a second agent.
8ed1d15 Step 10e: /api/warroom/standup endpoint + chat UI + run_agent.sh canonical agent invocation. Live verified end-to-end: orchestrator responded (4540ms, exit 0), rows landed in warroom_transcript / hive_mind / conversation_log / audit_log, dashboard /api/agents reflects last_seen. Per-agent 45s + watchdog 300s timeouts wired.
38815f6 Step 14: orchestrator agent (agent.yaml + CLAUDE.md). End-to-end smoketest passed: CLAUDE.md auto-loaded, agents/ enumerated, librarian correctly identified for inbox-sort routing.
b8e6bd5 Step 12: cron.txt with the 5 Phase 0 schedule lines. Operator must paste via crontab -e. All entries are local (no API cost). Note added for macOS Full Disk Access if Sequoia sandbox blocks cron.
a30ee05 Step 11: backup_db.sh (.backup, retain 28) + backup_daily.sh (rsync to /Volumes/AgentSSD) + backup_weekly.sh (rsync to /Volumes/WDArchive). Tested: snapshot integrity ok; daily/weekly fail gracefully when drives unmounted.
a67ece7 Step 10a (minimal): :3141 dashboard (Hono + node:sqlite). Endpoints: /api/health, /api/audit-log, /api/agents, /api/tokens. Token-guarded. dashboard.sh launcher. Used node:sqlite (Node 26 native) because better-sqlite3 fails to gyp against V8 26. Kanban/cost-chart/diary/war-room deferred to Phase 1 when there's data.
d0b0201 Step 10: rebuild_command_center.sh. Renders system status / today's spend / pending tasks / audit-log tail / today's diary. Cron lands in step 12.
4fa62c1 Step 9d: check_kill_switches.sh (mtime-refresh wrapper). Tested initial source + post-touch re-source + no-change skip.
1cc3333 Step 9c: skills/ bootstrap (README + sqlite SKILL.md). Path-scoped rules convention documented.
c8bd10d Step 9b: convert_dropzone.sh (pandoc/pdftotext/xlsx2csv/jq). Rewrote for bash 3.2 portability, explicit PATH for cron, robust mv. Installed pandoc + pipx + xlsx2csv. Tested with .txt.
9d18f94 Step 8: retrieve_context.sh (wiki + RAG stub + hybrid). Fixed wiki_search to tolerate empty matches under set -euo pipefail.
e65ba1e Step 7: init_db.sql (WAL + 10 tables + FTS5 with content-column-restricted UPDATE trigger). DB created at ~/store/aiteam.db; first audit row inserted.
c14687b Step 6: log_to_audit.sh helper (audit-log insert, returns correlation_id). Test deferred to step 7 after DB exists.
872c471 Step 5: notify.sh stub (echoes [notify stub] <msg> until grammy in Phase 2)
831e94f Step 4: ai-do.sh / ai-think.sh / ai-cheap.sh wrappers (tested, all 3 tiers return 'test')
9430db3 Step 3: .env scaffold (chmod 600, gitignored). Operator must fill ANTHROPIC_API_KEY and GEMINI_API_KEY before step 4 tests.
b0d019f Step 2: folder structure (~/brain, ~/agents, ~/store, ~/.claudeclaw-backups)
a0e9863 Step 1: verify Claude Code (v2.1.142)
```

### Files changed
```
A	_template/.gitkeep
A	.gitignore
A	config/.gitkeep
A	dashboard/package.json
A	dashboard/server.js
A	editor/.gitkeep
A	editor/agent.yaml
A	editor/CLAUDE.md
A	editor/rubric.md
A	editor/run_tier_test.sh
A	lib/.gitkeep
A	lib/ai-cheap.sh
A	lib/ai-do.sh
A	lib/ai-think.sh
A	lib/backup_daily.sh
A	lib/backup_db.sh
A	lib/backup_weekly.sh
A	lib/check_kill_switches.sh
A	lib/convert_dropzone.sh
A	lib/cron.txt
A	lib/dashboard.sh
A	lib/init_db.sql
A	lib/log_to_audit.sh
A	lib/notify.sh
A	lib/rebuild_command_center.sh
A	lib/retrieve_context.sh
A	lib/run_agent.sh
A	librarian/.gitkeep
A	librarian/agent.yaml
A	librarian/CLAUDE.md
A	librarian/run_librarian.sh
A	orchestrator/.gitkeep
A	orchestrator/agent.yaml
A	orchestrator/CLAUDE.md
A	orchestrator/write_diary.sh
A	skills/.gitkeep
A	skills/README.md
A	skills/sqlite/SKILL.md
A	STEP1_VERIFIED.md
D	config/.gitkeep
D	editor/.gitkeep
D	librarian/.gitkeep
D	orchestrator/.gitkeep
D	skills/.gitkeep
D	STEP1_VERIFIED.md
M	.gitignore
M	dashboard/server.js
```

### Diary entries since last save
- 2026-05-13.md: 2026-05-13
- 2026-05-14.md: 2026-05-14


### Agent activity (audit_log)
| id |         ts          |     actor_id     |         action          |                            target                            |
|----|---------------------|------------------|-------------------------|--------------------------------------------------------------|
| 55 | 2026-05-14 23:52:56 | phase_1          | phase_1_complete_v2     | aiteam_phase_1                                               |
| 54 | 2026-05-14 23:52:18 | phase_1          | phase_1_complete        | aiteam_phase_1                                               |
| 53 | 2026-05-14 23:45:43 | step_18          | step_18_completed       | warroom_discuss_mode                                         |
| 52 | 2026-05-14 23:44:51 | warroom          | discuss_completed       | 83a5434d-5007-402f-b749-47118860b15d                         |
| 51 | 2026-05-14 23:44:51 | warroom          | discuss_round_completed | 83a5434d-5007-402f-b749-47118860b15d                         |
| 50 | 2026-05-14 23:44:34 | warroom          | discuss_round_started   | 83a5434d-5007-402f-b749-47118860b15d                         |
| 49 | 2026-05-14 23:44:34 | warroom          | discuss_round_completed | 83a5434d-5007-402f-b749-47118860b15d                         |
| 48 | 2026-05-14 23:44:14 | warroom          | discuss_round_started   | 83a5434d-5007-402f-b749-47118860b15d                         |
| 47 | 2026-05-14 23:44:14 | warroom          | discuss_started         | 83a5434d-5007-402f-b749-47118860b15d                         |
| 46 | 2026-05-14 23:43:51 | warroom          | discuss_completed       | 95542664-3497-44e6-80e8-3b1ed7502480                         |
| 45 | 2026-05-14 23:43:51 | warroom          | discuss_round_completed | 95542664-3497-44e6-80e8-3b1ed7502480                         |
| 44 | 2026-05-14 23:43:32 | warroom          | discuss_round_started   | 95542664-3497-44e6-80e8-3b1ed7502480                         |
| 43 | 2026-05-14 23:43:32 | warroom          | discuss_round_completed | 95542664-3497-44e6-80e8-3b1ed7502480                         |
| 42 | 2026-05-14 23:42:56 | warroom          | discuss_round_started   | 95542664-3497-44e6-80e8-3b1ed7502480                         |
| 41 | 2026-05-14 23:42:56 | warroom          | discuss_started         | 95542664-3497-44e6-80e8-3b1ed7502480                         |
| 40 | 2026-05-14 23:42:23 | warroom          | discuss_completed       | c2545b6e-f235-410e-a750-b70543dc4443                         |
| 39 | 2026-05-14 23:42:23 | warroom          | discuss_round_completed | c2545b6e-f235-410e-a750-b70543dc4443                         |
| 38 | 2026-05-14 23:41:58 | warroom          | discuss_round_started   | c2545b6e-f235-410e-a750-b70543dc4443                         |
| 37 | 2026-05-14 23:41:58 | warroom          | discuss_round_completed | c2545b6e-f235-410e-a750-b70543dc4443                         |
| 36 | 2026-05-14 23:41:23 | warroom          | discuss_round_started   | c2545b6e-f235-410e-a750-b70543dc4443                         |
| 35 | 2026-05-14 23:41:23 | warroom          | discuss_started         | c2545b6e-f235-410e-a750-b70543dc4443                         |
| 32 | 2026-05-14 23:33:18 | step_19          | step_19_gaps_closed     | phase2_gaps_documented                                       |
| 28 | 2026-05-14 23:30:12 | orchestrator     | diary_written           | /Users/mmm2/brain/projects/aiteam/diary/2026-05-13.md        |
| 27 | 2026-05-14 23:26:38 | step_19          | step_19_completed       | failure_modes_docs                                           |
| 26 | 2026-05-14 23:14:49 | step_17b         | step_17b_completed      | warroom_slash_commands                                       |
| 25 | 2026-05-14 23:14:05 | warroom          | slash_completed         | 67af8e16-092b-4253-85c2-617a639e7119                         |
| 24 | 2026-05-14 23:13:56 | warroom          | slash_started           | 67af8e16-092b-4253-85c2-617a639e7119                         |
| 23 | 2026-05-14 23:13:56 | warroom          | slash_completed         | 83884a56-95cd-4465-9446-f23b753bb3c9                         |
| 22 | 2026-05-14 23:13:56 | warroom          | slash_started           | 83884a56-95cd-4465-9446-f23b753bb3c9                         |
| 21 | 2026-05-14 23:13:47 | warroom          | slash_unknown           | 051beccb-5b2f-4719-b895-44bde116b946                         |
| 20 | 2026-05-14 23:07:12 | step_10c         | step_10c_completed      | dashboard_cost_chart                                         |
| 19 | 2026-05-14 22:57:20 | step17_tier_test | tier_test_completed_v2  | editor_test_corpus                                           |
| 18 | 2026-05-14 22:57:20 | step17_tier_test | tier_test_retracted     | audit_log_id_17                                              |
| 17 | 2026-05-14 22:20:31 | step17_tier_test | tier_test_completed     | editor_test_corpus                                           |
| 16 | 2026-05-14 21:00:00 | warroom          | standup_completed       | c6a9a786-5af5-433b-9f15-a2c947117674                         |
| 15 | 2026-05-14 20:59:55 | warroom          | standup_started         | c6a9a786-5af5-433b-9f15-a2c947117674                         |
| 14 | 2026-05-14 20:59:54 | warroom          | standup_completed       | d0e9f0b5-40be-4a6d-8f01-3d3ac772d648                         |
| 13 | 2026-05-14 20:59:51 | warroom          | standup_started         | d0e9f0b5-40be-4a6d-8f01-3d3ac772d648                         |
| 12 | 2026-05-14 20:59:50 | warroom          | standup_completed       | 98536844-fb29-4af1-8287-89e6e91fa14c                         |
| 11 | 2026-05-14 20:59:47 | warroom          | standup_started         | 98536844-fb29-4af1-8287-89e6e91fa14c                         |
| 10 | 2026-05-14 20:59:24 | librarian        | sort_completed          |                                                              |
| 9  | 2026-05-14 20:59:24 | librarian        | sort_skip               | todo-bullets.md                                              |
| 8  | 2026-05-14 20:59:24 | librarian        | sort_skip               | loose-url.md                                                 |
| 7  | 2026-05-14 20:59:24 | librarian        | sort_move               | /Users/mmm2/brain/projects/aiteam/inbox/journal-fragment-202 |
|    |                     |                  |                         | 6-05-13.md -> /Users/mmm2/brain/projects/aiteam/wiki/notes/j |
|    |                     |                  |                         | ournal-fragment-2026-05-13.md                                |
| 6  | 2026-05-14 20:59:24 | librarian        | sort_move               | /Users/mmm2/brain/projects/aiteam/inbox/idea-membership-site |
|    |                     |                  |                         | -fragment.md -> /Users/mmm2/brain/projects/aiteam/wiki/oppor |
|    |                     |                  |                         | tunities/idea-membership-site-fragment.md                    |
| 5  | 2026-05-14 20:49:49 | orchestrator     | diary_written           | /Users/mmm2/brain/projects/aiteam/diary/2026-05-14.md        |
| 4  | 2026-05-14 20:45:50 | warroom          | standup_completed       | b9922d52-0ca1-4f52-9ae1-c8c326472df8                         |
| 3  | 2026-05-14 20:45:46 | warroom          | standup_started         | b9922d52-0ca1-4f52-9ae1-c8c326472df8                         |
| 2  | 2026-05-14 20:37:07 | orchestrator     | smoketest_passed        | step14                                                       |
| 1  | 2026-05-14 19:56:11 | chassis          | db_initialized          | /Users/mmm2/store/aiteam.db                                  |

### Token usage
|           model           | in_tok | out_tok | cost_usd |
|---------------------------|--------|---------|----------|
| claude-sonnet-4-6         | 61     | 15767   | 0.9864   |
| claude-haiku-4-5-20251001 | 67366  | 7005    | 0.2076   |

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session
- **Sonnet 4.6 as production editor judge** (audit row 19). Real Sonnet measured r=0.933 composite vs operator/Opus baseline; MAE 0.363. Sits in the published-empirical-judge band (0.85–0.95). Alternative considered: stay on Opus for r≈1.0 but pay 2.7× per scoring pass. Cost savings compound at scale; clarity-axis drop (0.982→0.623) reflects Sonnet's stingier use of top score, not discrimination failure.
- **Skip JSONL historical backfill** (step 10c). Operator chose log-forward-only after weighing complexity of project-path filtering, model attribution from session logs, and cache pricing edge cases — historical data was only one day anyway.
- **`ai-do.sh` pinned to `--model sonnet`** explicitly. Was silently defaulting to Opus 4.7 per the user's Claude Code config inheritance — invalidated step 17 v1's "Sonnet vs Opus r=0.995" finding (it was Opus vs Opus).
- **Slash commands auto-discover from filesystem** (step 17b). Drop a new `<name>.sh` into `~/agents/orchestrator/commands/`, no server restart needed. Less code in `server.js`.
- **Discuss-mode round-2+ prompts explicitly require engagement** (step 18, per operator note). Each agent must name a specific prior claim and either agree-with-detail, push-back, or build-on-it. Without this, multi-round devolves into parallel monologues — verified in test outputs.
- **Append-only audit log convention preserved when bugs are found** (step 19 gaps closed). The 16 corrupted-payload rows from the `${5:-{}}` brace bug were left as-is; corrective row 32 written instead of UPDATEing history. Same pattern applied to row 54 (empty payload from a python env-passing error) → superseded by row 55, not deleted.

## Lessons learned
- **Verify wrapper model attribution BEFORE shipping cost-sensitive decisions.** `ai-do.sh` had no `--model` flag and silently used `claude -p`'s default (which inherits whatever the parent Claude Code session is configured for — Opus 4.7 here). Made an entire tier test invalid. Now every wrapper has its model pinned explicitly. Future fix: add a runtime assertion that the returned `modelUsage` keys match the expected tier.
- **Bash heredoc `${5:-{}}` has brace-matching ambiguity.** When `$5` is non-empty, the expansion only consumes one closing brace and appends a literal `}` — corrupted 16 audit_log payloads silently. Use the explicit pattern: `PAYLOAD="${5-}" ; [ -z "$PAYLOAD" ] && PAYLOAD="{}"`.
- **`claude -p --output-format json` returns `total_cost_usd` and per-model breakdown directly in `modelUsage`.** No need to build a pricing map. Use Claude Code's own numbers — they're authoritative and update with model price changes automatically.
- **Claude Code routes some work to Haiku even on Sonnet/Opus calls.** The `modelUsage` dict has multiple keys per call (Haiku for the routing/compression step + the primary model for the response). Logging needs one row per model in `modelUsage`, not one row per call, or per-tier breakdown lies.
- **Operator-suggested fixes deserve verification before implementing literally.** The "export AGENT_ID env var before wrapper exec" suggestion wouldn't have stuck — wrappers do `AGENT_ID="${2:-shell}"` which shadows the env. Equivalent fix is passing as `$2`. Always check whether the suggested mechanism actually achieves the stated goal.
- **Schema mismatches in specs vs reality silently break SQL.** This ctx-system spec used `timestamp` / `cost_usd` / `title`; real schema is `ts` / `estimated_cost_usd` / `description`. Schema-check before writing queries, not after.
- **Empty-data graceful handling is load-bearing for credibility.** `/weekly_review` early-exits when <2 days of diary; doesn't fire LLM. Better to honestly say "not enough data" than hallucinate a review. Same pattern applies to "(no open mission tasks)" rather than a blank section.
- **macOS bash is 3.2.57.** No `${var,,}` lowercase, no associative arrays, no `mapfile`/`readarray`. `${var/pattern/}` works. Process substitution `< <(...)` works. `case` patterns work for filename matching.
- **Append-only audit logs are recoverable; mutable audit logs aren't.** When a bug corrupts history, write a corrective row pointing back. Never UPDATE or DELETE. The historical corruption is itself a fact worth recording.

## Operator corrections
- **Step 19 rubric tracking claim was wrong.** I documented `rubric.md` as "not git-tracked, recoverable only from backup." Operator said *"Add ~/agents/editor/rubric.md to git tracking. Whatever repo the brain or agents live in, make sure rubric.md is committed and shows up in git log."* — turned out it was already tracked (commit `8ffd923`). Lesson: verify before documenting gaps.
- **Step 17 first conclusion was invalid on premise.** I reported "Sonnet wins, 5× cheaper than Opus" with r=0.995. Operator's response was to lock the choice; only on the *next* step did I discover the silent-Opus bug and have to retract. Operator's note: *"Don't pre-commit to either."* — I'd jumped to a recommendation when the evidence was structurally compromised.
- **Step 18 cost cap discipline.** Operator capped testing at 2 runs / $0.90. I burned a 3rd run due to a client-side JSON parse mistake. Operator didn't push back but it's on me to count my own spend more carefully — would have stayed on budget by saving the response to a file the first time.
- **Step 18 round-2 prompt design.** Operator's note: *"include explicit instruction telling each agent to acknowledge or push back on at least one specific point from round 1 (not just 'respond to discussion'). Without this, multi-round risks devolving into parallel monologues."* — I would have written a weaker prompt without this nudge. Verified at test time: round-2 outputs cite specific prior claims by name.

## What's next
- **Phase 1 closed** (audit row 55). No active build threads.
- **10b (kanban) deferred** until `mission_tasks` has rows.
- **17a (silver platters) deferred** until first AITEAM site is live with 30+ days of GA4 data. Per operator: building summary jobs for empty data sources is premature.
- **Phase 2 gaps documented** at `~/brain/projects/aiteam/docs/phase2_gaps.md`: detection cron infrastructure, budget enforcement hook, per-agent switch bypass closure, `PUBLISHER_DEPLOY_ENABLED` enforcement.
- **Next session likely picks up** when first site is live, or when operator drops a fresh prompt with a Phase 2 trigger.

---

## Where to resume
**Last context save before this one:** (first save)
**Active HANDOFF.md state:** # AITEAM HANDOFF  Last updated: 2026-05-14  ## Where we are 
