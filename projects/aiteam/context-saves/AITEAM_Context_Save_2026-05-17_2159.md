# AITEAM Context Save — 2026-05-17_2159
**Generated:** 2026-05-17T21:59:26-0700
**Since last save:** 2026-05-17 19:22:54
**Session topic:** Keyword Registry agent build (single-source-of-truth markdown registry with alias collapse, 4 production hooks, nightly rebuild cron). Plus pain_points markdown log + digest archive for the Research/Opportunity Agent (earlier in same wall-clock window). Sister chat shipped Backlink Prospector v0.1 (b873996) — visible in git activity, not built by this chat.

---

## Mechanical record

### Git activity since last save
```
6047fe0 feat(keyword-registry): registry + backfill + writer hooks + reader shortcut + nightly rebuild cron
3f7145e feat(research-opportunity): brain-tracked pain_points log + digest archive + posts_scanned
b873996 feat(backlink-prospector): Backlink Agent Part 2 (resource-page prospector) v0.1
```

### Files changed
```
A	backlink-prospector/agent.yaml
A	backlink-prospector/CLAUDE.md
A	backlink-prospector/filter.sh
A	backlink-prospector/rank.sh
A	backlink-prospector/run.sh
A	backlink-prospector/search_patterns.yaml
A	backlink-prospector/search.sh
A	keyword-registry/CLAUDE.md
A	keyword-registry/lib/backfill_registry.sh
A	keyword-registry/lib/rebuild_registry.py
A	keyword-registry/lib/update_registry.py
A	lib/migrations/2026-05-17_outbound_link_prospects.sql
A	research-opportunity/lib/backfill_log.sh
A	research-opportunity/lib/write_pain_log.py
M	.gitignore
M	internal-link/apply_approval.sh
M	lib/cron.txt
M	research-opportunity/digest.sh
M	research-opportunity/extract.sh
M	ship-to-site/deploy_batch.sh
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
|  id  |         ts          |       actor_id       |            action            |                  target                  |
|------|---------------------|----------------------|------------------------------|------------------------------------------|
| 1369 | 2026-05-17 21:45:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 1368 | 2026-05-17 21:30:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 1367 | 2026-05-17 21:30:00 | assignment-drafter   | drafter_tick_started         |                                          |
| 1366 | 2026-05-17 21:18:02 | keyword-registry     | keyword_registry_rebuilt     |                                          |
| 1365 | 2026-05-17 21:18:01 | keyword-registry     | rebuild_tick_started         |                                          |
| 1364 | 2026-05-17 21:15:00 | watchdog             | watchdog_check_complete      | _summary                                 |
| 1363 | 2026-05-17 21:13:56 | keyword-registry     | registry_updated_via_shipped | test-slug                                |
| 1362 | 2026-05-17 21:13:56 | keyword-registry     | keyword_registry_rebuilt     |                                          |
| 1361 | 2026-05-17 21:13:18 | keyword-registry     | keyword_registry_rebuilt     |                                          |
| 1360 | 2026-05-17 21:13:17 | keyword-registry     | rebuild_tick_started         |                                          |
| 1359 | 2026-05-17 21:12:58 | keyword-registry     | keyword_registry_rebuilt     |                                          |
| 1358 | 2026-05-17 21:12:11 | keyword-registry     | keyword_registry_rebuilt     |                                          |
| 1357 | 2026-05-17 21:12:11 | keyword-registry     | keyword_registry_drift       |                                          |
| 1356 | 2026-05-17 21:00:04 | research-opportunity | poll_tick_completed          |                                          |
| 1355 | 2026-05-17 21:00:04 | research-opportunity | stackexchange_poll_completed | stackexchange                            |
| 1354 | 2026-05-17 21:00:01 | research-opportunity | reddit_poll_completed        | reddit                                   |
| 1353 | 2026-05-17 21:00:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 1352 | 2026-05-17 21:00:00 | assignment-drafter   | drafter_tick_started         |                                          |
| 1351 | 2026-05-17 21:00:00 | research-opportunity | poll_tick_started            |                                          |
| 1350 | 2026-05-17 20:45:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 1349 | 2026-05-17 20:30:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 1348 | 2026-05-17 20:30:01 | assignment-drafter   | drafter_tick_started         |                                          |
| 1347 | 2026-05-17 20:15:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 1346 | 2026-05-17 20:00:06 | ship-to-site         | preview_ping_sent            | _empty                                   |
| 1345 | 2026-05-17 20:00:06 | notify               | notify_sent                  | operator                                 |
| 1344 | 2026-05-17 20:00:05 | ship-to-site         | ship_to_site_skipped         | white-asbestos-vs-blue-asbestos          |
| 1343 | 2026-05-17 20:00:05 | ship-to-site         | ship_to_site_skipped         | when-was-asbestos-used-in-homes          |
| 1342 | 2026-05-17 20:00:05 | ship-to-site         | ship_to_site_skipped         | what-does-asbestos-siding-look-like      |
| 1341 | 2026-05-17 20:00:05 | ship-to-site         | ship_to_site_skipped         | vermiculite-insulation-guide             |
| 1340 | 2026-05-17 20:00:05 | ship-to-site         | ship_to_site_skipped         | transite-pipe-guide                      |
| 1339 | 2026-05-17 20:00:05 | ship-to-site         | ship_to_site_skipped         | is-popcorn-ceiling-asbestos              |
| 1338 | 2026-05-17 20:00:05 | ship-to-site         | ship_to_site_skipped         | how-to-test-popcorn-ceiling-for-asbestos |
| 1337 | 2026-05-17 20:00:04 | ship-to-site         | ship_to_site_skipped         | how-to-dispose-of-asbestos               |
| 1336 | 2026-05-17 20:00:04 | ship-to-site         | ship_to_site_skipped         | house-built-1976-asbestos                |
| 1335 | 2026-05-17 20:00:04 | ship-to-site         | ship_to_site_skipped         | friable-vs-nonfriable-asbestos           |
| 1334 | 2026-05-17 20:00:04 | ship-to-site         | ship_to_site_skipped         | does-plaster-have-asbestos               |
| 1333 | 2026-05-17 20:00:04 | ship-to-site         | ship_to_site_skipped         | chrysotile                               |
| 1332 | 2026-05-17 20:00:04 | ship-to-site         | ship_to_site_skipped         | black-mastic-guide                       |
| 1331 | 2026-05-17 20:00:04 | research-opportunity | poll_tick_completed          |                                          |
| 1330 | 2026-05-17 20:00:04 | research-opportunity | stackexchange_poll_completed | stackexchange                            |
| 1329 | 2026-05-17 20:00:04 | ship-to-site         | ship_to_site_skipped         | asbestos-vs-fiberglass                   |
| 1328 | 2026-05-17 20:00:03 | ship-to-site         | ship_to_site_skipped         | asbestos-under-carpet                    |
| 1327 | 2026-05-17 20:00:03 | ship-to-site         | ship_to_site_skipped         | asbestos-tile-guide                      |
| 1326 | 2026-05-17 20:00:03 | ship-to-site         | ship_to_site_skipped         | asbestos-siding-removal-cost             |
| 1325 | 2026-05-17 20:00:03 | ship-to-site         | ship_to_site_skipped         | asbestos-siding-guide                    |
| 1324 | 2026-05-17 20:00:03 | ship-to-site         | ship_to_site_skipped         | asbestos-shingles-guide                  |
| 1323 | 2026-05-17 20:00:03 | ship-to-site         | ship_to_site_skipped         | asbestos-roof-removal                    |
| 1322 | 2026-05-17 20:00:03 | ship-to-site         | ship_to_site_skipped         | asbestos-remediation-cost                |
| 1321 | 2026-05-17 20:00:02 | ship-to-site         | ship_to_site_skipped         | asbestos-popcorn-ceiling-vs-non-asbestos |
| 1320 | 2026-05-17 20:00:02 | ship-to-site         | ship_to_site_skipped         | asbestos-popcorn-ceiling-removal-cost    |

### Token usage
|           model           | in_tok | out_tok | cost_usd |
|---------------------------|--------|---------|----------|
| claude-haiku-4-5-20251001 | 21593  | 5327    | 0.1885   |
| claude-sonnet-4-6         | 3      | 1307    | 0.054    |

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

- **Single polymorphic registry table with markdown storage** (over SQLite). Operator pinned in pre-flight. Reasons: brain-tracked → auto-committed nightly to private GitHub vault, phone-readable in Obsidian, agents read/write via existing tmp+replace pattern. No new schema migration needed.
- **Source-of-truth model: hybrid** (Fork A). Write-through at ship/approval time + nightly drift-detection rebuild. Pure write-through risks silent drift if any agent fails; pure derivative would re-scan 35sec per read at 500-slug scale. Hybrid catches drift without read latency.
- **Aliases CANNOT be deferred** (Fork B). Data forced it: 26/32 slugs touch at least one asbestos-type term (chrysotile 5-15× per article in 20+ slugs). A registry treating "white asbestos" and "chrysotile" as separate would under-count by 80%+. Seeded 14 alias rules in v1.
- **Pre-Sonnet shortcut implemented conservatively** (Fork G). Operator said "skip Sonnet if registry says covered" but the only pre-Sonnet keyword candidate is the post title (Sonnet's `proposed_kw` is its OUTPUT, not input). Resolved by matching post title against multi-word primary keywords only (31 of 162 registry rows qualify). Single-word "asbestos" would over-match; 2+ word phrases keep false-positive risk low. **No live data tested yet** — first real-world test fires when extract.sh runs on freshly-triaged comments.
- **v1 update_registry delegates writes to full rebuild** (~0.5 sec). Per-row UPDATE primitives are deferred. Reason: 162-keyword regen is fast enough that per-row bookkeeping wasn't worth the complexity. Stable API surface so v2 can swap implementation without touching callers.
- **All 4 production hooks are best-effort.** deploy_batch.sh, apply_approval.sh, extract.sh shortcut: every registry update wrapped in `|| true` + audit. Registry is observability, not a gate. If write fails, the deploy/approval/extract still succeeds.
- **Deterministic digest beat Sonnet-synthesized digest on reflection** (Session 2 carry-over, mentioned for the pain_points log build). Pre-flight envisioned Sonnet writing the digest; in-build switched to Python template. $0/digest, no fence-wrapping risk, testable, trivially deterministic.

## Lessons learned

- **bash 3.2 chokes on `case` inside `while` inside `$()` with shell-special patterns.** The `*"$KW"*) printf ... ;;` construct produced `syntax error near unexpected token ';;'` even though the syntax is well-formed. Replaced with a Python heredoc one-liner — cleaner and bash-3.2-safe. General rule: when you find yourself nesting `case`/`while`/`$()` with quoted patterns, escape to Python early rather than fighting the parser.
- **A "pre-Sonnet shortcut" needs a pre-Sonnet keyword candidate.** The natural read of the operator's spec was "look up `proposed_kw` against registry before Sonnet" — but `proposed_kw` is what Sonnet generates. The only pre-Sonnet signal is the post title. Future shortcut specs should clarify: which field gets looked up, and at which point in the pipeline. (Title-based was the right resolution here; documenting in case a future shortcut spec hits the same ambiguity.)
- **Markdown table tolerates 162 rows fine in Obsidian.** Was a worry in pre-flight. Reality: file is 176 lines, sorts alphabetically by keyword, scans cleanly on phone. Won't need a split until ~1000+ keyword rows.
- **"Auto-maintained, do not edit" warnings at the top of brain files matter.** Operator workflow includes scrolling Obsidian and casually editing notes. Without an explicit warning, a hand-edit to keyword_registry.md would be silently overwritten on the next 03:00 cron tick. Standard pattern for all agent-owned brain files going forward: H1, then italicized "Auto-maintained by `<agent>`. Do not edit manually — your changes will be overwritten." before any content.
- **Alias-collapse is critical even at small scale.** v1 has 14 alias rules but they drive significant correctness. Without alias collapse, "chrysotile" would have ~120 mentions in registry (its surface form) and "white asbestos" would be a separate ~40-mention row, hiding the fact that they refer to the same thing. The collapsed total of 160 in one row matches operator's mental model of "site coverage of this concept."
- **`update_registry.py` as single helper with multiple verbs (shipped/refreshed/backlinked/lookup) is better than 4 separate scripts.** Bash callers don't need to know which file to invoke; one `update_registry.py <verb> <arg>` call site. Stable surface lets v2 swap full-regen-per-call for per-row UPDATE without touching any caller. Generalize: any agent that exposes 3+ related operations should consider single-script + verb instead of multi-script.
- **Proxy-cron-fire with restricted PATH continues to be the right Phase-N test.** Used same pattern as Session 2 (`PATH=/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin <script>`). Verified all 3 expected audit rows landed (`rebuild_tick_started`, `keyword_registry_rebuilt`, `registry_updated_via_shipped`). No need to wait 5+ hours for natural 03:00 cron fire.

## Operator corrections

- No mid-session corrections. Pre-flight forks A-I were pinned upfront. The one fork-vs-reality tension (Fork G "skip Sonnet" + "no pre-Sonnet keyword candidate" ambiguity) was resolved by building the title-based interpretation and reporting honestly in the summary; operator did not push back.

## What's next

- **Operator next moves:**
  1. Verify cron tick at next 03:00 PT — `tail -f ~/agents/keyword-registry/logs/rebuild.log` + `sqlite3 ~/store/aiteam.db "SELECT action, payload_json FROM audit_log WHERE actor_id='keyword-registry' AND ts > strftime('%s','now','-1 hour');"`
  2. Resolve `white-asbestos-vs-blue-asbestos` notes flag — either create the missing `keyword-configs/white-asbestos-vs-blue-asbestos.json` OR accept the h1-derived primary_kw and remove the notes flag (one-line edit to override at backfill).
  3. Watch for first `registry_shortcut_hit` audit row when extract.sh next processes a freshly-triaged Reddit/SE post with a title matching a primary multi-word keyword. If frequency is unexpectedly high → false-positive review.
  4. Watch the 2026-05-25 09:00 PT digest delivery to confirm Telegram (bot needs to be running by then).
- **Sister-chat watch:** `backlink-prospector/` (commit b873996) shipped this same wall-clock window from a sister chat — not inspected by this chat. Worth a recon next session to understand overlap with internal-link or keyword-registry.
- **Deferred (already in DEFERRED.md):** D078 Sonnet cost in digest header, D079 Haiku CLI overhead propagation, D080 YouTube triage threshold tune-up. No new deferred items from this session.
- **Blocked:** none.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-17_1921.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-17 19:25 (Research/Opportunity Agent Session 2 live: 3 sources, weekly digest, cron firing hourly) **Last session summary:** Shipped Session 2 of Research/Opportunity Agent. SE poller (149 questions ingested), YouTube auto-discovery (10 videos pinned, 330k-86k views), YouTube comment poller (529 comments, 1 video commentsDisabled handled), tightened YouTube triage prompt (Haiku praise-hallucination persists but Sonnet extract proven to act as reliable second gate — id 12 scored 0.10 with "No homeowner pain question present"). Weekly digest renders cleanly (10 pains across 3 sources, em-dash dividers, phone-readable). Cron wired: hourly poll + weekly Monday 09:00 digest + monthly auto-discovery. Master kill switch flipped `RESEARCH_OPPORTUNITY_ENABLED=true`. Proxy-cron-fire verified 4 audit_log rows with restricted PATH env. Bug fix: stray SHA on .env line 45 was breaking every `source ~/agents/config/.env` call across the fleet — normalized to `SERPER_API_KEY=<value>`. **Prior session summary (preserved):** Session 1 shipped Reddit-only vertical slice (14 posts, 11 triaged, 7 pain_points, /approve_pain handler) at commit 2413a94. D075 Opportunity Scout deferred at brain commit 83cb3ed. 7 architectural decisions pinned upfront, zero scope drift. 3 bash bugs fixed + documented (apostrophe-in-heredoc, sqlite-newlines, PyYAML-PEP668-venv). HANDOFF + LESSONS + ctx save committed at brain 1bf7c9e. Prior to Session 1: D068 watchdog hygiene + D067 SSG plan recovery + D056 editor production runner + GEO Optimizer Phases 1 & 2 + Internal Link Agent v1.  --- 
