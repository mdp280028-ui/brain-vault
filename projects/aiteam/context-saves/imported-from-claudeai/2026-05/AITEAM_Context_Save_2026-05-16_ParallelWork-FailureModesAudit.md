# AITEAM Context Save — Parallel Chat Coordination + Cross-Agent Failure Modes Audit

**Date:** 2026-05-16
**Session focus:** Run as one of three parallel chats. Closed 8 items on the parallel-work list, surfaced the cross-agent failure surface, brought brain docs current.

---

## TL;DR

This chat ran as one of three parallel Claude.ai sessions, each handing CC prompts. The track here closed eight items: Config Synthesizer build prompt → commit/triage → test-config cleanup → page.tsx template extraction → APPROVED_GUIDE_SLUGS verification → cross-agent failure modes audit → provenance strip → HANDOFF.md + DEFERRED.md update.

The cross-agent audit was the headline. It mapped the full agent fleet, documented 23 cross-agent failure modes (F1-F23), placed them on a 2×2 severity matrix, and surfaced two live bugs (F2 missing-config burn, F4 SQL injection in log_to_audit.sh) plus a cost-cap reconciliation gap. Ship-to-site was discovered to have never been in git history despite running in production for a full session — fixed via commit 3b43c11. HANDOFF.md (badly stale, said "Chassis build pending") and DEFERRED.md (didn't exist) brought current via commit e402c5a.

Three chats reached a clean coordinated stopping point at session end. Next chat queue: F2 fix, F4/D044 SQL escape, Escalation Triage build, Failure Pattern Reporter build, 7 operator-policy questions from cross-agent audit §7.

---

## WHAT GOT DONE THIS CHAT

### 1. Config Synthesizer build prompt (written here, built by Chat C)

Full CC build prompt written for Agent #2 in the pipeline-wrap stack. Sonnet tier, 3 input modes (single-flag, brief-file, batch queue), schema validation + audit_guide.py --validate-config-only dry-run. Chat C executed; agent landed at `~/agents/config-synthesizer/` with 10/11 sign-off. Build report: `~/brain/projects/aiteam/docs/config_synthesizer_build_2026-05-16.md`. Cost: $0.36 for full sign-off run.

### 2. Commit + triage of Config Synthesizer work

Two commits, two repos:
- Commit 69dd30f in `~/agents/` — config-synthesizer agent (16 files, 1,402 insertions)
- Commit e504793 in `~/projects/asbestos-contractors/` — audit_guide.py --validate-config-only flag (+96/-1, additive only)

Both commits scoped explicitly — neither swept in unrelated uncommitted work. Triage doc at `~/brain/projects/aiteam/docs/synth_configs_triage_2026-05-16.md` evaluated the 4 synthesized test configs against drafts/feedback/approved/assignment-batch/site-whitelist — all 4 came back TEST.

### 3. 4 test keyword-configs deleted

All 4 verified test artifacts with no downstream consumer; removed from `~/projects/asbestos-contractors/content/asbestos/keyword-configs/`. Working tree clean.

### 4. page.tsx template extraction

CC sampled 3 representative shipped slugs (chrysotile, asbestos-shingles-guide, how-to-test-popcorn-ceiling-for-asbestos). Surprise finding: all 31 per-slug wrappers are exactly 21 lines, byte differences are pure slug-length artifact. Diff reduces to 3 lines — same slug substring in JSON import path, alternates.canonical URL, openGraph.url. Zero hardcoded-per-page fields. Template at `~/agents/ship-to-site/templates/page-tsx-template.tsx`. Report at `~/brain/projects/aiteam/docs/page_tsx_template_extraction_2026-05-16.md`.

### 5. APPROVED_GUIDE_SLUGS verification

Already done during Ship-To-Site build per operator. No new work needed.

### 6. Cross-agent failure modes audit

The biggest deliverable. Read all current agents (orchestrator, librarian, ship-to-site, config-synthesizer, assignment-drafter in flight), shared lib helpers, build reports, production repos, SQLite schema. Output at `~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md`.

Seven sections: agent fleet map, 23 documented failure modes (F1-F23, exceeded the 18 minimum), 2×2 severity ranking, top 5 priority fixes, detection gaps, recommended monitoring/alerting spec, 7 open operator questions.

**High/high quadrant:** F2 (drafter fires without keyword-config), F14 (Haiku-tier false positives on quality gate), F17 (cost runaway in tight loop), F18 (no daily API spend hard stop).

**Live bugs surfaced:**
- F4 — SQL injection in `~/agents/lib/log_to_audit.sh`. Already hit by Chat C during Config Synthesizer build, defensively patched at caller layer. Matches D044 in memory but upgraded from "nice to fix" to "every agent calling this is silently broken."
- F2 — drafter fires without keyword-config check. One-line bash test in fire_pipeline.sh prevents $1-5/slug Opus burn.

**Cost cap disagreement:** Three numbers, none coherent. DAILY_API_BUDGET_USD=10 is observability-only. Drafter has its own $15/day cap enforced internally. Nothing watches rate-of-change.

### 7. Provenance comment strip

Edit to `~/agents/ship-to-site/lib/stage.sh` strips the 24-line provenance comment block (CC corrected my 22-line count) from the template before write. Verified byte-identical to a real shipped wrapper. Commit 3b43c11 in `~/agents/` — *which was also the first-ever commit of the entire ship-to-site/ directory*. CC caught this mid-execution and surfaced it: ship-to-site had been running autonomously in production for an entire session without ever being staged in git. Fixed by committing the whole directory with the strip tweak baked in.

### 8. HANDOFF.md + DEFERRED.md updates

`~/brain/projects/aiteam/HANDOFF.md` rewritten — 6 sections per spec (current phase, agents-live as 12-row table, production state, 6 open critical items, sign-off discipline rule, next-session opening move). Prior contents said "Chassis build pending" — wildly stale.

`~/brain/projects/aiteam/DEFERRED.md` created (didn't exist before). Priority view at top, all existing D-items carried forward, new D045 (ai-do.sh rewire) / D046 (notify.sh env override bug) / D047 (chrysotile draft decision) / D048 (provenance strip — now shipped) / D051 (~/agents/ repo hygiene) added, 23 F-items cross-referenced rather than duplicated. Grouped by category (Live bugs, Infrastructure, Pipeline hardening, Content/operational, Hygiene).

Commit e402c5a in `~/brain/` (+342 / -193).

---

## KEY FINDINGS / DECISIONS

### Autonomous chain went live mid-session

Yesterday's drafter patch 2 (per Chat C / operator messages) made the chain end-to-end autonomous: operator edits `drafter_queue.txt`, content is live on the internet ~30-90 min later, no human review gate. This radically changed the cross-agent failure surface. The audit was commissioned partly in response to this shift.

### Three parallel chats coordination worked

By session end the chats had cleanly split:
- This chat: documentation, audit, repo hygiene, prep work
- Chat B: SSG site data-driven rewrite plan (#11) → CONTENT_SPEC_GUIDE_SSG rewrite (#10)
- Chat C: Config Synthesizer build → Assignment Drafter build

One real collision avoided early: Chat C wanted to build Config Synthesizer while CC was finishing Ship-To-Site. Pushback worked. Subsequent overlap avoided by explicit chat coordination on the running tally.

### Ship-to-site discovery: not in git

Major silent failure. The Ship-To-Site agent built in a prior session was running production deploys (asbestos site auto-ships) without ever being in git history. Discovered when stage.sh edit was attempted. Resolved with 3b43c11 — but this is a pattern worth noting: builds end with "✅ complete" without verifying commit landed.

### Cost cap is the next foundation bug

Pre-audit, the system felt safe because the Max subscription absorbs API cost. The audit made it concrete: nothing actually prevents a runaway loop from spending $500 in an afternoon. The Max sub masks the problem rather than solving it. When the project moves off Max (D013), this becomes immediate.

### F2 fix is the single highest-value next action

One-line bash test in fire_pipeline.sh that checks for `content/asbestos/keyword-configs/${slug}.json` before launch. Prevents $1-5/slug burn on slugs that get into the queue without a config. Cheapest possible fix with biggest single-incident savings.

---

## OPEN QUESTIONS (UNCHANGED FROM CROSS-AGENT AUDIT §7)

Seven operator-policy questions surfaced by the audit, all unanswered:
1. Auditor false-positive vs false-negative tolerance — which error is more acceptable?
2. Daily API cap — what number? Hard stop or soft?
3. Bad-content rollback policy — how does a live-on-the-internet bad article get yanked?
4. Burn-in publication count for an approval gate — at what slug count do we add re-introduce review?
5. Concurrent run-batch.sh policy — allow or serialize?
6. Watchdog agent priority — build now or after F2/F4?
7. Editor agent (Sonnet, idle, tuned) — slot in as pre-ship gate?

---

## DEFERRED ITEMS (NEW + CARRIED)

### Live bugs (highest priority next session)
| ID | Item | Trigger | Notes |
|---|---|---|---|
| F2 | Drafter fires without keyword-config — $1-5/slug Opus burn | Pre-flight one-liner in fire_pipeline.sh | High/high quadrant |
| F4 / D044 | log_to_audit.sh SQL escape on apostrophes | Fix in shared lib, not at caller layer | Already hit, currently defensively patched per-caller |
| F14 | Haiku-tier auditor false positives | Consider editor (Sonnet, idle) as pre-ship gate | Cross-audit §4 |
| F17/F18 | Cost runaway, no daily hard stop | Reconcile three disagreeing cost caps | High/high quadrant |

### From this session
| ID | Item | Trigger |
|---|---|---|
| D045 (collision — see note) | Rewire run-batch.sh from `claude -p` → `ai-do.sh` (Opus → Sonnet, ~75% cost cut) | Next pipeline-touching session. **Context-incomplete — CC couldn't find specifics in build reports.** |
| D045-old | Older archived "Opus prompt-template" item from Cowen build | Renumber to D050 next time DEFERRED is touched |
| D046 | notify.sh env override bug — stops stray test Telegrams | Next notify-touching session |
| D047 | Chrysotile draft decision: clean markers and ship, or skip | Operator decision |
| D048 | Provenance strip — **SHIPPED in 3b43c11** | Done |
| D051 | `~/agents/` repo hygiene — 10+ uncommitted/untracked items at session start | Dedicated hygiene session |

### Open decisions
- 7 audit §7 operator questions (listed above)
- Whether to build a watchdog agent (audit §6 spec exists)

### From operator memory (carried)
D025, D028, D033, D039, D040, D041, D042, D043 — all carried into DEFERRED.md per category groups.

---

## OPERATOR CORRECTIONS / GOOD INSTINCTS THIS SESSION

1. **"Other chat wants to work on Config Synthesizer — thumbs up or down?"** — Operator instinctively checks parallel work for collisions before authorizing. This pattern caught the only real collision risk in the session.

2. **"I don't fucking care" / "wtf you just said"** — Operator pushed back on overcomplication twice. Both times, simpler explanation was correct. Lesson: when the operator asks "explain like I'm 14 under 200 words," he means it.

3. **Chose Option 2 on ship-to-site commit scope** without prompting — instinctively understood that committing only stage.sh would leave the whole agent unprotected in git.

4. **Pushed back on a 3rd chat working on Config Synthesizer mid-build** — would have created two CC sessions building the same agent in parallel. Caught it.

5. **"You are getting low on context, let's just do a few smaller items"** — operator self-managed context budget. Productive choice; both prompts landed cleanly without rushing.

6. **"I don't think I gave CC the second prompt"** — operator double-checking what CC did. Diagnosed correctly that both prompts went into one message; CC executed both. Useful pattern to know for future multi-prompt messages.

---

## FILES CREATED / MODIFIED THIS SESSION

In `~/brain/projects/aiteam/`:
- `HANDOFF.md` — rewritten (commit e402c5a)
- `DEFERRED.md` — created (commit e402c5a)

In `~/brain/projects/aiteam/docs/`:
- `synth_configs_triage_2026-05-16.md` — new
- `page_tsx_template_extraction_2026-05-16.md` — new
- `cross_agent_failure_modes_2026-05-16.md` — new

In `~/agents/`:
- Whole `config-synthesizer/` agent (commit 69dd30f)
- Whole `ship-to-site/` (commit 3b43c11 — first-ever commit of the directory) including stage.sh provenance-strip edit

In `~/projects/asbestos-contractors/`:
- `scripts/audit_guide.py` — added `--validate-config-only` flag (commit e504793)
- 4 test keyword-configs deleted (asbestos-attic-insulation, asbestos-window-glazing, asbestos-textured-paint, asbestos-cement-board)

---

## NEXT CHAT — WHERE TO RESUME

Read in order:
1. `~/brain/projects/aiteam/HANDOFF.md` (just updated, accurate)
2. `~/brain/projects/aiteam/DEFERRED.md` (just created, complete)
3. This context save
4. `~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md` (the audit deliverable that should drive next priorities)

Pick from this queue (in rough priority order):
- **F2 fix** — one-line bash test in fire_pipeline.sh. 10 min CC prompt. Highest dollar/effort ratio in the project right now.
- **F4 / D044 fix** — SQL escape in `~/agents/lib/log_to_audit.sh`. Every caller currently silently broken on apostrophes.
- **7 operator-policy questions** — walk through them, get operator answers, lock policy. Required before more autonomous infrastructure.
- **Cost cap reconciliation** — three disagreeing numbers, no enforcement. Spec the right answer.
- **Escalation Triage agent** — diagnose R3 failures (asbestos remaining ~20%)
- **Failure Pattern Reporter** — weekly audit-check digest
- **Renumber D045-old to D050** — small DEFERRED.md hygiene

Do NOT build new agents on the shared lib until F4 is fixed. Building more callers on a known-broken shared helper is the wrong order.

---

*End of context save. This chat reached a clean stopping point with eight items closed, three parallel chats coordinated, foundation hardened.*
