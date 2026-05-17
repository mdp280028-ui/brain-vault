# SSG Path Migration Inventory — `run-batch.sh`

**Date:** 2026-05-16
**Mode:** Recon only. No edits, no directory creation, no commits.
**Pairs with:** prior work in `ssg_content_spec_application_report_2026-05-16.md` (persona strings already replaced; commits `a397fc0`, `a6639d4`)

---

## Headline finding

**`content/asbestos/` does not exist in `~/projects/ssg-content/`.** This is not a directory-rename migration — it is a "repoint path constants to a partially-built `content/ssg/` directory structure" migration. The pipeline cannot run today because `ASSIGNMENT="content/asbestos/assignment-batch-${BATCH}.md"` (line 161) points at a non-existent directory.

**Three more blockers operator must decide on before any path swap:**

1. **Four content files referenced by the pipeline have NO SSG version anywhere.** The asbestos versions exist in the sibling asbestos repo but are 100-300 byte stubs, so copy-as-starting-point yields nothing useful. These need authored content:
   - `content/ssg/SSG_TEMPLATES.md` (asbestos version: 117 bytes, stub)
   - `content/ssg/SSG_Writer_Reference.md` (asbestos version: 293 bytes, stub)
   - `content/ssg/SSG_OUTPUT_FORMAT.md` (asbestos version: 121 bytes, stub)
   - `content/SSG_SITE_MAP.md` (asbestos version: 116 bytes, stub)

2. **Downstream blocker: `scripts/audit_guide.py` has 5 asbestos references** (docstring + branded comments). run-batch.sh calls this script directly. Migrating run-batch.sh paths without updating audit_guide.py leaves asbestos branding in audit output but does not break runtime — the asbestos refs in audit_guide.py are all in comments/docstrings, no functional paths.

3. **Downstream blocker: `content/AUDIT_SPEC.md` has 3 asbestos references.** This file is part of the `${assembled}` spec passed into the Writer agent. Operator should review whether SSG should keep or rewrite the audit spec.

---

## Full categorized table

44 total `asbestos`/`asbestosHQ` matches. Categories:

- **COMMENT** — comment text only, zero functional impact
- **STRING** — string literal in error/notification, cosmetic
- **PATH** — file/directory path in a variable assignment; functional

### COMMENT category (20 lines)

| Line | Text excerpt | Sub-type | Treatment |
|---|---|---|---|
| 2 | `# Asbestos Content Production, Automated Writer/Auditor Loop v10.13` | Header | Rewrite header |
| 14 | `Pairs with audit_guide.py v3.4, CONTENT_SPEC_GUIDE_ASBESTOS v2.7` | Header pairing | Update version ref |
| 23 | `Guide writer + auditor prompts updated for CONTENT_SPEC_GUIDE_ASBESTOS v2.4` | Changelog | Leave (historical) |
| 28 | `/find-asbestos-contractors/, /request-a-quote/, service-page, or` | Changelog | Leave (historical) |
| 35 | `prompts untouched (not active for phase 1 of AsbestosHQ.com)` | Changelog | Leave (historical) |
| 44 | `to match CONTENT_SPEC_GUIDE_ASBESTOS v2.2+ and audit_guide.py v3.1+` | Changelog | Leave (historical) |
| 52 | `at content/asbestos/feedback/<page>-r<round>.md before the file moves back` | Path doc | Update path ref |
| 65 | `per approved guide: once with shared content/asbestos/audit-configs/guide.json` | Path doc | Update path ref |
| 67 | `with per-slug content/asbestos/keyword-configs/<slug>.json (SEO1-5 primary` | Path doc | Update path ref |
| 76 | `Guide output lands in content/asbestos/approved/guides/` | Path doc | Update path ref |
| 77 | `Uses content/asbestos/CONTENT_SPEC_GUIDE_ASBESTOS.md v2.0 and G1-G5 audit` | Path doc | Update path ref |
| 78 | `Shared audit config at content/asbestos/audit-configs/guide.json` | Path doc | Update path ref |
| 84 | `with keyword config from content/asbestos/keyword-configs/<slug>.json` | Path doc | Update path ref |
| 90 | `# Usage: cd ~/Desktop/asbestos-contractors && bash content/run-batch.sh [batch-number] [options]` | Usage | Update cwd to `~/projects/ssg-content` |
| 131 | `# Service mode is UST-legacy and not active for AsbestosHQ Phase 1.` | Body | Update brand ref |
| 133 | `# tables that mis-audit asbestos content. CONTENT_SPEC_SERVICE_ASBESTOS.md is` | Body | Update brand + spec ref |
| 143 | `# CONTENT_SPEC_ASBESTOS.md — currently a 2-line stub. That produces garbage` | Body | Update spec ref (and note: `CONTENT_SPEC_SSG.md` IS still a 2-line stub) |
| 716 | `# Guide schema uses metaTitle (not seoTitle). Strict 60/160 caps per CONTENT_SPEC_GUIDE_ASBESTOS v2.0` | Body | Update spec version ref |
| 753 | `# Paragraph divisibility auto-merge (CONTENT_SPEC_GUIDE_ASBESTOS v2.2):` | Body | Update spec version ref |
| 1026 | `# Service/blog modes: per-slug config at content/asbestos/keyword-configs/<slug>.json, hard fail if missing` | Path doc | Update path ref |

### STRING category (4 lines)

| Line | Text excerpt | Treatment |
|---|---|---|
| 137 | `echo "AsbestosHQ Phase 1 ships guides only. Use --guide instead." >&2` | Replace `"AsbestosHQ Phase 1"` → `"SmartSourceGuide Phase 1"` |
| 146 | `echo "ERROR: no mode flag provided. Must pass --guide (asbestos GUIDE mode)." >&2` | Remove `"asbestos "` |
| 147 | `echo "BLOG mode default is unsafe: CONTENT_SPEC_ASBESTOS.md is a stub and produces garbage output." >&2` | `CONTENT_SPEC_ASBESTOS.md` → `CONTENT_SPEC_SSG.md` |
| 1337 | `with title \"Asbestos Content\"` (osascript notification) | `"Asbestos Content"` → `"SSG Content"` |

### PATH category (20 lines)

All under the BLOCKING tier — every one of these must be repointed for the pipeline to run.

| Line | Variable | Original path | SSG equivalent | SSG exists? | Notes |
|---|---|---|---|---|---|
| 161 | `ASSIGNMENT` | `content/asbestos/assignment-batch-${BATCH}.md` | `content/ssg/assignment-batch-${BATCH}.md` | **NO** | Per-batch file; created when batch is assigned |
| 164 | `CONTENT_SPEC` (service) | `content/asbestos/CONTENT_SPEC_SERVICE_ASBESTOS.md` | `content/ssg/CONTENT_SPEC_SERVICE_SSG.md` | YES (stub, 2 lines) | Service mode dormant in Phase 1 |
| 165 | `TEMPLATES` (service) | `content/TEMPLATES_ASBESTOS.md` | `content/TEMPLATES_SSG.md` | **NO** | Service mode dormant; asbestos version is 117-byte stub |
| 168 | `CONTENT_SPEC` (guide) | `content/asbestos/CONTENT_SPEC_GUIDE_ASBESTOS.md` | `content/ssg/CONTENT_SPEC_GUIDE_SSG.md` | YES (just written, 18 KB) | Populated |
| 169 | `TEMPLATES` (guide) | `content/asbestos/ASBESTOS_TEMPLATES.md` | `content/ssg/SSG_TEMPLATES.md` | **NO** | **CRITICAL — actively used by guide mode.** Asbestos version is 117-byte stub. |
| 173 | `REFERENCE` | `content/asbestos/ASBESTOS_Writer_Reference.md` | `content/ssg/SSG_Writer_Reference.md` | **NO** | **CRITICAL — actively used.** Asbestos version is 293-byte stub. |
| 174 | `OUTPUT_FORMAT` | `content/asbestos/ASBESTOS_OUTPUT_FORMAT.md` | `content/ssg/SSG_OUTPUT_FORMAT.md` | **NO** | **CRITICAL — actively used.** Asbestos version is 121-byte stub. |
| 177 | `SITE_MAP` | `content/ASBESTOS_SITE_MAP.md` | `content/SSG_SITE_MAP.md` | **NO** | **CRITICAL — actively used.** Asbestos version is 116-byte stub. |
| 178 | `SKELETONS` | `content/asbestos/skeletons/service-page-skeletons.json` | `content/ssg/skeletons/service-page-skeletons.json` | **NO** | Service mode dormant |
| 179 | `DRAFTS` | `content/asbestos/drafts` | `content/ssg/drafts` | **NO** (dir) | Runtime-creatable via mkdir |
| 180 | `FEEDBACK` | `content/asbestos/feedback` | `content/ssg/feedback` | **NO** (dir) | Runtime-creatable via mkdir |
| 181 | `APPROVED` (default) | `content/asbestos/approved` | `content/ssg/approved` | YES (dir, with `.gitkeep` and `guides/`) | |
| 182 | `NEEDS_REVIEW` | `content/asbestos/needs-review` | `content/ssg/needs-review` | **NO** (dir) | Runtime-creatable via mkdir |
| 183 | `INDEX` | `content/asbestos/approved-index-asbestos.md` | `content/ssg/approved-index-ssg.md` | **NO** | Created on first approval (pipeline appends) |
| 184 | `LOG_DIR` | `content/asbestos/logs` | `content/ssg/logs` | **NO** (dir) | Runtime-creatable via mkdir |
| 186 | `CACHE_DIR` | `content/asbestos/.cache` | `content/ssg/.cache` | **NO** (dir) | Runtime-creatable via mkdir |
| 190 | `APPROVED` (guide override) | `content/asbestos/approved/guides` | `content/ssg/approved/guides` | YES (dir) | |
| 1032 | `local guide_config` | `content/asbestos/audit-configs/guide.json` | `content/ssg/audit-configs/guide.json` | YES (1755 bytes, populated) | |
| 1033 | `local slug_config` | `content/asbestos/keyword-configs/${page}.json` | `content/ssg/keyword-configs/${page}.json` | YES (dir, per-slug files created per assignment) | |
| 1134 | `local config_path` | `content/asbestos/keyword-configs/${page}.json` | `content/ssg/keyword-configs/${page}.json` | YES (dir) | |

---

## Paths that need real directory/file work

### File content authoring blockers (CANNOT be solved by mkdir)

These four files are actively read by guide-mode runs. The asbestos versions in the sibling asbestos repo are tiny stubs — copying them yields no usable content. Operator must decide whether each is:

(a) **author SSG content from scratch** (recommended for `SSG_Writer_Reference.md` and `SSG_OUTPUT_FORMAT.md` if they should reflect SSG voice/schema)
(b) **port from another source** (do the equivalents exist in the UST pipeline or in the `~/projects/smartsourceguide/` site repo?)
(c) **defer until first batch is queued** (write only when first actual SSG batch needs them)

| Required file | Asbestos source size | Decision needed |
|---|---|---|
| `content/ssg/SSG_TEMPLATES.md` | 117 bytes (stub) | Author from scratch |
| `content/ssg/SSG_Writer_Reference.md` | 293 bytes (stub) | Author from scratch |
| `content/ssg/SSG_OUTPUT_FORMAT.md` | 121 bytes (stub) | Author from scratch — should match the guide JSON schema |
| `content/SSG_SITE_MAP.md` | 116 bytes (stub) | Author when site shape is decided (per CLAUDE.md, "Site build + deploy pipeline: NOT YET EXISTS") |

### File-creation-on-first-batch (resolved by assigning a batch)

| Required file | Lifecycle |
|---|---|
| `content/ssg/assignment-batch-${BATCH}.md` | Created per-batch; operator writes when defining each batch |
| `content/ssg/approved-index-ssg.md` | Created/appended by pipeline on first approval |
| Per-slug `content/ssg/keyword-configs/<slug>.json` | Created per-slug as part of batch prep |

### Dormant (Phase 1 service mode disabled)

`SKELETONS`, `TEMPLATES_SSG.md` (service-mode at content root), and `CONTENT_SPEC_SERVICE_SSG.md` are wired but service mode is hard-disabled (`run-batch.sh` lines 131-138 reject `--service`). Path updates can be applied for consistency without creating the underlying files.

### Directory-mkdir-only (no content needed)

Resolvable with a one-liner: `mkdir -p content/ssg/{drafts,feedback,needs-review,logs,.cache}`. The pipeline likely creates these itself on first run, but pre-creating avoids any first-run surprise.

---

## Pure text references (file edit only, no directory work)

### Comments — 20 lines

Edit-only. Zero runtime impact. Lowest priority.

Suggested approach: a single sed/regex pass to swap `asbestos` → `ssg`, `AsbestosHQ.com` → `SmartSourceGuide`, `CONTENT_SPEC_GUIDE_ASBESTOS` → `CONTENT_SPEC_GUIDE_SSG` would resolve most. Manual review needed for:
- Line 23-44: changelog entries (historical context; leave or annotate)
- Line 90: usage cwd reference (mechanical update)
- Line 143: stub-warning text accidentally still applies — `CONTENT_SPEC_SSG.md` IS a 2-line stub in ssg-content. Operator may want to update the spec OR update the warning to say `CONTENT_SPEC_GUIDE_SSG.md` (which is populated) instead.

### Strings — 4 lines

Edit-only. UX-cosmetic. Lines 137, 146, 147 are error messages users see in stderr; line 1337 is a macOS notification title.

---

## Recommended migration order

If operator authorizes a full migration in one pass:

1. **Prerequisite gate** — operator decides on the four CRITICAL missing content files (authored from scratch vs deferred). If deferred, pipeline still won't run; document this gap explicitly so downstream isn't surprised.
2. **Path constants (Tier A)** — repoint all 20 PATH-category lines to `content/ssg/...` equivalents. Single commit. Pipeline becomes runnable once required files exist.
3. **Pre-create runtime dirs** — `mkdir -p content/ssg/{drafts,feedback,needs-review,logs,.cache}`. Optional belt-and-suspenders.
4. **String messages (Tier B)** — 4 line edits. Single commit. Cosmetic.
5. **Comments (Tier C)** — 20 lines. Single commit. Cosmetic. Defer if low priority.
6. **Downstream `scripts/audit_guide.py`** — separate work item. 5 asbestos refs in docstring/comments. Same `simplify` treatment as run-batch.sh.
7. **Downstream `content/AUDIT_SPEC.md`** — separate work item. 3 asbestos refs. Operator should decide if SSG keeps the asbestos audit spec, modifies it, or rewrites it.

### Alternative phasing

If operator wants minimum-viable-pipeline ASAP:

1. **Path constants only** (skip comments/strings)
2. **Author the 4 missing content files as minimum-viable stubs** (enough to not crash; iterate on quality)
3. **Run one test page** to confirm pipeline works end-to-end
4. **Cosmetic cleanup** in a later pass

---

## Out-of-scope but worth flagging

### `scripts/ctx.sh` has 22 asbestos refs

The session-save tool is heavily asbestos-coupled. Not blocking SSG pipeline runs, but every `ctx` save in ssg-content currently writes "asbestos" branded session notes. Separate cleanup.

### `scripts/auto-harvest-to-library.sh` has 13 asbestos refs

Dev utility. Same flag — separate cleanup, not pipeline-blocking.

### `~/projects/smartsourceguide/` is a Next.js scaffold, not data

The persona prompts reference `~/projects/smartsourceguide/data/guides/<slug>.json` as the eventual destination. That path does NOT exist (the smartsourceguide repo has `app/`, `components/`, etc. but no `data/` dir). The phrasing in the persona prompts is aspirational — actual pipeline writes go to `content/ssg/approved/guides/` (inside this repo). Operator may want to either (a) build out the site repo to receive the data, or (b) update the persona prompt reference.

### Asbestos repo at `~/projects/asbestos-contractors/` is still production

Per the migration prompt's caveat — the asbestos pipeline at that path remains the live production system. The ssg-content repo SHOULD NOT read any files from `~/projects/asbestos-contractors/...` after migration. Confirmed: zero references to `~/projects/asbestos-contractors/` in `content/run-batch.sh` today. The persona work and this migration both keep ssg-content self-contained — good.

### Repo CLAUDE.md mentions paths that don't match reality

CLAUDE.md says ssg-content has "11 guide slugs in April 2026 batch" and "8 approved, 3 in needs-review, 1 stale draft" — but `content/ssg/approved/guides/` is empty in this repo and there's no `needs-review/` directory. Those numbers describe the asbestos pipeline state and were copied into the SSG CLAUDE.md without update. Worth flagging for separate operator review.

---

## Counts summary

- **Total matches:** 44 (vs the 36 reported in the prior application report; difference likely due to grep flag differences — this report uses `grep -n -i`)
- **PATH-category lines that BLOCK pipeline runs:** 20
- **PATHs whose SSG target exists:** 6 of 20
- **PATHs whose SSG target is missing:** 14 of 20
- **PATHs whose SSG target is missing AND is required content (not just a runtime dir):** 5
- **PATHs whose SSG target is missing AND is a critical guide-mode file:** 4
- **STRING-category lines (cosmetic):** 4
- **COMMENT-category lines (zero impact):** 20
- **Downstream files with asbestos refs that pipeline depends on:** 2 (`scripts/audit_guide.py`, `content/AUDIT_SPEC.md`)
- **Downstream files with asbestos refs that pipeline does NOT depend on:** 2 (`scripts/ctx.sh`, `scripts/auto-harvest-to-library.sh`)

---

*End of inventory. No edits made. Awaiting operator decision on:*
1. *Author the 4 missing content files from scratch, or defer?*
2. *Scope for next migration pass — paths only, paths + strings, or paths + strings + comments?*
3. *Treatment of downstream `audit_guide.py` and `AUDIT_SPEC.md` — bundle into same migration or separate?*
