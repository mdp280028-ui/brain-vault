# AITEAM Context Save — Watchdog Live + Sister Chat Foundation Sprint

**Date:** 2026-05-17 (morning into late morning)
**Session length:** ~3 hours
**Chat role:** One of two parallel chats. This chat owned the Watchdog agent build (Phase 3a). Sister chat owned foundation infrastructure (PRIME edit, GSC queue, F17/F18 cost cap, lockfile).
**Session character:** Two parallel chats, clean file-boundary split, zero collisions. This chat shipped Watchdog end-to-end (recon → build → cron activation → first-tick verification). Sister chat shipped 5+ commits across infrastructure foundation. Six top-10 priorities from session-start list moved off the queue.

---

## TL;DR

**Watchdog Phase 3a is live in production.** Detector agent running every 15 min, monitoring 7 cron-scheduled agents via three signal types (audit_log, logfile_mtime, sqlite_row). State-tracked dedup confirmed working. First incident written (orchestrator/diary_written missing since 2026-05-15). Telegram alert received and confirmed.

**Sister chat shipped lockfile + cost cap + GSC queue + PRIME.md edit + warroom dashboard work (previously uncommitted in production for multiple sessions).** Three of the original top-10 priorities closed in sister: F17/F18 cost cap (POLICY Q2 enforced for real), lockfile (POLICY Q5 enforced via per-site /tmp lockfile + trap-aware EXIT handler), GSC submission queue (table + dashboard panel + deploy_batch.sh hook).

**Big new finding: D066 — systemic untracked-production-code hygiene crisis.** F17/F18 pre-flight surfaced 15+ uncommitted files across `~/agents/`. Pattern has been caught 5+ times this month and is now blocking the meta-trust the rest of the system depends on. Headline next-session priority.

**Next chat priorities (in order):**
1. D056 — editor production runner (sharp operator + clean run-batch.sh = unblocked, this chat was about to start it)
2. D066 — dedicated 2-3 hr untracked-code hygiene audit
3. Inspector agent (Phase 3b) — defer ~1 week until Watchdog has caught 5-10 real incidents

---

## SESSION ARC

1. **Morning open** — Operator pasted v2.1 project instructions, asked for next-10 priority list from the three latest context saves. Generated and ranked.
2. **Side question on GSC** — answered (sitemap auto-ping primary, IndexNow API for instant Bing/Yandex, hybrid approach via deploy_batch.sh hook). Sister chat ended up building it.
3. **Watchdog explainer** — operator asked for 14-year-old-level explanation. Delivered ~400 words.
4. **Inspector/Maintainer architecture decision** — operator raised "shouldn't we also have an inspector?" Real architectural call. Split into Detector (Watchdog, cheap, dumb, always-on) vs Inspector (smart, slow, expensive, triggered-by-Watchdog). Phase 3a/3b separation.
5. **Model-tier escalation question** — operator asked if Inspector should escalate Sonnet → Opus for fix attempts. Real concern. Answered: **no — escalate for diagnosis only.** Whitelist + 1-attempt cap + confidence gate is the safety layer, not model tier. Locked as proposed Q8 policy when Inspector eventually builds.
6. **Watchdog recon (Step 0)** — CC pre-flight surfaced six HALT-worthy findings + five operator-input questions. Most important: ts is INTEGER epoch not datetime string, drafter has no per-tick heartbeat, three cron scripts don't audit at all, deploy_batch emits one of two acceptable actions.
7. **Operator-input answers** — Q1: alt signals for 3 silent scripts (logfile_mtime + sqlite_row); Q2: add drafter heartbeat; Q3: confirmed grace defaults (daily 30 min / */30 10 min / */5 15 min); Q4: state-flip dedup + 06:00 digest; Q5: analyst/briefer/curator excluded (on-demand, not scheduled).
8. **Watchdog build prompt** — comprehensive 7-step prompt with HALT after Step 6 for dry-run verification before cron activation. CC executed cleanly. Three reasonable deviations from spec, all defensible:
   - tg-monitor used `messages.db / poll_log.ts` (real schema) vs spec's assumed `monitor.db / messages.received_at`
   - Drafter heartbeat placed BEFORE kill switches (correct — heartbeat signals cron firing, not work decision)
   - brain-autocommit via logfile_mtime (matched recon, no audit_log hook needed)
9. **Step 6 HALT — dry-run output** — 6 healthy, 1 missing (orchestrator). Real failure exposed: diary_written last fired 2026-05-15, not the 7/7 nights expected. **Watchdog motivation confirmed in data before going live.**
10. **Step 7 cron activation** — green-lit. Crontab snapshot taken. Two cron lines added (`*/15 watchdog.sh`, `0 6 digest.sh`). Activation audit row id=1110.
11. **First tick verification** — fired at 11:15:00, ~2s execution. Audit rows: id=1112 watchdog_alert (orchestrator), id=1113 watchdog_check_complete. State.json populated with all 7 agents. Incident JSON written. Telegram alert received by operator.
12. **Sister chat handoff** — operator pasted sister's session summary. Five commits across multiple repos, including a notable D066 finding.
13. **Recommendation on next build** — D056 editor production runner. Sister cleared run-batch.sh of collision risk by closing lockfile build. Operator confirmed sharp, asked for ctx given my context burn.

---

## WHAT GOT SHIPPED — THIS CHAT

### Commits

**agents repo (`~/agents/`):**
- `fb53d04` — drafter: add per-tick heartbeat row for watchdog (D-WATCHDOG-PREP)
- `88f8d7a` — watchdog: add detector agent + expected schedule + state-tracked dedup

**brain repo (`~/brain/`):**
- `61ea5d1` — watchdog: scaffold incidents directory

**Crontab:**
- Added: `*/15 * * * *  /Users/mmm2/agents/watchdog/watchdog.sh >> /Users/mmm2/agents/watchdog/watchdog.log 2>&1`
- Added: `0 6   * * *  /Users/mmm2/agents/watchdog/digest.sh   >> /Users/mmm2/agents/watchdog/digest.log   2>&1`
- Snapshot: `/tmp/crontab-snapshot-pre-watchdog-1779040954.txt`

### Files in place

```
~/agents/watchdog/
├── digest.sh                 (3913 B, executable)
├── expected_schedule.yaml    (2199 B)
├── lib/.gitkeep
├── watchdog.sh               (12991 B, executable)
└── state.json                (created on first tick, populated 7 agents)

~/brain/projects/aiteam/incidents/
├── .gitkeep
└── 2026-05-17_111500_orchestrator.json   (first incident)
```

### Verified live

- First cron tick: 11:15:00 PT, fired exactly on schedule (~2s execution)
- Audit log row id=1112 (watchdog_alert, orchestrator) and id=1113 (watchdog_check_complete)
- One Telegram alert sent and received: ⚠️ WATCHDOG: orchestrator missing
- Dedup ready: orchestrator.status=missing in state.json; will not re-alert until state flips healthy
- 11:30 tick (drafter cron boundary) was the next test — no collision, no duplicate alert (operator did not confirm receipt of an 11:30 alert, expected behavior)

---

## WHAT GOT SHIPPED — SISTER CHAT

Pasted by operator at session close. Listed for cross-reference; sister will have its own context save with full details.

### Commits

**agents repo (`~/agents/`):**
- `c56209f` — feat(gsc-queue): asbestos-only GSC submission queue table + dashboard panel + deploy_batch.sh hook
- `60b2486` — multi-site upgrade (site column + UI badge)
- `af39f8d` — PRIME.md upgrade: added "Read first" section pointing at HANDOFF/POLICY/DEFERRED, fixed stale DAILY_API_BUDGET_USD rule
- `7446717` — log_token_usage.sh untracked-writer separation
- `54a69bd` — cost-cap (F17/F18, `check_cost_caps.sh` helper + post-INSERT hooks)
- `837081e` — D066 logging
- `a7c2c71` — warroom slash commands + multi-round discuss + token-spend chart (previously running uncommitted)

**asbestos-contractors + ssg-content repos:**
- `ac2f721` + `7dd5c27` — lockfile build (POLICY Q5, per-site /tmp lockfile + trap-aware EXIT handler)

**Brain commits:**
- ctx-related (sister's own context save will detail)

### What sister shipped

1. **GSC submission queue** — `gsc_submission_queue` table, dashboard panel with Copy + Mark Submitted per row, hooked into `deploy_batch.sh` at ship time. Asbestos-only first, then multi-site upgrade with site column + badge. SSG hook deferred as D065.
2. **PRIME.md upgrade** — future CC sessions now read HANDOFF.md/POLICY.md/DEFERRED.md at session start. Stale DAILY_API_BUDGET_USD rule removed.
3. **F17/F18 cost cap enforcement** — POLICY Q2 enforced for real. `$25 soft / $50 hard`. `check_cost_caps.sh` helper wired after every `token_usage` INSERT. Soft cap → `DRAFTER_PAUSED` flag + Telegram warning. Hard cap → rewrites `.env SYSTEM_PAUSED=true` (env-flip approach after CC pre-flight caught file-flag was inert) + both flags + Telegram. Daily 00:00 cron auto-resets. **Removed three pre-existing disagreeing caps** (orchestrator $10, drafter $15, $2.50/slug estimate) — `budget_check.sh` deleted.
4. **Pre-existing dashboard work committed** — 384/12 lines of warroom infrastructure that had been running uncommitted in production for multiple sessions. Shipped as a7c2c71 (operator accidentally pasted Watchdog Step 7 prompt into sister chat at one point — sister chat caught it).
5. **Lockfile for run-batch.sh** — POLICY Q5 serialization. Per-site `/tmp/run-batch-<site>.lock`. Trap-aware EXIT handler (CC self-caught a bare-trap clobber bug during testing — would have wiped existing spinner cleanup). Seven tests green. Per-site independence verified.

---

## KEY DECISIONS / FINDINGS

### Inspector escalation architecture (locked as proposed Q8)

Real architectural call from operator question. Three pieces:

1. **Sonnet escalates to Opus for diagnosis ONLY, never for fix attempts.** Smarter model with no guardrails = exactly the bad outcome to avoid. Whitelist is the safety layer, not tier.
2. **Fix-attempt cap: 1 per incident.** No loops. After one attempt, escalate to operator.
3. **Confidence gate: self-rated ≥4 + whitelist match required to attempt fix.** Below that = report only.

To be locked as Q8 in POLICY.md when Inspector eventually builds.

### Watchdog signal-type abstraction is the right shape

Three signal types (audit_log, logfile_mtime, sqlite_row) cover every monitored agent without forcing audit-log instrumentation everywhere. Avoided touching brain-autocommit / tg-monitor scripts. Cleaner build, smaller blast radius.

### Real failure exposed by Watchdog day 1

**orchestrator/diary_written has not fired since 2026-05-15.** This was invisible before Watchdog. Could be:
- D054 cron PATH issue (since fixed) — would have self-healed at 23:45 last night
- write_diary.sh script bug — would still be broken
- orchestrator agent itself failing
- something else

**Next session morning verification:** check Telegram for either ✅ recovery ping (self-healed) or silence (still broken, needs 20-min CC investigation).

### Drafter heartbeat placement principle (worth keeping)

When adding instrumentation, **signal that the script ran is separate from signal that work was attempted.** Heartbeat-before-kill-switches because Watchdog needs to know cron fired, not whether the script chose to do work. Operator pausing via halt.flag should NOT look like a misfire. This is generalizable to future instrumentation across the fleet.

### Sister chat caught trap-clobber bug self-tested

Bare `trap '...' EXIT` in lockfile build would have wiped existing spinner cleanup. CC found and fixed during testing. **Worth pulling forward into D056 build:** explicitly require additive trap pattern in any new bash agent. Add to LESSONS.md.

### Parallel-chat coordination worked clean

This session ran two parallel chats for ~3 hours with zero collision:
- Sister: infrastructure (PRIME, GSC, F17/F18, lockfile) — file scope: `~/agents/lib/`, dashboard, `run-batch.sh`, crontab additions
- This chat: agent build (Watchdog) — file scope: `~/agents/watchdog/` (new folder), `~/agents/assignment-drafter/drafter.sh` (1-line heartbeat)

Only file overlap was crontab (both added cron lines) and that was sequenced cleanly. The file-boundary-not-topic rule from v2.1 worked exactly as intended.

**One coordination glitch:** operator accidentally pasted Watchdog Step 7 prompt into sister chat. Sister chat caught it and didn't execute. No harm. Pattern note: when passing prompts between chats, verify which chat the prompt is for.

---

## D066 — UNTRACKED-CODE HYGIENE CRISIS (HEADLINE NEXT-SESSION ITEM)

Pattern has now been caught 5+ times this month:
- ship-to-site agent (May 16)
- assignment-drafter, config-synthesizer (May 16)
- brain-autocommit, tg-monitor reader/analyzer (May 17 = D060)
- warroom slash commands, multi-round discuss, token-spend chart (this session, a7c2c71)
- F17/F18 pre-flight surfaced **15+ more uncommitted files across `~/agents/`** (D066)

**This is structural risk, not bookkeeping.** When something breaks:
- Can't `git log` to find what changed
- Can't roll back
- Can't see actual production state
- **Watchdog can't audit code it doesn't know exists**

D066 is now blocking the meta-trust the rest of the system depends on. Needs a dedicated 2-3 hr audit session, not piecemeal fixes. Recommend doing it BEFORE D056 if operator agrees, AFTER if D056 takes priority.

**Audit shape:**
1. Find all untracked files in `~/agents/`, `~/projects/`, `~/brain/projects/aiteam/`
2. For each: commit / delete / refactor / leave (with rationale)
3. Add `.gitignore` entries where appropriate to prevent re-occurrence
4. Document the audit in a session-end report

---

## DEFERRED ITEMS LOGGED THIS SESSION

### Logged on disk (sister chat)
- **D065** — SSG GSC hook (multi-site upgrade landed asbestos-only first, SSG side deferred)
- **D066** — Systemic untracked-production-code hygiene crisis (15+ files, dedicated audit needed)

### Logged in this session as forward-references
- **D-WATCHDOG-PREP** — used as commit ref for drafter heartbeat (fb53d04). NOT in DEFERRED.md numbering scheme — one-off, commit history is the real record. Flagged for next session: either retroactively log as closed item or let stand.
- **Q8 (POLICY)** — Inspector escalation rules: Sonnet→Opus diagnosis-only, fix-attempt cap 1, confidence gate ≥4 + whitelist match. To be locked when Inspector builds.
- **LESSONS.md candidate** — Trap-clobber bug pattern (sister chat catch). Add to LESSONS.md next session: when adding `trap` to existing bash scripts, use additive pattern, never bare `trap '...' EXIT` that clobbers existing handlers.

---

## OPEN LOOPS / FLAGS

1. **Tonight 23:45 — diary writer test fire.** If diary_written audit row appears: Watchdog will auto-ping ✅ recovery for orchestrator, state.json flips, dedup arms for next failure. If silent: orchestrator stays missing through tomorrow's 06:00 digest. Either outcome is informative.

2. **Tomorrow 06:00 — first daily digest.** First proof that digest.sh works. Watch Telegram. Markdown rollup of healthy/missing/recovered states.

3. **D056 not started.** Plan was to start it after sister cleared run-batch.sh. Sister cleared it. Operator was sharp. My context burn forced the handoff. Next chat starts here.

4. **D066 is the bigger blocker.** D056 ships a new agent; D066 audits everything. Argument for D066-first: building D056 on top of a partially-untracked code base just adds to the audit pile. Argument for D056-first: shipping is fun, audits are slow, momentum matters. Operator's call.

5. **Inspector defer is intact.** Don't build before ~May 24 / ~5-10 caught incidents. Watchdog needs to observe first.

6. **Operator-track unchanged:**
   - Keyword research + enqueue 5-10 asbestos slugs into `drafter_queue.txt` (loop is live and starving)
   - AdSense application for asbestoshq.com (32 guides live, $0 monetization)
   - These two move the mission bar more than any agent build

7. **D061/D062** — gated on F17/F18 cost cap landing. **F17/F18 shipped this session.** D061/D062 are now unblocked. Should appear in next-session priority queue.

---

## LESSONS LEARNED

### Recon-first pattern continues to pay (4th week running)

Watchdog recon caught:
- ts is INTEGER epoch (would have caused silent always-healthy or always-missing on a wrong query form)
- drafter has no per-tick heartbeat (would have caused constant false alarms)
- Three cron scripts don't audit at all (would have been blind spots)
- deploy_batch emits one of two acceptable actions (would have caused false alerts on empty days)
- ship.sh disabled (would have alerted on intentionally-paused script)
- Lateness window needed (would have false-alerted on running-but-not-finished jobs)

Six real spec issues caught before any code shipped. The 5-10 min of recon repeatedly saves the entire build.

### Calibration ≠ production is now a project lesson

Watchdog needed a recon step (Step 0) before building. Cost cap needed a pre-flight (caught file-flag was inert). Editor production runner D056 will need the same. **Any agent transitioning from calibration / spec / "idea" to production needs explicit recon before build.** This pattern is locked.

### Self-caught bugs are a sign of healthy CC discipline

This session's two self-catches:
- Watchdog dry-run output revealed orchestrator real failure (would have shipped silent)
- Sister chat's lockfile bare-trap clobber (would have broken spinner cleanup)

Both caught by CC pre-shipping verification. Honest ⚠ at build time beats ✅ then 🔥 in production every time.

### Operator delegated to recommendation when policy was clear

When I asked "Option A or Option B for D056 collision avoidance" with a stated recommendation, operator said "go" without re-litigating. Same pattern as Q2/Q3/Q5/Q7 in POLICY.md lock session. **When the recommendation is clear and the rationale is sound, operator trusts the call.** Save the back-and-forth for genuine uncertainty.

### Context-burn handoffs are healthier than context-overload pushes

This chat hit ~80% context capacity around the time D056 was ready to start. Right call was the handoff, not "one more build." Future Claude (next chat) gets a clean context save + fresh budget. Operator's "you are getting low on context. let's continue this in another chat" is the right move — codify as a pattern.

---

## NEXT CHAT — WHERE TO RESUME

### Read in order
1. **This context save** (you're reading it)
2. `~/brain/projects/aiteam/POLICY.md` — 7 locked policies + Q8 (Inspector) when ready
3. `~/brain/projects/aiteam/PRIME.md` — recently updated, points at HANDOFF/POLICY/DEFERRED
4. `~/brain/projects/aiteam/HANDOFF.md` — current state
5. `~/brain/projects/aiteam/DEFERRED.md` — D065, D066 new this session
6. Sister chat's context save (separate file)

### Morning verification (passive, no CC needed)
1. **Check Telegram for 06:00 daily digest** — first proof digest.sh works
2. **Check for ✅ orchestrator recovery ping at 23:45 last night** OR confirm silence
3. **Check `~/brain/projects/aiteam/incidents/`** for any new JSONs overnight

### Recommended next builds (rough priority)

**Path A — D066 first (audit before build):**
1. D066 untracked-code hygiene audit (~2-3 hr CC + operator decisions)
2. Then D056 editor production runner (~60-90 min)
3. Then D061/D062 (now unblocked by F17/F18) (~30 min combined)

**Path B — D056 first (ship momentum):**
1. D056 editor production runner (~60-90 min)
2. Then D066 (still needs doing)
3. Then D061/D062

**Recommendation: Path A.** D066 has been caught 5+ times. Every additional build on top of untracked code makes the audit harder. Operator hasn't slept well into the project; better to do the unsexy audit before the next shiny build.

But operator's call.

### Do NOT
- Build Inspector (Phase 3b) — defer ~1 week, need real incident data
- Touch SSG run-batch.sh aggressively — site rewrite still pending
- Re-derive POLICY.md decisions — file is authoritative
- Bundle D066 audit with another build — needs its own session

### Critical context for next Claude
- **Watchdog is live. Every 15 min it checks 7 agents.** Don't treat audit_log queries the same way as before — watchdog's presence changes the meaning of "is X firing reliably."
- **F17/F18 is enforced.** `$25 soft / $50 hard`. Real kill switch. Cost runs are now bounded.
- **Lockfile is enforced on run-batch.sh.** Per-site, trap-aware. Concurrent runs of same-site pipeline now serialize.
- **PRIME.md points at POLICY.md.** Next CC sessions will read the policies automatically.
- **D-WATCHDOG-PREP commit ref** is one-off (not in DEFERRED.md numbering). Don't sweat it.
- **Trap-clobber lesson** — when adding `trap` to existing bash scripts, additive pattern, never bare overwriting trap.
- **Operator's research-before-building bias** — quiet today, but watch for it. Two consecutive build-heavy sessions; momentum is good.

---

## FILES CREATED / MODIFIED THIS CHAT

### In `~/agents/`
- `assignment-drafter/drafter.sh` — added per-tick heartbeat row at top (fb53d04)
- `watchdog/watchdog.sh` — created (88f8d7a)
- `watchdog/digest.sh` — created (88f8d7a)
- `watchdog/expected_schedule.yaml` — created (88f8d7a)
- `watchdog/lib/.gitkeep` — created (88f8d7a)
- `watchdog/state.json` — created on first tick (not tracked; runtime state)

### In `~/brain/projects/aiteam/`
- `incidents/.gitkeep` — created (61ea5d1)
- `incidents/2026-05-17_111500_orchestrator.json` — first incident (runtime, not tracked)

### Crontab
- Two lines added (`*/15 watchdog.sh`, `0 6 digest.sh`)
- Snapshot at `/tmp/crontab-snapshot-pre-watchdog-1779040954.txt`

### In this Claude.ai project
- `AITEAM_Context_Save_2026-05-17_Watchdog-Live.md` — this file

### Not touched
- Nothing in `~/projects/` (asbestos-contractors, ssg-content) — sister chat owned those
- HANDOFF.md — recommend update next session
- LESSONS.md — recommend update with trap-clobber pattern + Watchdog recon pattern + heartbeat-placement principle

---

## SIGN-OFF NOTE

Two parallel chats, ~3 hours, eight commits across four repos, zero collisions, zero premature ✅, two self-caught bugs (one per chat).

Watchdog is live and already exposing real failures (orchestrator). Foundation hardened (cost cap, lockfile, GSC queue). Top-10 priority list from session-start has six items closed.

D056 ready to build. D066 ready to audit. Operator sharp, context handing off cleanly. The pattern is working.

Mission bar: $0 / $200 mo. Loop is live, policies enforced, instrumented, locked. Revenue still gated on operator-side work (slugs + monetization).

---

*End of context save. Resume next session by reading this file first, then PRIME.md (which now points at POLICY/HANDOFF/DEFERRED automatically). Watchdog is your new always-on lens — trust it to surface real failures, don't ignore its alerts.*
