# AITEAM — Context Save 2026-05-30 — Payroll Site + Liveness Audit + Issues Severity Tiering

**Session date:** 2026-05-30
**Focus:** domain-hunter 4th site (payroll), full agent liveness/auto-restart audit, TG-spam fix via issues-capture severity tiering
**Parallel chats:** one D092 collision earlier in the week (sister chat claimed it for Issues Queue v1.1); not an issue this session.

---

## Headline

Three things shipped/closed tonight, plus two false alarms traced to root cause:

1. **domain-hunter now scores 4 sites** — added payrolldetective.com (id: payroll) as 4th topical-fit site. Live as of commits 11b4a07 / 5c35fd7 / f8e3a84.
2. **Full liveness + auto-restart audit** — system is healthy. Both daemons (grammy bot, dashboard) are under launchd with KeepAlive + RunAtLoad. Everything else is cron (survives reboot, fires on schedule). Bucket C (needs-launchd-but-isn't) is EMPTY. The auto-restart concern that prompted the audit is already solved.
3. **Issues-capture severity tiering** — TG now alerts ONLY on hard failures; soft issues go dashboard-only + a daily 06:05 digest. Fixes the ~90% TG-spam problem. Commits b2b5956 / d33e4e6.

Two "agents didn't run" scares both traced to **the Mac being powered off during the morning cron window** (booted 09:48 on 05-29), NOT bugs. Cron has no catch-up; missed runs are silently lost.

---

## Decisions Made

### domain-hunter payroll site
- Added payroll as 4th site in sites.yaml mirroring exact 3-field schema (id / domain / niche block scalar / related_topics list). id `payroll` lands in best_site_match.
- Gate unchanged: topical_fit_min = 0.3 for all 4 sites.
- Backfilled the 28 already-scored rows with payroll key (0 routed to payroll — existing rows are all pharma/foreign/spam junk; correct).
- The 46 NULL rows get full 4-site scoring naturally on next scan — did NOT force-score them.
- Payroll routability proven via test-mode (payrollexperts.com → payroll 0.9), NOT via a live scan. So payroll is wired but not yet proven end-to-end on a real fetched domain — first real test is whenever a payroll-relevant domain surfaces in a scan (days/weeks given narrow niches).

### Issues severity tiering (the TG-spam fix)
- Root problem: issues-capture's notify pipe fired on EVERY captured failure. With Reddit 403ing hourly, ~85-90% of TG alerts were the same dead Reddit failure.
- **Architecture chosen:** severity column on agent_issues, set at capture time. Hard → TG page. Soft → dashboard-only (still inserts to /issues, still writes new_issues.log for tail visibility) + daily digest.
- **Operator decision (ask_user_input):** ONLY hard failures (agent crash/timeout/pipeline death) buzz the phone. Everything else dashboard-only. Daily digest added so the silent pile isn't invisible.
- **Severity classifier (env-overridable, no code surgery to promote/demote):**
  - SUPPRESS_SOURCES = "reddit" (payload.source → always soft)
  - HARD_ACTIONS = "timeout_kill,pipeline_died,config_synth_failed"
  - CORE_PIPELINE = "writer,auditor,ship-to-site,assignment-drafter,config-synthesizer,orchestrator,content-loop" (substring match on agent_id)
  - Decision order, first match wins: suppress-source→soft; poll_source_failed/*_fetch_failed→soft; notify_failed→soft; HARD_ACTIONS→hard; error+core→hard; empty-output/FATAL→hard; *_failed+core→hard; default→soft (anti-spam bias).
  - is_core substring match means asbestos-writer→hard via "writer"; auto-covers future *-writer/*-auditor sites.
- Gate lives in notify_telegram.sh (reads severity, skips non-hard). Single patch point because new_issues.log is the ONLY thing feeding TG.
- Daily digest: issues-capture/digest.sh, cron `5 6 * * *`, reuses lib/notify.sh (watchdog digest pattern). Silent when 0 new soft AND 0 open hard.

---

## Files Created / Modified (Mac Mini)

- `~/agents/domain-hunter/sites.yaml` — added payroll 4th site
- `~/agents/domain-hunter/CLAUDE.md` — payroll changelog + ai-cheap cost-floor note; 3→4 site count fix
- `~/agents/issues-capture/capture.sh` — severity classifier
- `~/agents/issues-capture/notify_telegram.sh` — severity gate (skip non-hard)
- `~/agents/issues-capture/digest.sh` — NEW, daily soft-issue digest
- `~/agents/lib/cron.txt` — added digest cron line
- DB: `agent_issues` ALTER TABLE ADD COLUMN severity TEXT
- Safety backup left: `~/store/aiteam.db.bak-severity-*` (outside repo, deletable)

### Commits
- `11b4a07` feat(domain-hunter): add payrolldetective.com as 4th topical-fit site
- `5c35fd7` chore(domain-hunter): backfill 28 candidates with payroll topical scores
- `f8e3a84` docs(domain-hunter): log payroll site + ai-cheap cost-floor note; site count 3→4
- `b2b5956` feat(issues-capture): severity tiering — hard→TG, soft→dashboard-only
- `d33e4e6` feat(issues-capture): daily soft-issue digest

---

## Liveness Audit Findings (read-only, no changes)

- **Bucket A (launchd, survives reboot + auto-restart):** com.aiteam.grammy-bot (PID 1366), com.aiteam.dashboard (PID 1357). Both KeepAlive=SuccessfulExit=false + RunAtLoad. Plists in ~/Library/LaunchAgents/, actually loaded.
- **Bucket B (cron, survives reboot, fires on schedule, NO crash-restart, NO catch-up):** everything else — domain-hunter, market chain, idea-agent, ship-to-site, watchdog, issues-capture, research-opportunity, tg-monitor, drafter, autocommits, diary, keyword-registry, notes-agent.
- **Bucket C (needs launchd but isn't):** EMPTY. No always-on process is unsupervised.
- Folders not independently scheduled (pipeline-invoked or manual): editor, geo-optimizer, internal-link, librarian, config-synthesizer, content-loop, backlink-prospector.
- Optional future hardening (not built): dashboard watchdog probe (KeepAlive doesn't catch a wedged-but-alive HTTP server).

### Full agent schedule (all PT, Mac must be awake)
- 00:00 cost-cap reset | 03:00 keyword-registry backfill | 06:00 watchdog digest | 06:05 issues digest (NEW) | 06:30 domain-hunter
- 07:00 market scribe / tg-monitor analyzer / ship-to-site digest | 07:10 idea-agent | 07:15 market process_queue | 08:15 market curator | 08:45 market briefer
- 20:00 ship preview ping | 23:00 ship deploy batch | 23:45 orchestrator diary | 23:50 agents-autocommit | 23:55 brain-autocommit
- */5 tg-monitor reader + issues-capture | */15 watchdog | */30 assignment-drafter | hourly research-opportunity poll
- Mon 09:00 research weekly digest | Sun 10:00 notes-agent sort | monthly 1st 09:00 research YouTube discovery
- **Heavy cluster 06:30–08:45** — this whole block is lost if box is off mornings.

---

## Issues Queue Triage (the 109)

- 109 open, status distribution 100% open (nothing ever moved to resolved/wontfix before tonight).
- **92 (85%) = Reddit 403 noise** — reddit_fetch_failed (47) + poll_source_failed (45) are ONE root cause double-counted per hourly tick. Reddit blocks unauthenticated fetches now (policy, not a bug).
- **15 = one bad night (05-27/28 off-day cluster)** — timeout_kill, content-loop error, config_synth_failed, assignment_draft_failed, ship_to_site_stage_failed. Already self-resolved; root cause was AI_TIMEOUT_SEC, addressed by commit f0daccc (1200→1800s bump). Zero recurrence in 2+ days.
- **2 stragglers** — notify_failed (self-healed retry), digest_sent (success wrongly flagged).
- **0 active bugs.**
- Backlog closed: 93 wontfix (92 reddit + digest_sent), 16 resolved (15 cluster + notify_failed). New open count = 0.

---

## Cost Data Points (Max-sub value, real $ = $0)

- **ai-cheap.sh has a ~$0.018/call FLOOR** from the agent system-prompt cache overhead (cache_read ~30K + cache_write ~10K tokens per call) REGARDLESS of prompt size. Batch Haiku cost is **wrapper-dominated, not prompt-dominated.** The payroll backfill cost $0.5116 (28 × ~$0.018), ~50× over the "<$0.01 tiny prompt" estimate Claude + operator both locked in. Lesson: estimate batch Haiku jobs at ~$0.018/call, NOT by token count. Measure ONE call, read token_usage, extrapolate before running the batch. (Now in domain-hunter CLAUDE.md; belongs in LESSONS.md.)
- 7-day spend: $45.58 (Max-sub value). Sonnet $35.11, Haiku $6.58, Opus $3.89, 679 calls. Biggest: curator_cowen, asbestos writer/auditor.
- notify path health: 62 notify_sent vs 1 notify_failed in 24h — reliable.

---

## Operator Corrections to Claude This Session

- **The "v0.1.5/v0.2 mislabel" was a GHOST.** Claude carried a stale note saying domain-hunter CLAUDE.md mislabeled v0.1.5 items as v0.2. CC checked git history — labels were already correct (commits all 2026-05-24, attributed correctly). No edit needed; CC fixed only the real 3→4 site-count staleness. Lesson: don't carry stale memory notes into CC instructions as fact.
- **Stale-timestamp-read-as-today caused TWO false diagnoses.** Claude claimed "domain-hunter ran 06:33 today so box was on" — that 06:33 was YESTERDAY (05-28). The box was off until 09:48 on 05-29. This sent CC chasing a market-chain bug that didn't exist. **Lesson: always confirm the literal date on a last-run row before reasoning from it.** Both "agents didn't run" scares were off-days, not failures.
- Operator was right to be skeptical / want to just wait. Claude over-pushed to diagnose on a bad premise.

---

## DEFERRED ITEMS

- **D093 (existing) — domain-hunter v0.2:** ExpiredDomains.net Pro membership ($10/mo) for TF/CF/RDs + auction prices + direct URLs + backlink-weighted scoring. Gated on (a) 2-4 weeks of v0.1.5 signal AND (b) ≥1 domain acquired via current flow. UNCHANGED.
- **NEW — Reddit poller re-auth:** research-opportunity Reddit source is dark (403 on unauthenticated fetch). Needs Reddit API credentials to restore pain-point pulls. Optional — StackExchange + YouTube still work. Low priority; only matters if operator wants Reddit data back.
- **NEW — soft-issue dashboard accumulation:** soft issues (Reddit etc.) still insert to /issues silently, so the open count WILL climb again over coming days. By design. If it becomes annoying, lever is (a) periodic auto-close of soft issues older than N days, OR (b) suppress-at-capture so Reddit never hits dashboard at all. Don't build until digest-alone proves insufficient.
- **NEW — payroll end-to-end proof:** payroll routing proven in test-mode only; not yet proven on a real fetched domain via live scan. Confirm whenever first payroll candidate surfaces.
- **NEW — ai-cheap cost-floor → LESSONS.md:** the ~$0.018/call floor lesson is in domain-hunter CLAUDE.md; promote to LESSONS.md at a brain-touching session.
- **NEW (optional hardening) — dashboard watchdog probe:** com.aiteam.dashboard is KeepAlive-protected but has no expected_schedule entry, so a wedged-but-alive HTTP server wouldn't alert. Lowest-effort monitoring gap.
- **NEW — tg-monitor analyzer audit_log invisibility:** analyzer logs only to its own analyzer.log, writes no audit_log row, so it's invisible to audit-based monitoring (and the "registered agents" dashboard table). Cosmetic; runs fine.
- **Stale dashboard "Registered agents" table:** shows all agents as last-seen 5/14 with 0 events — broken/stale view, NOT evidence of dead agents (audit_log + token_usage prove daily runs). Cosmetic dashboard bug.

---

## Resume Point

System is healthy and quiet. Next natural threads (none urgent):
1. Watch for first real payroll candidate to prove domain-hunter 4th site end-to-end.
2. Confirm the 06:05 issues digest lands tomorrow morning with correct counts.
3. Optional: Reddit poller re-auth if pain-point data wanted back.
4. Promote ai-cheap cost-floor lesson to LESSONS.md next brain session.

Mission bar unchanged: $200/mo to break even on Max sub. Revenue still $0. No monetization enrolled (no AdSense, no affiliates).
