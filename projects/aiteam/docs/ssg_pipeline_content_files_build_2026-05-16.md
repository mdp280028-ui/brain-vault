# SSG Pipeline Content Files Build — Report

**Date:** 2026-05-16
**Operator:** ruralpropertyguide@gmail.com
**Built by:** Claude Code (Opus 4.7)
**Target repo:** `~/projects/ssg-content`
**Pairs with:** `ssg_content_spec_application_report_2026-05-16.md` (commits `a397fc0`, `a6639d4`); `ssg_path_migration_inventory_2026-05-16.md`

---

## Sign-off conditions (all 9 met)

| # | Condition | Status |
|---|---|---|
| 1 | All 4 files exist at pipeline-expected paths | ✅ |
| 2 | All 4 files have real content (not stubs, not TODOs) | ✅ |
| 3 | Backups of any pre-existing stubs preserved with `.bak` | ✅ (vacuously: no pre-existing stubs found at the SSG paths) |
| 4 | Each file's H1 matches `# <FILENAME>.md: <subtitle>` pattern | ✅ |
| 5 | Each file references `CONTENT_SPEC_GUIDE_SSG.md` as authority where applicable | ✅ (3 / 2 / 3 / 1 refs across the four files) |
| 6 | Zero asbestos references in any of the four new files | ✅ (0 / 0 / 0 / 0) |
| 7 | Zero em/en dashes in any of the four new files | ✅ (after 6-em-dash post-write cleanup in SSG_Writer_Reference.md, see "Deviations") |
| 8 | 4 commits made, clean revertibility | ✅ (SHAs below) |
| 9 | Build report at the specified path | ✅ (this file) |

**Overall verdict:** ✅ PASS

---

## Per-file status

| File | Status | Path | Lines | Bytes | Authority refs |
|---|---|---|---|---|---|
| SSG_OUTPUT_FORMAT.md | ✅ written | `content/ssg/SSG_OUTPUT_FORMAT.md` | 266 | 12,822 | 3 |
| SSG_SITE_MAP.md | ✅ written | `content/SSG_SITE_MAP.md` (root, NOT `ssg/`) | 135 | 7,326 | 1 |
| SSG_Writer_Reference.md | ✅ written | `content/ssg/SSG_Writer_Reference.md` | 177 | ~9,100 (post-fix) | 2 |
| SSG_TEMPLATES.md | ✅ written | `content/ssg/SSG_TEMPLATES.md` | 395 | 25,615 | 3 |
| **Total** | | | **973 lines** | **~55 KB** | |

### Path verification

Per the user's instruction, I matched the SSG-stub naming convention (already established in the repo for `CONTENT_SPEC_GUIDE_SSG.md`). All four paths reside under `content/ssg/` or `content/` to mirror the asbestos pipeline's path shape. NOTE: `run-batch.sh` still references the asbestos-named paths (e.g., `content/asbestos/ASBESTOS_TEMPLATES.md`) — repointing those constants is the separate next-session task per `ssg_path_migration_inventory_2026-05-16.md`. Once the path migration commits, the pipeline will read from these four new files.

---

## Commits

| # | SHA | Message |
|---|---|---|
| 1 | `d5d685d` | Author SSG_OUTPUT_FORMAT.md v1.0 |
| 2 | `b12ccf0` | Author SSG_SITE_MAP.md v1.0 |
| 3 | `9667efa` | Author SSG_Writer_Reference.md v1.0 |
| 4 | `d16e812` | Author SSG_TEMPLATES.md v1.0 |

Each file in its own commit per the prompt's revertibility requirement. Co-Authored-By trailer included per harness default.

---

## Source materials used (and not used)

### Used as authority

- `CONTENT_SPEC_GUIDE_SSG.md` v1.0 in `content/ssg/` (the spec I authored earlier; loaded into context for voice, banned phrases, audit philosophy, FAQ rules, link rules, FAQ + CTA + closing-paragraph patterns)
- GUIDE WRITER persona prompt in `content/run-batch.sh` lines 475-577 (the 15-field REQUIRED SCHEMA block, the full BANNED PHRASES list, the G1-G5 thresholds, the C1-C4 constraints, the V-series numeric caps, the closing-paragraph templates — these are what locked-in answer for Q1 specified as authoritative)
- GUIDE AUDITOR persona prompt in `content/run-batch.sh` lines 881-969 (vendor-neutrality check, vendor-marketing-language check, INLINE FIX list)
- SERVICE WRITER persona prompts in `content/run-batch.sh` lines 410-470 (service-page schema fields, content-paragraph rules, FAQ rules — used for SSG_TEMPLATES.md service-page section)
- BLOG WRITER persona prompt in `content/run-batch.sh` lines 569-592 (blog schema, link rules — used for SSG_TEMPLATES.md blog section)
- `~/projects/smartsourceguide/app/it-support/managed-it-services-pricing/page.tsx` (one full production article read to confirm rendered HTML structure and to make the SSG_OUTPUT_FORMAT.md worked example feel realistic)
- `~/brain/projects/aiteam/docs/ssg_articles_structure_dump_2026-05-16.md` (the article inventory — 14 articles confirmed, paths, titles, topical groupings, shim notes)

### Asbestos structural-model files attempted, found to be stubs

- `~/projects/asbestos-contractors/content/asbestos/ASBESTOS_TEMPLATES.md` → 117 bytes, literal "STUB" text
- `~/projects/asbestos-contractors/content/asbestos/ASBESTOS_OUTPUT_FORMAT.md` → 121 bytes, literal "STUB" text
- `~/projects/asbestos-contractors/content/ASBESTOS_SITE_MAP.md` → 116 bytes, literal "STUB" text
- `~/projects/asbestos-contractors/content/asbestos/ASBESTOS_Writer_Reference.md` → 293 bytes, marked "Deprecated 2026-04-24", redirects to `cc-context/05-REFERENCE/ported-from-ust-adapted/ASBESTOS_Guide_Creation_Workflow.md` and confirms runtime authority lives in `CONTENT_SPEC_GUIDE_ASBESTOS.md`

**Implication:** No structural model existed to "blind-copy with vocabulary swap" as the build prompt anticipated. I authored each file's structure from first principles, anchored to (a) what the live SSG persona prompts and content spec require, and (b) what each file's stated purpose in the build prompt asks for.

### Source NOT used (and why)

- `~/brain/projects/aiteam/docs/AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md` — file does not exist. Per Q1 lock-in, the GUIDE WRITER persona prompt was substituted as the schema authority. The persona prompt's REQUIRED SCHEMA section is the de-facto source of truth for the 15 JSON fields, since it is the literal text the Writer agent receives every batch.
- `~/brain/projects/aiteam/docs/ssg_content_spec_source_material_2026-05-16.md` (93 KB) — not consulted. The spec it sourced is already fully captured in `CONTENT_SPEC_GUIDE_SSG.md` and the persona prompts. Adding it would risk including pre-finalization variants.
- `scripts/audit_guide.py` — not consulted for implementation details. SSG_TEMPLATES.md documents WHICH checks apply per template, not HOW each check is implemented. The check IDs (V8, S5, G1, etc.) are cited from the persona prompts; the implementation lives in the Python file (5 asbestos refs flagged for the separate audit_guide.py migration session).

---

## Per-file design notes

### SSG_OUTPUT_FORMAT.md

- Lays out all 15 JSON schema fields with type, length cap, format constraint, and a per-field example value
- JSON validation rules JSON1 through JSON8 captured as a table with explicit failure modes
- String length constraints summary table covers metaTitle, metaDesc, h1, h2s, paragraph shape, etc.
- Worked example: synthetic `example-comparison-guide` JSON object with 4 h2s, 10 paragraphs (10 ÷ 5 = 2; passes JSON7 divisibility). Body content kept minimal because the example is a shape demo, not an audit-passing draft.
- Cross-references SSG_TEMPLATES.md and SSG_SITE_MAP.md at the footer.

### SSG_SITE_MAP.md

- Per Q3 lock-in: grouped by category (fleet-tracking, it-support, answering-services), 14 canonical URLs only, 2 shims explicitly forbidden
- Each entry shows BOTH the inline-link URL (with leading and trailing slashes) AND the `relatedLinks` slug (no slashes), with a usage example for each
- "Forbidden link targets" section lists the 2 shim URLs, the 3 category-root URLs that 404, the `/about/` page, and aspirational directory paths
- Includes maintenance protocol (how to update when new guides ship; what to do when a guide is retired)
- Counts summary at the end for quick scan

### SSG_Writer_Reference.md

- 90-second read target met: 177 lines, scannable structure with tables
- TL;DR voice rules condensed from CONTENT_SPEC §3 (8 bullets)
- Skeleton at a glance distilled from §4 (6 sections)
- Top 20 banned phrases (not full list; full list lives in CONTENT_SPEC §7 and SSG_TEMPLATES.md)
- Audit checks summary tables: G1-G5, C1-C4, V-series numeric caps, paragraph[0] hard-shape
- 10-item self-check list (condensed from the 15-item CONTENT_SPEC §12)
- House style quick hits at the end

### SSG_TEMPLATES.md

- Three template sections (`guide`, `service-page`, `blog`) with explicit JSON schema fields, structural thresholds, applied vs skipped checks
- `guide` template is the densest section (Phase 1 active type): all G1-G5, C1-C4, V-series, S-series (paragraph[0]), L-series (links), SEO-series, vendor-neutrality, vendor-marketing-language, closing-paragraph patterns, full banned-phrases list, INLINE FIX list
- `service-page` template lists the SKIPPED checks (S4-S6, S10, S12, V16, D1, D2, SEO3, L1-L5, AS2) per the SERVICE AUDITOR persona prompt; applied JSON1-JSON8 validation
- `blog` template captures the BLOG WRITER prompt requirements (1,400-1,800 words, 2+ one-sentence paragraphs, vendor link rule, etc.)
- Cosmetic Tolerance List authored from first principles (11 issues auto-normalized: whitespace, smart quotes, trailing periods, etc.)
- Verdict Rules with explicit decision tree (PASS / PASS WITH NOTES / PASS WITH FIXES / FAIL) and the ordered check sequence
- Anti-sameness AS1-AS5 checks documented (AS2 skipped for service-page per pattern)

---

## Deviations and unexpected issues

1. **Source authority doc missing.** `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md` did not exist. Operator was notified, locked in "use GUIDE WRITER persona prompt as authority" (Q1). All schema references in SSG_OUTPUT_FORMAT.md trace back to the persona prompt text in `run-batch.sh` lines 481-496, which is the literal contract the Writer agent receives at runtime. No fabrication.

2. **Asbestos structural-model files are stubs.** All four asbestos production files (TEMPLATES, OUTPUT_FORMAT, SITE_MAP, Writer_Reference) are 116-293 byte placeholders with "STUB" or "Deprecated" text. The build prompt's "use asbestos versions as structural model" rule had no real model to draw from. Operator was notified, locked in "write rich content per your build prompt" (Q2). The four SSG files were authored from the persona prompts + CONTENT_SPEC + smartsourceguide page.tsx structure rather than transcribed from asbestos source. The Cosmetic Tolerance List and Verdict Rules in SSG_TEMPLATES.md are first-principles authored.

3. **Six em dashes leaked into SSG_Writer_Reference.md on first write.** The "Article skeleton" list section used em dashes as the separator between bullet labels and descriptions. Verification grep caught them; fixed with a single `replace_all` edit (`** — ` → `**: `). All 6 occurrences resolved, format remained scannable, no content lost. Re-verification confirmed 0 em dashes across all 4 files before commit. Lesson: even with explicit "no em dashes" in the build prompt, list-separator em dashes are an easy reflex; verify before commit.

4. **`relatedLinks` slug shape decision.** The persona prompt specifies `relatedLinks` is `array of {title, slug}` but does not specify whether slug includes the category prefix or just the article name. I documented in SSG_OUTPUT_FORMAT.md that slug = path-segment-only without leading or trailing slash (e.g., `"it-support/managed-it-services-pricing"`), based on the nested URL structure observed in smartsourceguide/app/. This is a defensible inference but is technically an authored decision, not lifted from existing source. Flag for operator validation before first batch run; if convention differs, this is a 1-line edit in OUTPUT_FORMAT plus a SITE_MAP table column update.

5. **`reviewedBy` semantics.** Persona prompt says `reviewedBy: null` for initial Writer output. No specification of what populates this on subsequent review. I documented "placeholder for future human-reviewer attribution; the Writer agent never populates this." If a human-review workflow gets defined later, that workflow updates `reviewedBy` and `lastReviewed`. Same for `lastReviewed` after revision rounds: I documented "unchanged unless a human reviewer re-checked." These are operationally-sensible defaults but worth confirming when human review enters scope.

6. **Cosmetic Tolerance List and Verdict Rules invented.** No asbestos source had these; the build prompt asked me to "copy from asbestos version." I authored both sections from first principles based on what is typical for content-audit pipelines plus the persona prompts' "INLINE FIX" lists. Specifically:
   - Cosmetic Tolerance List (11 items): trailing whitespace, double spaces, smart quotes, trailing periods, empty paragraphs, `<br>` tags, etc.
   - Verdict Rules: 4-verdict structure (PASS / PASS WITH NOTES / PASS WITH FIXES / FAIL) with an explicit decision-tree pseudocode block
   
   Both are reasonable defaults. Operator should review the Verdict Rules decision tree before first audit run; if the production behavior should differ (e.g., different threshold for treating C-check failures as systematic), it's a small edit in SSG_TEMPLATES.md.

7. **Anti-sameness AS1-AS5 authored from `as_instruction` bash variable hints.** The persona prompts reference "anti-sameness checks (AS1-AS5)" without enumerating them. I authored the 5 checks based on the run-batch.sh `as_instruction` block context and what makes sense for comparison-driven B2B content (opener similarity, H2 reuse, vendor mention bias, closing pattern rotation, FAQ topic overlap). If `audit_guide.py` implements AS1-AS5 differently, the documentation in SSG_TEMPLATES.md needs to reconcile. Flag for the next audit_guide.py migration session.

8. **The "Run mechanical gate" line in SSG_TEMPLATES.md Verdict decision tree** assumes `audit_guide.py` IS the mechanical gate. Confirmed by reading the script header (Asbestos Guide Page Auditor v3.4 per the docstring). After audit_guide.py migration completes, the version string referenced in SSG_TEMPLATES.md ("audit_guide.py v3.4") may need to bump.

---

## DEFERRED items (out of scope for this session per the user's "What NOT to do" rules)

- `run-batch.sh` path migration: 20 PATH-category lines still reference `content/asbestos/...` — separate session.
- `scripts/audit_guide.py`: 5 asbestos refs in docstring/comments — separate session.
- `content/AUDIT_SPEC.md`: 3 asbestos refs — separate session.
- `scripts/ctx.sh`: 22 asbestos refs — separate cleanup, not pipeline-blocking.
- `~/projects/smartsourceguide/data/guides/`: aspirational path referenced in CONTENT_SPEC_GUIDE_SSG.md §16; doesn't exist. Will become real when the SSG site repo's data-driven rewrite ships. No action this session.
- Per-batch `content/ssg/assignment-batch-${BATCH}.md` files: created per actual batch assignment; nothing to do until first batch is queued.
- Per-slug `content/ssg/keyword-configs/<slug>.json` files: created per assignment; same as above.

---

## What's now unblocked

With these 4 files in place, the path migration session (the separate next session) is fully unblocked. After path migration:

1. `bash content/run-batch.sh batch1 --guide` with a real `assignment-batch-batch1.md` will succeed end-to-end (assuming audit_guide.py runs and AUDIT_SPEC is adapted).
2. The Writer agent has a real schema reference, a real site map of link targets, and a real cheat sheet.
3. The Auditor agent has a real template-overrides file with verdict rules and check enumeration.

The two remaining downstream pieces (audit_guide.py asbestos refs; AUDIT_SPEC.md asbestos refs) are cosmetic in terms of pipeline runtime — neither blocks execution, both should be migrated for brand consistency before going to scale.

---

*Build complete. Four commits land at d5d685d, b12ccf0, 9667efa, d16e812. Local-only; revertible with `git revert d16e812 9667efa b12ccf0 d5d685d` if needed.*
