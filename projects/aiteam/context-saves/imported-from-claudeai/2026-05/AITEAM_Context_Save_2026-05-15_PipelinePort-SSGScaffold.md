# AITEAM Context Save — Pipeline Port & SSG Scaffold
**Date:** 2026-05-15
**Session focus:** Decide adapt-vs-rebuild for content pipeline; port asbestos pipeline as SSG working scaffold; inventory production repos.

---

## TL;DR

Spent the session resolving "build new content/audit pipeline or reuse existing one." Resolution: **reuse, don't rebuild.** Cloned 5 production GitHub repos (UST-Contractors-, asbestos-contractors, asbestoshq-site, smartsourceguide, payrolldetective). Discovered UST and asbestos pipelines have **forked** rather than one being downstream of the other. Chose asbestos as the SSG template (more recently field-tested for domain adaptation; SSG is guide-only at launch so doesn't need UST's service pipeline yet). Ported asbestos pipeline to local working folder `~/projects/ssg-content/`. Moved 101 asbestos content artifacts to `~/projects/_asbestos-reference/` to preserve as gold standard. Did **not** push ssg-content to GitHub — destiny of that repo undecided until smartsourceguide site/deploy is understood.

No code shipped. No first article. Real work next session = SSG keyword research + understanding smartsourceguide deploy pipeline.

---

## Decisions made (and why)

### 1. Adapt existing pipeline, don't build new
The UST pipeline is ~6 months of hardened production: 30+ guides + 51 service pages shipped, 6 documented bugs found/fixed, lockstep spec discipline, ~30 mechanical audit checks. Building equivalent quality from scratch = months.

### 2. Use asbestos-contractors as the SSG template (not UST)
Pipelines have forked. Snapshot:

| File | UST version | Asbestos version | Winner for SSG |
|---|---|---|---|
| run-batch.sh | v10.11 (2026-04-29) | v10.13 (2026-04-24) | asbestos (newer minor) |
| audit_guide.py | v3.2 | v3.4 (adds G1 SEMANTIC-VARIETY) | asbestos |
| AUDIT_SPEC.md | v1.6.5 (has SPC-16 UMBRELLA-KEYWORD-GUARD) | v1.6.4 | UST has more, but SPC-16 is for service pages SSG doesn't need yet |
| CONTENT_SPEC_GUIDE_*.md | v2.3 | v2.7 | asbestos (newer pattern) |
| CONTENT_SPEC_SERVICE_*.md | v1.7 fully implemented | STUB | UST, but irrelevant for guide-only SSG launch |
| spc-audit.py | Present (service pages) | Absent | UST, irrelevant for now |

Asbestos won 4 of 6 axes. The 2 UST-only features (SPC-16, service pipeline) don't apply to SSG at launch.

**Reconciliation deferred.** Merging UST + asbestos into a unified "v11" is correct engineering but 1-2 days of work with risk of breaking either live pipeline. Will revisit when SSG ships service pages OR asbestos needs SPC-16.

### 3. Preserve asbestos artifacts, don't delete
101 asbestos files (34 approved guides + keyword research + 30 assignment batches + per-slug configs + reference docs) moved to `~/projects/_asbestos-reference/` rather than deleted. They're not SSG content but they're gold-standard examples of what the pipeline produces. Useful for prompt tuning later.

### 4. No GitHub push for ssg-content yet
Operator pushed back: "you don't know anything about that project yet." Correct call. The smartsourceguide site is 5 weeks dormant, has zero docs/specs/keywords/CLAUDE.md, just 14 mostly-empty page.tsx files across 4 categories. Before declaring "smartsourceguide-content" the canonical SSG content pipeline repo, need to understand how the site actually gets content + deploys.

### 5. Skip throwaway test article
Operator: "we will test it when we try it with our first article." Agreed — the first real article IS the validation test. No reason to run a separate smoke test.

### 6. claude -p → ai-do.sh rewire deferred
Operator's path: ship first article on existing `claude -p` (Opus default), then rewire to `ai-do.sh` (Sonnet 4.6) in week 2 to drop cost. The "Sonnet was too slow" verdict from old sessions is being held open for retest — likely a prompt-size problem, not a model-speed problem.

---

## Files created / moved / state

### Repos cloned to `~/projects/`
| Repo | Purpose | Last commit |
|---|---|---|
| UST-Contractors- | Live UST production pipeline | 2026-05-14 |
| asbestos-contractors | Asbestos production pipeline | 2026-04-27 |
| asbestoshq-site | Asbestos Next.js site | 2026-04-27 |
| smartsourceguide | SSG Next.js site (dormant) | 2026-04-10 |
| payrolldetective | Payroll detective Next.js site | 2026-04-26 |

### Created locally
- `~/projects/ssg-content/` — asbestos pipeline clone, content/asbestos → content/ssg, CONTENT_SPEC_*_ASBESTOS → CONTENT_SPEC_*_SSG. **Local only, no remote.** HEAD: e12b122.
- `~/projects/_asbestos-reference/` — 101 files (34 approved guide JSONs, 34 keyword configs, 28 assignment batches, 5 reference docs, 1 keyword research report, 2 site-level docs, plus 4 strategy-doc-looking files moved later)

### NOT cloned (deferred)
- `payroll-detective` (hyphenated) — repo does not exist on GitHub. Pipeline lives only at `~/Desktop/payroll-detective/` which is TCC-blocked.

### Inside ssg-content/content/ssg/ (final state)
- CONTENT_SPEC_GUIDE_SSG.md (25KB, **still asbestos-flavored v2.7** — needs SSG rewrite)
- CONTENT_SPEC_SERVICE_SSG.md (stub)
- CONTENT_SPEC_SSG.md (stub)
- approved/guides/.gitkeep (empty, ready)
- keyword-configs/.gitkeep (empty, ready)
- audit-configs/guide.json
- Zero assignment-batch-*.md files (4 stragglers moved at end of session)

### Inside ssg-content/ pipeline scaffold (still asbestos-named internally)
- content/run-batch.sh v10.13 — 8 hardcoded "AsbestosHQ.com" persona strings in writer/auditor prompts (lines 410, 444, 474, 569, 595, 847, 881, 972)
- scripts/audit_guide.py v3.4 — 5 asbestos references (comments/version headers)
- content/AUDIT_SPEC.md v1.6.4 — 3 asbestos references
- 62 asbestos references in run-batch.sh, 22 in scripts/ctx.sh, 13 in scripts/auto-harvest-to-library.sh

---

## Problems encountered & solutions

### macOS TCC blocking ~/Desktop/ from Terminal
**Problem:** Terminal showed "Operation not permitted" on `ls ~/Desktop/`, even with Full Disk Access toggled on. `sudo` didn't override.
**Tried:** Toggling FDA off/on, full Terminal Cmd+Q restart. Neither worked.
**Workaround used:** Finder can read what Terminal can't — operator dragged needed folders to ~/Desktop/AITEAM FILES/ and ~/Desktop/Desktop/ (still TCC-blocked though), and uploaded individual files (run-batch.sh, audit_guide-2.py, spc-audit.py, AUDIT_SPEC.md, AUDIT_SPEC-3.md) directly to Claude for inspection.
**Real fix:** Defer to end-of-day — try `sudo tccutil reset SystemPolicyAllFiles` + reboot + re-add Terminal to FDA. Not blocking.

### CC silently dropped Task 1 in early prompt
**Problem:** Multi-task CC prompt with payrolldetective clone as Task 1 + asbestos port as Task 2 — CC did Task 2 only, skipped Task 1.
**Solution:** Re-ran payrolldetective clone as a standalone task in later prompt. Cloned successfully.
**Lesson:** Multi-task CC prompts have a real failure rate on the first task. Either run tasks sequentially or explicitly call out completion at the end of each. Operator's project instructions already flag this pattern ("sign-off discipline").

### Confusion about what files were canonical
**Problem:** OldMacFiles + Downloads + uploads had multiple copies of run-batch.sh, audit_guide.py, AUDIT_SPEC.md at different version numbers. Couldn't tell which was current.
**Solution:** Stopped reading old copies. Cloned the live GitHub repos. Versions in cloned repos = canonical. Versions in OldMacFiles/Downloads = old snapshots, ignored.

### "Sonnet was too slow" verdict from prior sessions
**Likely cause:** UST pipeline assembles a giant static-specs.md blob (AUDIT_SPEC + APPENDICES + SITE_MAP) and feeds it to every `claude -p` call. Sonnet processes large prompts slower per token; Opus hides this by being smarter-per-token. The fix isn't picking Opus; it's trimming the prompt blob + capping MAX_ROUNDS + using --sonnet-audit. Not solved this session, but the diagnosis is on the record.

---

## Cost data points

- No API spend this session — all work was local file inventory + git operations.
- Earlier project context: UST pipeline ~$2-4 per article on Opus default. Sonnet 4.6 untested on current pipeline shape.

---

## DEFERRED ITEMS

1. **Desktop TCC fix** — `~/Desktop/OldMacFiles/`, `~/Desktop/AITEAM FILES/`, `~/Desktop/Desktop/`, `~/Desktop/payroll-detective/` all blocked. **Trigger:** when one of those folders contains something we need. Try `sudo tccutil reset SystemPolicyAllFiles` + reboot first.

2. **smartsourceguide site research** — Understand how content gets into the live site today, what the deploy pipeline is, and whether smartsourceguide.com is the right first AITEAM target or whether AITEAM should build on a fresh domain it controls end-to-end. **Trigger:** before first SSG assignment is written.

3. **ssg-content repo destiny decision** — Is this the canonical SSG content pipeline repo? Push to GitHub when? Or keep separate? Naming (smartsourceguide-content vs ssg-content vs something else)? **Trigger:** after #2.

4. **payroll-detective pipeline review** — Lives only at `~/Desktop/payroll-detective/` (TCC-blocked). Could be moved via Finder + pushed to GitHub if useful. **Trigger:** only if SSG hits a problem UST/asbestos pipelines didn't solve.

5. **6 strategy docs in cc-context/05-REFERENCE/ported-from-ust-adapted/** — Strategic_Prompts, Fake_Rotation_Strategy, Data_Processing_Playbook, Directory_Business_Context, State_Database_Research_Skill, Guide_Creation_Workflow. UST→asbestos adaptation playbooks. Possibly adaptable for SSG. **Trigger:** when writing the SSG CONTENT_SPEC_GUIDE.

6. **CONTENT_SPEC_GUIDE_SSG.md rewrite** — Currently still asbestos-flavored v2.7, 25KB, 367 lines. Largest piece of remaining adaptation work. **Trigger:** next SSG session, after keyword research.

7. **CLAUDE.md at ssg-content/ root** — Still says "Asbestos Project Context." References `~/Desktop/CC ASBESTOS FILES/` which is TCC-blocked. Needs SSG rewrite. **Trigger:** when SSG voice/scope is decided.

8. **8 hardcoded "AsbestosHQ.com" persona strings in run-batch.sh** — Lines 410, 444, 474, 569, 595, 847, 881, 972. **Trigger:** before first SSG writer fires (or it'll write as AsbestosHQ).

9. **62 asbestos references inside pipeline files** (run-batch.sh + audit_guide.py + AUDIT_SPEC.md + CONTENT_SPEC_GUIDE_SSG.md + audit-configs/guide.json + ctx.sh + auto-harvest-to-library.sh) — Targeted edits, not blind sed. **Trigger:** before first article runs.

10. **claude -p → ai-do.sh rewire** — Replace 8 direct `claude -p` calls in run-batch.sh with calls to `~/agents/lib/ai-do.sh` (or ai-think.sh for auditor). Drops cost from Opus to Sonnet 4.6. **Trigger:** after first SSG article ships successfully on current `claude -p` config.

11. **Pipeline reconciliation (UST v10.11 vs asbestos v10.13)** — Merge SPC-16 + G1 SEMANTIC-VARIETY + service pipeline into a unified pipeline. Risk: breaks either live pipeline during merge. **Trigger:** when SSG needs service pages OR asbestos needs SPC-16.

12. **Ahrefs keyword research for SSG** — Operator's planned next move. Across 4 categories (answering-services, fleet-tracking, it-support, plus pricing as sub-topic). **Trigger:** before writing first SSG assignment-batch.

13. **CC multi-task prompt reliability** — CC silently dropped Task 1 once this session. Either run tasks sequentially or build explicit "confirm Task N complete" before moving on. **Trigger:** ongoing operating principle.

---

## Open questions for operator

- **Is smartsourceguide.com the right first AITEAM target, or fresh domain?** This decision has been bouncing around without resolution. Worth a dedicated think.
- **What does "all 3 topics" mean for the first article?** Operator said first SSG article should cover "all 3 topics on the site." Pipeline produces one guide per slug. Need to clarify — does that mean one article per topic across the first batch of 3? Or one article that touches all 3?
- **Fleet-tracking** is the 4th category in the SSG repo. Operator only mentioned 3 (IT support, pricing, answering services). Was fleet-tracking forgotten or intentionally deprioritized?

---

## What was left mid-task

Nothing mid-task. Clean stopping point. Next session can start cold from the deferred items list.

---

## Resume instructions for next session

1. Read this file
2. Verify state: `cd ~/projects/ssg-content && git log -1` → expect `e12b122 cleanup: move 4 asbestos assignment stragglers`
3. Address deferred item #2 (smartsourceguide site research) before any more pipeline work
4. Or address deferred item #12 (Ahrefs keyword research) if site research is already done in operator's head
5. Do not start CONTENT_SPEC_GUIDE_SSG.md rewrite until keyword research + voice/scope decisions are in hand — without those it's a 25KB asbestos-find-and-replace and the result won't be useful SSG content
