# AITEAM Context Save — 2026-05-30_2238
**Generated:** 2026-05-30T22:38:53-0700
**Since last save:** 2026-05-30 22:06:25
**Session topic:** issues-capture severity tiering — hard failures alert Telegram, soft go dashboard-only + once-daily digest; plus bulk-close of the 109-issue backlog to a clean baseline. Preceded by two read-only diagnostic passes (morning cron-block liveness, issues-queue triage) that scoped the work.

---

## Mechanical record

### Git activity since last save
```
d33e4e6 feat(issues-capture): daily soft-issue digest (reuses watchdog notify pattern)
b2b5956 feat(issues-capture): severity tiering — hard failures alert TG, soft go dashboard-only
```

### Files changed
```
A	issues-capture/digest.sh
M	issues-capture/capture.sh
M	issues-capture/notify_telegram.sh
M	lib/cron.txt
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
|  id  |         ts          |      actor_id      |         action          |  target  |
|------|---------------------|--------------------|-------------------------|----------|
| 4597 | 2026-05-30 22:30:01 | watchdog           | watchdog_check_complete | _summary |
| 4596 | 2026-05-30 22:30:00 | assignment-drafter | drafter_tick_started    |          |
| 4595 | 2026-05-30 22:15:00 | watchdog           | watchdog_check_complete | _summary |
| 4594 | 2026-05-30 22:10:02 | notify             | notify_sent             | operator |
| 4593 | 2026-05-30 22:10:01 | notify             | notify_sent             | operator |

### Token usage
(no token usage since last save)

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

- **Severity tiering supersedes the earlier "suppress reddit at capture" plan.** Two turns earlier the operator had approved suppressing `source=reddit` rows entirely at capture (no issue, no alert). That edit was **never applied** (capture.sh was still original). This session the design evolved to a richer model: every captured issue is tagged `severity ∈ {hard,soft}` in a new `agent_issues.severity` column at capture time. Hard → Telegram alert; soft → still inserts into `agent_issues` (visible in `/issues`) but never pages. Reddit/fetch noise becomes *soft*, not *invisible* — surfaced via a daily digest instead of suppressed.
- **Single source of truth = the severity column; gate lives in the delivery step.** `capture.sh` is the only classifier; `notify_telegram.sh` reads `COALESCE(severity,'soft')` from `agent_issues` and skips anything not `hard` (advancing its high-water mark so soft rows aren't re-checked). Rejected the alternative of re-classifying inside notify (brittle, two code paths). Soft issues still get a `new_issues.log` line (tail-visibility preserved) — only the page is gated.
- **Classifier lists are env-overridable data, not hard-coded.** `SUPPRESS_SOURCES` (default `reddit`), `HARD_ACTIONS` (`timeout_kill,pipeline_died,config_synth_failed`), `CORE_PIPELINE` (writer/auditor/ship-to-site/assignment-drafter/config-synthesizer/orchestrator/content-loop). Promote/demote without code surgery.
- **`is_core` = substring match**, so derived agents (`asbestos-writer` → token `writer`) classify hard on `*_failed`. This is the over-suppression guard: a genuine pipeline-writer failure still pages.
- **Daily digest reuses `lib/notify.sh` exactly as `watchdog/digest.sh` does.** New `issues-capture/digest.sh`, cron `5 6 * * *` (just after watchdog's 06:00). One message: new-soft-24h count + still-open-hard count + top noise sources. **Silent on zero/zero** (no "all clear" spam). `DIGEST_DRYRUN=1` prints without sending.
- **Backlog: retro-tag then bulk-close to a clean baseline.** All 109 retro-tagged (15 hard cluster, 94 soft). Closed: 92 reddit → `wontfix` ("soft: external noise"); 15-row 05-27/28 cluster → `resolved` ("self-resolved off-day cluster (f0daccc)"); notify_failed → `resolved` ("self-healed on retry"); misfiled digest_sent → `wontfix`. **Open count 109 → 0.**
- **Deliberately did NOT fire a real test Telegram.** The whole task is anti-spam; firing a synthetic hard alert would mean polluting the real `audit_log`/`agent_issues` and pinging the phone. Tested gate *routing* on an isolated temp-DB copy and leaned on prod evidence that `notify.sh` is healthy (62 `notify_sent`/24h). Offered the live ping as opt-in rather than doing it unprompted.

## Lessons learned

- **A WAL-mode SQLite DB can't be copied with `cp` for testing** — schema/data changes still in the `-wal` file (e.g. a just-run `ALTER TABLE … ADD COLUMN`) won't be in the main `.db`. A plain `cp` produced a copy *missing the new `severity` column*. Use `sqlite3 src ".backup dst"` (or checkpoint first) for a consistent snapshot that includes WAL state. Check `PRAGMA journal_mode;` before assuming `cp` is safe.
- **Synthetic test rows must match how the real agent emits, or the capture filter silently drops them.** `issues-capture` keys on `payload_json LIKE '%"error"%'` (among others). A hand-crafted `timeout_kill` with `{"reason":…}` never matched and looked like a classifier bug; the *real* `timeout_kill` payload is `{"error":"claude -p exceeded …"}`. Pull a real backlog payload to shape test data — don't invent the JSON.
- **Verify a prior "approved" change actually landed before building on it.** The reddit-suppression the operator approved two turns earlier was never written to `capture.sh`; this task would have double-applied or conflicted if I'd assumed it was there. `grep` the file's current state first.
- **The gate belongs in the delivery step, reading a column the capture step wrote** — keeps one classifier and one source of truth. When two cron steps share state through a file (`new_issues.log`), decide explicitly which layer owns the decision; re-deriving it downstream invites drift.
- **Name the honest test boundary.** Gate routing was tested on an isolated temp DB; the sender's health came from prod audit rows — that is *not* the same as a literal end-to-end phone delivery of a synthetic issue. Marked exactly what executed and offered the missing 1% (a live ping) rather than claiming it.

## Operator corrections

- No factual correction this session. The operator reinforced a standing process bar verbatim: **"NEVER mark ✅ unless tested. Honest ⚠ beats premature ✅."** Honored by flagging the one untested edge (no real TG fired) instead of a blanket green check.
- The operator also enforced **pre-flight-report-before-edit gates** across turns (show the schema/notify mechanism/notify_failed health/counts and the exact suppression + gate conditions *before* touching anything). The design itself evolved between turns — from "suppress reddit entirely" to "severity tiering, soft stays in dashboard + digest" — a reminder that an earlier-approved plan is provisional until built.

## What's next

- **Watch the first real digest fire at 06:05 tomorrow** — confirm it sends one message with sane counts and that **no per-event reddit TG fires** through the night (the `*/5` notify_telegram path should now skip all soft).
- **Monitor `/issues` re-accumulation (by design).** Soft reddit issues will keep being captured (silently) — the dashboard open count will climb again. If it gets noisy, consider auto-aging/auto-closing soft issues older than N days; not built this pass.
- **Optional:** one-shot live TG test of the hard path if end-to-end certainty is wanted (insert a synthetic hard row → real capture → real notify_telegram → clean up).
- **Unchanged carry-overs (not touched this session):** fix the two silent-dead agents (**market-briefer** stale since 05-25, **orchestrator diary** since 05-24); add a watchdog liveness probe for `com.aiteam.dashboard`; reconcile the stale D093 clause; operator-side live-smoke of the three new phone agents.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-30_2204.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-30 22:04 (domain-hunter payroll site + 3 read-only fleet audits) **Last session summary:** Added `payrolldetective.com` as domain-hunter's 4th topical-fit site (id `payroll`, 3 commits) and ran three READ-ONLY fleet audits (agent liveness/auto-restart, stuck pause flags, market-chain 2026-05-29 skip RCA) — which surfaced two genuine silent-dead agents and confirmed the 05-29 market skip was a box-down event, not a chain bug.  **This session (domain-hunter payroll):** `sites.yaml` now has 4 topical-fit sites; the `payroll` block mirrors the exact `id`/`domain`/`niche`/`related_topics` shape and **needs no scorer code change** (`topical_fit.py` scales off the site list). 28 already-scored candidates backfilled payroll-only (existing ust/asbestos/ssg scores preserved); **0 routed to payroll** (all 28 are prior pharma/spam junk → correctly `none`). Routability proven via deterministic test-mode (`payrollexperts.com`→`payroll` 0.9), not a costly scan. Commits `11b4a07` (feat sites.yaml) / `5c35fd7` (chore backfill + committed `score/backfill_payroll.py`) / `f8e3a84` (docs). ctx: `context-saves/AITEAM_Context_Save_2026-05-30_2204.md`. 
