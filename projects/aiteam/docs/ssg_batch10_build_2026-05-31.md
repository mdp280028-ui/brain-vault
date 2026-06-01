# SSG Batch ssg10a — Build Report (2026-05-31)

**Pipeline:** Path B (audited) — `ssg-content/content/run-batch.sh ssg10a --guide --max-pages 10`
**Run window:** 2026-05-31 12:14:40 → 16:38 PDT · **wall time 263m 31s (4h 23m)**
**Models:** Sonnet writer + Sonnet auditor (via `~/agents/lib/ai-do.sh`) → `audit_guide.py` dual-pass mechanical gate.
**Exit code:** 2 (usage cap detected)
**Hard stop honored:** YES — nothing whitelisted, copied to the site repo, committed, or pushed. All artifacts remain in `ssg-content`.

## Result: 0 of 10 approved ❌

| Tally | Count |
|---|---:|
| ✅ Approved | **0** |
| ⚠️ Escalated → needs-review | 6 (2 retained a full draft, 4 lost to empty output) |
| ⏸️ Capped (empty writer response) | 4 |
| ⏭ Skipped | 0 |

`content/ssg/approved/guides/` holds only the pre-existing `answerconnect-review.json` — **zero ssg10a guides reached approved JSON.**

## Root cause: Anthropic usage cap / severe throttling

Not a content-design or config problem. Evidence:
- **Every round-1 writer call ran ~1800s (exactly 30 min) — the ai-do.sh hard timeout.** A normal Sonnet guide write is 1–3 min. Calls were crawling, i.e. throttled.
- Pages whose follow-up rounds returned empty in <10s tripped the script's CAP heuristic (4 pages).
- The script exited **code 2 = "usage cap detected → cooldown, do not immediately restart."**

So the cap throttled writer calls to their timeout, producing empty/thin drafts that couldn't pass audit. The 4h23m wall time is the symptom (10 pages × ~30-min timeouts, 2 at a time).

## Per-slug outcome

| # | slug (category) | outcome | draft retained | words | notes |
|---|---|---|---|---|---|
| 1 | answering-services/hipaa-compliant-medical-answering-service | ⏸️ capped | no | — | R1 writer hit 1800s timeout, R2 empty → cap |
| 2 | it-support/datto-alternatives | ⏸️ capped | no | — | same |
| 3 | it-support/manageengine-alternatives | ⏸️ capped | no | — | R1 empty → cap |
| 4 | it-support/atera-reviews | ⏸️ capped | no | — | R1 empty → cap |
| 5 | fleet-tracking/motive-vs-samsara | ⚠️ escalated (FAILED_AUDIT struct+slug) | **YES** `needs-review/motive-vs-samsara.json` | **1814** | full draft, 6 H2 / 3 FAQ — salvageable |
| 6 | it-support/ninjaone-vs-atera | ⚠️ escalated | no | — | R2/R3 empty (throttle) |
| 7 | fleet-tracking/field-service-management-software-quickbooks | ⚠️ escalated | no | — | R2/R3 empty |
| 8 | fleet-tracking/samsara-pricing | ⚠️ escalated | no | — | R2/R3 empty |
| 9 | it-support/ninjaone-reviews | ⚠️ escalated | no | — | R2/R3 empty |
| 10 | fleet-tracking/linxup-reviews | ⚠️ escalated (FAILED_AUDIT struct+slug) | **YES** `needs-review/linxup-reviews.json` | **1714** | full draft — failed only 2 trivial checks (below) |

### The two retained drafts are near-misses, not failures of substance
- **linxup-reviews (1714w, 6 H2, 3 FAQ):** round-3 mechanical fails were only **S6** (intro paragraph = 6 sentences; cap is ≤3) and **INTENT1** (1/5 intent concepts in first 150 words; min 2). Both are intro-paragraph tweaks. Content is otherwise complete.
- **motive-vs-samsara (1814w, 6 H2, 3 FAQ):** failed the dual mechanical gate (struct=1, slug=1); full-length, correct slug/category. Salvageable on a clean writer round.

## Built and staged (reusable, all in ssg-content)

- **10 keyword-configs** in `content/ssg/keyword-configs/` (bare-leaf filenames, validated JSON).
- **Assignment batch** `content/ssg/assignment-batch-ssg10a.md` (10 pages, full scaffolding, authority links, anti-cannibalization notes).

## Bug found + fixed (precache substring collision)

`run-batch.sh` precache assigns each slug the **first** page chunk containing the slug as a substring. Page 6 (ninjaone-vs-atera) literally referenced "ninjaone-reviews", so `assignment-ninjaone-reviews.md` got vs-atera content. **Fixed** by rewording cross-references in Pages 6/8/9 so no page embeds a later page's slug; re-simulated the splitter → all 10 map to their own chunk (ALL CLEAN). Fix is in place for the re-run.

## Recommended next step (NOT executed — your call)

1. **Wait for the usage cap to reset** (the script signals cooldown, not immediate restart). Re-running now would re-trip it.
2. Then re-run the **same** command — it's resumable (skip-if-approved) and the precache rebuilds from the fixed batch:
   `cd ~/projects/ssg-content && bash content/run-batch.sh ssg10a --guide --max-pages 10`
   Suggest chunking (`--max-pages 3` ×3–4) to stay clear of the cap, and run when API headroom is high.
3. The two retained drafts can alternatively be hand-fixed (linxup needs only intro sentence-split + 1–2 intent keywords up front) but the clean path is letting the re-run regenerate them.
4. STEP 4 (relatedLinks wiring) deferred — nothing approved yet to wire.

## Correction to a mid-run status note

An earlier update claimed pages 1–2 (hipaa-medical, datto) were "approved round 1." That was premature and wrong — they wrote round-1 drafts, failed audit, then fell into the capped set. Authoritative tally: **0 approved.**
