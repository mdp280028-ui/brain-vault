# SSG Pipeline Recon — 2026-05-16

Read-only factual map of the asbestos/SSG content pipeline and target sites. Recommendations live in the sibling `ssg_pipeline_automation_recommendations_2026-05-16.md`.

Repos inspected (all under `~/projects/`):

| Path | Role |
|---|---|
| `asbestos-contractors/` | Live source-of-truth content pipeline (git: `mdp280028-ui/asbestos-contractors`) |
| `ssg-content/` | Local-only fork of #1 for SSG (no remote on `origin`) |
| `asbestoshq-site/` | Live AsbestosHQ Next.js site (git: `mdp280028-ui/asbestoshq-site`) |
| `smartsourceguide/` | SSG Next.js site (git: `mdp280028-ui/smartsourceguide`) |
| `_asbestos-reference/` | Frozen reference snapshot (101 artifacts) |

Key version pins as of inspection: `run-batch.sh` v10.13, `audit_guide.py` v3.4, `AUDIT_SPEC.md` v1.6.4, `CONTENT_SPEC_GUIDE_ASBESTOS.md` v2.7. (CLAUDE.md in `asbestos-contractors` still cites v10.11 / v2.6 — drift.)

---

## 1. Pipeline flow diagram — `content/run-batch.sh` end-to-end

The script is 1,329 lines. Entry contract: `bash content/run-batch.sh <slug-or-batch-num> --guide [--force] [--sonnet] [--sonnet-audit] [--max-pages N]`. `--service` mode is hard-blocked in v10.13 (lines 135-140) because the asbestos service spec is a stub; `--guide` is the only viable mode.

Pseudocode:

```
1. Parse args; enforce GUIDE mode; exit on --service.
2. Set CONTENT_SPEC = content/asbestos/CONTENT_SPEC_GUIDE_ASBESTOS.md
   Set TEMPLATES   = content/asbestos/ASBESTOS_TEMPLATES.md     (currently a STUB)
   Set REFERENCE   = content/asbestos/ASBESTOS_Writer_Reference.md
   Set AUDIT_SPEC  = content/AUDIT_SPEC.md
   Set DRAFTS/FEEDBACK/APPROVED/NEEDS_REVIEW/LOG_DIR/CACHE_DIR paths
   For --guide: APPROVED becomes content/asbestos/approved/guides/
   mkdir -p those dirs; tee log to LOG_DIR.

3. Pre-flight: ensure `claude` CLI is on PATH; ensure the assignment file exists
   at content/asbestos/assignment-batch-<batch>.md; ensure all spec/template
   files exist.

4. Parse assignment file: extract every "**Target URL slug:**" line into PAGES array.

5. PRE-CACHE LOOP (lines 247-346):
   - Build STATIC_CACHE = concat(CONTENT_SPEC, TEMPLATES, OUTPUT_FORMAT).
   - Build CACHED_AUDIT = concat(AUDIT_SPEC, AUDIT_APPENDICES, ASBESTOS_SITE_MAP).
   - Build VALID_LINKS (blog mode only — skipped for guides).
   - For each page in PAGES:
     a. Carve a per-page assignment slice into .cache/assignment-<page>.md
        by splitting on "## Page \d+".
     b. Optionally slim REFERENCE to specified PARTs into .cache/ref-<page>.md.
     c. Assemble final writer prompt context into .cache/assembled-<page>.md
        (STATIC_CACHE + slim REFERENCE + per-page ASSIGNMENT).
   - reset PAGES_LAUNCHED=0 (this is the v10.13 bug-fix).

6. MAIN LOOP (lines 1196-1242) — parallel, MAX_PARALLEL=2:
   for each page in PAGES:
     if approved file already exists and not --force: SKIP, continue.
     wait until RUNNING < MAX_PARALLEL.
     spawn process_page(page) in background.

   process_page(page):
     round = 0; passed = false
     while round < 3 and not passed:
       round += 1
       step A. WRITER call:
         claude -p "<GUIDE WRITER PROMPT>" $WRITER_MODEL --dangerously-skip-permissions
         (round 1: full generation; round 2+: revise-from-feedback prompt)
         writer reads $assembled, writes JSON to DRAFTS/<page>.json
       step B. cap-detection: if writer returned in <10s with empty output,
         delete draft and write status=cap; return (NOT counted as a round).
       step C. POST-PROCESSOR (inline python3 heredoc):
         - strip em/en dashes from every string
         - truncate metaTitle/metaDesc to 60/160 cap
         - strip embedded newlines + double spaces
         - split paragraphs over 5 sentences in half
         - auto-merge paragraphs until count divisible by (h2_count+1),
           bail out if more than 3 merges needed.
       step D. AUDITOR call:
         claude -p "<GUIDE AUDITOR PROMPT>" $AUDITOR_MODEL
         auditor reads CACHED_AUDIT + TEMPLATES + draft + per-page assignment.
         if approved: auditor moves file DRAFTS -> APPROVED and appends to
            content/asbestos/approved-index-asbestos.md.
         if fail:    auditor writes content/asbestos/feedback/<page>-r<round>.md.

       step E. AuditKit mechanical gate (lines 1007-1163, GUIDE = dual-pass):
         pass 1 (structural): python3 scripts/audit_guide.py <draft>
                              --config <audit-configs/guide.json merged with
                              slug ngram_ignore>
         pass 2 (SEO):        python3 scripts/audit_guide.py <draft>
                              --config <keyword-configs/<slug>.json merged with
                              shared ngram_ignore>
         If either exit != 0: write feedback/<page>-r<round>.md with the FAIL
         lines from each pass; mv approved/<page>.json back to drafts/;
         passed = false.
         If both pass exit == 0: passed = true.

     if not passed after 3 rounds:
       mv drafts/<page>.json -> needs-review/<page>.json (escalation).
     write status file: approved | escalated | cap.

7. wait for all parallel jobs.

8. FAQ schema post-processing (lines 1267-1294):
   for each json in APPROVED:
     if it has faqs and no faqSchema: synthesize a schema.org FAQPage and inject.

9. Print summary, ls APPROVED + NEEDS_REVIEW, emit macOS notification, clean
   .cache/* assembled/ref/assignment files. Exit 0 normally; exit 2 if any page
   detected the usage cap (so run-overnight.sh can cooldown).
```

Inputs (per slug):
- `content/asbestos/assignment-batch-<slug>.md` — operator-authored brief.
- `content/asbestos/keyword-configs/<slug>.json` — per-slug primary/secondary/entities/secondary_nouns (33 files exist in the live repo as of inspection).
- `content/asbestos/audit-configs/guide.json` — shared G1-G5 audit config (one file).
- `content/asbestos/CONTENT_SPEC_GUIDE_ASBESTOS.md` — writer spec v2.7.
- `content/asbestos/ASBESTOS_Writer_Reference.md` — domain reference (live).
- `content/AUDIT_SPEC.md` (v1.6.4) + `content/AUDIT_APPENDICES.md` (v1.7).
- `content/asbestos/ASBESTOS_TEMPLATES.md`, `ASBESTOS_OUTPUT_FORMAT.md`, `ASBESTOS_SITE_MAP.md` — all currently STUB files in the live repo's tracked tree (live tree has them, but their contents in the reference snapshot are 1-line `# STUB` notes).

Outputs (per slug):
- `content/asbestos/approved/guides/<slug>.json` — final approved JSON (blog-posts.json schema).
- `content/asbestos/feedback/<slug>-r<round>.md` — round-N feedback for revisions.
- `content/asbestos/needs-review/<slug>.json` — escalated drafts after 3 failed rounds.
- `content/asbestos/approved-index-asbestos.md` — running index appended by the auditor.
- `content/asbestos/logs/batch-<batch>-<timestamp>.log` — full session transcript.

External binaries called:
- `claude -p "..."` (Claude Code CLI in non-interactive mode), invoked four times max per round per page (writer + auditor; round 1 + revision passes). Optional `--model sonnet` per `--sonnet`/`--sonnet-audit`.
- `python3` (inline heredoc post-processors + `scripts/audit_guide.py`).
- `osascript` (macOS Notification Center on completion).

Human-decision points (purely human, not pipeline-automated):
1. **Writing the assignment-batch markdown** — operator authors the slug brief, seed paragraph, keyword targets, link targets, regulatory entities, writer notes.
2. **Authoring the keyword-config JSON** — primary keyword, secondary ranges, ngram_ignore stopwords, intent_concepts, required_entities, secondary_nouns. Template at `keyword-configs/_TEMPLATE.json`.
3. **Operator review of `needs-review/` after 3 failed rounds** — pipeline never auto-approves an escalated guide.
4. **Operator-side copy of approved JSON onto the live site** — `content/asbestos/approved/guides/<slug>.json` → `asbestoshq-site/src/data/guides/<slug>.json`. There is no script in either repo that performs this transfer; the file hashes differ between the two trees (see section 11), confirming hand-edits during the copy step.
5. **Operator creation of the `src/app/guides/<slug>/page.tsx` route file** — 21-line wrapper per slug, must be added per new guide.
6. **Operator update of `APPROVED_GUIDE_SLUGS` set** in `src/components/GuideArticle.tsx` — list of 31 slugs hardcoded. New slugs must be added here or the inline-link rewriter strips their links to plain text.
7. **`git push`** to the site repo — triggers Vercel auto-deploy. The GitHub Action `schema-check.yml` runs 180s after deploy.
8. **Operator-side GSC submission** — sitemap + URL inspect / request-indexing for each new guide.

---

## 2. File inventory (every file the pipeline reads or writes)

### Pipeline scripts
- `content/run-batch.sh` (1,329 lines, v10.13) — orchestrator.
- `scripts/audit_guide.py` (543 lines, v3.4) — mechanical gate.
- `scripts/ctx.sh` (15,456 chars) — session snapshot / CC context save.
- `scripts/session-review.sh` (11,698 chars) — post-session review tool.
- `scripts/apply-approved-additions.sh` (8,601 chars) — appears to handle bulk approvals.
- `scripts/auto-harvest-to-library.sh` (4,748 chars) — archive harvest.

### Specs the pipeline reads every batch
- `content/asbestos/CONTENT_SPEC_GUIDE_ASBESTOS.md` (v2.7, 367 lines) — guide writer spec.
- `content/asbestos/CONTENT_SPEC_ASBESTOS.md` — **STUB** (2 lines, blog mode placeholder).
- `content/asbestos/CONTENT_SPEC_SERVICE_ASBESTOS.md` — **STUB** (2 lines, service mode placeholder).
- `content/AUDIT_SPEC.md` (v1.6.4, 619 lines) — universal auditor spec.
- `content/AUDIT_APPENDICES.md` (v1.7, 148 lines) — banned phrases + opener patterns.
- `content/asbestos/audit-configs/guide.json` — shared G1-G5 config (one file, ~40-line whitelist).
- `content/asbestos/keyword-configs/<slug>.json` — 33 per-slug files (live live tree); `_TEMPLATE.json` and `_REFERENCE.json` exist as scaffolds.
- `content/asbestos/asbestos-authority-links.md` (in reference snapshot, ~2.4KB) — canonical .gov authority link source.
- `content/asbestos/Asbestos_Keyword_Research_Report_April2026.md` (~62KB) — keyword research corpus.
- `content/asbestos/ASBESTOS_Writer_Reference.md` — domain reference (tracked, real content).
- `content/asbestos/ASBESTOS_TEMPLATES.md`, `ASBESTOS_OUTPUT_FORMAT.md`, `ASBESTOS_SITE_MAP.md` — all **STUB** in the reference snapshot. Pipeline still reads them via `STATIC_CACHE` / `CACHED_AUDIT` paths but they contribute nothing.

### Per-batch inputs (operator-authored)
- `content/asbestos/assignment-batch-<slug>.md` — 39 such files in reference snapshot, real ones live in `asbestos-contractors/content/asbestos/`.

### Pipeline outputs
- `content/asbestos/drafts/<slug>.json` (gitignored; transient).
- `content/asbestos/.cache/assembled-<slug>.md` / `assignment-<slug>.md` / `ref-<slug>.md` / `slug-merged-<slug>.json` / `struct-merged-<slug>.json` / `status-<slug>.txt` / `done-<slug>.txt` (transient, deleted at end of run).
- `content/asbestos/feedback/<slug>-r<round>.md` (gitignored; per-round writer feedback).
- `content/asbestos/approved/guides/<slug>.json` — 31 such files in the live tree.
- `content/asbestos/needs-review/<slug>.json` — escalations.
- `content/asbestos/approved-index-asbestos.md` — appended index of approved slugs.
- `content/asbestos/logs/batch-<batch>-<timestamp>.log` (gitignored).

### Site-side files (downstream, manual copy)
- `asbestoshq-site/src/data/guides/<slug>.json` — 32 such files (31 listed in APPROVED_GUIDE_SLUGS + one extra; the live data dir has all 32).
- `asbestoshq-site/src/data/blog-posts.json` — currently `[]`.
- `asbestoshq-site/src/data/static-pages.json` — driven by sitemap.
- `asbestoshq-site/src/app/guides/<slug>/page.tsx` — 31 per-slug route stubs (~21 lines each, copy-paste pattern).
- `asbestoshq-site/src/components/GuideArticle.tsx` — renderer; holds the `APPROVED_GUIDE_SLUGS` Set that gates inline link rewriting.

---

## 3. The 8 "AsbestosHQ" persona strings in run-batch.sh

Grep against `run-batch.sh` returns the 8 strings below. Lines 35, 131, 137 are operator-facing diagnostic strings, not LLM personas — listing them with the 5 actual personas for completeness as the prompt asked for "AsbestosHQ" strings.

**Line 35** (comment, header changelog):
> `#      prompts untouched (not active for phase 1 of AsbestosHQ.com).`

Role: documentation of which prompts are inactive in Phase 1.

**Line 131** (comment, guard rationale):
> `# Service mode is UST-legacy and not active for AsbestosHQ Phase 1.`

Role: warns operator that the SPC-1..14 checks carry UST oil-tank vocabulary that mis-audits asbestos content.

**Line 137** (operator-facing error message):
> `echo "AsbestosHQ Phase 1 ships guides only. Use --guide instead." >&2`

Role: hard guard against `--service` mode in the asbestos repo.

**Line 410** (SERVICE-MODE writer prompt, skeleton variant — currently unreachable since service mode is blocked):
> `claude -p "You are the WRITER for AsbestosHQ.com. You have a pre-filled JSON skeleton and reference specs.`

Role: skeleton-fill writer persona for state service pages. Inactive.

**Line 444** (SERVICE-MODE writer prompt, full-generation variant — also unreachable):
> `claude -p "You are the WRITER for AsbestosHQ.com. Read this file, then write the assigned service page for ${page}.`

Role: full-generation writer persona for state service pages. Inactive.

**Line 474** (GUIDE-MODE writer prompt — the ONLY active writer persona):
> `claude -p "You are the WRITER for AsbestosHQ.com. Read this file, then write the assigned GUIDE page for ${page}.`

Role: long-form national guide writer. Reads `${assembled}` (= STATIC_CACHE + slim REFERENCE + per-page ASSIGNMENT), writes blog-posts.json-schema JSON to `drafts/<slug>.json`. The writer is told the full v2.4 schema, numeric caps (V5/V7/V8/V9/V4c), G1-G5 structural rules, paragraph[0] hard-shape, link rules, closing-paragraph patterns, and C1-C4 human-voice constraints. ~90 lines of prompt body.

**Line 569** (BLOG-MODE writer prompt — currently unreachable because BLOG mode is the default branch and the mode-default is hard-blocked at lines 145-150 since `CONTENT_SPEC_ASBESTOS.md` is a stub):
> `claude -p "You are the WRITER for AsbestosHQ.com. Read this file, then write the assigned page for ${page}.`

Role: legacy 1,750-1,850-word blog post writer. Dormant.

**Line 847** (SERVICE-MODE auditor prompt — unreachable):
> `claude -p "You are the AUDITOR for AsbestosHQ.com content.`

Role: SPC-1..14 audit persona. Inactive.

**Line 881** (GUIDE-MODE auditor prompt — the ONLY active auditor persona):
> `claude -p "You are the AUDITOR for AsbestosHQ.com content.`

Role: long-form guide auditor. Reads `${CACHED_AUDIT}` + TEMPLATES + draft + per-page assignment. Applies G1-G5 structural, V-series voice/quality, L-series link, FRESH1/SPEC1, and C1-C4 soft-checks; moves to `approved/` or writes feedback file. ~95 lines of prompt body.

**Line 972** (BLOG-MODE auditor prompt — unreachable, paired with line 569 writer):
> `claude -p "You are the AUDITOR for AsbestosHQ.com content.`

Role: legacy blog auditor. Dormant.

In addition, the writer/auditor prompts reference the editorial team string verbatim in the schema requirement:
- Line 493 (writer prompt body): `- author: exactly the string \"The AsbestosHQ.com Editorial Team\"`
- Line 912 (auditor prompt body): `- E-E-A-T fields present: author = \"The AsbestosHQ.com Editorial Team\", lastReviewed = ISO date, reviewedBy = null`

Effective personas active in Phase 1: **2** (guide writer + guide auditor). The other 6 strings are inactive (3 service mode, 2 blog mode, 1 doc comment) or operator-facing (2 echo strings).

---

## 4. Audit checks catalog

Two layers — the LLM auditor runs `AUDIT_SPEC.md`, and the mechanical gate runs `audit_guide.py`. Several check IDs map across the two (V11↔V8, V12↔V9, V16↔D1, SEO5↔S7, SEO6↔S8, D3↔D4). The mapping is documented at the top of `AUDIT_SPEC.md`.

### `audit_guide.py` v3.4 — mechanical, binary

Thresholds and check list (verbatim from constants block, lines 70-90):

```
S1_WORD_RANGE             = (1800, 2800)
S3_H2_RANGE               = (4, 10)
S6_FIRST_PARA_MAX_SENT    = 3
S7_META_TITLE_MAX         = 65
S8_META_DESC_MAX          = 165
SEO5_RANGE                = (3, 12)
V5_HOWEVER_MAX            = 6
V7_SAME_WORD_STREAK_MAX   = 4
V8_SENTENCE_WORD_MAX      = 40
V9_PARA_SENT_MAX          = 6   (fail at 7+)
D1_ONE_SENT_PARA_MIN      = 1   (info-only, not enforced)
D4_EXCEPTION_MIN          = 1   (info-only, not enforced)
L1_INTERNAL_LINKS_RANGE   = (3, 12)
V4c_HEDGE_TOTAL_MAX       = 6
SPEC1_MIN                 = 3
FRESH1_MAX_DAYS           = 365
INTENT1_CONCEPTS_MIN      = 2
G1_SECONDARY_NOUNS_MIN    = 5
G1_TERM_FREQ_MIN          = 3
SAT1_THRESHOLD_MULTIPLIER = 1.3
```

Check inventory (all binary pass/fail, run on the approved JSON):

| Code | What it measures | Source |
|---|---|---|
| S1 | Total word count in paragraphs is 1800-2800 | `len(' '.join(paras).split())` |
| S2 | Paragraph count (informational, always passes) | `len(paras)` |
| S3 | H2 count is 4-10 | `len(h2s)` |
| S4 | Last section non-empty after `ceil(n/(h2+1))` distribution | derived |
| S5 | First sentence ≤ 15 words | regex split of `paras[0]` |
| S6 | First paragraph ≤ 3 sentences | sent_count(paras[0]) |
| S7 | `metaTitle` length < 65 | `len(d['metaTitle'])` |
| S8 | `metaDesc` length < 165 | `len(d['metaDesc'])` |
| SEO1 | Primary keyword in h1 | substring |
| SEO2 | Primary keyword in `metaTitle` | substring |
| SEO3 | Primary keyword in `metaDesc` | substring |
| SEO4 | Primary keyword in first 100 body words | substring |
| SEO5 | Total exact-match count of primary kw in 3-12 | `all_lower.count(pk)` |
| SEO5b | Primary keyword in N H2s (info-only) | |
| SEO5c | Max consecutive paras with primary ≤ 3 | streak check |
| SEO5d | No paragraph contains primary ≥ 2 times | |
| SEO6 | Secondary keyword counts (info-only) | from config |
| INTENT1 | ≥ 2 `intent_concepts` in first 150 body words | from config |
| TOPIC1 | `required_entities` matched ≥ `required_entities_min` (default 3) | from config |
| G1 | ≥ 5 secondary_nouns at ≥ 3 frequency (v3.4 mechanical) | from `secondary_nouns` |
| SPEC1 | ≥ 3 dollar/duration/year specifics (regex) | numeric regex |
| FRESH1 | `dateModified` parses and is < 365 days old | ISO date check |
| SAT1 | No bigram > 10 or trigram > 6 (after ignore-list filter) | calibrated +30% on base |
| V1 | Zero em dashes (`—`) anywhere | |
| V2 | Zero en dashes (`–`) | |
| V4 | None of the 88-entry BANNED_PHRASES list found in body | hardcoded list at lines 37-66 |
| V4c | `ultimately + essentially + fundamentally` combined ≤ 6 | |
| V4d | "not just X, it's Y" pattern (info-only) | regex |
| V5 | "however" count ≤ 6 | |
| V7 | Same-first-word paragraph streak ≤ 4 | |
| V8 | No sentence > 40 words | |
| V9 | No paragraph with > 6 sentences | |
| V10 | Zero `**bold**` markers in body | |
| D1 | One-sentence paragraph count (info-only) | |
| D4 | Exception/limiting sentence count (info-only) | |
| L1 | 3-12 internal links in body | `\]\(/...\)` regex |
| L2 | No duplicate internal link targets | |
| L4 | At least one `.gov` link present in authorityLinks or body | |
| L5 | No em dash in `authorityLink.name` | |

V3, V6, V11, D2, D3, V4b, V4e, L3 are all dropped (see code comments). Exit code is `1` if any FAIL, else `0`.

### `AUDIT_SPEC.md` v1.6.4 — LLM auditor rules (binary, with cosmetic tolerance band)

Structured into 9 sections + GSPC overrides for guide mode + verdict rules. Summary of every rule:

**Pre-audit setup** — skip anti-sameness (AS1-AS5) if `approved/` has < 4 files; if ≥4, load only the last 4 from `approved-index-asbestos.md`.

**Verdict rules** — PASS / PASS WITH NOTES (≤3 failures, all cosmetic) / FAIL. Cosmetic tolerance band defined for S1, S7, V11 (no band, hard at 41+), V12 (one 6-sent paragraph OK), V13, V14, SEO5, SEO6, L5, S12.

**Section 1: Structural** S1-S12 — word-count range, single H1, primary kw in h1, 4-10 H2s, no generic H2 names ("Introduction," "Conclusion," …), proper heading hierarchy, first-sentence ≤15 words, first sentence takes a position, primary kw in first sentence, final section is not "Conclusion," final section is action-oriented, FAQ schema present if FAQ.

**Section 2: SEO** SEO1-SEO9 — primary kw in h1 / first para / ≥1 H2; ≥8 secondary variants; metaTitle <65; metaDesc <165 with primary kw; slug lowercase + hyphens + 5-8 words; density ≤1 per 150 words.

**Section 3: Voice and Quality** V1-V18 — zero em/en dashes; zero hyphen-as-separator; zero banned phrases (V4 grep); zero banned openers (V5); zero bold in body (V6); zero bullets/lists (V7); zero images (V8); however ≤6 (V9); zero "Additionally" (V10); sentence ≤40w (V11); paragraph ≤5 sentences (V12); ≤3 consecutive paragraphs same-first-word (V13); ≤3 consecutive `This/The/It` sentences (V14); no section ending sentence restates section heading (V15); D1 count info-only (V16); ≥1 paragraph with ≥4 sentences (V17); not >80% paragraphs same length (V18).

**Section 4: Content Depth** D1-D6 — ≥1 gotcha/trap/myth section; D2 dropped; ≥3 exception sentences across H2s (template-overridable to 1-2); ≥1 tradeoff/limitation per H2 (D4); no H2 100% recommendation (D5); ≥1 state-specific detail (D6).

**Section 5: Links** L1-L7 — `relatedLinks` 3-5 entries; L2 deprecated (directory CTA, removed in v1.6.4); ≥1 .gov authority for regulatory content (L3); no Wikipedia/news/trade-assoc/vendor links (L4); ≥1 state page in relatedLinks if states discussed (L5); each relatedLinks slug exists in SITE_MAP — soft fail (L6); all authorityLinks are https + .gov (L7).

**Section 6: Anti-sameness** AS1-AS5 (skipped until 4+ approved) — opening sentence structure differs; no sentence >80% similar to last 4 approved; H2 style differs; CTA differs; paragraph flow pattern differs.

**Section 7: First-sentence flow test** FST1-FST2 — first-sentences-in-sequence form coherent progression; ≤2 in a row on same sub-topic.

**Section 8: Revision-round checks** R1-R4 (rounds 2-3 only) — all previous-round failures addressed; word count didn't drop >20% (deletion gaming); no new failures introduced; mode-specific checks re-run every round.

**Section 9: System integrity** SYS1-SYS3 — writer cited correct spec version; ≥10 consecutive zero-fail pages flagged as anomaly; cited cost/timeline/penalty data must be <6 months old or verdict becomes NEEDS REVIEW.

**Guide mode (GSPC) overrides** GSPC-1..14 — adapt SPC-1..14 for long-form. Active checks: GSPC-1 CANNIBAL (no overlap with service/state keyword ownership), GSPC-2 STUFFING (primary ≤15 across guide, ≤3 per paragraph), GSPC-3 TRAILING-SLASH, GSPC-4 BROKEN-LINKS, GSPC-5 MISSING-AUTHORITY (regulatory claim must have paired .gov in same paragraph), GSPC-6 SEMANTIC-VARIETY (≥5 secondary nouns at ≥3 occurrences — matches `audit_guide.py` G1), GSPC-7 EEAT-REG-DENSITY (≥4 specific citations + ≥3 concrete specifics), GSPC-8 FIRST-PERSON-EXPERTISE (no fake personal expertise), GSPC-9 CROSS-LINK-STRUCTURE (≥4 service-page links, ≥2 state-page links, ≥2 guide cross-links — note this conflicts with Phase 1 hidden-directory rules; LLM auditor is expected to apply the v2.4 override), GSPC-10 SECTION-BALANCE (40%-200% of average), GSPC-11 HEADING-HIERARCHY, GSPC-12 INTRO-HOOK (primary kw + concrete data in first 100 words), GSPC-13 dropped, GSPC-14 SCHEMA-MARKUP (FAQ JSON-LD if FAQ section).

**Service mode (SPC-1..14)** — UST legacy; not active for asbestos. Lists hub-mapping tables, primary-keyword ceilings, substitution test, etc. The auditor branch in `run-batch.sh` for service mode is hard-blocked.

**Scorecard output format** — JSON-like markdown block with PASS/FAIL listing only failing checks; verdict + ACTION line. Pre-defined.

**Batch pattern report** — after every 5 pages, summary of most-common failures saved to `feedback/batch-report-<date>.md`.

---

## 5. Content spec summary — what the writer is told to produce

`CONTENT_SPEC_GUIDE_ASBESTOS.md` v2.7 (367 lines). Identical bytes (MD5 3f5b84bf…) to `ssg-content/content/ssg/CONTENT_SPEC_GUIDE_SSG.md` — i.e., the SSG fork has not yet been re-skinned.

Writer is told to produce a single JSON object matching this schema:

```json
{
  "slug": "keyword-rich-slug",
  "metaTitle": "Under 60 chars with target keyword",
  "metaDesc": "Under 160 chars. Includes target keyword.",
  "h1": "Full Title With Target Keyword",
  "h2s": ["...", "..."],
  "paragraphs": ["...", "..."],
  "category": "Environmental Guides",
  "date": "April 2026",
  "dateModified": "YYYY-MM-DD",
  "authorityLinks": [{"name": "...", "url": "https://..."}],
  "relatedLinks": [{"title": "...", "slug": "..."}],
  "allText": "",
  "author": "The AsbestosHQ.com Editorial Team",
  "lastReviewed": "YYYY-MM-DD",
  "reviewedBy": null
}
```

Structural rules:
- **Length** writer 2000-2500 words; floor 1800 for niche; ceiling 2800; auditor accept 1700-3000.
- **Sections** 5-8 H2s, each tied to a real searchable sub-query, no generic ("Introduction," "Conclusion," etc.). Final H2 action-oriented.
- **Paragraph distribution** `len(paragraphs) == N × (h2_count + 1)`. Example 7 H2s × 4 = 32 paragraphs. Post-processor silently auto-merges ≤3 pairs to hit this.
- **Intro** 100-200 words before first H2; first 100 must contain primary keyword + concrete data point; no "In this guide we'll cover" framing.
- **Paragraph[0] hard shape** ≤3 sentences total; sentence 1 ≤15 words and contains primary keyword.
- **H2/H3 only** — renderer is flat, no H4.
- **No TOC** — renderer doesn't support jump links.
- **Authority links** ≥4 distinct .gov in `authorityLinks` array; every regulatory claim paired with a `.gov` inline link in the same paragraph; permanent pages only.
- **Internal links** ≥2 guide-to-guide cross-links; all trailing-slash; max 12 total. No `/find-asbestos-contractors/` or `/request-a-quote/` in Phase 1.
- **Tradeoff coverage** every major claim names a limitation; flag-don't-invent.
- **Closing paragraph** one of three patterns (hire-licensed-firm / verify-state-credentials / testing-first) — no directory CTA, no "browse our listings."

Mechanical numeric caps (also flagged in writer prompt body so the writer self-checks):
- Sentence word cap 40; paragraph sentence cap 6; "however" cap 6; same-first-word streak cap 4; hedge total cap 6.

Voice rules (C1-C4, LLM auditor soft):
- C1 sliding 4-sentence window must include a <10w or >35w sentence.
- C2 each H2 has one register-shift device.
- C3 ≥2 paragraphs with a non-fact transition/color sentence.
- C4 paragraph length distribution: ≥20% 1-2 sentences, ≥20% 5-6 sentences, rest 3-4.

Writer self-check (18 items, listed verbatim in the spec).

**SSG diff** — `CONTENT_SPEC_GUIDE_SSG.md` is byte-identical to the asbestos guide spec. Every asbestos-specific element will need rewriting:
- All references to "AsbestosHQ.com" (h1, intro, schema, prompts, brand string).
- The `author` field default `"The AsbestosHQ.com Editorial Team"`.
- The `category` default `"Environmental Guides"`.
- Banned-phrases list specific to asbestos/regulatory tone.
- G2 regulatory-citation examples (AHERA, NESHAP, 40 CFR 763, OSHA 1926.1101, ASTM E2356).
- G3 concrete-density examples (40 CFR 280, AHERA 1986, asbestos-specific dollar ranges).
- Closing paragraph patterns (hire-licensed-firm references state asbestos abatement agencies).
- Directory CTAs (`/find-asbestos-contractors/`, `/request-a-quote/`) — SSG has no such pages.
- 11-slug scope mention in §Scope.
- `cc-context` reading list triggers.

The CLAUDE.md in `ssg-content/` is also byte-identical to `asbestos-contractors/CLAUDE.md` — still describes the asbestos project. Not yet re-skinned.

---

## 6. Example assignment-batch file (full content)

`_asbestos-reference/assignment-batch-asbestos-shingles-guide.md`, 41 lines, verbatim:

```markdown
# Assignment Batch: asbestos-shingles-guide — Guide Page
# 1 page | Opus writer | Opus auditor | --guide mode
# Run: cd ~/Desktop/asbestos-contractors && bash content/run-batch.sh asbestos-shingles-guide --guide

---

## Page 1

**Target URL slug:** asbestos-shingles-guide
**Page type:** Long-form guide
**Template:** Environmental Guides (blog-posts.json schema)
**Word count range:** 1,600-1,800 (floor 1,500, ceiling 2,000)
**Primary keyword:** asbestos shingles
**Primary volume:** 1,500/mo | KD 0 | CPC $0.70
**SERP Status:** CLEAR (2026-04-22) — editorial + roofing contractor sites, clean KD 0 win
**Secondary keywords:** asbestos roof shingles (350/mo), asbestos roof tiles (250/mo), cement shingles, asbestos roof (1,400/mo)
**Audience:** Homeowners of pre-1980 homes assessing roof condition, roof replacement buyers.

**Seed:**
Cover history (1920s-1986), cement shingles vs asphalt shingles with asbestos, visual identification (rigid, chalky, gray, pre-drilled nail holes), manufacturers (Eternit, Johns-Manville), roof vs siding use, friability status (Category I non-friable until disturbed), testing, replacement cost overview. Bridges to higher-CPC roof removal guide.

**Internal link targets (minimum):**
- 4+ service page links
- 2+ guide cross-links (asbestos-siding-guide, friable-vs-nonfriable-asbestos)
- 1+ link to /find-asbestos-contractors/ or /request-a-quote/ (L3)
- All trailing-slash

**Authority links (minimum 4 distinct .gov):**
Pull from content/asbestos/asbestos-authority-links.md: EPA NESHAP, OSHA 1926.1101, EPA 1989 ban, EPA Asbestos in Your Home.

**Required regulatory entities (G2 min 4):** NESHAP, 40 CFR 61.141, 29 CFR 1926.1101, EPA 1989 ban

**Notes for writer:**
- Hook with specific identification (rigid gray shingle, chalky texture, pre-drilled nail holes) + date range (1920s-1986)
- Name specific manufacturers for G3: Eternit, Johns-Manville
- Distinguish cement asbestos shingles (roof + siding use) from asphalt shingles with asbestos fibers
- 5+ secondary nouns 3+ times: cement shingle, roof shingle, fiber cement roof, rigid shingle, gray shingle
- No affiliate CTAs
- Directory CTA only
- Final section action-oriented
```

Note: this reference snapshot still references the now-removed L3 directory CTA rule and the now-banned `/find-asbestos-contractors/` link target. The live spec dropped that requirement in v2.4 (2026-04-24). The pipeline runs against the live spec, not the reference snapshot.

Fields per page block:
- `Target URL slug` — output filename + slug field
- `Page type`, `Template`, `Word count range`, `Primary keyword`, `Primary volume`/`KD`/`CPC`, `SERP Status`, `Secondary keywords`, `Audience`
- `Seed` — 2-5 sentence prose seed of what to cover
- `Internal link targets` / `Authority links` / `Required regulatory entities`
- `Notes for writer` — bullet list of style/tactical guidance

What the human fills in for one assignment file:
- Slug, primary keyword, volume/KD/CPC/SERP status from keyword research
- Word count range (per niche)
- Secondary keyword list with volumes
- Audience persona (1 sentence)
- Seed paragraph (the "what to cover" brief, 80-200 words)
- Link target counts + cross-link slugs by name
- 4+ required entities for G2
- 5-8 bullet writer notes (hook angle, manufacturers, distinctions, etc.)

What is pipeline-generated: nothing — assignment-batch markdown is entirely operator-authored.

---

## 7. Example approved guide (structure)

`_asbestos-reference/approved-guides/asbestos-shingles-guide.json` — 96 lines (~14 KB). Structure inspected via `python3` (full file is too long to inline). Key fields:

- `slug: "asbestos-shingles-guide"`
- `metaTitle: "Asbestos Shingles: Identification and Removal Guide (2026)"` (60 chars)
- `metaDesc: "Asbestos shingles guide covering identification, NESHAP rules, testing, and the $8,000 to $20,000 cost to remove an asbestos cement shingle roof."` (155 chars)
- `h1: "Asbestos Shingles: A Homeowner's Identification and Removal Guide"`
- `h2s` (7 entries):
  1. "What Are Asbestos Shingles and When Were They Made?"
  2. "How to Identify Asbestos Roof Shingles vs Modern Materials"
  3. "Are Asbestos Cement Shingles Dangerous If Left Alone?"
  4. "Testing Suspect Asbestos Shingles for Confirmation"
  5. "Cost to Remove or Replace an Asbestos Shingle Roof"
  6. "Legal and Safety Rules for Asbestos Shingle Removal"
  7. "Your Next Step: Plan an Asbestos Shingle Project"
- `paragraphs` 32 strings — distribution matches `4 × (7+1) = 32`, satisfying the renderer formula.
- `category: "Environmental Guides"`
- `date: "April 2026"`, `dateModified: "2026-04-22"`, `lastReviewed: "2026-04-22"`, `reviewedBy: null`
- `author: "The AsbestosHQ.com Editorial Team"`
- `authorityLinks` 5 entries — EPA NESHAP, OSHA 1926.1101, EPA US Federal Bans on Asbestos, EPA Asbestos in Your Home, EPA AHERA. All `https://www.epa.gov/` or `https://www.osha.gov/`.
- `relatedLinks` 4 entries — `asbestos-siding-guide`, `friable-vs-nonfriable-asbestos`, `is-popcorn-ceiling-asbestos`, `when-was-asbestos-used-in-homes`.
- `allText: ""` (legacy field, no renderer uses it).

The paragraphs are professional, fact-dense, specific — e.g., paragraph 1: "Asbestos shingles are cement roof tiles bound with 14 to 30 percent chrysotile. Homeowners identify and test suspect roofs, then choose between management in place, encapsulation, or a $8,000 to $20,000 full abatement. The product is a Category I non-friable ACM under NESHAP 40 CFR 61.141, and worker exposure during removal is regulated by OSHA 29 CFR 1926.1101."

Inline links in body paragraphs use markdown `[text](/url/)` syntax for both internal (`/asbestos-inspection/`, `/asbestos-siding-guide/`, `/california-asbestos-contractors/`, `/find-asbestos-contractors/`) and external (full https://). The `GuideArticle.tsx` renderer rewrites only paths matching `APPROVED_GUIDE_SLUGS` into clickable `<Link>` elements; everything else (directory paths, service pages, state pages) is rendered as plain text in Phase 1.

The reference-snapshot file MD5 (`3d13672…`) differs from the same file in the live `asbestoshq-site/src/data/guides/asbestos-shingles-guide.json` (`de0f902…`) — the live one has had additional edits since the snapshot.

---

## 8. Example keyword config

`_asbestos-reference/keyword-configs/asbestos-shingles-guide.json`, full content:

```json
{
  "_comment": "Guide #6 — asbestos-shingles-guide. 1,500/mo, KD 0, $0.70 CPC, SERP CLEAR. [VERIFY] 1986 roofing phase-out + cement shingles volume dropped.",
  "primary": "asbestos shingles",
  "primary_target": [4, 10],
  "primary_in_h2s_target": [1, 2],
  "secondary": {
    "asbestos roof shingles": { "range": [1, 3] },
    "asbestos roof tiles": { "range": [1, 2] },
    "cement shingles": { "range": [1, 2] },
    "asbestos roof": { "range": [1, 3] }
  },
  "ngram_ignore": ["asbestos", "shingle", "shingles", "roof", "roofing", "cement", "fiber", "tile"],
  "intent_concepts": ["identify", "test", "replace", "assess", "estimate"],
  "required_entities": ["NESHAP", "40 CFR 61.141", "29 CFR 1926.1101", "EPA 1989 ban"],
  "required_entities_min": 4,
  "secondary_nouns": ["roof", "cement", "chrysotile", "ACM", "NESHAP", "CFR", "asphalt", "fiber", "abatement", "non-friable"]
}
```

Template at `_TEMPLATE.json` (15 lines) maps fields to TODO placeholders. The `_REFERENCE.json` companion sits next to it as a fully-populated example.

Relation to the assignment-batch file: the keyword config is the **mechanical-gate** companion to the assignment-batch's **writer-facing brief**.
- The assignment-batch tells the writer prose-style ("Primary volume: 1,500/mo", "Secondary keywords: asbestos roof shingles (350/mo)…", "Required regulatory entities: NESHAP, 40 CFR 61.141…").
- The keyword config tells `audit_guide.py` the exact thresholds in JSON ("primary: asbestos shingles", "required_entities: [...]", "secondary_nouns: [...]") so the SAT1/SEO1-5/INTENT1/TOPIC1/G1 checks have ranges and ignore-lists.

Both files are operator-authored. Pre-flight check in `run-batch.sh` lines 1119-1126 hard-fails any slug missing its `keyword-configs/<slug>.json` (MISSING_CONFIG status). The `audit-configs/guide.json` is a single shared file with ngram_ignore + intent_concepts + required_entities for the cross-cutting structural pass.

---

## 9. AsbestosHQ site map (page structure, deploy flow)

Stack: **Next.js 16.2.2 + React 19 + Tailwind 4** (per `asbestoshq-site/package.json`). Deployed to Vercel; auto-deploy on `git push` to `main`.

Notable config:
- `trailingSlash: true`, `skipTrailingSlashRedirect: true` in `next.config.ts` — site is canonical-trailing-slash.
- `output: undefined` (server-rendered, not static export).
- Security headers (`X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`) applied to all routes via `headers()`.
- `images.unoptimized: true`.
- Phase 1 redirect/header blocks for UST/contractor routes are commented out.

App-router layout (`src/app/`):

```
src/app/
├── page.tsx                                — homepage (hero + 3 entry cards)
├── layout.tsx                              — root layout
├── globals.css
├── not-found.tsx
├── robots.ts
├── sitemap.ts                              — generates homepage + static-pages + /guides/ hub + 31 guide URLs + blog-posts
├── blog/                                   — empty (`blog-posts.json` is `[]`)
├── guides/
│   ├── page.tsx                            — /guides/ hub, imports + lists all 31 guide JSONs
│   └── <slug>/page.tsx                     — 31 per-slug routes, each a 21-line wrapper that imports `@/data/guides/<slug>.json` and renders <GuideArticle data={guide} />
├── [slug]/                                 — dynamic slug catch-all (blog post style)
├── api/
├── contractor/                             — exists (Phase 2 placeholder)
├── find-asbestos-contractors/              — exists (Phase 2 placeholder)
├── request-a-quote/                        — exists (Phase 2 placeholder)
└── pricing/                                — exists (Phase 2 placeholder)
```

Content data (`src/data/`):
- `blog-posts.json` — currently `[]`. Used by sitemap + dynamic `[slug]` route. UST-era `blog-posts.ust-backup.json` retained for reference.
- `guides/<slug>.json` — 32 files. Each is a copy of an approved guide from `asbestos-contractors/content/asbestos/approved/guides/`. **Hashes differ between the two trees**, so there are edits made during the copy step.
- `service-pages.json`, `service-pages.ust-backup.json`, `service-pages-pre-patch-20260422.json` — UST legacy, not actively consumed in Phase 1.
- `static-pages.json` — drives sitemap static entries.
- `state-agencies.ts`, `state-content.ts`, `state-authority-links.json` — UST legacy.

Renderer: `src/components/GuideArticle.tsx`. Contains the hardcoded `APPROVED_GUIDE_SLUGS` set of 31 slugs. `parseInlineLinks()` walks each paragraph's markdown links and rewrites internal `[text](/slug/)` references to either:
- `<Link href="/guides/<slug>/">` if the slug is in `APPROVED_GUIDE_SLUGS`, OR
- plain text (no `<a>`) for every other internal slug (Phase 1 hidden-directory strip).
External `https://` links become `<a target="_blank" rel="noopener nofollow">`. This is how Phase 1 "hides" directory CTAs at the render layer without stripping them from the source JSON.

Deploy pipeline:
1. **Local edit** — operator copies `content/asbestos/approved/guides/<slug>.json` → `asbestoshq-site/src/data/guides/<slug>.json`. Operator creates `src/app/guides/<slug>/page.tsx` (21-line wrapper). Operator appends slug to `APPROVED_GUIDE_SLUGS` in `GuideArticle.tsx`.
2. **prebuild** — `npm run build` runs `npx tsx scripts/generate-redirects.ts` first.
3. **`git push origin main`** — triggers Vercel auto-deploy.
4. **GitHub Action `.github/workflows/schema-check.yml`** — sleeps 180s for Vercel to settle, then runs `python3 scripts/check_schema.py --all`, with one 60s retry. Opens an issue if both attempts fail.
5. **GitHub Action `.github/workflows/weekly-link-audit.yml`** — Mondays 9am PT link audit, opens PR with fixes.
6. **Sitemap** auto-generated at build time from `APPROVED_GUIDE_SLUGS` + `blog-posts.json` + `static-pages.json`. No manual sitemap edits needed.
7. **GSC submission** — operator-driven (not in repo).

`.vercel-trigger` file (`# Cache bust 2026-04-17T01:29:24Z`) used to force redeploys.

Notes:
- The CLAUDE.md in `asbestoshq-site/` is the **UST contractors** CLAUDE.md — still describes USTContractors.com / 5,136 Airtable records / SPC-1..14. Has not been re-skinned for AsbestosHQ.
- `add_analytics.py`, `add_blog_related_links.py`, `add_service_crosslinks.py`, `fix_blog_authority_links.py`, `fix_service_meta_titles.py`, `fix-footer.py`, `swap-footer-sections.py`, `update-footer.py` — UST-era root-level Python helpers; not actively run in Phase 1 but retained.
- `TASK_REPORT.md` and `SANITIZATION_NOTES.md` are the most recent operator notes on what's been cleaned up.

---

## 10. SmartSourceGuide site map (page inventory, categories, deploy flow)

Stack: **Next.js 14 + React 18** (per `smartsourceguide/package.json`). Configured as a **static export** (`output: 'export'`, `trailingSlash: true`).

Repo state: live in `~/projects/smartsourceguide/`; git remote `mdp280028-ui/smartsourceguide.git`.

Inventory — every `page.tsx` file with byte size and content state:

| Route | File | Bytes | Lines | State |
|---|---|---|---|---|
| `/` | `app/page.tsx` | 2,738 | 54 | **Real homepage** — hero, 3 featured-article cards linking to it-support/managed-it-vs-break-fix, managed-it-services-pricing, answering-services/best-answering-services. |
| `/about/` | `app/about/page.tsx` | 1,675 | 30 | **Real about page** — short prose about independent reviews, affiliate disclosure, contact email contact@smartsourceguide.com. |
| `/best-answering-services/` | `app/best-answering-services/page.tsx` | 376 | 14 | **Redirect stub** — meta refresh + noindex to `/answering-services/best-answering-services/`. |
| `/outsourced-it-support-cost/` | `app/outsourced-it-support-cost/page.tsx` | 341 | 14 | **Redirect stub** — meta refresh + noindex to `/it-support/managed-it-services-pricing/`. |
| `/fleet-tracking/best-fleet-gps-tracking/` | 15,471 | 131 | **Real article** |
| `/fleet-tracking/best-fleet-fuel-cards/` | 13,152 | 121 | **Real article** |
| `/fleet-tracking/best-fleet-dash-cam/` | 14,027 | 121 | **Real article** |
| `/fleet-tracking/best-fleet-maintenance-software/` | 14,217 | 125 | **Real article** |
| `/fleet-tracking/fleet-tracking-contracts/` | 13,342 | 125 | **Real article** |
| `/it-support/managed-it-vs-break-fix/` | 14,695 | 141 | **Real article** |
| `/it-support/best-it-support-law-firm/` | 14,442 | 123 | **Real article** |
| `/it-support/managed-it-services-pricing/` | 13,297 | 129 | **Real article** |
| `/it-support/outsource-help-desk-guide/` | 16,186 | 149 | **Real article** |
| `/answering-services/best-bilingual-answering-service/` | 14,289 | 131 | **Real article** |
| `/answering-services/answering-service-pricing/` | 13,925 | 117 | **Real article** |
| `/answering-services/best-ai-answering-service/` | 13,348 | 119 | **Real article** |
| `/answering-services/best-answering-services/` | 13,651 | 129 | **Real article** |
| `/answering-services/best-answering-service-law-firm/` | 13,739 | 133 | **Real article** |

**Correction to prompt premise:** the prompt described SSG as "dormant, ~14 mostly-empty page.tsx." The reality on disk: **19 page.tsx files; 15 are full-prose articles (117-149 lines, 13-16 KB each); 2 are real top-level pages (home + about); 2 are noindex redirect shims.** Each "real article" page.tsx file holds the entire body inline as JSX — there is no JSON data layer the way AsbestosHQ has. The article prose is hand-embedded in the route file.

Categories the SSG repo actually has (directory names under `app/`):
1. **answering-services/** — 5 articles
2. **fleet-tracking/** — 5 articles
3. **it-support/** — 4 articles
4. **about/** — 1 page

The prompt mentions "the 4 categories the SSG repo has." Three are content categories (answering-services, fleet-tracking, it-support); the fourth is `about/`. The header navigation (`components/Header.tsx`) only links to **3 sections**: IT support, Answering services, About. Fleet-tracking is not in the top-nav even though 5 fleet-tracking articles exist.

URLs live today vs placeholder:
- **Live**: every route in the table above (15 articles + home + about) plus the 2 redirect shims.
- **Placeholder**: none. There are no empty `page.tsx` stubs in this repo.

Static-export deploy:
- `npm run build` produces a static export.
- `vercel.json` has 4 redirect rules (2 pairs for slash/non-slash → canonical category-prefixed paths).
- No GitHub Actions in this repo (no `.github/` directory).
- No content data files (`src/data/blog-posts.json` etc.). Article content is inlined.

The SSG site at present has **zero connection to the SSG content pipeline** (`ssg-content/`). The 15 article pages were authored independently of `run-batch.sh`. There are no JSON files in `smartsourceguide/` matching the asbestos pipeline output schema, and no `src/components/GuideArticle.tsx`-style renderer.

---

## 11. Diffs: asbestos-contractors ↔ ssg-content, and asbestoshq-site ↔ smartsourceguide

### `asbestos-contractors/` vs `ssg-content/` (pipeline diff)

`diff -rq` returns only `.git/` differences (different remotes, different commit history) plus this content-tree diff:

- **Only in `asbestos-contractors/content/`**: `ASBESTOS_SITE_MAP.md`, `TEMPLATES_ASBESTOS.md`, `asbestos/` (the entire domain content subtree).
- **Only in `ssg-content/content/`**: `ssg/` (the SSG counterpart subtree).
- **In both** at the `content/` root, byte-identical: `AUDIT_APPENDICES.md`, `AUDIT_SPEC.md`, `run-batch.sh`.
- `scripts/` directory — byte-identical across both repos (apply-approved-additions.sh, audit_guide.py, auto-harvest-to-library.sh, ctx.sh, session-review.sh).
- `CLAUDE.md` — byte-identical; both still describe the asbestos project.

Inside the domain subtrees:

| File | asbestos-contractors | ssg-content |
|---|---|---|
| `CONTENT_SPEC_GUIDE_*.md` | v2.7, 367 lines, real | byte-identical (MD5 3f5b84bf…), 367 lines, real |
| `CONTENT_SPEC_SERVICE_*.md` | 2-line STUB | 2-line STUB |
| `CONTENT_SPEC_*.md` (blog) | 2-line STUB | 2-line STUB |
| `audit-configs/guide.json` | populated (ngram_ignore list of ~40 asbestos terms + entity sets) | populated, contents specifically asbestos (matches the source repo) |
| `keyword-configs/` | 33 populated `<slug>.json` files in live tree | empty in `ssg-content/content/ssg/keyword-configs/` |
| `approved/guides/` | 31 approved guides in live tree | empty |
| `assignment-batch-*.md` | present in live tree | none in ssg-content |
| `ASBESTOS_Writer_Reference.md` | real content | not present (no SSG equivalent yet) |
| `ASBESTOS_TEMPLATES.md` / `_OUTPUT_FORMAT.md` / `_SITE_MAP.md` | live tree (STUB content in reference) | not present in SSG fork |

**Conclusion** — `ssg-content/` is currently a **scaffolded fork**: same scripts, same orchestrator, byte-identical CONTENT_SPEC_GUIDE (still asbestos-flavored), but **all SSG-specific inputs missing**: no SSG keyword-configs, no SSG assignment-batches, no SSG writer reference, no SSG audit-config tuned to SSG vocabulary, no SSG approved guides. The pipeline is sitting in a "needs rebranding + content authoring" state.

### `asbestoshq-site/` vs `smartsourceguide/` (site diff)

Completely different stacks and content models:

| | asbestoshq-site | smartsourceguide |
|---|---|---|
| Next.js | 16.2.2 | 14.2.x |
| React | 19.2 | 18.3 |
| Build | server-rendered (Vercel) | `output: 'export'` static |
| Trailing slash | yes | yes |
| Content model | JSON data files (`src/data/guides/*.json`) + thin `page.tsx` wrappers | Prose embedded directly in each `page.tsx` as JSX |
| Renderer | shared `GuideArticle.tsx` component with markdown link parsing | none — each page is bespoke JSX |
| Guide count | 31 (in `APPROVED_GUIDE_SLUGS`) + 32 JSON files in `src/data/guides/` | 15 article pages + 2 real top-level pages |
| Sitemap | dynamic at build, driven by hardcoded `APPROVED_GUIDE_SLUGS` set | static export (no dynamic sitemap.ts) |
| GitHub Actions | `schema-check.yml`, `weekly-link-audit.yml` | none |
| Pipeline integration | content arrives as JSON, manually copied from `asbestos-contractors/content/asbestos/approved/guides/` | content authored directly in JSX; no pipeline connection |
| Phase status | Phase 1 with directory hiding | Phase 1 only (article-only, no directory at all) |
| CLAUDE.md | still UST-flavored (40K char USTContractors.com spec) | none committed locally |

Categories — AsbestosHQ has 31 guides under one category ("Environmental Guides"); SSG has 3 content categories (it-support, answering-services, fleet-tracking) of 4-5 articles each.

Maturity gap — AsbestosHQ has a data-driven renderer, a published-slug whitelist, and CI workflows; SmartSourceGuide is a hand-built marketing site with no data layer or CI, where each new page is a JSX file written from scratch. The pipeline output format (JSON matching `blog-posts.json` schema) is not consumed anywhere in SmartSourceGuide.

---

## 12. Human-driven steps in the current workflow (ordered by time investment)

Ranked by what the code shows is hand-authored, not pipeline-generated. (Time bands inferred from artifact size and complexity; not measured.)

1. **Author the assignment-batch markdown for each slug** — 30-90 min/slug. Requires keyword research, SERP scan, audience framing, seed prose, link-target planning, regulatory-entity selection, writer-notes drafting. Cumulatively the heaviest non-LLM step.
2. **Author the per-slug keyword-config JSON** — 10-20 min/slug. Primary keyword, secondary ranges, ngram_ignore stopword set, intent_concepts, required_entities, secondary_nouns (must be tuned per material — popcorn ceiling vocabulary ≠ transite pipe ≠ black mastic, per spec v2.5). The `_TEMPLATE.json` exists but every field is per-slug.
3. **Manual review of `needs-review/` escalations after 3 failed rounds** — variable. The pipeline does not auto-approve escalations; the operator either rewrites by hand, runs `--force` after fixing the assignment/config, or accepts the partial draft.
4. **Manual copy of approved JSON → asbestoshq-site `src/data/guides/<slug>.json`** — 1-2 min/slug. There is no script in either repo that does this transfer. Hash diff between the two trees confirms hand-edits land here.
5. **Create `src/app/guides/<slug>/page.tsx` route wrapper** — 2-3 min/slug. 21-line copy-paste pattern but operator-authored each time.
6. **Append slug to `APPROVED_GUIDE_SLUGS` Set in `GuideArticle.tsx`** — < 1 min/slug. One line.
7. **Operator-side `git push` + monitor `schema-check.yml`** — 5-10 min including watching the CI run land + the 180s Vercel-settle pause.
8. **GSC submit / URL inspect for each new guide** — 2-5 min/slug, off-repo.
9. **Pipeline orchestration choices on each run** — picking the slug, picking `--force` or not, picking writer/auditor model (`--sonnet` flags), watching the spinner, deciding whether to ctrl-C if a usage cap hits.
10. **Maintaining the spec / pipeline itself** — periodically tweaking thresholds (V5/V7/V8/V4c/G1) when failure patterns shift, updating CONTENT_SPEC_GUIDE on each version bump, syncing AUDIT_SPEC ↔ audit_guide.py constants. v10.13 added in 24 hrs of v2.7 — high iteration cadence.

Steps that are already pipeline-automated (operator does nothing here):
- Pre-cache assembly (STATIC_CACHE, CACHED_AUDIT, slim REFERENCE).
- Writer + auditor LLM calls.
- Post-processor dash-strip, meta-truncate, paragraph auto-merge.
- Mechanical audit gate (dual-pass for guides).
- AuditKit feedback file generation.
- FAQ schema JSON-LD injection.
- Round 1 → round 2 → round 3 escalation routing.
- macOS notification on completion.
- Vercel auto-deploy on push.
- `schema-check.yml` (Vercel deploy verification) + `weekly-link-audit.yml`.
- Sitemap regeneration on build.
