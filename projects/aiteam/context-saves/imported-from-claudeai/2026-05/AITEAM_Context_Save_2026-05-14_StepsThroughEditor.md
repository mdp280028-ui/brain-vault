# AITEAM Context Save — Steps Through Editor Tier Decision
**Date:** May 14, 2026 (~10 PM Pacific)
**Session duration:** ~2 hours
**Operator:** Stonecreed (Bonners Ferry, ID)
**Session API spend:** ~$1.40-1.65

---

## ONE-LINE SUMMARY

Closed 7 Phase 1 steps (14, 10e, 10f, 15, 16, 10d, 17), locked Sonnet as editor tier production judge, generated Project Instructions v2 and a comprehensive session handoff. CC is currently working on step 10c (cost chart).

---

## STEPS CLOSED THIS SESSION

| Step | Outcome | Cost | Commit |
|---|---|---|---|
| 14 — Orchestrator smoke test | ✅ CLAUDE.md loads, roster reads, routing coherent | ~$0.01 | committed |
| 10e — Chat input + standup endpoint | ✅ 4-table persistence verified, UI renders | ~$0.01 | committed |
| 10f — @mention parsing | ✅ Differential routing test passed | bundled w/16 | committed |
| 15 — Daily diary writer | ✅ 1455-byte diary written, audit_log id=5 | ~$0.02-0.03 | 491f5e8 |
| 16 — Librarian (Haiku) | ✅ Inbox sort + differential mention test | ~$0.01-0.02 | committed |
| 10d — Diary view | ✅ Dashboard renders existing diaries | $0 | 893fe97 |
| 17 — Editor tier test | ✅ **Sonnet wins, locked as judge** | ~$1.02 | id=17 audit_log |

**Step deferred:** 10b (kanban) — mission_tasks empty, revisit when an agent emits rows.

**Step in progress:** 10c (cost chart) — CC has the prompt.

---

## KEY DECISIONS MADE

### 1. Sonnet 4.6 is the production editor tier (LOCKED)
- Pearson r=0.995 vs operator manual (which equals Opus by operator instruction)
- Pearson r=0.841 for Haiku — systematically generous, ceiling-stuck
- Sonnet ≈ Opus at r=0.995 — Opus held in reserve only (5x more expensive for identical output)
- audit_log row id=17, correlation_id E7BE235B-C333-45BB-A95C-293B882C8C71

### 2. Manual scoring caveat (operator-acknowledged shortcut)
- `_manual_scores.csv` was set equal to `_opus_scores.csv` after CSV-editing blocker and operator's "~98% taste alignment with Opus" OOB confirmation
- The Sonnet-wins result is transitively guaranteed given construction
- Substantive validity depends on operator's OOB review being sound
- Noted in audit_log payload — not hidden

### 3. Cascade pattern identified, DEFERRED
- Production strategy that emerged: Haiku triage → Sonnet final gate
- Not built yet — waits for production volume to justify

### 4. TTWC footer disabled permanently
- Operator explicitly: "NO TTWC this session"
- Now codified in Project Instructions v2

### 5. Deferred-items system codified
- Single source of truth: end-of-session context saves under "DEFERRED ITEMS"
- No separate file — operator reviews at session start
- Now codified in Project Instructions v2

### 6. Project Instructions v2 generated
- File: `AITEAM_Project_Instructions_v2.md`
- Replaces v1 (which still referenced unbought hardware, CrewAI, Llama, etc.)
- Operator needs to drop into project knowledge to replace v1

---

## FILES CREATED OR MODIFIED THIS SESSION

### On Mac Mini
```
~/agents/editor/agent.yaml
~/agents/editor/CLAUDE.md
~/agents/editor/rubric.md
~/agents/editor/run_tier_test.sh
~/brain/projects/aiteam/diary/2026-05-14.md  (1455 bytes)
~/brain/projects/aiteam/raw/editor_test_corpus/
  ├── 01_slug.md through 27_slug.md  (27 article files)
  ├── _tier_intent.csv               (operator's intended tiers)
  ├── _manual_scores.csv             (= Opus by operator instruction)
  ├── _haiku_scores.csv              (preserved Haiku draft)
  ├── _sonnet_scores.csv             (production judge baseline)
  └── _opus_scores.csv               (reserve)
```

Plus standard git commits at `~/agents/` per step.

### Generated in Claude.ai outputs
- `AITEAM_Project_Instructions_v2.md` — replaces v1 in project knowledge
- `AITEAM_Session_Handoff_2026-05-14_StepsThroughEditor.md` — handoff for next chat

---

## NEW LEARNINGS / DATA POINTS

### Cost data
- Sonnet editor pass on 27 articles: **~$0.17**
- Opus editor pass on 27 articles: **~$0.83**
- Haiku editor pass on 27 articles: **~$0.02-0.05**
- WebFetch + Haiku extraction on 20 articles: **~$0.30-0.50** (and Haiku summarized instead of extracting — wasted spend)
- curl + trafilatura: **free, full-fidelity** — use this method going forward for corpus assembly

### Model behavior
- Haiku composite score systematically 0.63 points higher than Sonnet/Opus
- All 6 articles with >1.0 spread: Haiku was the high outlier, never the reverse
- Haiku is ceiling-stuck — bunches good content at 5/5/5/5/5, can't discriminate above threshold
- Hook is the most discriminating axis. Clarity is the least discriminating.
- Sonnet ≈ Opus is genuinely surprising — confirms Sonnet as cost-optimal for routine quality work

### Technical
- macOS bash is 3.2 not 4.x (already known from prev session — confirmed again)
- node:sqlite is the dashboard's DB driver (not better-sqlite3 — already known)
- CSV editing on macOS via Excel default-app routing is operator-friction — text editor (TextEdit, BBEdit) works fine but operator didn't know this
- WebFetch's "extraction" uses Haiku and summarizes; not appropriate for verbatim corpus assembly

### Operator interaction
- Operator gets frustrated with friction. When CSV editing blocked him, he chose "skip the integrity check" over "20 more minutes of manual work." Push back ONCE, then execute his call.
- "Explain like I'm 14" requests worked twice — use this when an explanation isn't landing
- Operator multitasks across many windows; keep responses short and visible
- Trail-off characters at message end ("g", "fg") are not meaningful — ignore

### Process
- CC tried to jump from step 17 → step 18, skipping 10c/10b/17a/17b. Re-prompted with locked sequence. **Watch for this pattern in future sessions.**
- Operator considered scoring articles himself, then changed his mind, then changed it again. Final approach: Opus scores as proxy for manual scores, with OOB validation by operator's word.

---

## DEFERRED ITEMS

This is the canonical list. Operator should review at start of next session.

| ID | Item | Why deferred | Trigger to revisit |
|---|---|---|---|
| D001 | Editor cascade pattern (Haiku triage → Sonnet final gate) | Production volume not high enough yet | When publishing ≥100 articles/week or seeing real Sonnet cost at scale |
| D002 | Kanban view (step 10b) | mission_tasks table still empty | When any agent emits mission_tasks rows |
| D003 | Editor tier test re-validation with operator-typed manual scores | CSV editing blocker + operator burnout | Quarterly confirmation, or if Editor's published output starts failing operator's taste |
| D004 | HANDOFF.md on Mac Mini says "Chassis build pending" — stale | Mid-session, no time to update | Update at start of next CC session |
| D005 | First site target decision (PLR/Etsy/Gumroad leaning) | Doesn't block Phase 1 | When Phase 2 worker agents begin |
| D006 | Voices seed list (5+ names) | Not blocking | When voices ingestion is next on build plan |
| D007 | Persona project | Operator: "soonish, depends" | When operator raises it explicitly |
| D008 | launchd service strategy (one or multiple) | Defer until Phase 2 | When grammy Telegram bot port begins |
| D009 | Backup verification cadence | No first restore has happened yet | After first successful test restore |
| D010 | Step 17a (silver platters) | May not have real data yet | After 10c and 17b, evaluate then |
| D011 | UST tier intent re-labeling | Operator hasn't spot-checked whether his UST pieces are "middling" or different | Optional, low priority |
| D012 | gwern proto_essays.md swap-out in corpus | Was index page not actual essay; CC flagged but unfixed | Only if running another corpus test |
| D013 | Replace AITEAM_Project_Instructions v1 with v2 in project knowledge | Generated this session, not yet swapped in | Operator action — drop file into project knowledge |

---

## WHAT'S LEFT MID-TASK

**Step 10c (cost chart) is in progress.**

- CC received the 10c prompt asking for `/api/tokens` endpoint, chart rendering with Haiku/Sonnet/Opus breakdown, total spend, and a design-choice surfacing (line chart vs stacked area vs total + breakdown table)
- ⏱️ estimate requested
- Sign-off bar: chart must render in browser with real data from today's runs (not just JSON endpoint returning)

**Next chat resumes by:**
1. Reading CC's response to the 10c prompt (most likely)
2. OR: Operator confirms session is done for the night
3. OR: Operator drops Project Instructions v2 into project knowledge before logging off

---

## WHERE TO RESUME

Next Claude session should read:
- `AITEAM_Project_Instructions_v2.md` (assuming it's now in project knowledge)
- `AITEAM_Session_Handoff_2026-05-14_StepsThroughEditor.md` (this handoff)
- `AITEAM_Session_Handoff_2026-05-14_Step14-Orchestrator.md` (previous handoff for builder profile, locked substrate, Mark calibrated values, day-one gotchas)

Then:
- Opening move: "Paste CC's response to step 10c and we'll continue."

---

## OPERATOR CORRECTIONS TO CLAUDE THIS SESSION

1. **TTWC footer** — operator killed it explicitly. Codified.
2. **Deferred items file → context save approach** — operator preferred context save over separate file.
3. **Manual scoring shortcut** — operator overrode Claude.ai's pushback on test integrity. Lesson: push back once clearly, then execute.
4. **"STOP WASTING MY FUCKING TIME"** — explicit signal to drop the discussion and act. Don't litigate the same point three times.

---

*End of context save. Operator can resume in a new chat by pasting this handoff or the longer session handoff file into project knowledge.*
