# AITEAM Context Save — SSG Pipeline Migration (Full Session)

**Date:** 2026-05-16 → 2026-05-17 (session bridged midnight)
**Session topic:** Make the ssg-content pipeline runnable on SSG content; coordinate parallel chats; capture deploy throttle outcome
**This chat's track:** SSG pipeline migration (recon → spec rewrite → persona strings → 4 content files → AUDIT_SPEC adaptation → path migration)
**Parallel chats handled:** Asbestos deploy throttle, verdict persistence build, Ship-To-Site supporting tasks (Config Synthesizer spec, Assignment Drafter spec, page.tsx template extraction, APPROVED_GUIDE_SLUGS verification, failure modes pre-draft, Telegram slash commands, daily digest mockup, HANDOFF.md update, DEFERRED.md creation)

---

## TL;DR

The ssg-content pipeline at `~/projects/ssg-content/` is now ready to run on SSG content. 8 commits landed in `ssg-content/`. Single remaining blocker for first batch run is operator-authored assignment-batch file plus a 5-minute SITE_MAP grep fix.

In parallel, the asbestos deploy throttle shipped in another chat (commit `3ed57dc` in `~/agents/`). Verdict persistence shipped too (auditor_verdicts SQLite table).

SSG deploy throttle correctly deferred (D059) — building a throttle for a pipeline that doesn't ship yet is solving an imaginary problem.

---

## What got done in this chat (chronological)

### 1. SSG articles structure recon (CC)
- File: `~/brain/projects/aiteam/docs/ssg_articles_structure_dump_2026-05-16.md` (2,400 lines, 224K)
- 14 articles under `app/` at repo root (NOT 15, NOT `src/app/`)
- Bare Next 14 + React 18 + TS, no Tailwind, hand-rolled CSS
- Universal rigid skeleton: article-header → takeaway-box → h2/p body → faq-section (exactly 3) → cta-box
- 4 articles flagged for CTA/link mismatches
- Categories: 5 fleet-tracking, 4 it-support, 5 answering-services

### 2. SSG data-driven rewrite plan
- File: `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md`
- Converts 14 JSX-inlined → JSON + dynamic route
- Adds `zod` as only new dep
- Per-article commits for clean revertibility
- 4 open questions locked: Author = "SmartSourceGuide Editorial"; datePublished = git log first-commit date; canary = CC picks; timing = after Ship-To-Site ✅
- NOT yet executed — separate future session

### 3. SSG content spec source material recon (CC)
- File: `~/brain/projects/aiteam/docs/ssg_content_spec_source_material_2026-05-16.md` (1,288 lines)
- Confirmed existing `CONTENT_SPEC_GUIDE_SSG.md` was byte-identical to asbestos version

### 4. CONTENT_SPEC_GUIDE_SSG.md v1.0 rewrite
- File: `AITEAM_CONTENT_SPEC_GUIDE_SSG_v1.md`
- 16 sections, full B2B services voice
- Both Comparison/Pricing/"Best X" AND How-to/Process/Decision modes
- Word counts: 1,400-1,800 (comparison) / 1,200-1,600 (how-to)
- Banned phrases adapted for B2B (AI-powered, cloud-based, enterprise-grade, etc.)
- Vendor-neutrality rules added
- Closing patterns: compare-providers / evaluate-criteria / get-quotes

### 5. 8 persona string replacements (drafted + applied)
- File: `AITEAM_SSG_PersonaString_Replacements_2026-05-16.md`
- All 8 `claude -p` blocks in run-batch.sh replaced
- Commits: `a397fc0` (CONTENT_SPEC), `a6639d4` (persona strings)
- Issue caught + fixed: bare `$79` would be eaten as variable expansion. CC fixed to `\$79`.

### 6. Path migration inventory (CC)
- File: `~/brain/projects/aiteam/docs/ssg_path_migration_inventory_2026-05-16.md`
- Original count 36, actual count 44 (20 PATH blocking, 4 STRING, 20 COMMENT)
- Surprise finding: asbestos "production reference" files were 116-293 byte STUBS

### 7. Session A — 4 SSG pipeline content files authored (CC)
- SSG_OUTPUT_FORMAT.md (266 lines)
- SSG_SITE_MAP.md (135 lines)
- SSG_Writer_Reference.md (177 lines)
- SSG_TEMPLATES.md (395 lines)
- Commits: `d5d685d`, `b12ccf0`, `9667efa`, `d16e812`
- Authored from first principles (asbestos sources were stubs)
- CC caught + fixed 6 em dashes in SSG_Writer_Reference.md before commit

### 8. Sessions B + C — AUDIT_SPEC adaptation + path migration (CC)
- AUDIT_SPEC.md: G2 = PRICING-DENSITY, authority links = 3+ vendor/industry-research, closing patterns updated
- run-batch.sh: all 20 PATH refs migrated via sed, missing dirs created (`content/ssg/keyword-configs/`, `approved/guides/`, `drafts/`, `feedback/`), filename refs updated
- Commits: `1c8c41a` (AUDIT_SPEC), `100bff1` (path migration)
- 11 asbestos refs remain — all COMMENT lines per the "leave comments untouched" rule
- `bash -n` clean

### 9. SSG deploy throttle — HALTED then DEFERRED
- CC pre-flight halted correctly on 6 of 8 failed checks
- Timeline drift between parallel chats: asbestos throttle DID ship just AFTER CC's snapshot
- Both chats had accurate disk states at different moments
- SSG throttle correctly deferred — would gate a pipeline that doesn't ship yet
- Tracked as D059

---

## Commits this session (`~/projects/ssg-content/`)

| SHA | Description |
|---|---|
| `a397fc0` | Replace CONTENT_SPEC_GUIDE_SSG with SSG-native v1.0 spec |
| `a6639d4` | Replace 8 asbestos persona strings with SSG-native versions |
| `d5d685d` | Author SSG_OUTPUT_FORMAT.md v1.0 |
| `b12ccf0` | Author SSG_SITE_MAP.md v1.0 |
| `9667efa` | Author SSG_Writer_Reference.md v1.0 |
| `d16e812` | Author SSG_TEMPLATES.md v1.0 |
| `1c8c41a` | Adapt AUDIT_SPEC.md from asbestos to SSG rules |
| `100bff1` | Migrate run-batch.sh paths from asbestos to ssg |

All 8 commits in `~/projects/ssg-content/`. Clean per-change history. Easy revertibility.

## Commits from other chats (referenced for context)

- **`3ed57dc`** (in `~/agents/`) — Asbestos deploy throttle live: 20:00 PT preview ping + 23:00 PT batch deploy, `*/15` ship.sh disabled (commented, reversible)
- Verdict persistence build — auditor_verdicts SQLite table + audit_guide.py persistence (build report: `~/brain/projects/aiteam/docs/verdict_persistence_build_2026-05-17.md`)

---

## Pipeline state at session end

| Aspect | State |
|---|---|
| run-batch.sh ready to invoke | YES |
| `bash -n content/run-batch.sh` | CLEAN |
| `${assembled}` files all exist | YES |
| Writer/Auditor persona prompts | SSG-native |
| Output directories | Created and empty |
| Asbestos deploy throttle | LIVE (20:00 preview / 23:00 deploy) |
| SSG ship enabled | NO (`enabled: false` in ssg.yaml — needs data-driven renderer first) |
| Last blocker for first SSG batch run | Operator authors `content/ssg/assignment-batch-<batch>.md` + D-SSG-05 fix |

---

## Key decisions captured

| Decision | Rationale |
|---|---|
| Build per-site config from day one (not refactor later) | Multi-site agent stack from day one |
| Asbestos stays content-only for now (no programmatic SEO yet) | Compete invisibly while pages age into rankings |
| Ship-To-Site: no per-ship approval, auditor IS the gate | Periodic audit oversight, not per-item friction |
| Ship-To-Site: cron-driven polling every 15 min | Decoupled from run-batch.sh — superseded by deploy throttle |
| Asbestos deploy throttle: 20:00 preview + 23:00 batch, 3-hr review window | Human review before content goes live; temporary safety scaffolding |
| SSG deploy throttle deferred to D059 | Building throttle for pipeline that doesn't ship is solving imaginary problem |
| SSG site migration is data-driven, not JSX-emitter | Affiliate site model + future programmatic SEO both require data-driven |
| Author migration from scratch (not transcribed from asbestos) | Asbestos source files were stubs; no structural model existed |
| Comments in run-batch.sh left alone | Zero-impact; editing risks breaking line numbers we'll reference in future sessions |
| Service-mode and blog-mode persona strings replaced too | Consistency — dormant code that activates later shouldn't carry asbestos branding |
| relatedLinks slug shape: category-prefixed, no slashes | CC-authored; awaiting confirmation |

---

## DEFERRED items (final state)

### Blocking-ish (do before first SSG batch run)
- **D-SSG-05:** SSG_SITE_MAP.md uses markdown table format; run-batch.sh line 269 `grep -E '^\/'` won't extract URLs from `|` cells. VALID_LINKS file will be empty. Writer still gets full SITE_MAP via `${CACHED_AUDIT}`, so this is Writer-instruction completeness gap, not crash. 5-min fix.
- **D-SSG-02:** Operator-review SSG_TEMPLATES.md Cosmetic Tolerance List + Verdict Rules. CC authored from first principles. Verify before first audit run.
- **D-SSG-03:** Confirm relatedLinks slug shape. CC chose `"it-support/managed-it-services-pricing"` (category-prefixed, no slashes). 1-line edit if convention differs.

### Triggered by future events
- **D-SSG-04:** Reconcile AS1-AS5 anti-sameness checks between SSG_TEMPLATES.md and audit_guide.py. Trigger: audit_guide.py SSG migration session.
- **D-SSG-06:** AUDIT_SPEC.md SPC-1..14 (~160 lines) still has UST/asbestos vocabulary. Gated as inactive. Trigger: when SSG service mode activates.
- **D059:** SSG deploy throttle (pattern-copy of asbestos throttle at commit `3ed57dc`). Reference: `~/brain/projects/aiteam/docs/deploy_throttle_build_2026-05-17.md`. Trigger: `ssg.yaml` flips to `enabled: true` AND SSG ships its first batch. Est. 30-45 min CC session.
- **D058:** Vercel preview branches not wired for asbestos preview URLs (referenced in deploy throttle build, currently no preview URL in 20:00 ping). Trigger: operator decides to enable preview links.

### Cosmetic / convenience (no trigger urgency)
- **D-SSG-07:** Stale "2,000-2,500 words" echo on run-batch.sh line ~357. Cosmetic console output. No functional impact.
- **D-SSG-08:** ctx.sh has 22 asbestos refs. Session-save tool. Convenience cleanup.
- **D-SSG-09:** audit_guide.py has 5 asbestos refs in docstring/comments. Cosmetic.

### Non-pipeline (still on board)
- SSG site data-driven migration (per `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md`). Trigger: after Ship-To-Site ✅, before SSG content production resumes.
- 11 remaining asbestos comment refs in run-batch.sh. Convenience cleanup, no functional impact.

---

## Lessons learned this session

1. **Bash prompt rewrites must escape literal `$` inside `claude -p "..."` blocks.** Bare `$79` becomes empty string at runtime. Use `\$79`. Caught in persona string apply session.

2. **Asbestos "production reference" files were stubs.** Don't assume legacy files have substance to port. Always verify file size/content before planning a transcription approach. 4 critical pipeline content files were 116-293 byte placeholders. Forced first-principles authoring.

3. **Markdown table cells starting with `|` break naïve grep regexes that expect `^\/`.** Sanity-check any bash grep against the format of the file it parses.

4. **The "original count" of asbestos references was wrong (36 vs actual 44).** Quick counts during early scans aren't reliable. When recon flags "X references," budget for X+20%.

5. **CC's "stop and report" instinct prevented multiple migration disasters.** Twice this session CC paused before blind execution (missing source files, asbestos stubs as model). Both pauses surfaced material findings that changed the approach.

6. **Comments are cheap to leave alone.** 20 zero-impact COMMENT refs stayed untouched. Preserved line-number stability for future sessions.

7. **Parallel chats can produce timeline drift.** Both chats can have accurate snapshots at different moments. Pre-flight halt discipline catches this cleanly — CC's "verify on disk before executing" caught a hand-off that was correct-when-written but stale-at-execution. Pattern worth keeping: when a hand-off claims X exists, CC verifies before building on top of X.

8. **Building infrastructure for pipelines that don't ship yet is solving imaginary problems.** SSG throttle was correctly deferred. The blocker isn't lack of throttling — it's lack of pipeline output AND lack of data-driven SSG site. Throttle becomes valuable AFTER those land.

---

## Files created/modified this session

### In `~/projects/ssg-content/`
- `content/ssg/CONTENT_SPEC_GUIDE_SSG.md` (replaced) + `.bak`
- `content/run-batch.sh` (8 persona blocks + 20 paths) + `.bak`
- `content/ssg/SSG_OUTPUT_FORMAT.md` (authored)
- `content/SSG_SITE_MAP.md` (authored)
- `content/ssg/SSG_Writer_Reference.md` (authored)
- `content/ssg/SSG_TEMPLATES.md` (authored)
- `content/AUDIT_SPEC.md` (adapted) + `.bak`
- `content/ssg/keyword-configs/`, `content/ssg/approved/guides/`, `content/ssg/drafts/`, `content/ssg/feedback/` (created, empty)

### In `~/agents/` (other chat)
- Asbestos deploy throttle scripts at commit `3ed57dc`
- Crontab updated (continuous ship.sh disabled, 20:00/23:00 entries added)

### In `~/brain/projects/aiteam/docs/`
- `ssg_articles_structure_dump_2026-05-16.md` (CC recon)
- `ssg_content_spec_source_material_2026-05-16.md` (CC recon)
- `ssg_path_migration_inventory_2026-05-16.md` (CC recon)
- `ssg_content_spec_application_report_2026-05-16.md` (CC application report)
- `ssg_pipeline_content_files_build_2026-05-16.md` (CC Session A report)
- `ssg_pipeline_migration_complete_2026-05-16.md` (CC Sessions B+C report)
- `deploy_throttle_build_2026-05-17.md` (other chat)
- `verdict_persistence_build_2026-05-17.md` (another chat)
- `ssg_throttle_build_halt_2026-05-17.md` (CC halt report)

### In this chat's project (Claude.ai)
- `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md`
- `AITEAM_CONTENT_SPEC_GUIDE_SSG_v1.md`
- `AITEAM_SSG_PersonaString_Replacements_2026-05-16.md`
- `AITEAM_CCPrompt_SSG_PipelineContentFiles_2026-05-16.md`
- `AITEAM_CCPrompt_SSG_AuditSpec_PathMigration_2026-05-16.md`
- `AITEAM_Context_Save_2026-05-16_SSG_PipelineMigration.md` (mid-session save)
- `AITEAM_Context_Save_2026-05-16_SSG_PipelineMigration_Final.md` (this file)

---

## Resume points (where to pick up next session)

### If goal is "run the SSG pipeline end-to-end"
1. Fix D-SSG-05 (SITE_MAP grep regex, 5 min)
2. Operator reviews D-SSG-02 (Cosmetic Tolerance + Verdict Rules) and D-SSG-03 (slug shape)
3. Author one `content/ssg/assignment-batch-<batch>.md` for a single test slug
4. Run `bash content/run-batch.sh <batch> --guide` — first end-to-end SSG pipeline run
5. Validate output: drafts/ → approved/ → audit_log row
6. If clean: pipeline is production-ready for SSG (still won't deploy until ssg.yaml: enabled: true)

### If goal is "ship more agents in the fleet"
Other chats have build prompts ready for: Config Synthesizer, Assignment Drafter, Ship-To-Site supporting tasks. Stack-ranked recommendation in `ssg_pipeline_automation_recommendations_2026-05-16.md`.

### If goal is "enable SSG to actually deploy content"
1. Execute SSG site data-driven migration per `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md`
2. Flip `ssg.yaml: enabled: true`
3. Execute D059 (build SSG throttle as pattern-copy of asbestos)
4. SSG pipeline + SSG ship + SSG throttle all live

### If goal is "monitor the new asbestos deploy throttle"
First 20:00 PT preview ping → check Telegram for slug list + word count + rubric score
First 23:00 PT batch deploy → check live asbestos site for new slugs
3-hour review window between is when operator yanks bad slugs if any

---

## Sign-off note

Every step of this session followed locked sign-off discipline: ✅ only when pass conditions executed and produced expected result. No premature ✅. CC's reports honest about first-principles content and flagged 9 deferred items needing operator review or future cleanup.

The SSG pipeline is migrated. The asbestos deploy throttle is live. The SSG throttle is correctly deferred. The verdict persistence layer is in place.

Real test of all this: tomorrow's first 20:00 PT preview ping + 23:00 PT batch deploy on the asbestos side. And eventually, the first SSG batch run.

---

*End of context save. Resume next session by reading this file first, then the DEFERRED section. The DEFERRED list is the single source of truth for "things we said we'd do later."*
