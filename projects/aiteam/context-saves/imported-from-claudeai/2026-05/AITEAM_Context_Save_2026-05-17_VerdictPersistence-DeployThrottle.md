# AITEAM Context Save — Verdict Persistence + Asbestos Deploy Throttle

**Date:** 2026-05-17
**Session length:** ~3 hours, late evening
**Chat role:** One of three parallel chats; this chat handled audit-derived foundation work (verdict persistence + deploy throttle)

---

## TL;DR

Two real builds shipped, both unblocked the autonomous pipeline's safety posture. Started by trying to build Escalation Triage agent — hit a hard wall when CC's pre-flight discovered the R1/R2/R3 tiering referenced in yesterday's cross-agent audit doesn't actually exist in code. Switched tracks to build the verdict-persistence layer (the missing foundation), shipped the audit_guide.py half, deferred the editor half as D056. Then shipped the asbestos deploy throttle — autonomous chain now has a 3-hour human review window (20:00 preview ping → 23:00 batch deploy) instead of continuous */15 deploys. SSG throttle correctly halted by parallel chat; deferred until SSG actually ships its first batch.

Three doc-vs-reality findings surfaced tonight, all worth correcting in next session's audit: R1/R2/R3 is planning-doc concept only; editor agent is calibration-only, not the "production judge" project instructions claim; deploy mechanism is git push → Vercel, not Cloudflare Pages.

---

## SESSION ARC

1. Opened with parallel-chat coordination (3 active chats, this one taking Escalation Triage)
2. Got 4 design answers from operator on triage agent (cron cadence, authority model, output target, escalation chain)
3. CC HALTED on pre-flight: R3 tiering doesn't exist in code
4. Switched tracks to verdict persistence layer
5. CC HALTED again on editor half: no production runner, no documented pass threshold
6. Shipped audit_guide.py persistence only, deferred editor as D056
7. Operator requested deploy throttle — autonomous chain → 3-hour review window
8. Wrote throttle CC prompt (Cloudflare-assumed version)
9. **TIMELINE CONFUSION:** Operator pasted prompt; CC executed it with its own corrections (Cloudflare→Vercel, no preview URLs) while this chat wrote a corrected version assuming nothing had shipped. Sister chat (SSG throttle) halted accurately on snapshot showing no throttle existed — but their snapshot predated CC's completion. Cleared up cleanly when CC's report and operator's "I haven't pasted yet" message got reconciled.
10. Asbestos throttle confirmed live; SSG throttle correctly deferred

---

## WHAT GOT SHIPPED

### 1. Verdict persistence layer (audit_guide.py half)

Schema migration: `~/agents/lib/migrations/2026-05-17_auditor_verdicts.sql` — new table `auditor_verdicts` with columns for slug, scorer, passed, composite_score (NULLable, reserved for editor), rubric_json, reasoning, tier (NULL for now per D056 dependency), triaged_at (NULL — populated by future Escalation Triage).

Indexes: by slug, by triaged_at WHERE NULL, by passed+ts WHERE passed=0.

audit_guide.py modifications: `persist_verdict()` helper using parameterized sqlite3 queries (not concatenation), called from `__main__` after `audit()` returns. `passed = (result['fail_count'] == 0)`. Also emits audit_log row via `log_to_audit.sh` with correlation_id linking the two records.

**Commits:**
- `4e2fe0a` in `~/agents/` — schema migration
- `0e8089b` in `~/projects/asbestos-contractors/` — audit_guide.py persistence

**Tests passed:** 3 of 5 applicable (Test 2 row persists, Test 3 apostrophe/F4 regression, Test 4 idempotent migration). Editor tests (1, 5) correctly skipped per deferred scope.

**Build report:** `~/brain/projects/aiteam/docs/verdict_persistence_build_2026-05-17.md`

### 2. Asbestos deploy throttle

Disabled `*/15 * * * * ship.sh` cron (commented, not deleted — reversible). Added two new cron lines:
- `0 20 * * * preview_ping.sh` — Telegram preview message at 20:00 PT
- `0 23 * * * deploy_batch.sh` — batch deploy at 23:00 PT

Both scripts use `ship.sh --dry-run` as queue source (prevents drift between preview and deploy). Queue definition: files in `content/asbestos/approved/guides/*.json` not in APPROVED_GUIDE_SLUGS whitelist AND not in `state/needs_review_queue.txt` AND site repo clean.

Telegram preview format: slug + word count + composite score (em-dash for NULL) + audit reasoning excerpt truncated to ~60 chars. Empty queue: terse "📋 No slugs ready for tonight's deploy."

**Spec drift from original CC prompt:** CC's pre-flight caught Cloudflare→Vercel mismatch and no-preview-URL gap; applied Option 1 corrections (drop URLs entirely) on its own. Same corrections this chat later wrote in a "corrected" prompt that turned out to be redundant.

**Commit:** `3ed57dc` in `~/agents/`

**Tests passed:** Empty-queue preview ping (audit row 1031, http 200), empty-queue deploy_batch (audit row 1064, no Telegram noise), reversibility (DISABLED comment block preserved). Populated path verified offline against asbestos-shingles-guide (2006 words, em-dash for NULL composite) and asbestos-don't-demolish-test (60-char truncation, apostrophes preserved through F4 escape path). Populated deploy_batch deferred to first real 23:00 fire.

**Build report:** `~/brain/projects/aiteam/docs/deploy_throttle_build_2026-05-17.md`

**Crontab snapshot pre-change:** `/tmp/crontab-snapshot-pre-throttle-1778996892.txt` (30-day retention or until /tmp wipe).

**First verification milestones:**
- 20:00 PT today (May 17) — first real preview ping fires
- 23:00 PT today — first real batch deploy

---

## KEY FINDINGS / DECISIONS

### Doc-vs-reality gaps surfaced (THREE)

**Finding 1: R1/R2/R3 tiering is planning-doc concept only.** Yesterday's cross-agent failure modes audit (`cross_agent_failure_modes_2026-05-16.md`) referenced R3 failures as if they were a real artifact. CC's recon proved they aren't — editor scores to stdout, audit_guide.py prints pass/fail to stdout, neither persists. The terms R1/R2/R3 appear nowhere in `~/agents/` or `~/projects/`. The cross-agent audit needs a minor correction next session.

**Finding 2: Editor agent is calibration-only, not "production judge."** Project instructions call editor "the production judge (per step 17 tier test)" but `~/agents/editor/` contains only `run_tier_test.sh` (explicitly a calibration script that loops over a fixed test corpus running Haiku + Sonnet twice). There is no script that takes one real draft, scores it, and persists a verdict. Phrasing in project instructions is misleading.

**Finding 3: Deploy mechanism is Vercel, not Cloudflare Pages.** Multiple spec references throughout planning docs assume Cloudflare Pages. Actual deploy path: `lib/git_ops.sh` does `git push` to `mdp280028-ui/asbestoshq-site main`, Vercel auto-deploys. No wrangler.toml, no .cloudflare/ dir. Tracked as D058 (Vercel preview URLs deferred).

### Architectural decisions locked

- **Slow only the final deploy step, not the whole chain.** Drafter + audit_guide.py keep current cadence. Drafts pile up in "ready to ship" state. Single batch deploys at 23:00. Reasoning: content velocity preserved (would drop ~10x if whole chain throttled), only deployed content touches the live internet.
- **23:00 PT deploy, 20:00 PT preview.** 3-hour review window. Latest deploy time that still gets overnight Google indexing.
- **No preview URLs in preview ping.** Vercel preview branches not wired. Score + word count + audit excerpt is sufficient triage signal. URLs deferred as D058.
- **`ship.sh --dry-run` as the single queue source.** Both preview_ping.sh and deploy_batch.sh call it. Guaranteed no drift between what's previewed and what ships.
- **Empty-queue handling differs by script.** Preview ping sends terse "no slugs" message. Deploy batch sends no Telegram on empty (avoid noise), only audit_log entry.
- **Throttle is reversible by design.** Original cron line commented with explicit DISABLED note + restore instructions. Single uncomment flips back to 15-min cadence.
- **SSG throttle deferred until SSG ships.** ssg.yaml: enabled: false; content/ssg/approved/guides/ empty. Build a throttle when there's something to throttle.

### Verdict-persistence design call

- **New table `auditor_verdicts`, not extending audit_log.** Cleaner separation of structured verdict data from generic event log. Both populated — verdict in dedicated table, summary event in audit_log with correlation_id.
- **Editor persistence deferred (D056), not built tonight.** Two blocking gaps: no production runner exists (`run_tier_test.sh` is calibration only), no pass threshold documented anywhere. Both gated on operator-policy Q7 from yesterday's cross-agent audit.
- **tier column stays NULL.** No R1/R2/R3 rubric defined yet. Field exists for future Triage agent to populate.
- **F4 regression verified under a new caller.** audit_guide.py's persist_verdict() exercises the patched log_to_audit.sh through Python's subprocess. Apostrophes in slug, source_path, rubric, and reasoning all preserved verbatim. F4 fix holds.

---

## OPERATOR CORRECTIONS / GOOD INSTINCTS

1. **"This seems like a waste of time right now"** on Escalation Triage — instinct was right. If R3s would just sit until human review, the agent's value drops to "saves you a 2-min SQL query." The honest cost-benefit reframe led to dropping the build.

2. **"I would have Opus eyeball them and go with its decision anyways"** — reframed the value: the triage agent IS that workflow on autopilot. Cron beats remember-to-check.

3. **"I don't want it asking me questions all the time"** — clarified the design principle: agent decides autonomously, posts decision (FYI), only escalates to human when both Sonnet AND Opus can't resolve. Removed the human-in-loop default from the triage spec.

4. **"What do you recommend"** at three decision points (cadence, output target, preview URL) — clean delegation when uncertain. Each recommendation came with reasoning.

5. **"This is only temporary till the system gets its legs"** on the throttle — framed it as scaffolding, not architecture. The right mindset: ship-fast-and-throttle while quality stabilizes, then remove brake.

6. **Caught my soft sign-off twice tonight.** First when I assumed a written prompt was a shipped build (asbestos throttle). Second when I assumed CC's later completion report was about my "corrected" prompt rather than the original. Both are exactly the pattern flagged in project instructions. Lesson re-internalized.

---

## DEFERRED ITEMS (NEW THIS SESSION)

| ID | Item | Trigger | Notes |
|---|---|---|---|
| D052 | Verdict-persistence layer architecture / R1/R2/R3 rubric | After operator answers cross-audit Q1, Q4, Q7 | Currently editor persistence is the missing half — D056 picks that up specifically |
| D056 | Editor verdict persistence | After Q7 answered (editor as pre-ship gate?) AND production editor runner exists | Pre-reqs documented in verdict_persistence_build_2026-05-17.md |
| D057 | Remove asbestos deploy throttle | After 2-3 weeks of clean preview pings, no manual skips needed | Reversal procedure in deploy_throttle_build_2026-05-17.md |
| D058 | Vercel preview URLs in preview_ping.sh | If 2-3 weeks of throttle shows score-based triage is insufficient | Currently messages have slug + word count + score + audit excerpt, no URLs |
| D059 | SSG deploy throttle (pattern-copy of asbestos) | After ssg.yaml: enabled: true AND SSG ships its first batch | Pattern reference: deploy_throttle_build_2026-05-17.md |

**Cross-agent audit correction (no D-item — needs to happen next session):**
- R3 reference in `cross_agent_failure_modes_2026-05-16.md` should be corrected — F2 was described as preventing "R3 burn" but R3 doesn't exist. Real failure mode is "drafter fires without keyword-config check" regardless of audit-tier model. Audit content stays valid; the R-tier framing needs a footnote.

**Project instructions correction (no D-item — needs to happen next session):**
- "Production judge: Sonnet 4.6 via ai-do.sh" phrasing in AITEAM_Project_Instructions.md should be clarified to "Editor agent (Sonnet 4.6 via ai-do.sh) is the calibration-tested scorer; not currently wired as a production gate. Wiring deferred per D056."

---

## OPEN LOOPS / FLAGS

1. **First real 23:00 deploy hasn't fired yet.** Empty-queue path verified; populated-queue verification pending the first non-empty real fire. If anything breaks, log is at `~/agents/ship-to-site/deploy_batch.log`.

2. **20:00 preview ping at first non-empty fire is the user-facing verification.** Watch your Telegram tonight at 20:00 PT for the first real preview message — confirm format renders correctly, scores populate, audit excerpts truncate cleanly.

3. **The seven operator-policy questions from cross-audit §7 are still open.** Verdict-persistence work tonight surfaced that several of them (Q1, Q4, Q7) actually block the editor persistence half. They're not just "nice to answer" — they're foundation gates for the next layer of pipeline hardening.

4. **DEFERRED.md numbering may need a sync pass.** Three chats tonight added D-items independently. D045/D046/D047/D048 added in sister chat earlier today, this chat added D052/D056/D057/D058/D059. Each chat used "next free number per file's authoring rule" but parallel writes may have collided. Worth a quick `cat DEFERRED.md | grep "^| D" | sort` at next session start.

5. **Parallel-chat timeline drift is now a documented failure mode.** Two close calls tonight (asbestos throttle "did it ship?" and SSG halt "should it stand down?"). Worth adding to lessons learned formally.

---

## LESSONS LEARNED

### Prompt-written ≠ build-shipped

Made this mistake twice tonight. Once when I produced a "summary for the other chat" describing asbestos throttle infrastructure as live before confirming the operator had pasted the prompt and CC had executed it. Again when I read CC's completion report and started writing "corrected" follow-up work, not realizing CC's report was about an earlier paste. Both are soft sign-off patterns the project instructions explicitly flag.

**Going forward:** before writing follow-up work that depends on a build, explicitly confirm execution. Cheapest signal: ask the operator "is CC done with that build?" or watch for the operator quoting CC's actual completion report.

### Parallel chat snapshots drift in minutes

What's true at 22:14 in one chat may not be true at 22:18 in another. When pulling status from a sister chat:
- Ask when their snapshot was taken
- Don't assume their "no throttle exists" check is still current 4 minutes later
- Cross-reference with operator's most recent statement

### CC's pre-flight discipline is doing real work

Tonight CC caught three real spec errors I missed:
- R1/R2/R3 doesn't exist in code (Escalation Triage HALT)
- Editor has no production runner (verdict persistence editor-half HALT)
- Deploy is Vercel not Cloudflare (throttle build pre-flight)

In each case CC reported the gap clearly with evidence (grep output, file listings, schema dumps) and waited for direction. The sign-off discipline rule is paying for itself. Reinforces: never relax pre-flight requirements to "speed up" a build.

### Build-against-stub-data is the wrong instinct

When CC HALTed on missing R3 verdicts, one menu option was "build against stub source — agent works against fake data, real upstream wires in later." Tempting because it lets the build ship tonight. Rejected because: schema assumptions made against fake data almost certainly won't match real upstream when it ships. Better to fix the upstream first (verdict persistence) and build the dependent agent on real data later.

### Doc-vs-reality is real here

Project instructions, cross-agent audit, and build plans all referenced infrastructure (R3 tiering, editor-as-production-judge, Cloudflare deploy) that doesn't exist in code. None of these are mistakes by anyone in particular — they're the natural drift between planning docs and implementation in a fast-moving solo project. The fix is periodic recon (CC's pre-flight pattern), and documenting gaps as they surface (this session's contribution).

---

## FILES CREATED / MODIFIED THIS SESSION

In `~/agents/`:
- `lib/migrations/2026-05-17_auditor_verdicts.sql` — new schema migration (commit 4e2fe0a)
- `ship-to-site/preview_ping.sh` — new (commit 3ed57dc)
- `ship-to-site/deploy_batch.sh` — new (commit 3ed57dc)

In `~/projects/asbestos-contractors/`:
- `scripts/audit_guide.py` — added `persist_verdict()` helper + call from `__main__` (commit 0e8089b)

In `~/brain/projects/aiteam/docs/`:
- `verdict_persistence_build_2026-05-17.md` — new
- `deploy_throttle_build_2026-05-17.md` — new
- `ssg_throttle_build_halt_2026-05-17.md` — new (sister chat output, referenced)

In `~/brain/projects/aiteam/`:
- `DEFERRED.md` — D056, D057, D058, D059 added (D052 referenced but not authored this session)
- `HANDOFF.md` — item 0 (half-live verdict layer), item 0a (asbestos throttle live)

System state:
- Crontab — `*/15 ship.sh` commented out (preserved with DISABLED note), `0 20 preview_ping` + `0 23 deploy_batch` added
- `auditor_verdicts` table — live, populating from audit_guide.py runs
- `audit_log` rows 1031 (preview_ping_sent), 1064 (deploy_batch_complete empty) — written during manual tests

---

## NEXT CHAT — WHERE TO RESUME

Read in order:
1. This context save
2. `~/brain/projects/aiteam/HANDOFF.md` (just updated)
3. `~/brain/projects/aiteam/DEFERRED.md` (D052/D056-D059 new)
4. `~/brain/projects/aiteam/docs/verdict_persistence_build_2026-05-17.md`
5. `~/brain/projects/aiteam/docs/deploy_throttle_build_2026-05-17.md`

**Morning verification (May 18):**
1. Check Telegram for the 20:00 preview ping from yesterday — did first real fire land?
2. Check Telegram for 23:00 deploy summary — did first real batch deploy?
3. Check audit_log for the corresponding rows: `sqlite3 ~/store/aiteam.db "SELECT id, ts, actor_id, action, target FROM audit_log WHERE actor_id='ship-to-site' AND action IN ('preview_ping_sent','deploy_batch_complete') ORDER BY ts DESC LIMIT 10;"`
4. Spot-check `~/agents/ship-to-site/preview_ping.log` and `deploy_batch.log` for any errors

**Highest-leverage next builds (rough priority):**
1. **Answer the 7 operator-policy questions** from cross-agent audit §7. Specifically Q1 (FP vs FN tolerance) and Q7 (editor as pre-ship gate) unblock D056 (editor verdict persistence).
2. **Correct the cross-agent audit** — minor footnote on R3 framing.
3. **Correct project instructions** — clarify editor's calibration-only status.
4. **DEFERRED.md numbering sync** — `cat ... | grep "^| D" | sort` and renumber any collisions.
5. **Once D056 ships and editor verdicts persist** — Escalation Triage agent becomes buildable. Real R-tier rubric or confidence-based triage can be designed at that point with real data.

**Do NOT:**
- Build Escalation Triage until verdict persistence is full (both audit_guide and editor)
- Build SSG throttle until SSG actually ships content
- Treat "prompt written" as "build shipped" — always confirm CC executed before downstream work

---

*End of context save. Three real builds shipped (verdict persistence audit_guide half, asbestos deploy throttle, doc-vs-reality findings). Two lessons learned and documented. Three parallel chats coordinated cleanly despite one timeline-drift confusion. Foundation is harder than it was 3 hours ago.*
