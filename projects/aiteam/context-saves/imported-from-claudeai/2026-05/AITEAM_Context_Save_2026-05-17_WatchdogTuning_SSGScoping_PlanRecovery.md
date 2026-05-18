# AITEAM Context Save — Watchdog Tuning + SSG Scoping + Plan Recovery

**Date:** 2026-05-17 (afternoon, ~13:00–15:00 PT)
**Session length:** ~2 hours
**Chat role:** Primary chat working from operator's 5-item afternoon list
**Session character:** Two clean closures + one substantive recovery. Watchdog tuning pass complete (4 commits, 4 pre-flight HALTs catching real issues). SSG scoping complete with site repo located + lost plan doc recovered from conversation history and saved as project artifact. D066 spillover deferred to next chat for context reasons.

---

## TL;DR

**Watchdog tuning pass: complete.** Four commits, zero scope creep, zero bundled changes, zero premature ✅. Real safety bug (corrupted state.json alert storm) closed. Real semantics bug (`grace_minutes` discarded) closed. Drafter window tightened from 2h to 1h. LESSONS.md doc entries landed.

**SSG scoping: complete.** Site repo found at `~/projects/smartsourceguide/` (not "ssg-*" — recon's earlier grep missed it). Lost `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md` recovered from conversation history via `conversation_search` and saved as `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-17.md` (Claude project artifact; needs to be saved to `~/brain/projects/aiteam/docs/` next session). 3-lane scoping framework established (Content / Site / Wire-up).

**D066 spillover: not started.** Operator paused due to context burn. Next chat opens with the `~/projects/` half of D066.

Mission bar: $0 / $200 mo. Foundation harder than session start. Revenue still gated on operator-side work (slugs into drafter_queue.txt + AdSense enrollment).

---

## SESSION ARC

1. Opened with operator's 5-item working list (subset of 10-item priority list from session start):
   1. Morning cron verification (read-only)
   2. GSC queue end-to-end smoke test
   3. Watchdog tuning pass
   4. SSG data-driven rewrite scoping
   5. D066 spillover / `~/projects/` half if parallel
2. #1 skipped — not enough time since cron entries landed (only ~hours).
3. #2 parked — `gsc_submission_queue` empty, no shipped slug yet to test against.
4. #3 Watchdog tuning pass executed in full. Four CC build cycles, each with recon → HALT → pre-flight findings → operator GO → build → test → commit → sign-off. Four pre-flight HALTs caught real issues.
5. #4 SSG scoping started chat-side. CC recon of `~/projects/ssg-content/` confirmed pipeline-ready but content inputs zero. CC could not find SSG site repo locally — operator's strongest guess was GitHub. CC ran second recon and found `~/projects/smartsourceguide/` already on disk (sibling repo to ssg-content, not named "ssg-*"). All 14 article markers matched the 2026-05-16 dump. No clone needed.
6. Plan doc `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md` not on disk, not in Claude project. Claude searched conversation history via `conversation_search` — found the authoring session (chat `5c03da96-ae1c-4431-851e-41a618741049`, 2026-05-16). Reconstructed plan verbatim into `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-17.md` with recovery note. Presented as project artifact.
7. #5 D066 spillover paused — operator chose to ctx + continue in fresh chat for context reasons. Sound call.

---

## WHAT GOT SHIPPED — WATCHDOG TUNING PASS

### Watchdog fix #5 — corrupted state.json guard
**Commit:** `1053913 fix(watchdog): guard against corrupted state.json — single alert + flag dedup`

**Bug:** state.json corruption (invalid JSON) silently returns empty string from `state_get` via `2>/dev/null`. Empty prev_status + current "missing" triggers healthy→missing transition every 15-min tick — alert storm, indefinitely.

**Fix:** 25-line validity gate after bootstrap. `jq -e 'type == "object"'` rejects null/arrays/strings (tighter than plain `jq empty`). Dedup via flag file at `~/store/flags/WATCHDOG_STATE_CORRUPT`. Forensic payload `{"size": N, "head200": "..."}`. DRY-RUN-aware.

**Tests:** All 14 sub-checks green (a–i plus j–n for empty-file case). Exactly 2 Telegram pings during test sequence (corruption events), 0 during dedup verifications.

### Watchdog fix #3 — grace_minutes wired into threshold math
**Commit:** `38ed525 feat(watchdog): wire grace_minutes into threshold math`

**Bug:** `grace_minutes` field present in `expected_schedule.yaml` for all 7 agents but read into a variable and discarded. Threshold math used only `NOW - check_window_hours * 3600`. Field looked load-bearing in YAML but wasn't.

**Fix:** 10-line patch. All four threshold sites (3 probes + main loop) now subtract `${grace_minutes:-0} * 60`. Probes need grace too — un-graced probe threshold would drop grace-eligible signals before they reach the main loop. Defensive `${VAR:-0}` for bash 3.2 safety on missing field.

**Provable evidence:** e1/e2 boundary test — same NOW, same code, same YAML except `grace_minutes`. e1 (grace=0): drafter=missing. e2 (grace=35): drafter=healthy. Flip on grace alone proves the field is now consulted.

### Watchdog fix #2 — tighten assignment-drafter check_window
**Commit:** `26d3311 tune(watchdog): tighten assignment-drafter check_window to 1h`

**Tuning (not bug):** With grace now wired, drafter at `check_window=2h + grace=10m` had 2h10m tolerance — could miss 3 consecutive `*/30` ticks before alerting. Tightened to `check_window=1h + grace=10m` = 1h10m tolerance. Single missed tick still healthy; second missed tick triggers alert.

**Behavior change:**
| Silence | Old (2h+10m) | New (1h+10m) |
|---|---|---|
| 30m | healthy | healthy |
| 60m | healthy | alerts |
| 90m | healthy | alerts |
| 2h | alerts | alerts |

### Watchdog fix #1 — LESSONS.md doc entries
**Commit:** `1e90165 docs(lessons): audit_log column name + heartbeat-before-killswitch validation`

**Entries:**
1. `audit_log` column is `payload_json`, not `payload`. Cross-referenced existing line 87 (yesterday's same lesson); flagged as second occurrence so future Claude/CC writes correct column name in spec queries from the start.
2. Drafter heartbeat fires BEFORE kill switches consult flags. Observed in the wild 2026-05-17 13:02:02 — manual off-cycle run wrote `drafter_tick_started` (id 1149) before `skipped_pause_flag` (id 1150). Watchdog correctly saw drafter as alive throughout. Commit reference: fb53d04 (heartbeat-add). Generalizes to future agents.

New `## 2026-05-17` heading opened. Sub-section `### Watchdog hardening + tuning (2026-05-17 11:00–13:15 window)`. Bold-lead-in style matching recent entries.

### Side note — D064 closed by sister chat
Between commits, sister/operator landed `ebb0507 fix(lib): D064 — log_to_audit.sh arg-shape validation`. Closes the positional-vs-keyvalue malformation pattern from 2026-05-17 evening. Verify it actually rejects ACTOR_TYPE containing `=` next session.

### Three nits logged, not bundled
1. Four duplicate threshold computations in watchdog.sh (3 probes + main loop) — could be a single helper. Units-divergence risk now that formula has two terms.
2. Unused `_stale` action suffix returned by probe_logfile_mtime/probe_sqlite_row — main loop reads only the timestamp and re-evaluates. Dead information.
3. state.json reparsed 14× per tick (7 actors × 2 fields × jq subprocess). Already flagged in earlier recon. Refactor, not fix.

All three out of scope for tuning pass. Surface in a future watchdog hygiene session.

---

## WHAT GOT SHIPPED — SSG SCOPING

### CC recon findings (2 passes)
- `~/projects/ssg-content/` — pipeline machinery, fork-complete, byte-current with asbestos improvements (D061 routes through ai-do.sh, D-SSG-05 SITE_MAP grep fixed, lockfile/throttle hooks, Sonnet pinned, D062 cleanup). 5 SSG-native spec markdowns authored v1.0. Content inputs zero. Drafter gated OFF.
- `~/projects/smartsourceguide/` — Next.js site repo, found on second recon pass. All 14 article markers from 2026-05-16 dump match: `/app/` at root (not `src/app/`), 5 fleet-tracking + 4 it-support + 5 answering-services, bare Next 14, no Tailwind, `vercel.json` present, remote `git@github.com:mdp280028-ui/smartsourceguide.git`, last commit `c05c1f6` 2026-05-16.
- The `ls ~/projects/ | grep -i ssg` from earlier recon missed it — repo is named `smartsourceguide`, not `ssg-*`. Sister repos by design.

### 3-lane scoping framework
| Lane | State | Blockers |
|---|---|---|
| **A — Content pipeline** (`ssg-content/`) | ~95% ready | D-SSG-02 sign-off (cosmetic tolerance + verdict rules), D-SSG-03 sign-off (relatedLinks slug shape), 6 input files to author (`keyword_research.md`, `approved-index.md`, `authority-links.md`, `drafter_queue.txt`, 2+ exemplar `assignment-batch-*.md`) |
| **B — Site data-driven rewrite** (`smartsourceguide/`) | 0% executed; plan recovered | Execute Phase 1 chassis → canary → 13 articles → validation → enablement |
| **C — Wire-up** | 0%, gated on A + B | Flip `ssg.yaml: enabled: true`, D059 (SSG deploy throttle pattern-copy from asbestos `3ed57dc`), D065 (SSG GSC submission queue hook) |

**Critical scoping insight:** Lane A produces JSON (per `SSG_OUTPUT_FORMAT.md v1.0`). Lane B's purpose is making the site consume that JSON. **Lane B must land before Lane A's output has anywhere to go.** Lane A can produce drafts/approved guides without B, but they sit in `approved/guides/` until B is done.

### Plan doc recovery
- Original `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md` lost — not on Mac, not in Claude project files
- Found in chat `5c03da96-ae1c-4431-851e-41a618741049` (2026-05-16 authoring session) via `conversation_search`
- Reconstructed verbatim into `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-17.md`
- Includes: JSON schema (full), template code (`app/[slug]/page.tsx`), helper code (`lib/guides.ts`), 5-phase migration approach (chassis → canary → 13 articles → validation → enablement), component responsibilities table, JSON-LD plan, CSS approach, 4 locked operator answers preserved from 2026-05-16 context save
- Added: per-phase CC time estimates, pre-execution checklist, risks section, recovery note
- **State:** saved as Claude project artifact; NEEDS TO BE SAVED TO `~/brain/projects/aiteam/docs/` next session

### 4 locked operator answers (preserved verbatim from 2026-05-16)
| # | Question | Locked answer |
|---|---|---|
| 1 | Author name to backfill all 14 | `"SmartSourceGuide Editorial"` (single persona, can fragment later) |
| 2 | datePublished backfill | CC pulls git log first-commit date per file; fallback `"2026-04-01"` |
| 3 | Canary article pick | CC picks — 2-level breadcrumb, NOT on the 4-mismatch list, ~130 lines |
| 4 | Migration timing | After Ship-To-Site ✅ (now true since 2026-05-16 commit `3b43c11`) |

---

## KEY FINDINGS / DECISIONS

### Pre-flight HALTs paid for themselves four times this session
1. **#5 — `jq -e 'type == "object"'` tighter than `jq empty`.** CC's pre-flight proposed the refinement; would have shipped a guard that accepted `null`, `[]`, strings, numbers as valid state.json roots otherwise.
2. **#5 — empty-file (0 bytes) case not in original test plan.** Operator added tests j–n on the GO message; CC confirmed `jq -e` rejects empty input as expected.
3. **#3 — probes need grace too.** Original spec only patched line 301 (main loop). CC's pre-flight surfaced that probes filter signals before they reach the main loop, so un-graced probe threshold would drop grace-eligible signals; main loop would mark missing regardless of grace.
4. **#3 — e1/e2 pair makes the flip provable.** Original spec's step e (`check_window=0, grace=1`) was not provable in isolation — result is "missing" whether grace is consulted or not (unless we land inside the 60s window). CC proposed e1 (grace=0) → missing AND e2 (grace=35) → healthy with everything else held constant. The flip on grace alone is direct evidence.

### "Calibration ≠ production" is now a project pattern lesson
Watchdog tuning surfaced same pattern as the cost-cap session: any agent transitioning from spec/calibration/idea to production needs explicit pre-flight recon before build. Four real bugs caught this session via pre-flight before any code shipped. Don't relax pre-flight to "speed up" a one-line change.

### conversation_search rescued the SSG plan
The plan doc was assumed lost (not in Claude project, not on disk, not findable in `/mnt/project/`). `conversation_search` against the authoring chat ID + targeted queries pulled the full content back in pieces. Lesson: when a critical doc is missing, search conversation history before reconstructing from memory or asking operator to re-author. Documented chat IDs help.

### "Repo not found" was wrong — was just named differently
SSG site recon assumed missing-from-disk and prepared a GitHub locator prompt. The repo had been on disk since 2026-05-15 — wasn't `ssg-*`, was `smartsourceguide`. Lesson: when an `ls | grep` returns nothing, also try `ls` alone and scan by metadata (recent activity, file structure) rather than name pattern. Conserve hops.

### Off-cycle observation strengthened a design
At 13:02:02 someone (operator or another shell) ran drafter manually off-cycle while DRAFTER_PAUSED was set. Heartbeat fired BEFORE kill switch silenced the work — confirming the heartbeat-before-killswitch design from earlier in the day (commit fb53d04). Watchdog stayed correct. Documented as LESSONS.md entry.

---

## FILES CREATED / MODIFIED

### In `~/agents/`
- `watchdog/watchdog.sh` — 25-line state.json validity gate added (1053913); 10-line grace_minutes patch (38ed525)
- `watchdog/expected_schedule.yaml` — assignment-drafter `check_window_hours: 2 → 1` (26d3311)

### In `~/brain/`
- `projects/aiteam/LESSONS.md` — `## 2026-05-17` heading + sub-section + 2 entries (1e90165)

### In this Claude project
- `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-17.md` — recovered plan, presented as project artifact (NOT yet saved to `~/brain/projects/aiteam/docs/`)
- `AITEAM_Context_Save_2026-05-17_WatchdogTuning_SSGScoping_PlanRecovery.md` — this file

### Crontab
No changes this session.

### System state
- `~/store/flags/WATCHDOG_STATE_CORRUPT` — created during test, removed after recovery. No flag present at sign-off.
- Audit log — multiple new rows from watchdog tests + production ticks. All clean.

### Touched by sister chat (not this chat)
- `~/agents/lib/log_to_audit.sh` — D064 arg-shape validation (ebb0507). Verify next session.

---

## DEFERRED ITEMS

### New this session
- **D067 (proposed)** — Save recovered SSG rewrite plan to disk: `cp /path/to/AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-17.md ~/brain/projects/aiteam/docs/ && cd ~/brain && git add . && git commit -m "docs(ssg): recovered SSG data-driven rewrite plan from 2026-05-16 chat history"`. Trigger: opening of next session.
- **D068 (proposed)** — Watchdog hygiene pass: collapse 4 duplicate threshold computations into single helper; remove unused `_stale` action suffix; refactor state.json single-load-at-tick-start to eliminate 14× jq reparse. Trigger: after D066 closure; not urgent.

### Carried forward, no change
- D025, D026, D028, D033, D039, D040, D041, D042, D043, D052, D053, D055, D056, D057, D058, D059, D060, D063, D065, D066
- D-SSG-01, D-SSG-02, D-SSG-03, D-SSG-04, D-SSG-06, D-SSG-07, D-SSG-08, D-SSG-09

### Closed since last context save
- D064 (log_to_audit.sh arg-shape validation) — closed by sister chat (`ebb0507`)
- Watchdog tuning recon items #1, #2, #3, #5 — all four closed this session

---

## HANDOFF — NEXT CHAT

### Read in this order
1. **This context save** (you're reading it)
2. `~/brain/projects/aiteam/PRIME.md` — points at HANDOFF / POLICY / DEFERRED
3. `~/brain/projects/aiteam/HANDOFF.md` — current state
4. `~/brain/projects/aiteam/POLICY.md` — 7 locked operator policies
5. `~/brain/projects/aiteam/DEFERRED.md` — D066 + D067 (proposed) + D068 (proposed) are the freshest items

### Opening moves

**1. Save the recovered SSG plan to disk (D067).** ~5 min. Operator has `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-17.md` from this Claude project. Copy to `~/brain/projects/aiteam/docs/`, add + commit in brain repo. After this, `brain-autocommit.sh` keeps it versioned automatically.

**2. D066 spillover — `~/projects/` half.** This was the deferred item from end of this session. Scoped per operator: `~/projects/` repos only (NOT the full `~/agents/` audit — that's the dedicated session). Repos to check:
   - `~/projects/asbestos-contractors/`
   - `~/projects/asbestoshq-site/`
   - `~/projects/ssg-content/`
   - `~/projects/smartsourceguide/` (newly relevant after this session's recon)
   - `~/projects/_asbestos-reference/`, `~/projects/friend-bot/`, `~/projects/payrolldetective/`, `~/projects/UST-Contractors-` (verify state, cheap scan)
   
   Format: recon first (read-only inventory of uncommitted/untracked state), then a commit-grouping plan, then execute groups one at a time with chat-side approval.
   
   Estimated 20-40 min CC + 10-15 min chat-side review. Smaller than full D066 (which is 60-90 min including `~/agents/`).

**3. Then morning cron verification.** Now wide enough time window for a useful query — 06:00 digest, 20:00 preview, 23:00 deploy_batch, 00:00 cost-cap reset should all have fired. Single SQL query against audit_log.

**4. Then GSC queue smoke test.** Only if a slug shipped overnight (check `gsc_submission_queue` count first).

### Open work units, in priority

| Item | Type | Estimated | Status |
|---|---|---|---|
| D067 — save SSG plan to disk | Hygiene | 5 min CC | Ready |
| D066 — `~/projects/` half | Audit | 30-55 min | Ready (this is opener) |
| Morning cron verification | Read-only | 5 min | Ready |
| GSC smoke test | Operator-side | 10 min | Gated on shipped slug |
| D056 — editor production runner | Build | 60-90 min CC | Ready (POLICY Q1/Q4/Q7 answered) |
| D066 — `~/agents/` half | Audit | 60-90 min CC | Ready (dedicated session) |
| SSG Lane B execution | Build | 3-5 hours CC | Plan recovered, ready |
| SSG Lane A content inputs | Operator | varies | Operator-only (D-SSG-02/03 sign-offs + 6 files) |

### Do NOT
- Build new features until D066 `~/projects/` half closes (per project rule, every new build adds to hygiene crisis surface area). D066 `~/agents/` half can also wait, but `~/projects/` is bite-sized and should land before more builds in those repos.
- Execute SSG Lane B (data-driven rewrite) until plan is on disk (D067 closes first). Spec-on-disk is the standard before multi-phase migrations.
- Touch the three watchdog nits (duplicate thresh, unused _stale suffix, state.json reparse) — D068, not urgent.
- Treat the D064 sister-chat closure as verified until next session confirms behavior.
- Defer to "Cloudflare Pages" anywhere — Vercel, always. (Already corrected on-disk per 2026-05-18 doc-vs-reality fixes; flagging for context-burned future Claude.)

### Critical context for next Claude
- **Watchdog is now fully tuned.** State-corruption safe. Grace-aware. Drafter window tightened. Don't treat audit_log queries the same way — watchdog's reliability data is now usable.
- **Sister chats may have run between this save and next session.** Cross-reference operator's most recent statement before acting on any sister-chat claim. Periodic D-number sync via `grep -oE "\bD[0-9]+\b" ~/brain/projects/aiteam/DEFERRED.md | sort -u -V` catches collisions.
- **SSG site is at `~/projects/smartsourceguide/`, not anything `ssg-*`.** Don't repeat the earlier-recon miss.
- **Plan doc is recovered but lives only in Claude project until D067.** Saving it to disk is the first move next session.
- **Operator's research-before-building bias** — quiet today, but the session was build-heavy. Two consecutive build-heavy sessions before this one. Momentum is good; don't let the next chat slide into open-ended planning.

---

## SIGN-OFF NOTE

Four commits across two repos. Four pre-flight HALTs caught real issues. Zero premature ✅. Zero misleading commit messages. Watchdog tuning closed cleanly. SSG plan recovered via conversation history — would have been lost otherwise. 3-lane SSG scoping framework documented.

Mission bar: $0 / $200 mo. Watchdog is materially more reliable than 2 hours ago. SSG has a recovered execution plan and a confirmed site repo. Revenue is still gated on operator-side work (slugs into drafter_queue.txt + AdSense enrollment).

D066 `~/projects/` half is the natural opener next chat. Then morning cron verification when window is wider. Then editor production runner (D056) or SSG Lane B, depending on appetite.

---

*End of context save. Resume next session by reading this file first, then PRIME.md (which points at POLICY/HANDOFF/DEFERRED automatically). Save the SSG plan to disk before anything else.*
