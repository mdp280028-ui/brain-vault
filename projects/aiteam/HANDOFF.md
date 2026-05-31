# AITEAM Project Handoff
**Last updated:** 2026-05-30 22:38 (issues-capture severity tiering + 109-backlog close)
**Last session summary:** Shipped severity tiering for issues-capture — hard failures fire Telegram, soft (reddit/fetch noise, transient) go dashboard-only and are summarized by a new once-daily digest — then retro-tagged and bulk-closed the entire 109-issue backlog to a clean baseline (open 109 → 0). Two read-only diagnostic passes (morning cron-block liveness, issues-queue triage) preceded and scoped the build.

**This session (issues-capture severity tiering):** New `agent_issues.severity ∈ {hard,soft}` set at capture time (`ALTER TABLE`, additive). `capture.sh` classifies via env-overridable lists (`SUPPRESS_SOURCES=reddit`, `HARD_ACTIONS=timeout_kill,pipeline_died,config_synth_failed`, `CORE_PIPELINE=writer,auditor,ship-to-site,…`; `is_core` = substring match so `asbestos-writer`→hard). `notify_telegram.sh` pages **only** `severity='hard'` (reads the column as source of truth, advances HWM past skipped soft). Soft still inserts to `/issues` + still writes `new_issues.log` (tail-visibility) — only the page is gated. New `digest.sh` (cron `5 6 * * *`, reuses `lib/notify.sh` like `watchdog/digest.sh`) sends one daily soft-pile summary, silent on zero. Commits `b2b5956` (severity tiering: capture.sh + notify_telegram.sh) / `d33e4e6` (digest.sh + lib/cron.txt). Backlog close was a DB-only op (no repo artifact): 109→0 open. ctx: `context-saves/AITEAM_Context_Save_2026-05-30_2238.md`.

**This session (read-only diagnostics, report-only):** (1) **Morning cron-block liveness** → box was AWAKE the whole 06:00–09:00 window (booted 05-29 09:48, no sleep/wake transitions, caffeinate holding it up); all 8 morning agents ran clean; not a failure. (2) **Issues-queue triage** → 109 open = 92 reddit-403 noise (one root cause, double-logged) + 15-row self-resolved 05-27/28 cluster + 2 stragglers; **0 active bugs**. This directly scoped the tiering work above.

---

## Current phase
**issues-capture severity tiering shipped + backlog at clean baseline (0 open). Fleet-reliability lane unchanged: two genuine silent-dead agents (market-briefer, orchestrator-diary) still awaiting a fix pass; prior hardening lane still awaiting operator-side live smoke.**

## Completed (cumulative)
- [x] **issues-capture severity tiering (hard→TG, soft→dashboard-only + daily digest)** — `severity` column; env-overridable classifier; notify gate on `severity='hard'`; `digest.sh` (`5 6 * * *`, silent on zero). Backlog 109→0 (92 reddit `wontfix`, 15 cluster `resolved`, 2 stragglers). `b2b5956/d33e4e6`. Tested: classifier + gate on isolated temp DB (timeout_kill/pipeline_died/content-loop-error/asbestos-writer→PAGE; reddit→SKIP), digest dry-run 48/0 matches manual count. NOT tested: literal end-to-end phone ping (deliberate, anti-spam).
- [x] **Two read-only diagnostic passes** — morning cron-block liveness (box awake, all agents clean) + issues-queue triage (109 = 92 noise + 15 stale + 2, zero active bugs). Report-only.
- [x] **domain-hunter payroll site (4th topical-fit target)** — `payrolldetective.com` / `id: payroll` in `sites.yaml`; gate unchanged (0.3); no scorer code change; 28 rows backfilled (0→payroll, all junk); test-mode routability proven. `11b4a07/5c35fd7/f8e3a84`.
- [x] **Three read-only fleet audits (prior session)** — liveness A/B/C, pause-flag check, market 05-29 skip RCA (box-down, not a bug).
- [x] **D092 — issues nested log paths + Telegram pager.** `resolve_agent_dir` (bare-name→one-level glob); `notify_telegram.sh` (`*/5`, high-water); dashboard `cd` fix. `20611fd/24b7cd7/eaa4fe9`, brain `1d89771`.
- [x] **D026 — dashboard launchd auto-restart.** `com.aiteam.dashboard.plist`; `lib/dashboard.sh`→launchctl; verified respawn ~1s. `fb33435`, brain `adfe1e5`.
- [x] **D094 — hybrid lockfile + timeout.** `shlock` locks + perl-alarm timeout in the 3 `ai-*.sh` wrappers (`AI_TIMEOUT_SEC` default 1200, later 1800). `633f6dc`, brain `904992f`.
- [x] **D091 closed** — render-aware `audit_links.sh`. `ff63975`, brain `e4d132e`.
- [x] **Internal-link SSG-fy v2** — multi-site schema migration, `site_lib.sh`, all scripts `--site`-aware. `934dc76`.
- [x] **F17/F18 closed**, D045→D050 rename, **D087** weekly→daily (`6192499`), **D093** added. brain `1c35674`.
- [x] **Orchestrator conversation memory** (`9f7c5c4/dc1dfc4`), **typing indicator** (`1627643`), **dropzone** (in `74c1cde`) + D025.
- [x] **Prior (carried):** idea-agent v1, notes-agent v1, /model picker, market pipeline cron (D033), watchdog, Issues Queue v1, domain-hunter v0.1.5, SSG pipeline, ship-to-site, orchestrator/telegram bot, D066/D072.

## In progress
- [ ] **Watch first real soft-issue digest fire at 06:05 (05-31)** — confirm one message, sane counts, and that NO per-event reddit TG fires overnight via the `*/5` notify path (all reddit should now be soft→skipped).
- [ ] **Fix the two silent-dead agents** (carried, NOT reboot-related): **market-briefer** stale since 2026-05-25 (no `briefer_run` on 05-28 when scribe/curator ran); **orchestrator diary** (`45 23` daily) last ran 05-24. Real execution gaps.
- [ ] **Operator-side live smoke of the three new phone agents** (carried) — `/model` round-trip; notes text+image + forward /yes /no; spot-check idea-agent proposals.
- [ ] **Forwarded-photo dropzone archive** (carried) — committed, never run live; confirm `image_received forwarded:true` on next forward.

## Blocked
- none

## Known bugs
| Bug | Severity | File(s) | Notes |
|---|---|---|---|
| **market-briefer silently stale since 2026-05-25** | medium | `market/briefer/brief.sh` | Did NOT emit `briefer_run` in the 05-28 cycle when scribe/curator ran. Chain's TG-delivery endpoint — investigate next. |
| **orchestrator diary silent-dead (~6d)** | medium | `orchestrator/write_diary.sh` | `45 23` daily; last `diary_written` 2026-05-24. Watchdog flags `missing`. |
| `com.aiteam.dashboard` has no watchdog liveness probe | minor | `watchdog/expected_schedule.yaml` | KeepAlive-protected but a wedged-but-alive server wouldn't alert. Add a `launchctl_label` probe. |
| `/issues` re-accumulates soft (reddit) rows over time | by design | `issues-capture/capture.sh` | Soft issues are captured silently (not suppressed); dashboard open count climbs, surfaced via daily digest. If noisy, add auto-aging of old soft rows. |
| Memory: vision/photo turns get no conversation context | minor (v1.1) | `lib/run_agent.sh`, `telegram/bot.js` | Text path only. |
| `audit_links.sh` no dedup | minor (v1.1) | `internal-link/audit_links.sh` | Repeated audits accumulate duplicate `stripped` rows. |
| `idea_calibration` duplicate per-week rows under daily cadence | minor | `idea-agent/calibrate.sh` | D087. |
| D044 `log_to_audit.sh` apostrophe escape | live workaround | `lib/log_to_audit.sh` | APOS scrub works; parameterized rewrite deferred. |
| Photo album `media_group_id` not in inbox markdown | minor (v1.1) | `notes-agent/capture.sh`, `bot.js` | D089. |
| approved-index drift on escalated slugs | medium | `content/run-batch.sh` | Open as **D090**. Check before next batch. |
| domain-hunter godaddy fetcher non-producing | by design | `domain-hunter/fetch/godaddy.py` | Akamai 403; exits 4, continues on expireddomains. |
| `/issues` clipboard copy unverified headlessly | unknown | `dashboard/server.js` | Needs an operator browser tap. |

## Architecture decisions (recent)
| Decision | Reasoning | Date |
|---|---|---|
| issues-capture: severity tiering, soft = captured-but-silent (not suppressed) | hard failures must page; reddit/fetch noise must not spam but must stay visible — `/issues` + daily digest, not dropped at capture | 2026-05-30 |
| Severity classifier lists are env-overridable (`SUPPRESS_SOURCES`/`HARD_ACTIONS`/`CORE_PIPELINE`) | promote/demote actions, agents, sources without code surgery | 2026-05-30 |
| TG gate reads `agent_issues.severity` in the notify step; capture is the sole classifier | one source of truth, no driftable second classification in delivery | 2026-05-30 |
| `is_core` = substring token match (`asbestos-writer`→`writer`) | derived per-niche agents still page on real `*_failed` (over-suppression guard) | 2026-05-30 |
| domain-hunter adds sites by config only (`sites.yaml`), no scorer code change | `topical_fit.py` scales off the site list | 2026-05-30 |
| Estimate batch Haiku at ~$0.018/call (wrapper floor), not by prompt size | ai-cheap.sh logs ~30K cache_read + ~10K cache_write per call regardless of prompt | 2026-05-30 |
| Diagnose "skipped agent" by checking box uptime FIRST | macOS cron has no catch-up; a reboot across the window silently drops the job | 2026-05-30 |
| AI-call timeout centralized in the 3 `ai-*.sh` wrappers; `AI_TIMEOUT_SEC` (now 1800) | one change covers all agents | 2026-05-25 |
| Lock+timeout via `shlock`+`perl alarm`, zero new deps | gtimeout/flock/coreutils absent on this Mac | 2026-05-25 |
| Agent perm allow-rules in cwd-local `<agent>/.claude/settings.json` | `run_agent.sh` cd's into agent dir; git-root settings ignored (D025) | 2026-05-25 |

## Deferred items
(See `DEFERRED.md` for full context.)
**Closed prior period:** D092, D026, D094, D091, F17/F18. D045→D050. D083 → PARTIAL.
**Needs reconcile:** **D093** — its note claims "CLAUDE.md still mislabels v0.1.5 as v0.2." Verified false 2026-05-30 — drop that clause (the v0.2 Pro-membership gate itself still stands).
**Open/carried:** F2 (drafter pre-flight keyword-config), D044 (log_to_audit apostrophe rewrite), F14 (pre-ship approval gate), D087 (idea-agent daily calibration), D088 (/model v1.1), D089 (notes-agent v1.1), D090 (approved-index drift), D073/D074 (editor rubric + live-gate).

## Next session should
1. **Confirm the 06:05 soft-issue digest fired** with sane counts and that no reddit per-event TG went out overnight.
2. **Fix market-briefer** — why it stopped emitting `briefer_run` after 2026-05-25 (last chain stage before Telegram delivery). Real execution gap, not box-down.
3. **Fix orchestrator diary** — `write_diary.sh` hasn't emitted since 05-24; check the script/cron path.
4. **Add a watchdog probe for `com.aiteam.dashboard`** (`launchctl_label`, mirrors the grammy-bot probe).
5. **Reconcile D093** in `DEFERRED.md` — remove the false "v0.1.5/v0.2 mislabel" clause.
6. **(Optional)** one-shot live TG test of the hard path for end-to-end certainty; **live-smoke the three phone agents** (carried).

## Files to reference
- `~/agents/issues-capture/{capture.sh,notify_telegram.sh,digest.sh}` — severity classifier (capture), the `severity='hard'` page gate (notify), the daily soft-pile digest. Classifier lists are env-overridable.
- `~/agents/lib/cron.txt` — doc-snapshot of issues-capture cron lines (capture `*/5`, notify `*/5`, digest `5 6`). NOT a full crontab mirror (drifted).
- `~/store/aiteam.db` — `agent_issues` now has a `severity` column (WAL mode: use `sqlite3 … ".backup"` to copy, not `cp`). Backup left at `~/store/aiteam.db.bak-severity-*` (harmless, deletable).
- `~/agents/watchdog/{expected_schedule.yaml, state.json}` — market-briefer + orchestrator show `missing`.
- `~/agents/market/briefer/brief.sh` + `market/scribe/*` — chain to debug (briefer stale since 05-25).
- `~/agents/lib/{ai-do,ai-think,ai-cheap}.sh` — perl-alarm timeout + the ~$0.018/call wrapper floor.
- `~/agents/domain-hunter/{sites.yaml, score/topical_fit.py, score/backfill_payroll.py, CLAUDE.md}` — 4-site config + cost-floor note.

## Gotchas for future me
- **`agent_issues` severity model: soft ≠ suppressed.** Reddit/fetch/transient rows are still captured (silently) and re-accumulate in `/issues`; they surface via the `5 6 * * *` digest, not a TG page. Hard = `HARD_ACTIONS` ∪ (`error`/`*_failed` from a `CORE_PIPELINE` substring-matched agent) ∪ payload `empty output`/`fatal`. Tune via env vars, no code change.
- **aiteam.db is WAL mode — `cp` is NOT a faithful copy.** Uncommitted-to-main changes (e.g. a fresh `ALTER TABLE`) live in the `-wal` file. Use `sqlite3 src ".backup dst"` for tests. Check `PRAGMA journal_mode;` first.
- **`/issues` surfacing is payload-shaped** — capture matches `action LIKE '%_failed'/'%error%'` OR payload containing `"error"`/`"failed"`/`fatal`/`traceback`. A clean action name (`timeout_kill`) only surfaces if its payload carries one of those keys (real ones carry `"error"`). Shape synthetic test rows from a real payload.
- **macOS cron does NOT catch up missed firings** after sleep/reboot — check `kern.boottime`/`uptime`/`last reboot` FIRST when an agent "skipped." launchd `RunAtLoad` *does* recover (dashboard + grammy-bot covered; cron-only agents not).
- **A "last run HH:MM" can be a prior day** — read `datetime(ts,'unixepoch','localtime')`, confirm the DATE before concluding cron fired today.
- **`ai-cheap.sh`/agent-wrapped Haiku calls cost ~$0.018 each** regardless of prompt size (~30K cache_read + ~10K cache_write/call). Estimate batches at `$0.018 × n`; measure ONE call before a batch under a cost cap.
- **shlock + perl-alarm are the macOS zero-dep lock/timeout** — `gtimeout`/`flock`/coreutils NOT installed. `claude -p` has no wall-clock timeout flag.
- **launchd `KeepAlive` fights `kill`** — stop dashboard via `dashboard.sh stop` (bootout); bot restart `launchctl kickstart -k gui/$UID/com.aiteam.grammy-bot`.
- **Agent perm allow-rules are cwd-local** — `~/agents/<agent>/.claude/settings.json`, NOT git-root [D025].
- **23:50 PT auto-commit sweeps uncommitted work** under `agents: auto-commit <date>` — `git add` exact paths, never `-A`; commit before then for descriptive messages.
- **When a task premise is wrong** (mislabel that isn't there, a run that didn't happen, an "approved" edit that never landed) — report it and make the deliverable accurate; don't fabricate the requested fix. Verify earlier-approved changes are actually in the file before building on them.
- **Name the honest test boundary** — "gate routing tested on a temp DB + sender healthy in prod" is not "fired a real alert end-to-end." Mark what executed; offer the gap.
- **`log_to_audit.sh` is positional + echoes the corr id** — `actor_type actor_id action target payload_json [corr]`; `audit_log` uses `actor_id`/`action`/`payload_json` (+ `ts` epoch, not `created_at`).
