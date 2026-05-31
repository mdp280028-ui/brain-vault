# AITEAM Context Save — 2026-05-30_2204
**Generated:** 2026-05-30T22:04:57-0700
**Since last save:** 2026-05-25 09:38:32
**Session topic:** domain-hunter payroll site (4th topical-fit target) + 3 read-only fleet audits (agent liveness/auto-restart, stuck pause flags, market-chain 2026-05-29 skip RCA)

---

## Mechanical record

### Git activity since last save
```
f8e3a84 docs(domain-hunter): log payroll site + ai-cheap cost-floor note; site count 3->4
5c35fd7 chore(domain-hunter): backfill 28 candidates with payroll topical scores
11b4a07 feat(domain-hunter): add payrolldetective.com as 4th topical-fit site
f0daccc fix(ai-do): bump default AI_TIMEOUT_SEC 1200 -> 1800
be2fe28 agents: auto-commit 2026-05-27
dd8c867 ship-to-site(ssg): rich-schema bring-up - dual-mode scorers, gate skips, awk type-annotation fix
2eb9e26 ship-to-site(ssg): template-skip + category-subdir + slug regex superset
```

### Files changed
```
A	content-loop/.gitignore
A	content-loop/agent.yaml
A	content-loop/CLAUDE.md
A	content-loop/lib/_helpers.sh
A	content-loop/lib/pick_candidate.sh
A	content-loop/lib/run_one.sh
A	content-loop/loop_driver.sh
A	domain-hunter/score/backfill_payroll.py
M	domain-hunter/CLAUDE.md
M	domain-hunter/sites.yaml
M	editor/score.sh
M	geo-optimizer/score.sh
M	lib/ai-do.sh
M	ship-to-site/config/ssg.yaml
M	ship-to-site/lib/rollback.sh
M	ship-to-site/lib/stage.sh
M	ship-to-site/lib/validate.sh
M	ship-to-site/ship.sh
```

### Diary entries since last save
- 2026-05-27.md: (no heading)
- 2026-05-29.md: 2026-05-29


### Agent activity (audit_log)
|  id  |         ts          |       actor_id       |            action            |                  target                  |
|------|---------------------|----------------------|------------------------------|------------------------------------------|
| 4592 | 2026-05-30 22:00:03 | research-opportunity | poll_tick_completed          |                                          |
| 4591 | 2026-05-30 22:00:03 | research-opportunity | stackexchange_poll_completed | stackexchange                            |
| 4590 | 2026-05-30 22:00:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 4589 | 2026-05-30 22:00:01 | research-opportunity | poll_source_failed           | reddit                                   |
| 4588 | 2026-05-30 22:00:01 | research-opportunity | reddit_fetch_failed          | reddit                                   |
| 4587 | 2026-05-30 22:00:00 | assignment-drafter   | drafter_tick_started         |                                          |
| 4586 | 2026-05-30 22:00:00 | research-opportunity | poll_tick_started            |                                          |
| 4585 | 2026-05-30 21:45:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 4584 | 2026-05-30 21:30:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 4583 | 2026-05-30 21:30:00 | assignment-drafter   | drafter_tick_started         |                                          |
| 4582 | 2026-05-30 21:15:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 4581 | 2026-05-30 21:10:02 | notify               | notify_sent                  | operator                                 |
| 4580 | 2026-05-30 21:10:01 | notify               | notify_sent                  | operator                                 |
| 4579 | 2026-05-30 21:00:03 | research-opportunity | poll_tick_completed          |                                          |
| 4578 | 2026-05-30 21:00:03 | research-opportunity | stackexchange_poll_completed | stackexchange                            |
| 4577 | 2026-05-30 21:00:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 4576 | 2026-05-30 21:00:01 | research-opportunity | poll_source_failed           | reddit                                   |
| 4575 | 2026-05-30 21:00:01 | research-opportunity | reddit_fetch_failed          | reddit                                   |
| 4574 | 2026-05-30 21:00:00 | assignment-drafter   | drafter_tick_started         |                                          |
| 4573 | 2026-05-30 21:00:00 | research-opportunity | poll_tick_started            |                                          |
| 4572 | 2026-05-30 20:45:00 | watchdog             | watchdog_check_complete      | _summary                                 |
| 4571 | 2026-05-30 20:30:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 4570 | 2026-05-30 20:30:00 | assignment-drafter   | drafter_tick_started         |                                          |
| 4569 | 2026-05-30 20:15:01 | watchdog             | watchdog_check_complete      | _summary                                 |
| 4568 | 2026-05-30 20:10:02 | notify               | notify_sent                  | operator                                 |
| 4567 | 2026-05-30 20:10:01 | notify               | notify_sent                  | operator                                 |
| 4566 | 2026-05-30 20:00:07 | research-opportunity | poll_tick_completed          |                                          |
| 4565 | 2026-05-30 20:00:07 | research-opportunity | youtube_poll_completed       | youtube                                  |
| 4564 | 2026-05-30 20:00:05 | ship-to-site         | preview_ping_sent            | _empty                                   |
| 4563 | 2026-05-30 20:00:05 | notify               | notify_sent                  | operator                                 |
| 4562 | 2026-05-30 20:00:04 | ship-to-site         | ship_to_site_skipped         | answerconnect-review                     |
| 4561 | 2026-05-30 20:00:04 | ship-to-site         | ship_to_site_skipped         | white-asbestos-vs-blue-asbestos          |
| 4560 | 2026-05-30 20:00:04 | ship-to-site         | ship_to_site_skipped         | when-was-asbestos-used-in-homes          |
| 4559 | 2026-05-30 20:00:04 | ship-to-site         | ship_to_site_skipped         | what-does-asbestos-siding-look-like      |
| 4558 | 2026-05-30 20:00:04 | ship-to-site         | ship_to_site_skipped         | vermiculite-insulation-guide             |
| 4557 | 2026-05-30 20:00:04 | ship-to-site         | ship_to_site_skipped         | transite-pipe-guide                      |
| 4556 | 2026-05-30 20:00:04 | ship-to-site         | ship_to_site_skipped         | is-popcorn-ceiling-asbestos              |
| 4555 | 2026-05-30 20:00:03 | ship-to-site         | ship_to_site_skipped         | how-to-test-popcorn-ceiling-for-asbestos |
| 4554 | 2026-05-30 20:00:03 | ship-to-site         | ship_to_site_skipped         | how-to-dispose-of-asbestos               |
| 4553 | 2026-05-30 20:00:03 | ship-to-site         | ship_to_site_skipped         | house-built-1976-asbestos                |
| 4552 | 2026-05-30 20:00:03 | ship-to-site         | ship_to_site_skipped         | friable-vs-nonfriable-asbestos           |
| 4551 | 2026-05-30 20:00:03 | ship-to-site         | ship_to_site_skipped         | does-plaster-have-asbestos               |
| 4550 | 2026-05-30 20:00:03 | ship-to-site         | ship_to_site_skipped         | chrysotile                               |
| 4549 | 2026-05-30 20:00:03 | research-opportunity | youtube_comments_disabled    | 5f7medwuIMA                              |
| 4548 | 2026-05-30 20:00:03 | ship-to-site         | ship_to_site_skipped         | black-mastic-guide                       |
| 4547 | 2026-05-30 20:00:03 | ship-to-site         | ship_to_site_skipped         | asbestos-vs-fiberglass                   |
| 4546 | 2026-05-30 20:00:02 | ship-to-site         | ship_to_site_skipped         | asbestos-under-carpet                    |
| 4545 | 2026-05-30 20:00:02 | research-opportunity | stackexchange_poll_completed | stackexchange                            |
| 4544 | 2026-05-30 20:00:02 | ship-to-site         | ship_to_site_skipped         | asbestos-tile-guide                      |
| 4543 | 2026-05-30 20:00:02 | ship-to-site         | ship_to_site_skipped         | asbestos-siding-removal-cost             |

### Token usage
|           model           | in_tok | out_tok | cost_usd |
|---------------------------|--------|---------|----------|
| claude-sonnet-4-6         | 442    | 306700  | 12.9791  |
| claude-haiku-4-5-20251001 | 761350 | 90856   | 2.4915   |

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

**domain-hunter — payroll site (commits `11b4a07` / `5c35fd7` / `f8e3a84`):**
- Added **`payrolldetective.com` as the 4th topical-fit site** (`id: payroll`) in `sites.yaml`, mirroring the exact 3-field shape (`id`/`domain`/`niche` block scalar/`related_topics` list). **No scorer code change** — `topical_fit.py` `load_sites()`+`build_prompt()` scale off the site list, so a 4th entry auto-becomes a 4-line prompt/output. Gate unchanged (`topical_fit_min` stays 0.3 for all sites). Verified via test-mode: `payrollexperts.com` → `payroll` 0.9, `ust…` → `ust`, junk → `none`.
- **Backfill scope = payroll-only merge on the 28 already-scored rows** (preserve existing ust/asbestos/ssg scores; merge `payroll` key into `topical_fit_json`; recompute `best_site_match` across all 4). The 46 `topical_fit_max IS NULL` rows left for natural full-4-site scoring on next scan — deliberately NOT force-scored. Result: **0 of 28 routed to payroll** (all 28 are prior pharma/foreign/spam junk — correctly still `none`). Persisted the one-off as committed `score/backfill_payroll.py` (precedent: `backfill_topical.py`).
- **Skipped the costly `scan.sh --dry-run`** — after the cost surprise (below) the operator chose to commit on the deterministic test-mode routability proof instead, saving ~$0.27–0.50.
- **Docs commit message made accurate, not as-briefed** — the requested "fix the v0.1.5/v0.2 mislabel" was a **no-op: no such mislabel exists**. The 4 referenced commits (5a31df0/8c4ec29/4f554d9/c2d5a49) are already correctly attributed to CLAUDE.md's `## v0.1.5 changes` section; the only `v0.2` ref is the genuinely-future Pro-membership item. Did the *real* staleness fix instead (three `3 sites` → `4 sites` references) + added the ai-cheap cost-floor note.

**Three read-only audits (report-only, zero changes — operator scoped each as READ-ONLY):**
- **Agent liveness/auto-restart audit** → A/B/C bucketing. Key finding: **bucket C is empty** — the grammy Telegram bot (operator's stated top concern) is *already* under launchd (`com.aiteam.grammy-bot`, PID live, exit 0, KeepAlive+RunAtLoad). Dashboard likewise. Everything else is correctly cron (bucket B).
- **Stuck-pause-flag check** → nothing gating. `~/store/flags/` empty, `SYSTEM_PAUSED=false`, `LLM_SPAWN_ENABLED=true`, spend $0.58/$1.00. Cleared nothing (nothing stale). `PUBLISHER_DEPLOY_ENABLED=false` is an intentional Phase-2 gate, not a stale pause.
- **Market-chain 2026-05-29 skip RCA** → **(c) the box was DOWN** 07:00–08:45 (booted 09:48:10; first audit row all day = 10:00:01). NOT a chain bug — no scribe error, no YouTube 403, no queue issue; the script never executed. macOS cron has no catch-up. The task's premise ("domain-hunter ran 06:33 today") was wrong — that run was **05-28**.

## Lessons learned

- **`ai-cheap.sh` has a ~$0.018/Haiku-call cost floor** from the wrapper's agent system prompt: every call logs `cache_read ≈ 30K + cache_write ≈ 10K tokens` regardless of how tiny the user prompt is. Batch Haiku cost is **wrapper-dominated, not prompt-dominated** — estimate batch jobs at **~$0.018 × n calls**, never by prompt size. The 28-row payroll backfill cost **$0.51**, not the "<$0.01" a prompt-size estimate suggested (~50× off, blew a $0.15 cap). Now documented in `domain-hunter/CLAUDE.md`.
- **Before running an N-call LLM batch under a cost cap, run ONE call and read `token_usage` to extrapolate** real cost. I (and the operator) locked in a `<$0.01` premise that was wrong by ~50×; a single-call measurement would have caught `28 × $0.018 = $0.51` before spending.
- **macOS cron does NOT catch up missed firings** (no anacron). A reboot/sleep across a job's window silently drops it. **Always check box uptime (`kern.boottime` / `uptime` / `last reboot`) before diagnosing a "skipped agent" as a code/quota failure.** The 2026-05-29 market "failure" was just the box down 07:00–08:45 (booted 09:48). (launchd `RunAtLoad` *does* recover — dashboard + grammy-bot are covered; cron agents are not.)
- **A "last run 06:33" timestamp can be yesterday's** — confirm the DATE (`datetime(ts,'unixepoch','localtime')`), not just the time, before concluding cron fired today. The "domain-hunter ran 06:33 today" premise was actually 05-28.
- **For idle long-poll bots, use a `launchctl_label` PID probe, not logfile-mtime** for liveness — the grammy bot reads healthy via PID even though it writes no logs while idle (the watchdog already does this correctly).
- **When a task's premise is wrong** (a mislabel that doesn't exist, a run that didn't happen), report the discrepancy and make the deliverable/commit message accurate rather than fabricating the requested fix. Honest ⚠ beats premature ✅.

## Operator corrections

- **Pre-flight gate respected then approved:** operator had me STOP and report the 4 pre-flight items before any edit, then locked: payroll-only backfill on the 28 rows; use REAL-token cost not the $0.02/call nominal guard. (The real cost then surprised us both — see lessons.)
- **After the cost surprise, operator chose "option 3 — commit on test-mode proof, no scan."** Skip the costly dry-run; rely on deterministic test-mode routability proof. (I had surfaced the overrun via AskUserQuestion before spending more.)
- **The "v0.1.5/v0.2 mislabel" doesn't exist** — I reported back that CLAUDE.md already attributes the 4 commits to v0.1.5 correctly; the only `v0.2` reference is the legitimately-future item. The operator's `DEFERRED.md` **D093** note still asserts "CLAUDE.md still mislabels v0.1.5 as v0.2" — that belief is stale and worth reconciling.

## What's next

- **domain-hunter:** payroll site is live; first organic payroll match will appear when a payroll-relevant domain is fetched (none in the current 74-row set — all 28 scored junk routed `none`).
- **Two genuine reliability follow-ups surfaced by the audits (NOT reboot-related):**
  1. **market-briefer silently stale since 2026-05-25** — it did *not* emit `briefer_run` in the 05-28 cycle when scribe/curator did. Separate from the 05-29 box-down skip. Needs a look (last in chain to deliver to Telegram).
  2. **orchestrator diary (`45 23` daily) last ran 05-24** (~5 days) — silent-dead; watchdog flags `missing`.
- **Optional hardening:** add a watchdog probe for `com.aiteam.dashboard` (KeepAlive-protected but no liveness probe — a wedged-but-alive server wouldn't alert). Already noted in `dashboard/CLAUDE.md` as a hardening follow-up.
- **Reconcile D093** in `DEFERRED.md` — drop the "CLAUDE.md mislabels v0.1.5 as v0.2" clause (verified false this session).
- **Blocked:** none.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-25_0935_hardening.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-25 09:40 (TWO parallel chats merged: **hardening lane** [D092 issues nested-logs+TG pager / D026 dashboard launchd / D094 lockfile+timeout] + **verification/build lane** [outstanding-work audit → DEFERRED bookkeeping / internal-link SSG-fy v2 + D091 / orchestrator memory / typing indicator / dropzone + D025]) **Last session summary:** Reliability + operator-facing hardening across the fleet — cron hybrids now self-serialize and bound hung claude calls, the dashboard auto-restarts under launchd, nested-agent failures surface correctly on /issues and page Telegram, and the orchestrator gained conversation memory + a typing indicator + a safe write/photo dropzone.  **Hardening lane:** Three recon-gated builds. **D092** — issues-capture resolves nested-agent log paths (recon found actor_ids are the BARE child name `scribe`, not `market/scribe`, and nested agents write `*.log` in-dir not under `log/`): `resolve_agent_dir` globs one level deep; same `cd` bug fixed in the dashboard CC-prompt route. `notify_telegram.sh` pages `new_issues.log` to Telegram (high-water on `audit_log_id`, 10/tick, **no dashboard link** per operator), `*/5` cron live. **D026** — dashboard (last always-on process on nohup) → launchd `com.aiteam.dashboard`; `lib/dashboard.sh` wraps `launchctl`; logs → `dashboard/log/`. **D094** — lockfile + timeout, **zero new deps** (`shlock`+`perl alarm`): timeout centralized in the 3 `ai-*.sh` wrappers (`AI_TIMEOUT_SEC` default **1200s**), shlock locks on drafter + market chain + idea-agent. Commits: agents `20611fd/24b7cd7/eaa4fe9/fb33435/633f6dc`; brain `1d89771/adfe1e5/904992f`. ctx: `context-saves/AITEAM_Context_Save_2026-05-25_0935_hardening.md`. 
