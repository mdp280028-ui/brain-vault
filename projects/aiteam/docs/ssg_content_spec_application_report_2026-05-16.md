# SSG Content Spec + Persona Strings — Application Report

**Date:** 2026-05-16
**Operator:** ruralpropertyguide@gmail.com
**Applied by:** Claude Code (Opus 4.7)
**Source docs:** `AITEAM_CONTENT_SPEC_GUIDE_SSG_v1.md`, `AITEAM_SSG_PersonaString_Replacements_2026-05-16.md` (both written to `~/brain/projects/aiteam/docs/` by operator-paste in this session, then applied)
**Target repo:** `~/projects/ssg-content`

---

## Part 1 — CONTENT_SPEC_GUIDE_SSG.md replacement

**Status:** SUCCESS

- Backup created: `content/ssg/CONTENT_SPEC_GUIDE_SSG.md.bak` (25,979 bytes, original asbestos spec preserved)
- New spec copied from `AITEAM_CONTENT_SPEC_GUIDE_SSG_v1.md` (full body, H1 to end-of-file marker)
- Old spec completely replaced

### Verification

**Head check (new H1 present):**
```
# CONTENT_SPEC_GUIDE_SSG.md: Guide Page Writer Spec

**Version:** 1.0 (initial SSG-native rewrite)
**Audience:** Writer agent producing guide content for SmartSourceGuide
**Last updated:** 2026-05-16
```

**Asbestos grep:** `grep -i "asbestos" content/ssg/CONTENT_SPEC_GUIDE_SSG.md` → **zero matches**

---

## Part 2 — run-batch.sh persona string replacements

**Status:** SUCCESS (with 1 documented no-op + 1 documented scope deferral)

Backup created: `content/run-batch.sh.bak` (64,754 bytes, original asbestos persona preserved)

### Per-block application log

| # | Line range (orig) | Block name | Action | Notes |
|---|---|---|---|---|
| 1 | 410-441 | SERVICE WRITER SKELETON | Replaced | SSG B2B-services skeleton prompt. Dormant in Phase 1 (`$SERVICE = false`). |
| 2 | 444-470 | SERVICE WRITER FULL | Replaced | SSG full-generation service prompt. Dormant in Phase 1. |
| 3 | 474-566 | **GUIDE WRITER (active)** | Replaced | Main writer prompt. Bare `$79`/`$149` price examples escaped to `\$79`/`\$149` to prevent bash variable expansion. |
| 4 | 569-592 | BLOG WRITER | Replaced | SSG blog-mode prompt. Active if blog branch fires. |
| 5 | 595-602 | REVISE (round 2+) | **NO-OP (skipped)** | Source doc's SSG version is byte-identical to existing brand-neutral prompt. Per source doc: "Keep as-is or apply the trivial formatting clean-up above. Operator's call." Chose to leave untouched. |
| 6 | 847-878 | SERVICE AUDITOR | Replaced | SSG service-page auditor. Dormant in Phase 1. |
| 7 | 881-969 | **GUIDE AUDITOR (active)** | Replaced | Main auditor prompt. Bare `$79`/`$149` escaped to `\$79`/`\$149`. |
| 8 | 972-1000 | BLOG AUDITOR | Replaced | SSG blog auditor. Active if blog branch fires. |

### Verification

**Bash syntax check:** `bash -n content/run-batch.sh` → **clean, exit 0**

**Asbestos grep on persona strings:** all 8 prompt blocks contain zero asbestos references (confirmed by reading the post-edit file state).

**Asbestos grep on entire file:** `grep -in "asbestos\|asbestosHQ" content/run-batch.sh` → **36 matches remaining**, ALL outside the persona blocks. Categorized below.

### Out-of-scope asbestos references (flagged, NOT modified)

The apply prompt's rule was: *"should return zero matches (or only intentional comments/path references — flag any you find)."* The 36 remaining matches break down as:

| Location | Count | Type | Recommended treatment |
|---|---|---|---|
| Lines 2-90 | 14 | File header / version history / usage comments | Rewrite when SSG header is authored. Out of scope for "persona string" replacement. |
| Lines 131-147 | 6 | SERVICE-mode disable logic + error messages mentioning "AsbestosHQ Phase 1" | Rewrite when service-mode is re-enabled or removed for SSG. Currently no functional impact (service mode is blocked). |
| Lines 161-190 | 11 | Path constants: `content/asbestos/...`, `ASBESTOS_TEMPLATES.md`, `ASSIGNMENT="content/asbestos/..."`, etc. | **Functional dependency.** These point to real directories that exist. Migrating these requires moving/renaming directory structure and is a separate workstream — flagging for follow-up. |
| Lines 716, 753 | 2 | Inline comments referencing `CONTENT_SPEC_GUIDE_ASBESTOS v2.0` / `v2.2` | Stale comment refs; update opportunistically. |
| Lines 1026-1134 | 3 | Audit-config path comments + assignments | Same path-dependency issue as 161-190. |
| Line 1337 | 1 | osascript notification title `"Asbestos Content"` | Cosmetic; update when notification UI is touched. |

**Recommendation:** A follow-up pass should address the path constants (lines 161-190 + 1026-1134) — these are the only functionally meaningful asbestos references left. The header/comment ones are cosmetic.

### Visual diff sanity check

`git diff content/run-batch.sh` showed clean prompt-only changes with bash structure (`if/elif/else`, variable expansions like `${page}`, model flags, hereforce arguments) all preserved across all 7 edited blocks. Line count delta: 1329 → 1343 (+14 lines, reasonable for prompts that grew with SSG-specific content).

---

## Part 3 — Commits

Two separate commits, clean revertibility:

| # | SHA | Message |
|---|---|---|
| 1 | `a397fc0` | Replace CONTENT_SPEC_GUIDE_SSG with SSG-native v1.0 spec |
| 2 | `a6639d4` | Replace 8 asbestos persona strings with SSG-native versions |

Co-Authored-By trailer included per harness default. `.bak` files NOT deleted (operator removes after validation).

---

## Deviations and unexpected issues

1. **Source files missing on first attempt.** The two `~/brain/projects/aiteam/docs/` source files (`AITEAM_CONTENT_SPEC_GUIDE_SSG_v1.md` and `AITEAM_SSG_PersonaString_Replacements_2026-05-16.md`) did not exist when the apply prompt was first issued. Per the apply prompt's own guard rule, CC stopped and asked. Operator then pasted both file bodies in chat; CC wrote them and re-ran the apply. **No fabrication occurred.**

2. **Block 5 (REVISE) no-op.** The source doc's SSG version of Block 5 is byte-identical to the existing brand-neutral REVISE prompt. Source doc explicitly said operator's call; CC left it untouched. No risk — content is already correct.

3. **Bash dollar-sign escaping in Blocks 3 and 7.** The SSG content for these blocks contains bare price examples (`$79`, `$149`) that bash would interpret as variable expansions (`$7` → empty positional, then `9`). CC escaped these to `\$79` and `\$149` so they pass through to Claude verbatim. This is the only deviation from a strictly mechanical paste; the underlying intent is preserved and the output the Writer/Auditor agents see will be the intended `$79-$149/user/month`. The original asbestos prompts had no bare dollar amounts so this issue didn't arise before.

4. **Em dashes in replacement prompts.** The pasted SSG prompts contain em dashes (e.g., `"V4d 'not just X, it's Y' — INFO-ONLY"`). The project CLAUDE.md bans em dashes in published content, but the original asbestos prompts in run-batch.sh also contained em dashes (e.g., original line 511) — so this is precedent, not a new violation. Em dashes inside prompts to the Writer agent do not appear in published guide output. No action taken; flagging for awareness.

5. **Out-of-scope asbestos references left in run-batch.sh.** 36 matches remain (categorized above). These are header comments, error strings, and `content/asbestos/...` path constants pointing to real, existing directories. Migrating the path constants requires moving directories and updating multiple touch-points — a separate workstream that the apply prompt explicitly did not authorize. **Flagging this as the most important follow-up.**

6. **Unrelated dirty files in repo.** `cc-context/00-CURRENT-STATE/*`, `cc-context/06-HISTORY/SESSION_HANDOFF.md`, and a `cc-context/06-HISTORY/snapshots/2026-05-16_1943.md` snapshot were modified before this session started (presumably by `ctx.sh`). CC did NOT touch or commit these — only the two target files were staged and committed.

7. **No pipeline execution.** Per the apply prompt's "Don't run the pipeline. Don't audit anything." rule, CC did not invoke `run-batch.sh` against any page. Runtime correctness of the new prompts is unverified until operator runs a test page.

---

## Next steps for operator

1. **Validate by running a single page** through the new pipeline (`bash content/run-batch.sh <batch> --guide` against a test slug) to confirm Writer and Auditor agents receive coherent SSG instructions.
2. **Decide on the path-constant migration** (`content/asbestos/...` → `content/ssg/...` or similar). This is the highest-impact remaining asbestos coupling.
3. **Remove `.bak` files** once validation passes: `rm content/ssg/CONTENT_SPEC_GUIDE_SSG.md.bak content/run-batch.sh.bak`.
4. **Optional polish:** rewrite the file-header comments (lines 2-90) and the notification title (line 1337) to drop the asbestos branding.

---

*Report end. Both commits are local-only; operator can revert with `git revert a6639d4 a397fc0` if anything goes sideways.*
