# SSG Content Spec — Source Material Dump

**Date:** 2026-05-16
**Purpose:** Surface source material for rewriting `CONTENT_SPEC_GUIDE_SSG.md` from asbestos voice to B2B services voice.
**Mode:** Read-only recon. No edits, no recommendations.

---

## §1 — Current `CONTENT_SPEC_GUIDE_SSG.md` (verbatim)

**Path:** `~/projects/ssg-content/content/ssg/CONTENT_SPEC_GUIDE_SSG.md`
**Note:** File is titled `CONTENT_SPEC_GUIDE_SSG.md` but the H1 inside still reads `CONTENT_SPEC_GUIDE_ASBESTOS.md`. Content is byte-identical to §2 below.

```markdown
# CONTENT_SPEC_GUIDE_ASBESTOS.md: Guide Page Writer Spec

**Version: 2.7**
**Status: Active**
**Last updated: 2026-04-24**
**Scope:** All AsbestosHQ.com guide pages (0 live). Current 11-slug batch (April 2026):
- Approved (8): asbestos-shingles-guide, asbestos-siding-guide, black-mastic-guide, friable-vs-nonfriable-asbestos, how-to-test-popcorn-ceiling-for-asbestos, is-popcorn-ceiling-asbestos, transite-pipe-guide, vermiculite-insulation-guide
- Needs-review (3): asbestos-ceiling-tile-guide, asbestos-tile-guide, house-built-1976-asbestos
**Audit script:** scripts/audit_guide.py v3.4 (L3 removed, BANNED_PHRASES_UST dropped, G1 SEMANTIC-VARIETY enforced mechanically from secondary_nouns)

---

## Changelog

- **v2.7 (2026-04-24):** Added `## Voice Guidance` subsection (prose tone rules: Confident / Specific / Honest about tradeoffs / Occasionally informal / Opinionated / Reader first, plus six voice do-nots). Added `### SEO Metadata` subsection under Schema & Deployment (metaTitle ≤60, metaDesc ≤160, primary-keyword placement in h1 + intro + metaDesc). Added generic-heading ban under `### Sections`. Strengthened `### Intro` rule: first 2-3 sentences must directly answer the core question, no throat-clearing or "In this guide we'll cover…" framing. Added `### Tradeoff Coverage` prose rule (non-mechanical, LLM-auditor review only) — every major claim gets a limitation callout, flag-don't-invent. Added `## Editor Pass Discipline` section. Folded stale `audit_guide.py v3.3` header reference to v3.4 to match HEAD. Ported from UST Blog_Voice_And_Style_Guide.md + Blog_Post_Creation_Playbook.md. No mechanical thresholds changed; no auditor script changes required.
- **v2.6 (2026-04-24):** Human-voice constraints C1-C4 canonized in the spec (previously only in run-batch.sh v10.11 prompts). C1 sentence rhythm variation, C2 register shifts, C3 unequal information density, C4 paragraph architecture variation. All four are LLM-auditor SOFT checks; audit_guide.py does not enforce them. Added to target the uniform-AI-cadence pattern observed in the first 11-guide batch. Pairs with run-batch.sh v10.11 writer and auditor prompts.
- **v2.5 (2026-04-24):** G1 placeholder replaced with per-slug secondary_nouns reference. The list now lives in `content/asbestos/keyword-configs/<slug>.json` under the `secondary_nouns` key, populated from actual 3+ frequency terms in each shipped guide. Spec stops carrying a single asbestos-wide example list because the real vocabulary diverges by material (popcorn ceiling vs. transite pipe vs. black mastic vs. vermiculite). No threshold change, no schema change.
- **v2.4 (2026-04-24):** Directory CTA requirement removed. Phase 1 of AsbestosHQ.com ships guide content only; no `/find-asbestos-contractors/` or `/request-a-quote/` pages exist yet, so the L3 mechanical gate and the corresponding writer rule both become dead weight. Closing-paragraph guidance rewritten: guides now end on hire-licensed-firm, verify-state-credentials, or testing-first prose, with no internal link to a directory page. Three E-E-A-T schema fields added (append-only, existing field order unchanged): `author` (string), `lastReviewed` (ISO date), `reviewedBy` (string or null). Writer self-check item 17 removed. §Scope updated with the real 11-slug April batch. Pairs with audit_guide.py v3.3, AUDIT_SPEC v1.6.4, AUDIT_APPENDICES v1.7, run-batch.sh writer/auditor prompt update.
- **v2.3 (2026-04-19):** D4 (exception/limiting sentences) demoted to info-only to match audit_guide.py v3.2. Guide spec never required exception markers; the v3.1 min-1 floor was producing false FAILED_AUDIT states on guides that had no narrative reason for "unless / does not apply / fails" language. Added explicit annotation near G1-G5 that those checks are LLM-auditor-only and cannot be deferred to the mechanical gate (audit_guide.py TOPIC1 checks entity count via required_entities_min, not semantic variety or citation specificity). No threshold changes.
- **v2.2 (2026-04-19):** Relax mechanical thresholds after R1 pass-rate analysis showed v3.0 FAILs concentrated on non-ranking-critical rules. V5 "however" cap 3 to 6, V7 same-first-word streak 2 to 4, V8 sentence word cap 30 to 40, V4c hedge cap 3 to 6, D4 exception markers 3 to 1, L1 link count max 10 to 12. V4d ("not just X, it's Y") and D1 (one-sentence paragraph) become info-only, surfaced in reports but not gating. Paragraph count divisibility by (h2_count+1) moves from writer-enforced to post-processor auto-merge (silent repair). All SEO, structural (S1-S8), TOPIC1, SPEC1, FRESH1, INTENT1, SAT1, L2-L5, V1, V2, V4, V4b, V9, V10 unchanged.
- **v2.1 (2026-04-17):** Makes D1 (one-sentence paragraph minimum) and L3 (conversion-path link requirement) explicit writer rules. Previously these mechanical gate checks had no spec-level backing, so writers consistently missed them and only discovered the failures at the AuditKit gate after the LLM auditor approved. Adds a "Paragraph Rhythm" structural section and two new items to the Writer Self-Check.
- **v2.0 (2026-04-17):** Full rewrite. Adapts to existing production system (blog-posts.json flat schema, ceil-distributed paragraphs, audit_guide.py v3.0). Drops v1.0 requirements that don't match renderer (H4 hierarchy, TOC jump links, FAQ schema as writer-controlled). Word count aligned to proven production range (2,000-2,500). Adds 5 high-signal checks from v1.0 that survived the renderer reality check.
- **v1.0 (2026-04-17):** Initial release. Deprecated same day. Built against assumed renderer capabilities rather than actual production system.

---

## Purpose

Guide pages are long-form national content targeting topic-authority keywords (cost guides, process guides, regulatory explainers, training guides). This spec produces guides that rank competitively in the 300-5,000 monthly-search range while keeping token spend low and pipeline throughput high.

## Voice Guidance

The guide voice is a polished version of a knowledgeable professional explaining something to a smart friend. Not academic. Not salesy. Not casual texting. Somewhere between "experienced practitioner answering your question over coffee" and "well-written trade-magazine article."

### Voice Characteristics

- **Confident.** State facts directly. Don't hedge with "it's important to consider" or "you may want to think about." Say what's true.
- **Specific.** Use real numbers, real ranges, real timelines. Vague statements ("costs can vary significantly") are useless. Give the range.
- **Honest about tradeoffs.** Every recommendation has a downside. Name it. Readers trust writers who acknowledge complexity.
- **Occasionally informal.** A short punchy sentence. A one-word paragraph. A rhetorical question. These are fine and encouraged. But the baseline is polished, not slangy.
- **Opinionated where warranted.** If one option is clearly better for most readers, say so. Don't present three equal options when one is obviously the right call for 80% of readers.
- **Reader-first.** Every sentence should help the reader make a decision or understand something. If a sentence exists only to sound smart or fill space, cut it.

### Voice Do-Nots

- Never sound like a textbook.
- Never sound like a marketing brochure.
- Never sound like an AI summarizing search results.
- Never talk down to the reader. They've already done basic research.
- Never use filler phrases to sound authoritative. Be authoritative by being specific.
- Never summarize what you just said at the end of a section.

Voice Guidance operates at the tone/prose layer above the mechanical banned-phrase list in AUDIT_APPENDICES.md. Banned phrases catch specific strings; Voice Guidance shapes cadence, stance, and reader relationship.

## Schema & Deployment

Guides are JSON objects appended to `src/data/blog-posts.json`. There are NO markdown files, NO MDX, NO HTML files. The Next.js renderer handles all styling and structure.

**Canonical schema**:

```json
{
  "slug": "keyword-rich-slug",
  "metaTitle": "Under 60 chars with target keyword",
  "metaDesc": "Under 160 chars. Include target keyword.",
  "h1": "Full Title With Target Keyword",
  "h2s": ["Section 1", "Section 2", ...],
  "paragraphs": ["Paragraph 1", "Paragraph 2", ...],
  "category": "Environmental Guides",
  "date": "April 2026",
  "authorityLinks": [{"name": "Display Name", "url": "https://..."}],
  "relatedLinks": [{"title": "Related Post", "slug": "related-slug"}],
  "allText": "",
  "author": "The AsbestosHQ.com Editorial Team",
  "lastReviewed": "2026-04-24",
  "reviewedBy": null
}
```

**E-E-A-T fields (v2.4+, append-only):**

- `author` — always the string `"The AsbestosHQ.com Editorial Team"` in phase 1. No individual author names. No fabricated credentials.
- `lastReviewed` — ISO date string (`YYYY-MM-DD`). On initial write, matches `dateModified`. Bump on any substantive factual review.
- `reviewedBy` — `null` in phase 1. Reserved for a future named-reviewer program; do not populate with placeholder names.

### SEO Metadata

- **metaTitle:** ≤60 characters. Primary keyword near the beginning.
- **metaDesc:** ≤160 characters. Starts with the direct answer to the searcher's query. Includes the primary keyword naturally. Written to earn the click, not to stuff keywords.
- **Primary keyword placement:** must appear in h1, within the intro's first 100 words (already G5-enforced), and in metaDesc.
- **Secondary keywords:** surface naturally across h2s and body — not forced into every section, not absent from the back half.

**Renderer distribution formula** (critical):

**Target paragraph count** = N × (h2_count + 1), where N is paragraphs per section.

Example: 7 h2s + 4 paras per section = 32 paragraphs total (4 intro, 4 per section).

Writer MUST compute this math before writing and confirm after. One extra paragraph shifts content under wrong headings.

## Structural Requirements

### Length

- **Writer target: 2,000-2,500 words** (proven production range, all 12 live guides)
- **Acceptable floor: 1,800 words** for niche guides (KD 0-5, volume under 300/mo)
- **Ceiling: 2,800 words** for highest-competition guides only (requires justification)
- **Auditor pass range: 1,700-3,000 words**

### Sections

- **5-8 H2 sections** (not 3, not 15)
- **Each H2 must correspond to a real searchable sub-query** of the main topic
- Writer lists the sub-query each section targets as a draft note before writing
- **Generic H2 titles banned:** "Introduction," "Overview," "Conclusion," "Summary," "Tips," "Key Takeaways," "In Conclusion." Every H2 names a specific user question or topic.
- No "In today's fast-paced world" filler. No "In conclusion" wrap-ups.
- Final section should be action-oriented (e.g., "Your Next Step"), not a summary

### Paragraph Rhythm

- **Recommended** (no longer gated): include at least one single-sentence paragraph somewhere in the guide for reading rhythm. As of v2.2 D1 is info-only; the audit reports the count but does not fail on zero.
- Vary paragraph length: mix 3-5 sentence paragraphs with at least one 1-sentence punch paragraph where it reads naturally
- The punch paragraph can be a verdict, a rule restatement, a numeric takeaway, or a pivot line; it does not have to sit in any specific position
- Avoid every paragraph landing in the same 3-5 sentence band

### Intro

- **100-200 words** (before first h2)
- **First 100 words must contain primary keyword AND a concrete data point** (dollar figure, specific regulation name, time range)
- Answer the reader's core question in the first sentence
- No generic openers or framing: no "In this guide we'll explore…", no "Everyone knows…", no "In today's world…", no scene-setting before the answer. The first 2-3 sentences give the answer; the rest of the guide supports and qualifies it.

### H2/H3 Only

Renderer handles H2 and H3 only. **No H4 or deeper.** Renderer is flat-paragraph. H3 subheads are rendered as bolded text within the paragraph flow, not as separate section markers for the distribution algorithm.

### No TOC

Renderer does not support TOC jump links. **Skip the TOC entirely.** Section structure reveals itself through H2s.

### FAQ Section

- **4-8 FAQ items** (optional but recommended for guides over 2,200 words)
- FAQs go in the final h2 section as prose, not as separate schema-marked items
- FAQ schema markup is NOT writer-controlled in current renderer. Skip.

### Authority Links

- **Minimum 4 distinct .gov authority links** in the `authorityLinks` array
- Every regulatory claim paired with its .gov source in the same paragraph via inline link: `[text](/url/)` for internal, full https:// for external
- Link to permanent pages only (program pages, regulation text, fact sheets)
- Never: press releases, news articles, Wikipedia, vendor pages, PDFs

### Internal Links

- **2+ other guide links** (cluster building, content-to-content only)
- **All internal links use trailing slashes** (site-wide rule)
- Use `[text](/url/)` inline syntax. No footer dump.
- **Total internal link count cap: 12** (mechanical L1 range is 3-12); target 3-6 guide cross-links in phase 1.
- **No directory CTAs in phase 1.** `/find-asbestos-contractors/` and `/request-a-quote/` pages do not exist yet. Do not link to them from guide bodies. Writer spec v2.4 and audit_guide.py v3.3 both drop the L3 requirement.
- Service-page and state-page cross-linking will be re-enabled in phase 2 when those pages ship. Until then, link only to other guides.

### Tradeoff Coverage

Every major claim or recommendation includes honest limitations: situations where it doesn't apply, cost or schedule downsides, who it's the wrong fit for. Guides read like a practitioner giving real advice, not a promotional pitch.

**Flag-don't-invent:** if a specific number, timeline, or limitation isn't known, say "verify with your state environmental agency" or "this varies by site and material condition" — never fabricate a figure to make a paragraph feel concrete. G3 CONCRETE-DENSITY covers real specifics; Tradeoff Coverage covers honest unknowns.

This is a SOFT prose rule. The LLM auditor reviews it during the quality pass but `audit_guide.py` does not enforce it mechanically.

### Closing Paragraph

The final paragraph closes on informational, action-oriented prose tied to the topic. Acceptable patterns:

- **Hire-licensed-firm:** "Hiring a state-licensed asbestos abatement firm with current certification is non-negotiable for removal work. Verify credentials with your state environmental agency before signing a contract."
- **Verify-state-credentials:** "Check the abatement contractor's license against your state asbestos program before any work begins. Most state agencies maintain a public license-status lookup."
- **Testing-first:** "Testing is quick, inexpensive, and the only reliable way to replace guessing with an answer. Pull a sample before planning any renovation, sale, or repair that would disturb the material."

Do NOT close on:
- Any internal link to a directory page (removed in v2.4)
- "Contact us" or "browse our listings" phrasing (no directory yet)
- Generic wrap-ups ("In conclusion," "To summarize," "All in all")

---

## CRITICAL: READ BEFORE DRAFTING

EVERY GUIDE WILL BE AUDITED AGAINST THE CHECKS BELOW. FAILURES TRIGGER REWRITES. REWRITES BURN TOKENS.

MOST COMMON FAILURE MODES:

- SECTIONS DRIFT THIN IN THE BACK HALF (G4 SECTION-BALANCE)
- INTROS OPEN WITH FILLER INSTEAD OF KEYWORD + DATA (G5 INTRO-HOOK)
- REPEATING THE PRIMARY KEYWORD INSTEAD OF USING VARIED TERMINOLOGY (G1 SEMANTIC-VARIETY)
- NAMING "REGULATIONS" INSTEAD OF SPECIFIC STATUTES (G2 REG-DENSITY)
- GENERIC EXAMPLES INSTEAD OF CONCRETE DOLLAR FIGURES OR NAMED STANDARDS (G3 CONCRETE-DENSITY)
- PARAGRAPH COUNT DOESN'T DIVIDE CLEANLY BY (H2 COUNT + 1), CONTENT LANDS UNDER WRONG HEADINGS

IF A SECTION DOESN'T CORRESPOND TO A REAL SEARCH QUERY, CUT IT OR REPLACE IT.

SELF-CHECK ALL G1-G5 BEFORE SAVING. THE AUDIT IS BACKUP, NOT PROOFREADER.

---

## The 5 Guide-Specific Checks (G1-G5)

These extend the standard service-page checks (SAT1, SEO, V, D, L series) that already run via audit_guide.py v3.2. Guide writers must pass both the standard checks AND G1-G5.

> **ENFORCEMENT NOTE (v2.3):** G1, G2, G4, and G5 are enforced by the LLM auditor only. `audit_guide.py` TOPIC1 checks entity COUNT (min 4 via `required_entities_min` in `audit-configs/guide.json`), not semantic variety, citation specificity, section balance, or intro-hook composition. The auditor cannot defer these to the mechanical gate — if the LLM auditor approves a draft that fails G1/G2/G4/G5, the gate will not catch it. G3 is partially backed by SPEC1 (≥3 specifics, audit_guide.py line 219) but G3's definition (dollar/timeframe/standard/scenario) is richer than SPEC1's regex; the LLM auditor remains authoritative on G3 too.

### G1 SEMANTIC-VARIETY

**Rule:** At least 5 distinct secondary nouns each appear 3+ times across the full guide.
**Fail condition:** Fewer than 5 secondary nouns meeting the 3-occurrence threshold.
**Fix:** G1 requires ≥5 distinct secondary nouns, each appearing 3+ times in the guide. The canonical secondary-noun list for each guide lives in keyword-configs/<slug>.json under the `secondary_nouns` key. Writer must select ≥5 from that list and use each ≥3 times. The list is slug-specific because asbestos secondary vocabulary varies by material type (popcorn ceiling ≠ transite pipe ≠ black mastic).

### G2 REG-DENSITY (EEAT)

**Rule:** At least 4 distinct specific regulatory citations appear across the guide.
**Fail condition:** Fewer than 4 citations, or citations are generic ("federal regulations," "state rules") rather than specific.
**Fix:** Valid citations include statute name with year ("e.g., AHERA 1986"), regulation code ("e.g., 40 CFR 763, 29 CFR 1926.1101"), standard name ("e.g., ASTM E2356"), program name ("e.g., NESHAP", "EPA Worker Protection Rule"), or PEL reference ("OSHA PEL 0.1 f/cc"). Generic references count for zero.

### G3 CONCRETE-DENSITY

**Rule:** At least 3 concrete specifics in the guide, from one or more of these forms:
- Specific dollar figures ("$2,000-$4,000")
- Specific timeframes ("180 days under ASTM E1527-21")
- Specific standard/code names ("40 CFR 280")
- Specific example scenarios ("K-12 school with documented floor tile ACM, pre-1981 construction, AHERA management plan required")

**Fail condition:** Fewer than 3 concrete specifics. "Expensive," "takes time," "common" count for zero.
**Fix:** Replace vague claims with numbers, named regulations, or specific scenarios.

### G4 SECTION-BALANCE

**Rule:** Every H2 section is between 40% and 200% of the average section word count.
**Fail condition:** Any section under 40% (thin) or over 200% (bloated) of average.
**Fix:** Redistribute content. Thin sections get cut or merged; bloated sections get split or trimmed. Measured in words, excluding H2 headings themselves.

### G5 INTRO-HOOK

**Rule:** First 100 words contain the primary keyword AND a concrete data point.
**Fail condition:** Primary keyword only appears after word 100, OR no concrete data point in first 100 words, OR opener is generic filler ("In today's world," "Everyone knows").
**Fix:** Lead with a specific fact or number, state the primary keyword naturally within the first 2-3 sentences.

---

## Human-Voice Constraints (C1-C4)

Added in v2.6 to break uniform AI-cadence patterns that emerged across the first 11-guide batch. C-checks are SOFT. The auditor may pass a draft with 1-2 C-failures if overall quality is strong. Systematic failure across 3-4 C-checks triggers revision. C-checks are not enforced by audit_guide.py.

### C1 Sentence Rhythm Variation

**Rule:** Every sliding window of 4 consecutive sentences in body paragraphs must include at least one sentence under 10 words OR one sentence over 35 words. Headers and list items do not count toward the window.
**Fail condition:** Any 4-sentence window has no sentence under 10 words AND no sentence over 35 words.
**Enforcement:** LLM auditor SOFT check. Pipeline prompt: run-batch.sh v10.11 GUIDE AUDITOR.

### C2 Register Shifts

**Rule:** Each H2 section must include at least one register-shift device: (a) a rhetorical question, (b) direct reader address, or (c) a sentence fragment used for emphasis. Max one register-shift device per paragraph.
**Fail condition:** Any H2 section contains zero register-shift devices.
**Enforcement:** LLM auditor SOFT check. Clusters (multiple devices in one paragraph) are flagged INFO, not failed.

### C3 Unequal Information Density

**Rule:** At least 2 paragraphs per guide include a transition or color sentence that does not carry a hard fact. Acceptable non-fact sentences: restate the stakes in different words; acknowledge the reader's likely mental state; or preview what the next paragraph covers. These bridge between fact-dense prose and do not count toward G3 CONCRETE-DENSITY.
**Fail condition:** Fewer than 2 paragraphs contain a qualifying non-fact transition/color sentence.
**Enforcement:** LLM auditor SOFT check.

### C4 Paragraph Architecture Variation

**Rule:** Target distribution across the guide's `paragraphs[]`:
- At least 20% of paragraphs at 1-2 sentences (punchy or transitional)
- At least 20% of paragraphs at 5-6 sentences (deeper explanatory)
- Remaining 60% at 3-4 sentences

Not every paragraph opens with a topic sentence. Some should open with a specific fact, question, or scenario and build to the point.
**Fail condition:** Fewer than 20% of paragraphs are 1-2 sentences, OR fewer than 20% are 5-6 sentences.
**Interaction with V9:** V9 hard-fails paragraphs with 7+ sentences. C4's 5-6 sentence band sits at or below the V9 ceiling — a 6-sentence paragraph is the long-tail maximum.
**Enforcement:** LLM auditor SOFT check.

---

## Inherited Checks (from existing production system)

Guides also pass all service-page checks enforced by audit_guide.py v3.4:

- **SAT1 (saturation):** No phrase/transition repeats above threshold (calibrated +30% for v1 launch)
- **SEO5 (primary keyword occurrences):** 3-12 occurrences across guide
- **L1 (internal link cap):** 3-12 per guide (mechanical range); phase 1 target 3-6 guide-to-guide cross-links
- **L3 (directory CTA):** REMOVED in v3.3 — directory pages do not exist in phase 1
- **D1 (one-sentence paragraph):** Info only in v3.1+. Recommended but not gating.
- **D4 (exception/limiting sentences):** Info only in v3.2+. Reported, not gating. Guide spec does not require exception markers.
- **V4d ("Not just X, it's Y" pattern):** Info only in v3.1+. Reported, not gating.
- **V-series (voice):** Zero banned phrases (see AUDIT_APPENDICES.md for full list including the 14 long-form additions)
- **No dashes:** Zero em dashes, en dashes, or hyphen-as-separator anywhere

---

## Banned Phrases (writer-facing summary)

Standard service-page bans PLUS these long-form additions:

- "In today's fast-paced world"
- "In today's world"
- "It's important to note that"
- "It's worth noting that"
- "In conclusion"
- "At the end of the day"
- "When all is said and done"
- "In this comprehensive guide"
- "We'll dive deep into"
- "Let's explore"
- "Without further ado"
- "Rest assured"
- "Needless to say"
- "It goes without saying"

Full banned phrase list lives in content/AUDIT_APPENDICES.md.

---

## Writer Self-Check (before saving draft)

1. Primary keyword identified and not owned by a service or state page
2. Primary keyword appears 4-10 times, not stuffed
3. At least 5 secondary nouns appear 3+ times each (G1)
4. At least 4 specific regulatory citations (G2)
5. At least 3 concrete dollar figures, timeframes, or named standards (G3)
6. All H2 sections within 40%-200% of average length (G4)
7. First 100 words contain primary keyword + concrete data point (G5)
8. 5-8 H2 sections, each corresponding to a real searchable sub-query
9. Paragraph count = N × (h2_count + 1), distribution math confirmed
10. 2+ guide cross-links (content-to-content), all trailing-slash. No directory CTAs in phase 1.
11. 4+ distinct .gov authority links in authorityLinks array
12. Zero dashes anywhere
13. Zero banned phrases (standard + long-form additions)
14. JSON validates and follows blog-posts.json schema exactly (including `author`, `lastReviewed`, `reviewedBy` as of v2.4)
15. Word count 2,000-2,500 (or justified within 1,800-2,800)
16. Recommended: at least one single-sentence paragraph present somewhere in `paragraphs[]` (D1 is info-only in v3.1; writers should still aim for one for reading rhythm)
17. Closing paragraph follows hire-licensed-firm / verify-state-credentials / testing-first pattern. No internal directory links. No "browse our listings" phrasing.
18. `author` = `"The AsbestosHQ.com Editorial Team"`; `lastReviewed` matches `dateModified` on initial write; `reviewedBy` = `null` in phase 1.

---

## Editor Pass Discipline

This applies to anyone editing a draft after the writer produces it — LLM auditor rewrite passes, operator manual fixes, or future human editors.

When editing a draft — manual pass, auditor-rewrite response, or operator-flagged fix — do not change meaning, do not add new content, do not significantly change length. The editor's job is to tighten sentence rhythm, remove banned phrases, fix cadence, and vary paragraph length — not to rewrite or expand. If a section needs more factual depth or better specifics, escalate back to the writer rather than inventing material.

---

## Failure Escalation

- Round 1 fail: writer rewrites flagged sections
- Round 2 fail: writer rewrites full guide
- Round 3 fail: escalate to operator for manual review, no further automated rewrites

---

## Related Specs

- Audit script: scripts/audit_guide.py v3.4
- Pipeline / writer + auditor prompts: content/run-batch.sh v10.11
- Banned phrases (full): content/AUDIT_APPENDICES.md (v1.7)
- Universal auditor spec: content/AUDIT_SPEC.md (v1.6.4)
```

---

## §2 — Current `CONTENT_SPEC_GUIDE_ASBESTOS.md` (verbatim)

**Path:** `~/projects/asbestos-contractors/content/asbestos/CONTENT_SPEC_GUIDE_ASBESTOS.md`
**Note:** Byte-identical to §1 above. Both files share the same 368-line content. Reproduced once for parity; treat §1 as the canonical text.

```markdown
# CONTENT_SPEC_GUIDE_ASBESTOS.md: Guide Page Writer Spec

**Version: 2.7**
**Status: Active**
**Last updated: 2026-04-24**
**Scope:** All AsbestosHQ.com guide pages (0 live). Current 11-slug batch (April 2026):
- Approved (8): asbestos-shingles-guide, asbestos-siding-guide, black-mastic-guide, friable-vs-nonfriable-asbestos, how-to-test-popcorn-ceiling-for-asbestos, is-popcorn-ceiling-asbestos, transite-pipe-guide, vermiculite-insulation-guide
- Needs-review (3): asbestos-ceiling-tile-guide, asbestos-tile-guide, house-built-1976-asbestos
**Audit script:** scripts/audit_guide.py v3.4 (L3 removed, BANNED_PHRASES_UST dropped, G1 SEMANTIC-VARIETY enforced mechanically from secondary_nouns)

[FULL TEXT IDENTICAL TO §1 — reproduction skipped to keep this doc readable. Verified byte-identical via comparison; both files have 368 lines, same H1 (`CONTENT_SPEC_GUIDE_ASBESTOS.md: Guide Page Writer Spec`), same Voice Guidance, same G1-G5, same C1-C4, same Writer Self-Check, same Editor Pass Discipline. If diff needed, run: `diff ~/projects/ssg-content/content/ssg/CONTENT_SPEC_GUIDE_SSG.md ~/projects/asbestos-contractors/content/asbestos/CONTENT_SPEC_GUIDE_ASBESTOS.md`]
```

---

## §3 — Representative SSG Article (verbatim)

**Path:** `~/projects/smartsourceguide/app/it-support/managed-it-services-pricing/page.tsx`
**Why this one:** Mid-size (129 lines), covers cost/pricing — analog to asbestos cost guides. Demonstrates the target B2B voice: confident numbers, named tradeoffs, direct reader address, takeaway box + FAQ + CTA box pattern.

```tsx
import type { Metadata } from 'next';
import Link from 'next/link';

export const metadata: Metadata = {
  title: 'Managed IT Services Pricing: What Small Businesses Actually Pay (2026)',
  description: 'Managed IT services cost $100-$250 per user per month in 2026. Here\'s how pricing models work, what\'s included, and where providers hide extra charges.',
  openGraph: {
    title: 'Managed IT Services Pricing: What Small Businesses Actually Pay (2026)',
    description: 'Managed IT services cost $100-$250 per user per month in 2026. Here\'s how pricing models work, what\'s included, and where providers hide extra charges.',
    type: 'article',
  },
};

export default function ManagedITServicesPricing() {
  return (
    <>
      <div className="article-header">
        <p className="article-breadcrumb">
          <Link href="/">Home</Link> &rsaquo; IT support
        </p>
        <h1>Managed IT services pricing: what small businesses actually pay in 2026</h1>
        <p className="article-meta">Last updated April 2026</p>
      </div>

      <div className="article-body">

        <div className="takeaway-box">
          <strong>Key takeaway</strong>
          <p>Most small businesses pay $100 to $250 per user per month for fully managed IT services. A 20 person company should expect $2,000 to $5,000 per month. The monthly number on the proposal matters less than what is actually included in it.</p>
        </div>

        <h2>The number you came here for</h2>

        <p>Most small businesses pay $100 to $250 per user per month for fully managed IT services in 2026. A 20 person company should expect to spend $2,000 to $5,000 per month. A 50 person company lands between $5,000 and $12,500.</p>

        <p>That range is wide because "managed IT" can mean wildly different things depending on the provider. Some quotes at $100 per user include monitoring, help desk, and basic security. Others at the same price cover monitoring only, then charge separately for every help desk ticket, every security tool, and every after hours call. The monthly number on the proposal matters less than what is actually included in it.</p>

        <h2>The four pricing models MSPs use</h2>

        <p>Not every managed IT provider prices the same way. Understanding the model tells you more about what you will actually pay than the number on the first page of the quote.</p>

        <p><strong>Per user pricing</strong> is the most common model for small business IT support. You pay a flat rate for each employee who uses technology. This usually covers their workstation, laptop, mobile device, email, help desk access, and security tools. The typical range is $100 to $250 per user per month. Per user pricing is simple to budget and scales predictably as you hire. The downside is that companies with employees who barely touch technology (warehouse workers, field crews) end up paying the same rate for people who use a computer two hours a week as for people who live in spreadsheets and email all day.</p>

        <p><strong>Per device pricing</strong> charges based on the number of endpoints the MSP manages: laptops, desktops, servers, firewalls, switches. Rates typically run $30 to $100 per device per month, with servers at the higher end. This model works for businesses where employee count and device count are very different, like a 15 person company running 40 devices across multiple locations. It can also get expensive quickly if your environment is device heavy.</p>

        <p><strong>Tiered or bundled pricing</strong> packages services into levels. A basic tier might cover monitoring and help desk for $75 per user. A mid tier adds security, backup, and patch management for $150. A premium tier includes everything plus virtual CIO consulting and after hours support for $250. Tiered pricing gives you control over what you pay for, but it also creates upsell pressure. The basic tier often excludes things most businesses need (like security monitoring), which means the real price is the mid tier, not the number in the headline.</p>

        <p><strong>All inclusive flat rate pricing</strong> wraps everything into one monthly fee regardless of tickets, incidents, or hours consumed. This is the cleanest model from a budgeting perspective. You know exactly what you pay every month. The trade off is that all inclusive contracts tend to price higher because the MSP is absorbing the risk of high ticket volume months. For businesses that generate a lot of support requests, this model usually saves money over time. For businesses with few issues, you are overpaying for peace of mind.</p>

        <h2>What should be included at every price point</h2>

        <p>The line between "included" and "extra" is where managed IT pricing gets deceptive. Some MSPs advertise $100 per user and then charge separately for items that should be standard at that price.</p>

        <p>At $100 to $150 per user, you should expect: 24/7 network and endpoint monitoring, help desk support during business hours, patch management and software updates, basic antivirus and endpoint protection, backup management and verification, and monthly reporting. If a provider at this price point charges extra for patching or backup monitoring, their real price is higher than $100 and they are counting on you not noticing until the first invoice.</p>

        <p>At $150 to $200 per user, you should also get: advanced cybersecurity tools (EDR, email filtering, DNS protection), after hours <Link href="/it-support/outsource-help-desk-guide/">help desk</Link> support, vendor management (the MSP calls your internet provider or software vendor on your behalf), and some level of technology planning or quarterly review meetings.</p>

        <p>Above $200 per user, the package should include everything listed above plus: virtual CIO or strategic technology advisory, compliance support for regulated industries (HIPAA, PCI, SOC 2), on site support visits as needed, and priority response SLAs with guaranteed resolution times.</p>

        <p>If you are getting a quote above $200 per user and the proposal does not include strategic planning, you are paying premium prices for standard service.</p>

        <h2>Where the extra charges hide</h2>

        <p>Every MSP has a scope of work document. The things outside that scope are where surprise invoices come from. Knowing what to ask about before signing saves you from the "I thought that was included" conversation three months in.</p>

        <p><strong>Projects vs support</strong> is the most common boundary. Your monthly fee covers day to day support: fixing things that break, monitoring, maintenance. It does not cover projects: server migrations, office moves, new software deployments, network redesigns. Projects are billed separately, usually at $150 to $250 per hour. This distinction is reasonable. What is not reasonable is when routine work gets reclassified as a "project" to generate additional billing. Ask the provider to define exactly where they draw the line, and get examples of what they have classified as a project for similar clients.</p>

        <p><strong>After hours and weekend support</strong> is sometimes included, sometimes billed at 1.5x the normal rate. If your business operates outside traditional 9 to 5 hours or has any customer facing systems that need to stay up overnight, confirm this before signing.</p>

        <p><strong>New employee onboarding and offboarding</strong> can be included or billed at $100 to $300 per event. For a growing company hiring 10 people a year, that is an extra $1,000 to $3,000 annually that did not appear in the monthly quote.</p>

        <p><strong>Hardware procurement</strong> is usually handled at cost plus a markup of 5 to 15 percent, or the MSP may require you to purchase through their preferred vendors. This is not inherently bad (they can often get volume discounts you cannot), but it removes your ability to shop around. Ask if you can source your own hardware and still receive full support.</p>

        <h2>The hidden cost nobody talks about</h2>

        <p>The biggest managed IT expense is not on the invoice. It is the switching cost.</p>

        <p>Once an MSP has been managing your environment for six months, they know your network, your users, your quirks, your vendor relationships. Leaving means a new provider has to learn all of that from scratch, and you pay for their onboarding process while they do. Some MSPs make this worse by using proprietary tools or keeping documentation locked in their own systems. If you leave, your network documentation leaves with them.</p>

        <p>Before signing with any provider, ask two questions. First: will you provide us with complete, current documentation of our environment at any point we request it? Second: if we terminate the contract, what is the transition process and what do we receive?</p>

        <p>The answer to those questions tells you whether you are entering a partnership or a trap.</p>

        <h2>How managed IT pricing compares to the alternatives</h2>

        <p>The monthly cost of managed IT services feels significant until you compare it to the realistic alternatives.</p>

        <p>A single full time IT generalist in the US costs $11,500 to $15,600 per month fully loaded. That one person cannot cover 24/7 monitoring, deep cybersecurity, strategic planning, and daily help desk support simultaneously. They also take vacations, get sick, and eventually leave, at which point you start over with a new hire who does not know your environment.</p>

        <p><Link href="/it-support/managed-it-vs-break-fix/">Break-fix IT support</Link> runs $100 to $149 per hour with no monitoring, no prevention, and no one watching your systems between calls. A single serious incident can cost more than six months of managed services fees.</p>

        <p>The internal hire makes sense when you pass roughly 75 employees, at which point you need someone who lives inside the business full time and can be supplemented by an MSP for security and after hours coverage. Below that threshold, managed IT is almost always the more cost effective model.</p>

        <h2>How to read an MSP quote without getting burned</h2>

        <p>When a proposal lands in your inbox, skip the cover page and go straight to the scope of work. Read every exclusion. Then ask the provider to walk you through three scenarios: a normal month with typical support requests, a bad month with a server failure and a security incident, and an employee onboarding month where you hire five people at once. Ask what your total invoice would be in each scenario.</p>

        <p>If the answer to all three is your monthly rate, you have an all inclusive contract. If the answer to the bad month scenario is "that depends," ask depends on what. The specifics of that answer will tell you more about your true cost than anything else in the proposal.</p>

        <p>Get outsourced IT support pricing comparisons from at least three providers before committing, and always compare total annual cost including estimated project work, not just the monthly per user rate.</p>

        <div className="faq-section">
          <h2>FAQ</h2>

          <div className="faq-item">
            <h3>What is the average cost of managed IT services for a small business?</h3>
            <p>Most small businesses pay $100 to $250 per user per month for fully managed IT services in 2026. The exact cost depends on what is included: basic monitoring and help desk sits at the lower end, while comprehensive packages with cybersecurity, compliance support, and strategic planning run toward the upper end. A 25 person company should budget $2,500 to $6,250 per month.</p>
          </div>

          <div className="faq-item">
            <h3>Is managed IT cheaper than hiring an IT person?</h3>
            <p>For companies under 75 employees, managed IT is almost always less expensive. A full time IT hire costs $138,000 to $187,000 per year including salary, benefits, and overhead. A managed IT contract for 25 users runs $30,000 to $75,000 per year and gives you a team instead of a single person. The internal hire starts making sense when your organization is large enough to need someone embedded full time who is supplemented by an MSP for specialized work.</p>
          </div>

          <div className="faq-item">
            <h3>What is not included in managed IT services?</h3>
            <p>Most managed IT contracts exclude project work (server migrations, office relocations, new software deployments), hardware purchases, and sometimes after hours support. These items are billed separately, typically at $150 to $250 per hour for project work. Always read the scope of work document and ask the provider to identify every category of work that falls outside the monthly fee.</p>
          </div>
        </div>

        <div className="cta-box">
          <h3>Comparing outsourced IT options?</h3>
          <p>See our guide on the <Link href="/it-support/managed-it-vs-break-fix/">best outsourced IT support</Link> providers for small business to compare managed services, help desk, and hybrid models.</p>
        </div>

      </div>
    </>
  );
}
```

---

## §4 — Representative Approved Asbestos Guide (verbatim)

**Path:** `~/projects/_asbestos-reference/approved-guides/asbestos-inspection-cost.json`
**Why this one:** Mid-size approved guide (15,197 bytes — sits at the median of 30 approved guides). A cost guide, so directly comparable to the SSG article in §3. Demonstrates what the current asbestos spec produces.

```json
{
  "slug": "asbestos-inspection-cost",
  "metaTitle": "Asbestos Inspection Cost: 2026 Pricing and Scope Guide",
  "metaDesc": "Asbestos inspection cost runs $231 to $776 nationally with a $483 average. Per-sample fees, AHERA inspector credentials, and PLM lab analysis explained.",
  "h1": "Asbestos Inspection Cost: A 2026 Pricing and Scope Guide",
  "h2s": [
    "What Does an Asbestos Inspection Cost in 2026?",
    "How Much Does Asbestos Inspection Cost Per Sample?",
    "Whole Home vs Pre Renovation Survey Pricing",
    "What Drives Asbestos Inspection Cost in Your Market",
    "What Does a Certified AHERA Inspector Actually Do?",
    "PLM Lab Analysis, NVLAP Accreditation, and Report Turnaround",
    "How to Hire a Licensed Asbestos Inspector"
  ],
  "paragraphs": [
    "What an asbestos inspection costs in 2026: $231 to $776 nationally, $483 average. Per-sample fees fall in the $75 to $150 range, while pre-renovation surveys run $600 to $1,500 and whole-home inspections push $1,200 to $2,000 or more. Cost depends on sample count, accessibility, lab turnaround, and whether the inspector holds AHERA accreditation under 40 CFR 763.",
    "What the inspector does on site, including walkthrough, homogeneous area mapping, bulk sample collection, PLM lab submission, and a written report, drives every line item on the bill. Most homeowners discover the cost question during a renovation, after a contractor flags a popcorn ceiling or a vermiculite attic. The number that comes back depends less on the building and more on how an inspector scopes the job.",
    "Federal AHERA rules under 40 CFR 763 set the accreditation floor for any inspector working in a K-12 school, and most states extend the same training requirement to residential work. OSHA 29 CFR 1926.1101 governs worker protection during sample collection. Inspections that follow ASTM E2356 use a structured homogeneous area sampling plan, which drives sample count and therefore lab cost. State surcharges in California, New York, and New Jersey add 15 to 25 percent on top of the national average.",
    "This guide breaks the pricing apart by line item. Per-test pricing, pre-renovation survey scope, whole-home survey scope, AHERA inspector credentials, NVLAP-accredited PLM lab analysis, and report turnaround each carry their own number. Knowing what each step costs lets you compare three written quotes without getting lost in scoping confusion.",
    "Professional asbestos inspection cost in 2026 runs $231 to $776 nationally per Angi 2026 contractor data, with a $483 weighted average across regions. iBuyer 2026 numbers track close behind at $250 to $850, and HomeAdvisor's residential dataset places the median right at $483. The spread reflects scope variation, not chaotic pricing. A homeowner ordering three samples for one bathroom remodel sits at the bottom. A pre-purchase whole-home survey on a 1976 colonial sits at the top.",
    "Cheap inspections almost always mean too few samples.",
    "An inspector who quotes $150 for one popcorn ceiling sample is delivering a single data point, not a survey. EPA AHERA sampling guidance for popcorn ceilings calls for three samples per homogeneous area, the minimum needed to clear or condemn the material under regulatory scrutiny. A real $483 asbestos inspection covers site time, three to seven samples, PLM lab analysis at a NVLAP-accredited lab, and a written inspection report listing each sample location, lab result, and management recommendation.",
    "What if you only want one sample tested? Most labs sell direct mail-in kits at $25 to $60 per sample with no inspector required, useful when you have one suspect material and no plan to disturb anything else. The trade-off is no walkthrough, no scope review, and no written report tied to a credentialed inspector signature, all of which matter at sale or permit time. Our [how to test popcorn ceiling for asbestos](/how-to-test-popcorn-ceiling-for-asbestos/) guide covers the DIY mail-in path in detail.",
    "Per-sample asbestos inspection cost runs $75 to $150 in residential markets and $50 to $100 in commercial markets where volume drives lower unit pricing. The per-test number bundles bulk sample collection by the inspector, lab handling, PLM analysis under EPA Method 600/R-93/116, and reporting. Some firms quote a flat trip fee of $200 to $400 plus a per-test charge, which can cost less than an all-in survey on a small scope.",
    "Sample count is the top variable. A bathroom remodel with one popcorn ceiling and one floor tile material needs six samples, three per homogeneous area. A whole-home pre-purchase survey on a pre-1980 home commonly pulls 15 to 30 samples covering ceiling textures, floor tiles, mastic, pipe wrap, vermiculite insulation, and plaster. Commercial pre-renovation survey work on a 50,000 square foot office can require 60 to 120 samples or more. Match samples to scope.",
    "AHERA defines a homogeneous area as material with the same color, texture, and date of installation. Each one needs its own three sample minimum to clear or condemn under PLM analysis. A finished basement with two ceiling phases, two floor tile patterns, and one stair tread material is five homogeneous areas. That's 15 samples at minimum. An inspector who quotes a sample count without a walkthrough is guessing at the scope.",
    "Two survey scopes, two different prices. A pre-renovation survey targets only materials that will be disturbed by the planned renovation, which keeps sample count and turnaround tight. Cost typically lands between $600 and $1,500 for a residential scope and between $1,500 and $5,000 for a commercial demolition scope under NESHAP 40 CFR 61 Subpart M. Whole-home surveys cover everything accessible. Cost runs $1,200 to $2,000 or more for a typical residential building, climbing for larger or older homes. Different scope, different number.",
    "The product is a baseline document that survives the building's ownership and any future renovation. Real estate buyers, lenders, and listing agents commonly request whole-home surveys on pre-1980 properties before a transaction closes. Asbestos doesn't move once it's installed, so a clean whole-home survey at year zero saves the cost of a fresh pre-renovation survey at year five when the kitchen gets remodeled.",
    "Demolition surveys are the most expensive of the three. Federal NESHAP requires the building owner to identify all regulated asbestos containing material in any commercial or institutional structure before demolition or major renovation. Cost runs $0.10 to $0.30 per square foot for commercial buildings, putting a 100,000 square foot warehouse demolition survey between $10,000 and $30,000. The scope is exhaustive because an inspector signs off on the absence of ACM in any area not pulled as a sample. Buy the survey scope your project actually needs, not the biggest one available.",
    "Five factors move an asbestos inspection quote: sample count, accessibility, geographic market, rush turnaround, and inspector credential level. Sample count is the largest single mover because each sample carries a $40 to $60 lab fee on top of the on-site collection rate. Accessibility hits next, because crawlspace, attic, and finished ceiling cavity work add labor time.",
    "Geographic market sets the floor. NYC, San Francisco, Boston, and Los Angeles run 15 to 25 percent above the national average on asbestos inspection cost. Rural markets in the Midwest and South often sit 10 to 20 percent below. State NESHAP filing fees, license bonding requirements, and the local supply of AHERA accredited inspectors all push the local rate up or down.",
    "Rush turnaround is the most negotiable line item. Standard PLM lab turnaround runs 3 to 5 business days. A 24 hour rush adds 50 to 100 percent to the lab portion of the bill, and same day priority can double the standard rate. Plan around standard turnaround when the renovation timeline allows, because the rush surcharge buys nothing the regulator cares about.",
    "Inspector credential level is the variable most homeowners overlook. An AHERA inspector under EPA Worker Protection Rule 40 CFR 763 Subpart E carries three day building inspector training plus annual refresher hours, and the rate reflects the credential. A handyman with a single weekend training certificate is cheaper but cannot legally sign off on a NESHAP demolition survey or a pre-1981 school AHERA management plan.",
    "The cheapest inspection in town is often delivered by someone whose report won't satisfy the building department, the state air agency, or the buyer's lender. Verify the credential before booking. Our [friable vs nonfriable asbestos](/friable-vs-nonfriable-asbestos/) guide explains why credential level matters more on friable material than non-friable.",
    "An AHERA inspector arrives with a clipboard, a sampling kit, and a calibrated approach to identifying suspect materials. The walkthrough starts with construction era cues. A 1976 home with original textured ceiling, vinyl floor tile, and pipe wrap insulation gets a different sample plan than a 2005 home with one suspect attic vermiculite layer. The AHERA inspector flags every homogeneous area, photographs each location, and logs it on a site map before pulling any samples.",
    "Sample collection follows EPA AHERA protocol. The inspector wets the suspect material with an amended water solution, cuts a small chunk into a labeled bag, decontaminates the tool, and seals each bag with a chain of custody label. A typical residential walkthrough collects samples across 5 to 12 homogeneous areas. The whole site visit usually takes 60 to 120 minutes for a single family home.",
    "Curious what a 90 minute walkthrough actually feels like? Calmer than most homeowners expect, with the inspector working room by room without disturbing the materials being sampled.",
    "Inspectors sample and scope, not remove. Removal is the abatement contractor's role under separate licensing, separate insurance, and separate NESHAP notifications. A reputable inspector refers out, which is the right answer because conflict of interest rules in many states bar the same firm from inspecting and abating the same building. Independent inspectors who accept no abatement work in the same household are the gold standard credential for unbiased reporting. Confirm before booking.",
    "PLM analysis is the workhorse method for bulk sample identification at NVLAP accredited labs, costing $25 to $50 per sample. Any lab not on the NVLAP roster is a red flag because state agencies and most building departments will not accept results from unaccredited labs.",
    "PLM analysis detects chrysotile, amosite, and crocidolite in bulk samples at the 1 percent threshold. EPA Method 600/R-93/116 is the governing PLM standard. Below 1 percent, the sample reads PLM negative even when low concentrations of chrysotile are present. TEM is the upgrade method, costing $100 to $300 a sample, used when the regulatory standard requires sub 1 percent sensitivity. Schools under AHERA, vermiculite under the EPA Libby protocol, and post abatement clearance air samples typically rely on TEM.",
    "Which method fits your project? Standard turnaround is 3 to 5 business days for PLM at the typical NVLAP lab. Same day priority runs $90 to $150 a sample and is rarely worth the markup.",
    "A 24 hour rush sits between. PCM (Phase Contrast Microscopy) is the air sample method under NIOSH 7400, used during and after abatement, and runs $25 to $40 a sample at standard turnaround. Each method has its own scope, so lab choice should match the regulatory question, not the price tag.",
    "Reports end the inspection. A complete inspection report includes a site map, a sample log, photographs, lab certificates with the NVLAP accreditation number, and a management recommendation per homogeneous area. The recommendation falls into removal, encapsulation, enclosure, or operations and maintenance. A report that ends at lab results without a written recommendation is incomplete. Ask for a sample report before booking.",
    "Verify the AHERA inspector's credential against your state asbestos program before scheduling the walkthrough. Most states publish a public license status lookup with current expiration dates, training records, and any disciplinary history. The credential search takes five minutes and surfaces the cheapest red flag in the industry: an expired or suspended license. License first.",
    "Get three written quotes. Quotes should itemize the per-test fee, the lab turnaround tier, the trip fee, and the inspection report deliverable. A flat all in number with no line items is a red flag at every price point. Ask whether the inspector subcontracts the lab to a NVLAP accredited facility, then request the lab's accreditation certificate.",
    "How do you know the inspector is truly independent? Independent inspectors avoid the most common pricing pressure conflict in the industry. A firm that inspects, finds, and bids the abatement work has a financial interest in expanding the scope. Our [asbestos air quality test](/asbestos-air-quality-test/) and [house built 1976 asbestos](/house-built-1976-asbestos/) guides cover related pre-renovation testing.",
    "Independence beats convenience on inspection booking. A bulk sample run at a NVLAP accredited lab costs less than an hour of contractor labor and returns a written number instead of a guess. Schedule sampling before signing a renovation contract or accepting a real estate disclosure that assumes material safety without proof."
  ],
  "category": "Environmental Guides",
  "date": "April 2026",
  "dateModified": "2026-04-26",
  "authorityLinks": [
    {
      "name": "EPA AHERA (Asbestos Hazard Emergency Response Act, 40 CFR 763)",
      "url": "https://www.epa.gov/asbestos/asbestos-hazard-emergency-response-act-ahera"
    },
    {
      "name": "NVLAP Lab Accreditation (NIST, National Voluntary Laboratory Accreditation Program)",
      "url": "https://www.nist.gov/nvlap"
    },
    {
      "name": "OSHA Asbestos Standard for Construction (29 CFR 1926.1101)",
      "url": "https://www.osha.gov/laws-regs/regulations/standardnumber/1926/1926.1101"
    },
    {
      "name": "EPA Asbestos in Your Home (Homeowner Guidance)",
      "url": "https://www.epa.gov/asbestos/protect-your-family-exposures-asbestos"
    },
    {
      "name": "EPA NESHAP Asbestos (40 CFR 61 Subpart M)",
      "url": "https://www.epa.gov/stationary-sources-air-pollution/asbestos-neshap"
    }
  ],
  "relatedLinks": [
    {
      "title": "How to Test Popcorn Ceiling for Asbestos",
      "slug": "how-to-test-popcorn-ceiling-for-asbestos"
    },
    {
      "title": "Asbestos Air Quality Test",
      "slug": "asbestos-air-quality-test"
    },
    {
      "title": "Friable vs Nonfriable Asbestos",
      "slug": "friable-vs-nonfriable-asbestos"
    },
    {
      "title": "House Built 1976 Asbestos Risk",
      "slug": "house-built-1976-asbestos"
    }
  ],
  "allText": "",
  "author": "The AsbestosHQ.com Editorial Team",
  "lastReviewed": "2026-04-26",
  "reviewedBy": null
}
```

---

## §5 — SSG Self-Description

**Sources searched in `~/projects/smartsourceguide/`:**
- ✓ `app/page.tsx` (home, 54 lines) — present
- ✓ `app/about/page.tsx` (30 lines) — present
- ✓ `app/layout.tsx` (37 lines) — present
- ✓ `components/Header.tsx` — present
- ✓ `components/Footer.tsx` — present
- ✗ README — **not present** in repo root
- ✗ CLAUDE.md — **not present** in repo root

### 5.1 Home (`app/page.tsx`)

```tsx
import type { Metadata } from 'next';
import Link from 'next/link';

export const metadata: Metadata = {
  title: 'SmartSourceGuide — Independent Reviews of Outsourced Business Services',
  description: 'Unbiased comparisons, real pricing data, and honest reviews of outsourced IT support, answering services, and fleet tracking for small and mid-size businesses.',
  openGraph: {
    title: 'SmartSourceGuide — Independent Reviews of Outsourced Business Services',
    description: 'Unbiased comparisons, real pricing data, and honest reviews of outsourced IT support, answering services, and fleet tracking for small and mid-size businesses.',
    type: 'website',
  },
};

export default function Home() {
  return (
    <>
      <section className="hero">
        <p className="hero-label animate-fade-up animate-fade-up-1">Independent reviews &amp; comparisons</p>
        <h1 className="animate-fade-up animate-fade-up-2">
          Find the right business services<br />
          <span className="gradient-text">without the sales pitch</span>
        </h1>
        <p className="animate-fade-up animate-fade-up-3">Unbiased comparisons, real pricing data, and honest reviews of outsourced business services for small and mid-size companies.</p>
      </section>

      <div className="section-divider"><hr /></div>

      <section className="articles-section">
        <p className="articles-section-label animate-fade-up animate-fade-up-3">Popular guides</p>
        <div className="articles-grid">

          <Link href="/it-support/managed-it-vs-break-fix/" className="article-card animate-fade-up animate-fade-up-3">
            <p className="article-card-category">IT support</p>
            <h3>Managed IT services vs break-fix: which actually costs less?</h3>
            <p>The real math behind monthly contracts versus paying per incident.</p>
          </Link>

          <Link href="/it-support/managed-it-services-pricing/" className="article-card animate-fade-up animate-fade-up-4">
            <p className="article-card-category">Pricing</p>
            <h3>Managed IT services pricing: what small businesses actually pay (2026)</h3>
            <p>How pricing models work, what should be included at each tier, and where providers hide extra charges.</p>
          </Link>

          <Link href="/answering-services/best-answering-services/" className="article-card animate-fade-up animate-fade-up-5">
            <p className="article-card-category">Answering services</p>
            <h3>Best answering services for small business compared</h3>
            <p>Live vs virtual vs AI — pricing, features, and who each type works best for.</p>
          </Link>

        </div>
      </section>
    </>
  );
}
```

### 5.2 About (`app/about/page.tsx`)

```tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'About SmartSourceGuide',
  description: 'SmartSourceGuide provides independent, unbiased reviews and comparisons of outsourced business services. No sponsored rankings. No sales pitches.',
};

export default function About() {
  return (
    <div className="about-content">
      <h1>About SmartSourceGuide</h1>

      <p>SmartSourceGuide provides independent, research-driven comparisons of outsourced business services. We help small and mid-size companies find the right providers without the sales pitch.</p>

      <p>Every guide on this site is based on real pricing data, actual service evaluations, and honest analysis. We don&apos;t accept payment for rankings and we don&apos;t let providers influence our recommendations.</p>

      <h2>How we make money</h2>

      <p>Some links on this site are affiliate links, which means we may earn a commission if you sign up with a provider through our link. This never affects our rankings or recommendations — we recommend the best option regardless of whether we have an affiliate relationship with them.</p>

      <h2>Our process</h2>

      <p>For every category we cover, we research pricing from multiple providers, compare features and contract terms, read customer reviews across multiple platforms, and verify claims directly with providers when possible. We update our guides regularly to keep pricing and recommendations current.</p>

      <h2>Contact</h2>

      <p>Questions, corrections, or suggestions? Email us at <a href="mailto:contact@smartsourceguide.com">contact@smartsourceguide.com</a>.</p>
    </div>
  );
}
```

### 5.3 Layout (`app/layout.tsx`)

```tsx
import type { Metadata } from 'next';
import Header from '@/components/Header';
import Footer from '@/components/Footer';
import '@/styles/globals.css';

export const metadata: Metadata = {
  title: {
    default: 'SmartSourceGuide — Independent Business Service Reviews & Comparisons',
    template: '%s | SmartSourceGuide',
  },
  description: 'Unbiased comparisons, real pricing data, and honest reviews of outsourced business services for small and mid-size companies.',
  metadataBase: new URL('https://www.smartsourceguide.com'),
  openGraph: {
    type: 'website',
    locale: 'en_US',
    siteName: 'SmartSourceGuide',
  },
  robots: {
    index: true,
    follow: true,
  },
  verification: {
    google: 'N9un8xrRDLK5pBQ6Y4TlW-kMmHhvEw6mCGdXOkogi-I',
  },
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Header />
        <main>{children}</main>
        <Footer />
      </body>
    </html>
  );
}
```

### 5.4 Header (`components/Header.tsx`)

```tsx
import Link from 'next/link';

export default function Header() {
  return (
    <header className="site-header">
      <div className="header-inner">
        <Link href="/" className="site-logo">SmartSourceGuide</Link>
        <nav>
          <ul className="site-nav">
            <li><Link href="/it-support/managed-it-vs-break-fix/">IT support</Link></li>
            <li><Link href="/answering-services/best-answering-services/">Answering services</Link></li>
            <li><Link href="/about/">About</Link></li>
          </ul>
        </nav>
      </div>
    </header>
  );
}
```

### 5.5 Footer (`components/Footer.tsx`)

```tsx
import Link from 'next/link';

export default function Footer() {
  return (
    <footer className="site-footer">
      <div className="footer-inner">
        <p className="footer-copy">SmartSourceGuide.com &mdash; Independent business service reviews</p>
        <ul className="footer-links">
          <li><Link href="/about/">About</Link></li>
          <li><a href="mailto:contact@smartsourceguide.com">Contact</a></li>
        </ul>
      </div>
    </footer>
  );
}
```

### 5.6 SSG self-description summary (extracted verbatim)

Strings describing SSG's purpose, audience, and angle across the files above:

- **Tagline:** "Independent reviews & comparisons"
- **Hero H1:** "Find the right business services / without the sales pitch"
- **Hero body:** "Unbiased comparisons, real pricing data, and honest reviews of outsourced business services for small and mid-size companies."
- **Metadata title (root):** "SmartSourceGuide — Independent Business Service Reviews & Comparisons"
- **Metadata title (home):** "SmartSourceGuide — Independent Reviews of Outsourced Business Services"
- **Metadata description:** "Unbiased comparisons, real pricing data, and honest reviews of outsourced IT support, answering services, and fleet tracking for small and mid-size businesses."
- **About H1:** "About SmartSourceGuide"
- **About §1:** "SmartSourceGuide provides independent, research-driven comparisons of outsourced business services. We help small and mid-size companies find the right providers without the sales pitch."
- **About §2:** "Every guide on this site is based on real pricing data, actual service evaluations, and honest analysis. We don't accept payment for rankings and we don't let providers influence our recommendations."
- **About §3 (monetization):** Affiliate commissions, "This never affects our rankings or recommendations — we recommend the best option regardless of whether we have an affiliate relationship with them."
- **About §4 (process):** "research pricing from multiple providers, compare features and contract terms, read customer reviews across multiple platforms, and verify claims directly with providers when possible. We update our guides regularly to keep pricing and recommendations current."
- **Footer copy:** "SmartSourceGuide.com — Independent business service reviews"
- **Verticals shown in nav/home:** IT support, Answering services, Fleet tracking, About
- **Article H3 voice samples (from home cards):**
  - "Managed IT services vs break-fix: which actually costs less?" / "The real math behind monthly contracts versus paying per incident."
  - "Managed IT services pricing: what small businesses actually pay (2026)" / "How pricing models work, what should be included at each tier, and where providers hide extra charges."
  - "Best answering services for small business compared" / "Live vs virtual vs AI — pricing, features, and who each type works best for."

---

## §6 — AsbestosHQ persona strings from `run-batch.sh` (verbatim with context)

**Path:** `~/projects/ssg-content/content/run-batch.sh` (v10.12 per file header; 1329 lines total)
**Note:** The prompt called out 8 line numbers — 410, 444, 474, 569, 595, 847, 881, 972. Mapping below. Line 595 is the round 2+ revise prompt; it's not headed by `claude -p "You are the WRITER/AUDITOR..."` but is the eighth `claude -p` invocation in this block. All eight are reproduced with full surrounding context per the prompt request.

### 6.1 Line 410 — SERVICE WRITER SKELETON MODE (`claude -p`)

```bash
        if [ $round -eq 1 ]; then
            if [ "$SERVICE" = true ]; then
                local skeleton_path="${CACHE_DIR}/skeleton-${page}.json"
                if [ -f "$skeleton_path" ]; then
                    # SKELETON MODE: Pre-filled JSON, writer only fills content
                    claude -p "You are the WRITER for AsbestosHQ.com. You have a pre-filled JSON skeleton and reference specs.

1. Read specs: ${assembled}
2. Read skeleton: ${skeleton_path}

Here is a pre-filled JSON skeleton. ALL fields are final except contentParagraphs, faqs, and subtitle.
Do NOT modify slug, metaTitle, metaDesc, h1, contentHeading, stateAgency, or statePageSlug.

YOUR TASK — fill in these 3 fields only:

contentParagraphs: Write 4 unique state-specific paragraphs. Each paragraph must be 4-6 sentences.
- Replace the CROSS_LINK_PHASE1 placeholder with a natural inline link to /phase-1-environmental-site-assessment-cost/ using varied anchor text. Do NOT use 'site assessment' as anchor text. Vary it: Phase I ESA, environmental evaluation, property assessment, contamination assessment, ESA process, etc.
- Replace the CROSS_LINK_HAZWOPER placeholder with a natural inline link to /hazwoper-training-guide/ using varied anchor text. Do NOT use 'state certification' as anchor text. Vary it: HAZWOPER training, safety certification, hazardous materials training, worker safety certification, etc.
- Each cross-link goes in a SEPARATE paragraph. Max one link per paragraph.
- Links use markdown syntax: [anchor text](/url/)

faqs: Write 5 FAQ Q&A pairs relevant to this service type in this state.
- Each answer 4-6 sentences.

subtitle: Write a subtitle with 3-4 relevant city names and service keywords for this state.

QUALITY REQUIREMENTS:
- 400-600 words across contentParagraphs (under 400 = instant fail)
- Zero em dashes or en dashes anywhere
- No embedded newline characters in any string value
- First sentence of contentParagraphs[0] under 15 words, takes a position, contains primary keyword
- At least 3 real cities named in contentParagraphs[1]
- Specific dollar ranges in contentParagraphs[2]
- contentParagraphs[3] ends with actionable advice, not a summary

Save the complete JSON object (skeleton fields + your filled content) to: ${DRAFTS}/${page}.json
Do not explain. Just write the page." $WRITER_MODEL --dangerously-skip-permissions
```

### 6.2 Line 444 — SERVICE WRITER FULL GENERATION MODE (`claude -p`)

```bash
                else
                    # FULL GENERATION MODE: No skeleton available
                    claude -p "You are the WRITER for AsbestosHQ.com. Read this file, then write the assigned service page for ${page}.

1. ${assembled}

Output a single valid JSON object per the CONTENT SPEC. Save to: ${DRAFTS}/${page}.json

YOUR DRAFT WILL BE AUDITED. GET THESE RIGHT THE FIRST TIME:
- statePageSlug must match the assignment sheet's statePageSlug value EXACTLY. Do not guess. Copy it from the assignment. This is the #1 mechanical error in service pages.
- 400-600 words across contentParagraphs (under 400 = instant fail)
- Exactly 4 contentParagraphs
- Exactly 5 FAQs with question and answer fields
- Each FAQ answer 4-6 sentences
- Zero em dashes or en dashes anywhere
- metaTitle under 65 chars, metaDesc under 165 chars
- No embedded newline characters in any string value
- stateAgency must match the assignment sheet exactly (use null if specified)
- slug must match pattern [serviceType]-[stateSlug]
- serviceType must be exactly as specified in assignment

Before saving, also verify:
1. Valid JSON with no trailing commas
2. First sentence of contentParagraphs[0] under 15 words, takes a position, contains primary keyword
3. At least 3 real cities named in contentParagraphs[1]
4. Specific dollar ranges in contentParagraphs[2]
5. contentParagraphs[3] ends with actionable advice, not a summary

Do not explain. Just write the page." $WRITER_MODEL --dangerously-skip-permissions
                fi
            elif [ "$GUIDE" = true ]; then
```

### 6.3 Line 474 — GUIDE WRITER (v10.12, `claude -p`)

```bash
            elif [ "$GUIDE" = true ]; then
                # GUIDE WRITER PROMPT (v10.12: paragraph[0] hard-shape constraint added; v2.4 schema + C1-C4 human-voice constraints)
                claude -p "You are the WRITER for AsbestosHQ.com. Read this file, then write the assigned GUIDE page for ${page}.

1. ${assembled}

Output valid JSON matching the blog-posts.json schema. Save to: ${DRAFTS}/${page}.json

REQUIRED SCHEMA (all fields, v2.4):
- slug
- metaTitle (under 60 chars, contains primary keyword)
- metaDesc (under 160 chars, contains primary keyword)
- h1 (full title with primary keyword)
- h2s (array of 5-8 section headings, each a real searchable sub-query)
- paragraphs (array; count = N x (len(h2s) + 1) for even section distribution)
- category: Environmental Guides
- date (e.g. April 2026)
- dateModified: ISO date string (YYYY-MM-DD), today's date
- authorityLinks (array of at least 4 distinct .gov links; each {name, url})
- relatedLinks (array of 2-4 internal links to OTHER GUIDES; each {title, slug})
- allText: empty string
- author: exactly the string \"The AsbestosHQ.com Editorial Team\"
- lastReviewed: ISO date string (YYYY-MM-DD), same value as dateModified on initial write
- reviewedBy: null (literal JSON null, not the string \"null\")

YOUR DRAFT WILL BE AUDITED (G1-G5 STRUCTURAL + V-SERIES). GET THESE RIGHT THE FIRST TIME:
- 2,000-2,500 words target (floor 1,800 for niche; ceiling 2,800 justified only)
- 5-8 H2 sections, each corresponding to a real searchable sub-query
- Intro 100-200 words (before first h2); first 100 words MUST contain primary keyword AND a concrete data point (dollar figure, regulation code, timeframe, or named standard)
- Paragraph count = N x (len(h2s) + 1). Example: 7 h2s, 4 paragraphs each = 32 paragraphs total.
- Zero em dashes, en dashes, or hyphen-as-separator anywhere in any field
- Zero banned phrases: In todays world, In conclusion, Its important to note, Its worth noting, At the end of the day, Lets explore, Rest assured, Needless to say, It goes without saying, In this comprehensive guide, Well dive deep into, Without further ado, When all is said and done

NUMERIC CAPS (audit_guide.py v3.3 hard fails — exceed any of these and the mechanical gate blocks approval):
- Sentence word cap: 40 words max. 41+ hard fails V8.
- Paragraph sentence cap: 6 max. 7+ hard fails V9.
- However cap: 6 max across doc. 7+ hard fails V5.
- Same-first-word paragraph streak: 4 max consecutive. 5+ hard fails V7.
- Hedge word cap (ultimately/essentially/fundamentally combined): 6 max. 7+ hard fails V4c.
- V4d 'not just X, it's Y' — INFO-ONLY, not banned. Use sparingly but does not block approval.

GUIDE-SPECIFIC STRUCTURAL CHECKS (all must pass):
- G1 SEMANTIC-VARIETY: At least 5 distinct secondary nouns each appear 3+ times. Rotate variants (asbestos-containing material, ACM, friable asbestos, abatement, encapsulation, asbestos removal), do not repeat the primary keyword.
- G2 REG-DENSITY: At least 4 distinct SPECIFIC regulatory citations. Valid forms: statute+year (AHERA 1986), regulation code (40 CFR 763), OSHA standard (29 CFR 1926.1101), standard name (ASTM E2356), agency program (NESHAP 40 CFR 61 Subpart M). Generic phrasing (federal regulations, state rules) counts for zero.
- G3 CONCRETE-DENSITY: At least 3 concrete specifics. Dollar figures, named standards, timeframes, or specific scenarios. Vague words (expensive, common, takes time) count for zero.
- G4 SECTION-BALANCE: Every H2 section between 40% and 200% of the average section word count. No thin back-half sections.
- G5 INTRO-HOOK: First 100 words contain primary keyword + concrete data point. No generic openers.

PARAGRAPH[0] HARD SHAPE CONSTRAINT (mechanical gate S5 and S6, failure forces a round restart):
- paragraph[0] MUST be 2-3 sentences total. 4+ hard fails S6.
- Sentence 1 MUST be ≤15 words. 16+ hard fails S5. Count words before submitting.
- Sentence 1 MUST contain the primary keyword verbatim (this is also how G5 is satisfied early).
- Sentence 1 MUST be a clean factual lead, not a question, and not an opener like 'Many homeowners...' or 'If you have ever wondered...'.
- Sentences 2 and 3 should each be ≤25 words. Vary length across the three.
- Verify before saving: count sentences, count words in sentence 1, confirm primary keyword in sentence 1.
- Reference: the 8 first-batch approved guides all have paragraph[0] = 3 sentences with sentence 1 ≤14 words. See asbestos-shingles-guide and asbestos-siding-guide for tone.

LINK REQUIREMENTS (v2.4 — directory CTAs removed):
- 2+ guide-to-guide cross-links inline as [text](/url/) in body paragraphs (content-to-content only)
- All internal links use trailing slashes
- 4+ DISTINCT .gov authority links in the authorityLinks array (not Wikipedia, not news, not vendor)
- Max 12 internal links total in body (L1 range is 3-12; phase 1 target 3-6 guide cross-links)
- Primary keyword 3-12 occurrences total (SEO5 range)
- NO links to /find-asbestos-contractors/ or /request-a-quote/. Directory pages do not exist in phase 1.
- NO service-page or state-page links in phase 1. Cross-link only to other guides.

CLOSING PARAGRAPH (v2.4 — required pattern):
The final paragraph closes on one of these patterns. No internal directory links. No 'browse our listings' phrasing.
- Hire-licensed-firm: Hiring a state-licensed asbestos abatement firm with current certification is non-negotiable for removal work. Verify credentials with your state environmental agency before signing a contract.
- Verify-state-credentials: Check the abatement contractor's license against your state asbestos program before any work begins. Most state agencies maintain a public license-status lookup.
- Testing-first: Testing is quick, inexpensive, and the only reliable way to replace guessing with an answer. Pull a sample before planning any renovation, sale, or repair that would disturb the material.

HUMAN-VOICE CONSTRAINTS (C1-C4, v10.11 — writer must satisfy all four; the auditor will treat these as soft checks but systematic failure triggers a rewrite):

- C1 SENTENCE RHYTHM: Within every 4 consecutive sentences in body paragraphs (sliding window, not fixed blocks), include at least one sentence under 10 words OR at least one sentence over 35 words. This breaks uniform cadence. H2 headings and list items do not count toward the 4-sentence window.
- C2 REGISTER SHIFTS: Each H2 section must include at least one register-shift device from: (a) a rhetorical question ('Is this always a removal job?'); (b) direct reader address ('If your home was built before 1978, the texture above you is suspect until proven otherwise.'); or (c) a sentence fragment used for emphasis ('The answer is not obvious. Not always.'). Max one register-shift device per paragraph. Do not cluster them.
- C3 UNEQUAL INFORMATION DENSITY: At least 2 paragraphs per guide must include one transition or color sentence that carries no hard fact. Acceptable non-fact sentences: restate the stakes in different words; acknowledge the reader's likely mental state ('Most homeowners discover this during a renovation.'); or preview what the next paragraph covers. These are not filler — they bridge between fact-dense prose and give the reader a moment to absorb. They do not count toward G3 CONCRETE-DENSITY.
- C4 PARAGRAPH ARCHITECTURE VARIATION: Do not make every paragraph 3-4 sentences opening with a topic sentence and closing with a takeaway. Distribution targets: at least 20% of paragraphs are 1-2 sentences (punchy or transitional), at least 20% are 5-6 sentences (deeper explanatory), remaining 60% can be 3-4 sentences. Some paragraphs should open with a specific fact, question, or scenario and build to the point rather than lead with the takeaway. (Note: 6-sentence paragraphs are at the V9 ceiling — a 7+ sentence paragraph still hard-fails.)

Before saving, verify:
1. Valid JSON with no trailing commas
2. len(paragraphs) divisible by (len(h2s) + 1)
3. First 100 words of intro contain primary keyword + concrete data point
4. At least 4 .gov authority links in authorityLinks
5. Zero dashes anywhere (em, en, or hyphen-separator)
6. category is exactly: Environmental Guides
7. author, lastReviewed, reviewedBy fields present with correct values (author string exact, lastReviewed ISO date, reviewedBy literal null)
8. No /find-asbestos-contractors/ or /request-a-quote/ links anywhere
9. Closing paragraph follows hire-licensed-firm / verify-state-credentials / testing-first pattern
10. C1 sentence rhythm satisfied across every sliding 4-sentence window
11. C2 register-shift device present in every H2 section
12. C3 at least 2 non-fact transition/color sentences across the guide
13. C4 paragraph-length distribution hits the 20% / 20% / 60% targets

Do not explain. Just write the page." $WRITER_MODEL --dangerously-skip-permissions
            else
```

### 6.4 Line 569 — BLOG WRITER (`claude -p`)

```bash
            else
                # BLOG POST WRITER PROMPT (unchanged from v10.1)
                claude -p "You are the WRITER for AsbestosHQ.com. Read this file, then write the assigned page for ${page}.

1. ${assembled}

Output valid JSON per the OUTPUT FORMAT section. Save to: ${DRAFTS}/${page}.json

YOUR DRAFT WILL BE AUDITED. GET THESE RIGHT THE FIRST TIME:
- 1,750-1,850 words (under 1,700 = instant fail)
- 2+ one-sentence standalone paragraphs scattered through the piece
- Every state mentioned by name must have a markdown link to its state page
- Zero em dashes or en dashes anywhere in any field
- metaTitle under 60 chars, metaDesc under 160 chars
- Final section must be action steps with concrete next moves, not just information
- 4-8 relatedLinks using only slugs from the Valid Internal Link Slugs list
- Max 8 inline internal links in body text
- Primary keyword max 1 per 150 words (over = instant fail)

Before saving, also verify:
1. Valid JSON with no trailing commas
2. First paragraph under 15 words, takes a position, contains primary keyword
3. Primary keyword in h1, at least one h2, and metaDesc
4. At least 8 secondary keywords woven in naturally

Do not explain. Just write the page." $WRITER_MODEL --dangerously-skip-permissions
            fi
        else
```

### 6.5 Line 595 — ROUND 2+ REVISE PROMPT (`claude -p`)

```bash
        else
            claude -p "Read the feedback and revise the draft:

1. Feedback: ${FEEDBACK}/${page}-r$((round - 1)).md
2. Draft: ${DRAFTS}/${page}.json
3. Specs: ${assembled}

Fix every flagged issue. Do not delete content. Do not drop word count more than 20%.
Save to same location. Do not explain." $WRITER_MODEL --dangerously-skip-permissions
        fi

        stop_spinner
        local writer_time=$(( SECONDS - step_start ))
        local word_count=$(count_words "${DRAFTS}/${page}.json")
        echo "  ✏️  [$page_num/$total] WRITER done — ${word_count} words — ${writer_time}s"
```

### 6.6 Line 847 — SERVICE PAGE AUDITOR (`claude -p`)

```bash
        if [ "$SERVICE" = true ]; then
            # SERVICE PAGE AUDITOR PROMPT
            claude -p "You are the AUDITOR for AsbestosHQ.com content.

1. Read: ${CACHED_AUDIT}
2. Read template overrides from: ${TEMPLATES}
   Use template type: service-page
3. Audit: ${DRAFTS}/${page}.json
4. Compare against assignment: ${CACHE_DIR}/assignment-${page}.md
5. ${as_instruction}

The draft is a service page JSON object. Apply ALL service-page overrides from the templates file above:
- S1: 400-600 words across contentParagraphs only
- S4-S6, S10, S12, V16, D1, D2, SEO3, L1-L5, AS2: SKIP
- Run JSON1-JSON8 validation checks
- Check exactly 4 contentParagraphs and exactly 5 FAQs

Apply Cosmetic Tolerance List and Verdict Rules.

INLINE FIX: If the ONLY failures are mechanical, fix them yourself and approve:
- metaTitle/metaDesc too long → shorten
- Sentence over 35 words → split
- Embedded newlines in strings → remove
- Consecutive same-start paragraphs → reword opener
Note fixes as FIXED in scorecard.

If ANY failure needs content judgment → write feedback for writer.

Output FAILURES ONLY.

If PASS or PASS WITH NOTES or PASS WITH FIXES: move file to ${APPROVED}/${page}.json and append to ${INDEX}
If FAIL: write feedback to ${FEEDBACK}/${page}-r${round}.md

Do not explain. Just audit, fix what you can, output scorecard." $AUDITOR_MODEL --dangerously-skip-permissions
        elif [ "$GUIDE" = true ]; then
```

### 6.7 Line 881 — GUIDE AUDITOR (v10.11, `claude -p`)

```bash
        elif [ "$GUIDE" = true ]; then
            # GUIDE AUDITOR PROMPT (v10.11 — G1-G5 + C1-C4 human-voice soft checks; v2.4 schema)
            claude -p "You are the AUDITOR for AsbestosHQ.com content.

1. Read: ${CACHED_AUDIT}
2. Read template overrides from: ${TEMPLATES}
   Use template type: guide
3. Audit: ${DRAFTS}/${page}.json
4. Compare against assignment: ${CACHE_DIR}/assignment-${page}.md
5. ${as_instruction}

The draft is a long-form GUIDE page JSON object matching the blog-posts.json schema (slug, metaTitle, metaDesc, h1, h2s, paragraphs, category, date, dateModified, authorityLinks, relatedLinks, allText, author, lastReviewed, reviewedBy).

Apply ALL guide-specific requirements from CONTENT_SPEC_GUIDE_ASBESTOS.md v2.4.

GUIDE-SPECIFIC STRUCTURAL CHECKS (G1-G5, all must pass):
- G1 SEMANTIC-VARIETY: At least 5 distinct secondary nouns each appear 3+ times across the guide. Fewer = FAIL.
- G2 REG-DENSITY: At least 4 distinct SPECIFIC regulatory citations. Valid: statute+year (AHERA 1986), regulation code (40 CFR 763), OSHA standard (29 CFR 1926.1101), standard name (ASTM E2356), agency program (NESHAP 40 CFR 61 Subpart M). Generic (federal regulations, state rules) counts for zero.
- G3 CONCRETE-DENSITY: At least 3 concrete specifics (dollar figures, timeframes, named standards, specific scenarios). Vague words (expensive, common, takes time) count for zero.
- G4 SECTION-BALANCE: Every H2 section between 40% and 200% of the average section word count. Measured in words, excluding H2 headings.
- G5 INTRO-HOOK: First 100 words contain primary keyword AND concrete data point. Generic opener (In todays world, Everyone knows, In this comprehensive guide) = FAIL.

STRUCTURAL REQUIREMENTS:
- Word count 1,800-2,800 (target 2,000-2,500) — matches audit_guide.py S1
- 4-10 H2 sections (target 5-8), each a real searchable sub-query — matches audit_guide.py S3
- len(paragraphs) divisible by (len(h2s) + 1)
- metaTitle under 65 chars (target 60), metaDesc under 165 chars (target 160) — matches audit_guide.py S7/S8
- category is exactly: Environmental Guides
- At least 4 DISTINCT .gov authority links in authorityLinks array
- 2+ guide-to-guide cross-links in body, all trailing-slash
- Max 12 inline internal links in body
- Primary keyword 3-12 occurrences total (SEO5)
- Zero em dashes, en dashes, or hyphen-as-separator anywhere
- E-E-A-T fields present: author = \"The AsbestosHQ.com Editorial Team\", lastReviewed = ISO date, reviewedBy = null

DIRECTORY-CTA CHECK (v2.4 — hard fail if present):
- FAIL any draft containing /find-asbestos-contractors/ or /request-a-quote/ links. Phase 1 of the site has no directory pages; directory CTAs from pre-v2.4 guides must be stripped and rewritten per the closing-paragraph pattern below.
- FAIL any service-page or state-page link (/california-asbestos-contractors/, /asbestos-inspection/, etc.) in phase 1. Guide-to-guide cross-links only.

CLOSING PARAGRAPH CHECK (v2.4):
The final paragraph must close on one of these patterns, with no internal directory link:
- Hire-licensed-firm prose (verify credentials with state environmental agency)
- Verify-state-credentials prose (state license-status lookup)
- Testing-first prose (sample before planning renovation)
FAIL if the closing paragraph references 'our directory', 'browse', 'find a contractor', 'request a quote', or any internal link to directory pages.

NUMERIC CAPS (audit_guide.py v3.3 hard fails — flag any draft that exceeds these):
- Sentence word cap: 40 words max. 41+ hard fails V8.
- Paragraph sentence cap: 6 max. 7+ hard fails V9.
- However cap: 6 max across doc. 7+ hard fails V5.
- Same-first-word paragraph streak: 4 max consecutive. 5+ hard fails V7.
- Hedge word cap (ultimately/essentially/fundamentally combined): 6 max. 7+ hard fails V4c.
- V4d 'not just X, it's Y' — INFO-ONLY, not banned. Note in scorecard if used heavily but do not fail.

INHERITED STANDARD CHECKS:
- SAT1 saturation (calibrated v1 thresholds)
- V-series: zero banned phrases (see APPENDICES for the 14 long-form additions)
- V4b: REMOVED in v3.3 (BANNED_PHRASES_UST dropped)
- L3: REMOVED in v3.3 (directory CTA no longer required)
- D1: one-sentence paragraphs are info-only, not enforced (v3.1+)
- D4: exception/limiting sentences are info-only, not enforced (v3.2+; guide spec does not require exception markers)
- L1: no more than 12 internal links (mechanical range 3-12)

HUMAN-VOICE CONSTRAINTS (C1-C4, v10.11 — SOFT checks, LLM-auditor-only, NOT enforced by audit_guide.py):

- C1 SENTENCE RHYTHM: scan sliding 4-sentence windows across body paragraphs. FAIL any window that has no sentence under 10 words AND no sentence over 35 words. Exclude H2 headings and list items from the window. Report number of failing windows out of total.
- C2 REGISTER SHIFTS: scan each H2 section. FAIL any section missing all three register-shift devices (rhetorical question, direct reader address, sentence fragment for emphasis). A section with at least one device passes. Max one device per paragraph within a section — flag clusters as INFO but do not fail.
- C3 INFORMATION DENSITY: count paragraphs that contain a non-fact transition/color sentence (restates stakes, acknowledges reader mental state, or previews next paragraph). FAIL if fewer than 2.
- C4 PARAGRAPH ARCHITECTURE: count paragraph sentence-length distribution. FAIL if fewer than 20% of paragraphs are 1-2 sentences OR fewer than 20% are 5-6 sentences.

Report each as C1 / C2 / C3 / C4 PASS or FAIL in the scorecard with a one-line reason. These are SOFT: a draft MAY pass with 1 or 2 C-check failures if overall quality is strong. Systematic failure across 3 or 4 C-checks is a reject signal — write feedback and send back for revision. Do not treat C1-C4 as binary gates.

Apply Cosmetic Tolerance List and Verdict Rules.

INLINE FIX: If the ONLY failures are mechanical, fix them yourself and approve:
- metaTitle/metaDesc too long = shorten
- Sentence over 40 words = split
- Missing .gov authority link = add to authorityLinks
- Missing trailing slash on internal link = add it
- Consecutive same-start paragraphs = reword opener
- Missing or incorrect author/lastReviewed/reviewedBy = set to canonical values
Note fixes as FIXED in scorecard.

If ANY failure needs content judgment (G1-G5 failure, thin section, missing concrete data, banned phrase in meaning, section not matching a searchable sub-query) = write feedback for writer.

Output FAILURES ONLY.

If PASS or PASS WITH NOTES or PASS WITH FIXES: move file to ${APPROVED}/${page}.json and append to ${INDEX}
If FAIL: write feedback to ${FEEDBACK}/${page}-r${round}.md

Do not explain. Just audit, fix what you can, output scorecard." $AUDITOR_MODEL --dangerously-skip-permissions
        else
```

### 6.8 Line 972 — BLOG AUDITOR (`claude -p`)

```bash
        else
            # BLOG AUDITOR PROMPT (unchanged from v10.1)
            claude -p "You are the AUDITOR for AsbestosHQ.com content.

1. Read: ${CACHED_AUDIT}
2. Read template overrides from: ${TEMPLATES}
3. Audit: ${DRAFTS}/${page}.json
4. Compare against assignment: ${CACHE_DIR}/assignment-${page}.md
5. ${as_instruction}

The draft is JSON. Check paragraphs for voice/quality. Check h2s for structure. Check relatedLinks and authorityLinks for links.

Apply template-specific overrides, Cosmetic Tolerance List, and Verdict Rules.

INLINE FIX: If the ONLY failures are mechanical, fix them yourself and approve:
- seoTitle/metaDesc too long → shorten
- Sentence over 35 words → split
- Missing .gov authority link → add to authorityLinks
- Missing state page link → add to relatedLinks
- Internal link slug wrong → correct it
- Consecutive same-start paragraphs → reword opener
Note fixes as FIXED in scorecard.

If ANY failure needs content judgment (weak gotcha, missing tradeoff, banned phrase in meaning, thin section) → write feedback for writer.

Output FAILURES ONLY.

If PASS or PASS WITH NOTES or PASS WITH FIXES: move file to ${APPROVED}/${page}.json and append to ${INDEX}
If FAIL: write feedback to ${FEEDBACK}/${page}-r${round}.md

Do not explain. Just audit, fix what you can, output scorecard." $AUDITOR_MODEL --dangerously-skip-permissions
        fi
```

---

## End of recon dump

**File map:**
- §1 `~/projects/ssg-content/content/ssg/CONTENT_SPEC_GUIDE_SSG.md` — verbatim, 368 lines
- §2 `~/projects/asbestos-contractors/content/asbestos/CONTENT_SPEC_GUIDE_ASBESTOS.md` — byte-identical to §1; full text not duplicated
- §3 `~/projects/smartsourceguide/app/it-support/managed-it-services-pricing/page.tsx` — verbatim, 129 lines
- §4 `~/projects/_asbestos-reference/approved-guides/asbestos-inspection-cost.json` — verbatim, 30-guide median size
- §5 SSG self-description — home, about, layout, header, footer (verbatim); no README or CLAUDE.md exists
- §6 8× `claude -p` persona/prompt blocks from `~/projects/ssg-content/content/run-batch.sh` (lines 410, 444, 474, 569, 595, 847, 881, 972) with surrounding context
