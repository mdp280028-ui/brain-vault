# AITEAM Context Save — Doc-Reality Fixes + Operator Policy Lock

**Date:** 2026-05-18
**Session length:** ~3 hours, morning into afternoon
**Chat role:** One of two parallel chats; this chat handled doc/policy/bookkeeping foundation work while sister chat (per operator) handled separate items
**Session character:** Foundation hardening. No new agents built. Five brain commits + one project-file update. Three pre-flight HALTs caught real issues that would have produced wasted or misleading work.

---

## TL;DR

Cleaned up four threads of accumulated foundation debt: DEFERRED.md was missing 10 D-items discussed in sister chats; project planning docs had three doc-vs-reality drifts (R1/R2/R3, editor-as-judge, Cloudflare→Vercel); D045's "Opus → Sonnet cost lever" premise turned out to be stale (already on Sonnet); and the 7 operator-policy questions from the cross-agent failure modes audit were finally answered and locked to disk.

**Five brain commits, zero pipeline code touched, three real spec errors caught by CC pre-flight.** Project instructions on Claude.ai bumped to v2.1 with substantial expansion (g-rule, parallel-chat coordination, Max-sub nuance, 7 policy questions listed, cross-project boundary, autonomous loop status).

POLICY.md is the durable artifact. It unblocks D056 (editor verdict persistence), F17/F18 (cost cap enforcement), watchdog agent build, and run-batch.sh lockfile work.

---

## SESSION ARC

1. Opened with a 10-item priority list derived from three context saves (16th SSG migration, 16th autonomous loop live, 17th verdict persistence).
2. Operator split work across two parallel chats by file boundary (this chat = docs + DB queries; sister chat = CC work on agents).
3. Morning verification (#1) was attempted but operator bailed — only ~3 hours had passed since the cron entries landed, not enough activity to query usefully.
4. DEFERRED.md sync (#4): grep revealed 10 missing entries (D059 + D-SSG-01 through D-SSG-09) discussed in sister chats but never written. CC appended cleanly to a new "Late additions" section.
5. Doc-vs-reality fixes (#5): three gaps from the May 17 verdict-persistence findings. Two of the three were already corrected on-disk in active docs (only stale in historical context-saves); one (R1/R2/R3) needed a correction box prepended to the cross-agent audit doc.
6. Operator delegated full v2.1 project-instructions rewrite to this chat after the on-disk corrections landed. Generated comprehensive v2.1 with substantial additions beyond the three doc-vs-reality fixes.
7. Comprehensive project-file sweep before finalizing v2.1 — caught operator-style guidance gaps and codified them (g-rule clarified, TTWC permanently disabled, parallel-chat coordination as behavioral rule, Max-sub nuance, 7 policy questions listed, cross-project boundary).
8. Option A (D045 rewire): CC pre-flight HALTed with material finding — writer/auditor already on Sonnet via `--model sonnet` hardcoded at lines 107-108. D045's premise stale. Operator chose Option 1: close obsolete, open properly-scoped successor.
9. D061/D062 successor items written. D060 number collision caught by CC (already used for untracked-scripts entry); CC requested confirmation before deviating from prompt. Used D061/D062 instead.
10. Option B (7 operator-policy questions): walked through Q1-Q7 conversationally. Operator answered three directly (Q1, Q4, Q6); delegated remaining four to recommendation. Locked all 7 to POLICY.md.

---

## WHAT GOT SHIPPED

### Brain commits (5)

| SHA | Description |
|---|---|
| `b2a6fb0` | DEFERRED: sync D059 and D-SSG-01..09 from sister chats (2026-05-16/17) — 10 entries, 73 insertions |
| `bcc16d5` | Add cross_agent_failure_modes doc (was untracked) — 865 lines, file rescue |
| `38f77d0` | Doc-vs-reality: correct R1/R2/R3, editor production judge, Cloudflare→Vercel references (synced from 2026-05-17 verdict persistence findings) — 7 lines correction box |
| `9818e20` | DEFERRED: close D045 obsolete (writer/auditor already on Sonnet); open D061 (ai-do.sh rewire) + D062 (vestigial flag cleanup) — 9 insertions, 5 deletions |
| `6e7fb52` | POLICY.md: lock 7 operator-policy answers from cross-agent audit §7 — 54 lines, 3525 bytes |

### Project-file update (Claude.ai)

`AITEAM_Project_Instructions.md` bumped from v2.0 (May 14) to v2.1 (May 18). Substantial additions:
- The `g` rule codified: "shortest reply that gets the message across, 6 sentences max"
- TTWC permanently disabled (explicit, separate from "never do" list)
- Parallel-chat coordination as new behavioral subsection (4 documented patterns)
- Mission-bar Max-sub nuance: dashboard "API cost" is Max-sub burn, not real $
- Autonomous loop status: `drafter_queue.txt` → live in 30-90 min, throttle's 3-hour window
- The 7 operator-policy questions listed in full, with which gate which downstream work
- Cost cap reconciliation added as Open Decision (3 disagreeing caps, no enforcement)
- Cross-project boundary with SMART explicitly stated
- Phase 1 step progress updated — 10c, 17b, 18, 19 all marked ✅
- Lessons learned sections added for May 15-16 (autonomous loop) and May 17 (verdict persistence)
- Three doc-vs-reality fixes (editor as calibrated-not-production-judge, Vercel not Cloudflare, R1/R2/R3 only in plans)
- Commit-message discipline as locked rule (new)
- Updated agent fleet count (9+), site count (1 live), articles (32)
- Cloudflare Pages added to Killed list
- DEFERRED.md confirmed as primary source of truth (corrects v2.0's "no separate file needed" line)

### No agent code touched
Zero changes to `~/agents/` or `~/projects/`. All work was bookkeeping + docs + policy.

---

## KEY FINDINGS / DECISIONS

### Three pre-flight HALTs paid for themselves
1. **D045 premise stale.** CC's pre-flight discovered run-batch.sh lines 107-108 hardcode `--model sonnet` as default for WRITER_MODEL and AUDITOR_MODEL. The "Opus → Sonnet" cost lever already happened (history unclear when, but it's done). Saved a ~15 min CC rewire that would have shipped no cost reduction, then a misleading commit message claiming ~75% savings.
2. **D060 collision.** CC's pre-flight on the closure prompt caught that D060 was already assigned to the untracked-scripts entry at line 380. Avoided a silent renumber that would have orphaned references.
3. **Doc-vs-reality already-corrected on disk.** CC's grep for "production judge" found zero active-doc matches — all 5 hits were in historical context-saves (skipped per rule). Gap 2 was already corrected in active docs. Same for Cloudflare→Vercel (LESSONS.md, HANDOFF.md, deploy_throttle_build_2026-05-17.md all already accurate). Saved Claude from inventing edits where none were needed.

### Pattern worth keeping: pre-flight discipline catches doc/code drift
Same pattern as the May 17 session caught (R1/R2/R3 doesn't exist, editor has no production runner, deploy is Vercel not Cloudflare). The fix isn't more careful planning — it's CC verifying premises before executing. This session reinforces: never relax pre-flight to "speed up" a one-line change.

### Operator-policy decisions (locked to POLICY.md)

| Q | Policy |
|---|---|
| Q1 | False positives much worse — gates lean strict. Accept ~5 good losses per bad-block. |
| Q2 | $25/day soft warning (Telegram + drafter pause) + $50/day hard stop (SYSTEM_PAUSED). |
| Q3 | Manual page-only rollback. Auto-strip of links/sitemap deferred to 3rd incident. |
| Q4 | 4-20 clean slugs burn-in before evaluating review-window reduction. |
| Q5 | Serialize run-batch.sh via lockfile. One pipeline at a time per site. |
| Q6 | Build watchdog agent now (not deferred). |
| Q7 | Editor runs AFTER audit_guide.py as second gate. Both must pass. |

### D045 successor items (D061 + D062)

**D061** — Route run-batch.sh through ai-do.sh for kill-switch enforcement + per-call token attribution. Real remaining value after D045's cost premise turned stale. Trigger: after F17/F18 cost cap built (Q2 needs kill-switch hooks ai-do.sh provides). Requires:
- ai-do.sh accepts `--dangerously-skip-permissions` pass-through (pipeline is cron, non-interactive)
- `AGENT_MAX_TURNS` made env-overridable for writer/auditor (current cap 30, calls typically 10-20 turns)
- Pass agent_id ("asbestos-writer" / "asbestos-auditor") as $2 for clean token_usage attribution
- SSG fork has same 8 callsites, separate commit/scope

**D062** — Remove vestigial `--sonnet` / `--sonnet-audit` CLI flags + WRITER_MODEL/AUDITOR_MODEL indirection. Dead code; both set what's already hardcoded as default. Bundle with D061.

---

## DEFERRED ITEMS LOGGED OR FLAGGED THIS SESSION

### Logged on disk
- **D061** — ai-do.sh rewire for kill-switch + attribution (successor to D045)
- **D062** — vestigial flag cleanup in run-batch.sh

### Flagged but not formally logged
- **Pre-existing D045 collision** — CC reported D045 number is also used by a Cowen archived item (line 167) and an authoring-notes mention (line 415). Both pre-existing; file already documents the collision. Not blocking, but worth a future hygiene pass to renumber.
- **POLICY.md not yet referenced from PRIME.md or HANDOFF.md** — future CC sessions should treat POLICY.md as authoritative, but won't know to read it unless PRIME.md is updated. Worth a 1-minute edit to PRIME.md to add POLICY.md to "read at session start" list.

---

## OPEN LOOPS / FLAGS

1. **Morning verification skipped.** The 07:00 digest, 20:00 preview ping, 23:00 deploy_batch from May 17 night were never confirmed to have fired. The audit_log query returned zero rows for the relevant actors in 18 hours — could mean cron entries never installed, or could mean operator's clock was off. Worth a confirmation tomorrow morning with a wider time window query.

2. **Project instructions v2.1 not yet applied in Claude.ai UI.** File generated and presented to operator. Operator needs to manually paste into the project file in the Claude.ai right-side panel for v2.1 to be active in future sessions.

3. **The next-builds queue is now well-defined and unblocked.** D056 (editor verdict persistence) is the highest-value next build. Watchdog and F17/F18 follow. Lockfile is small but tied to Q5.

4. **Sister chat ran in parallel** — split was clean (this chat = docs/policy, sister = agent code). Cross-checked operator's most recent statements before acting on sister-chat-derived claims (per parallel-chat-coordination rule now codified in v2.1). No collisions surfaced.

---

## LESSONS LEARNED

### Pre-flight catches stale premises, not just spec errors

D045 sat in the priority list for two days with a "~75% cost reduction" estimate. The lever was already pulled. Without CC's pre-flight grep, this session would have shipped a no-op rewire and a misleading commit message. The lesson reinforces May 17's lesson: doc-vs-reality drift is the norm in a fast-moving solo project, and the fix is verification-before-execution, not more careful planning.

### Operator delegated three Q-answers to recommendation — worked well

For Q2/Q3/Q5/Q7, operator said "what do you recommend on all the other questions, give me short explains like I'm 14." Got clean recommendations with rationale. Operator approved all four in one message ("ok this looks good"). Pattern worth keeping: when policy questions have defensible defaults and operator is making 7 decisions in a row, recommendations > options for fatigue management.

### D-number collision detection is now a documented routine

This session ran the `grep -oE "\bD[0-9]+\b" ~/brain/projects/aiteam/DEFERRED.md | sort -u -V` check explicitly, caught the D060 collision via CC's own pre-flight, and added D061/D062 cleanly. The detection routine works; parallel chats can keep adding items as long as the routine is run before any new D-numbered append. Now codified in v2.1's parallel-chat-coordination subsection.

### "Already corrected" is a valid CC report — don't force edits

CC's Gap 2 + Gap 3 findings on the doc-vs-reality task came back as "zero editable matches needing change" (active docs already accurate, only stale in historical context-saves which are skipped per rule). The right move was to accept the report, not to ask CC to find something to edit. Forced edits would have degraded already-accurate docs. Trust the recon.

### Project instructions are now substantially denser

v2.1 is ~50% longer than v2.0 and contains material that will keep future sessions on-rails: the g-rule, parallel-chat coordination, the 7 policy questions, Max-sub nuance, cross-project boundary. The cost of length is reading-time; the value is fewer re-derivations and fewer doc-vs-reality drifts. Watching for whether it's too dense in practice.

---

## FILES CREATED / MODIFIED

### In `~/brain/projects/aiteam/`
- `DEFERRED.md` — 10 entries added via D059 + D-SSG-01..09 sync (commit b2a6fb0); D045 closed obsolete + D061/D062 added (commit 9818e20)
- `POLICY.md` — created (commit 6e7fb52)
- `docs/cross_agent_failure_modes_2026-05-16.md` — added (was untracked, commit bcc16d5); correction box prepended (commit 38f77d0)

### In this Claude.ai project
- `AITEAM_Project_Instructions.md` — v2.1 generated (operator applies via UI)
- `AITEAM_Context_Save_2026-05-18_DocRealityFix-PolicyLock.md` — this file

### Not touched
- Nothing in `~/agents/`
- Nothing in `~/projects/` (asbestos-contractors, asbestoshq-site, ssg-content all untouched)
- HANDOFF.md (deferred — recommend a 1-line PRIME.md edit next session to add POLICY.md to session-start reading list)
- LESSONS.md (deferred — entries from this session can land via brain-autocommit if logged in diary)

---

## NEXT CHAT — WHERE TO RESUME

Read in order:
1. This context save
2. `~/brain/projects/aiteam/POLICY.md` (new authoritative source for the 7 policy answers)
3. `~/brain/projects/aiteam/HANDOFF.md` (current state)
4. `~/brain/projects/aiteam/DEFERRED.md` (D061, D062 new)

### Highest-leverage next builds (rough priority, all now unblocked by POLICY.md)

1. **D056 — editor verdict persistence.** Q1 + Q4 + Q7 answered. Build the editor production runner: takes one real draft, scores it (Sonnet via ai-do.sh), persists to `auditor_verdicts` table with composite_score populated. Pass threshold = operator-tunable env var, FP-leaning default per Q1. Slots AFTER `audit_guide.py` per Q7. Highest-value next build.

2. **Watchdog agent.** Q6 = build now. Spec exists in cross-agent audit §6. Monitors other agents — alive/dead checks, failure alerts, cron-fired-but-no-completion-row, audit-log anomalies.

3. **F17/F18 — cost cap enforcement.** Q2 answered ($25 soft / $50 hard). After every `INSERT` into `token_usage`, query today's sum; if > soft cap → Telegram warning + drafter cron pause; if > hard cap → touch sentinel that `check_kill_switches.sh` treats as `SYSTEM_PAUSED=true`.

4. **Lockfile in run-batch.sh.** Q5 = serialize. ~10 lines bash. Check `/tmp/run-batch-<site>.lock` at start; exit-with-skip if present; create on entry; EXIT trap deletes. Per-site lockfile so asbestos and ssg can run independently.

5. **D061 + D062.** After F17/F18 (cost cap needs the kill-switch hooks D061 unlocks). Both bundled into one commit since same callsites.

### Operator-only follow-ups

- Apply project instructions v2.1 manually in Claude.ai UI (paste into project file)
- Morning verification when wider time window available (was 3 hours, not 18 — query for first real 20:00 ping + 23:00 deploy_batch confirmation)
- Keyword research + enqueue 5-10 asbestos slugs into `drafter_queue.txt` (loop is live; without slugs, $0 revenue)
- AdSense enrollment / affiliate enrollment for asbestos (no monetization yet, content live)

### Do NOT

- Build D061 before F17/F18 — gating dependency is real
- Build editor cascade (Haiku triage → Sonnet final) until production volume justifies — deferred per project rules
- Touch SSG run-batch.sh until SSG site rewrite is done (separate scope)
- Treat the 7 policies as re-derivable in conversation — POLICY.md is authoritative

---

## SIGN-OFF NOTE

Five brain commits, one project-file update. No premature ✅. Three pre-flight HALTs caught real issues (D045 stale premise, D060 collision, doc-vs-reality already-corrected). Zero misleading commit messages. Parallel-chat coordination held — no collisions with sister chat.

The foundation is materially harder than it was 3 hours ago. POLICY.md alone unblocks D056 + watchdog + F17/F18.

Mission bar: $0 / $200 mo. Loop is live, policy is locked, next builds are queued. Operator-side work (keyword research, monetization enrollment) is still the rate-limiting step for revenue.

---

*End of context save. Resume next session by reading this file first, then POLICY.md, then HANDOFF.md, then DEFERRED.md.*
