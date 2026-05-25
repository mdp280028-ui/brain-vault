# AITEAM Context Save — SSG Lane C Pass 1 + Pass 2

**Date:** 2026-05-23 (evening session, post-Lane-B-Phase-2-shipped)
**Session topic:** SSG pipeline activation — Lane C wire-up + content input authoring
**Predecessor context:** `AITEAM_Context_Save_2026-05-23_SSG_LaneB_Complete.md` (earlier same day)

---

## TL;DR

SSG pipeline activation is now ~85% complete. Two passes shipped:

- **Pass 1** — Config + code wire-up (ship-to-site/config/ssg.yaml replaces stale TODOs with Phase-2 paths; deploy_batch.sh gets SSG branch in gsc_submission_queue INSERT; live_base_url fixed to include www.)
- **Pass 2** — 5 content input files authored (keyword_research.md, approved-index.md, authority-links.md STARTER, assignment-batch-001.md, assignment-batch-002.md) totaling 768 lines

**Pre-Pass-3 blockers** (all operator-side, not CC):
1. D086 — operator creates ssg-content GitHub remote + pushes 5 local commits (CRITICAL: 768 lines local-only)
2. Operator review of all 5 Pass-2 files (especially authority-links.md STARTER URLs + batch seed paragraphs)
3. Operator creates content/ssg/drafter_queue.txt with batch order
4. Operator flips ~/agents/assignment-drafter/config/ssg.yaml `enabled: true`

Both master switches (drafter + ship-to-site) remain `enabled: false`. Pipeline gated.

---

## Session arc

1. **Opening** — Operator asked for "next 5 articles to write." Pivoted to 20 over the conversation. Built tiered ranking (Tier 1 vendor reviews → Tier 4 fleet alternatives) with cannibalization analysis against the 14 migrated articles.
2. **Cadence decision** — 3-5 articles/week × ~5 weeks. Week 1 = 4 answering-services vendor reviews (Moneypenny, AnswerConnect, Smith.ai, Voicenation-vs-Patlive).
3. **First article setup** — Wrote a CC research+draft prompt for Moneypenny review. Operator picked "no affiliate CTA — internal links only" per Claude's recommendation (premature monetization risk, bait-and-switch readability, indexing comes before affiliate revenue).
4. **The key inflection** — Operator: "don't we have agents that can write these articles?" → Surfaced that SSG pipeline was 95% built but Lane C (wire-up) was deferred to a separate session. Operator chose to finish the pipeline rather than hand-write 20 articles.
5. **Lane C recon** → Pass 1 verification → Pass 1 build → Pass 2 (5 files, one at a time with operator sign-off between each) → close-out push + 3 new DEFERRED items.

---

## What got built

### Lane C Pass 1 (commit d3025ae in ~/agents)

**~/agents/ship-to-site/config/ssg.yaml** — replaced pre-Phase-2 TODOs with real Phase-2 paths:
- `site_data_dir: data/guides` (was: `src/data/guides`)
- `site_pages_dir: app` (was: `src/app` — dynamic [category]/[slug] handles all)
- `slugs_whitelist_file: lib/site-config.ts`
- `slugs_whitelist_var: APPROVED_GUIDE_SLUGS`
- `live_base_url: https://www.smartsourceguide.com/` (was missing www. — production redirect)
- `template_path:` and `ci_workflow_name:` left blank (data-driven, no template; Vercel auto-deploys)
- `enabled: false` preserved

**~/agents/ship-to-site/deploy_batch.sh** — added SSG branch to gsc INSERT (after asbestos branch, line 103):
```bash
elif [ "${site}" = "ssg" ] && [ "${kind}" = "new" ]; then
  slug_esc="${slug//\'/\'\'}"
  sqlite3 "${STORE_ROOT}/aiteam.db" \
    "INSERT OR IGNORE INTO gsc_submission_queue (slug, url, site, published_at) VALUES ('${slug_esc}', 'https://www.smartsourceguide.com/${slug_esc}/', 'ssg', strftime('%Y-%m-%dT%H:%M:%fZ','now'));" \
    >/dev/null 2>&1 || true
```
Pattern matches asbestos-new exactly (same SQL escape, same error handling). Diverges intentionally: site literal `'ssg'`, SSG live_base_url, omits propose_backlinks + keyword-registry (asbestos-only infrastructure — captured as D083).

**Verification before commit:** curl https://www.smartsourceguide.com/answering-services/answering-service-pricing/ → HTTP 200 confirms URL pattern works.

### Lane C Pass 2 (5 commits in ~/projects/ssg-content, local-only)

| # | SHA | File | Lines | Notes |
|---|---|---|---|---|
| 1 | ef32bc3 | keyword_research.md | 137 | 90 keywords (30 it-support + 30 answering + 30 fleet), 3 cluster tables, status legend |
| 2 | 3b0ac75 | approved-index.md | 53 | 14 live slugs with title + primary KW + status; 4 confirmed exact-match, 10 UNCONFIRMED |
| 3 | 1a89e11 | authority-links.md | 171 | 33 sources, STARTER pending operator review; .gov + industry assocs + trade pubs; aggregators excluded |
| 4 | 2d3c2da | assignment-batch-001.md | 199 | moneypenny-review + answerconnect-review (single-vendor Mode A) |
| 5 | 8742be1 | assignment-batch-002.md | 208 | smith-ai-review (single-vendor) + voicenation-vs-patlive (comparison Mode A variant) |

**Schema extensions vs asbestos exemplars (all documented in-file):**
- approved-index.md adds Title + PrimaryKW + Status columns (cannibalization signal)
- authority-links.md adds source-quality bar policy section + anti-coverage list (no Capterra/G2/Software Advice — vendor-marketing channels, not editorial authority)
- assignment-batch files drop asbestos-specific gate fields (G2/G3 regulatory entities, secondary-noun counts) — SSG-native fields drawn from CONTENT_SPEC_GUIDE_SSG §5/§6/§7/§8

---

## Decisions made (and why)

### "Don't enroll affiliates before traffic exists" — locked
Operator: "I'll work on affiliate enrollment when we start to see site traffic." Per asbestos pattern (no AdSense before traffic). Pass 2 CTA pattern for all 4 vendor-review pages = internal-link only, NO affiliate. Affiliate CTA wire-in is a future revision once enrollments are operationally tested. Article body stays useful either way.

### "Build SSG-native assignment-batch fields, drop asbestos regulatory fields" — locked
Question G from File 4 pre-flight. CC's recommendation: Option A. Assignment-batch is writer input, not audit artifact. Asbestos's "G2 required regulatory entities" field has no B2B-services analog — preserving it with N/A markers would be cargo-culting. Build from CONTENT_SPEC §5 (Mode A typical H2s) + §6 (voice) + §7 (banned phrases) + §8 (CTA rules).

### "Capture D086 as new item, don't reopen D066" — clean audit trail
CC asked whether to mutate D066 (closed 2026-05-17 covering local-commits-only scope) into "needs remote creation" status, or create D086 as fresh item citing D066 as predecessor. Closed-items-stay-closed wins. D086 captures both gh CLI path (after D071 closes) + operator-manual path for now.

### "Lane C Pass 1 fires safely before content inputs exist"
Flipping ship-to-site/config/ssg.yaml `enabled: true` would be harmless because approved/guides/ is empty (ship.sh skips). But drafter-side flip without content inputs = errors. So Pass 1 config landed safely with both master switches still false. Pass 2 content inputs landed without flipping anything. Pass 3 is the actual on-switch.

### "Asbestos GSC hook is UNTESTED but structurally correct" — pattern-copy safe
Recon found 0 rows in gsc_submission_queue and 0-byte deploy_batch.log since 2026-05-16. Investigation: the */15 ship cron was disabled by the throttle commit the same day, only one ship event ever (white-asbestos-vs-blue-asbestos 2026-05-17) and it failed at build + rolled back. Hook has had zero opportunities to fire. Not broken — untested. Captured as D081 (canary check after first SSG ship).

---

## New DEFERRED items captured this session

| ID | Topic | Trigger |
|---|---|---|
| D081 | deploy_batch.log 0-byte canary check | After first SSG slug ships — confirm log gets populated |
| D082 | ship.sh pushes before detecting build failure | Future ship.sh hardening session (out of Lane C scope) |
| D083 | SSG-fy propose_backlinks.sh + keyword-registry update_registry.py | After 5+ SSG slugs ship and internal-link coverage debt matters |
| D084 | keyword_research.md misses high-CPC/low-SV opportunities | After first batch ships — add "Sub-threshold high-CPC" section filtering by CPC≥$30 |
| D085 | Cross-link backfill from 5 existing answering-services articles to new vendor pages | After batch 001+002 ships live, ~30 min sed-driven backfill |
| D086 | ssg-content GitHub remote creation (operator manual via web UI) | CRITICAL — 768 lines pipeline operating instructions local-only |

All committed to ~/brain/projects/aiteam/DEFERRED.md and pushed to brain-vault remote.

---

## What was left mid-task

**Pre-Pass-3 operator checklist (in priority order):**

1. **D086 — push ssg-content to remote.** 768 lines pipeline operating instructions are local-only. Operator action: create `mdp280028-ui/ssg-content` (private) on github.com manually, then:
   ```bash
   cd ~/projects/ssg-content
   git remote add origin git@github.com:mdp280028-ui/ssg-content.git
   git push -u origin main
   ```

2. **Review the 5 Pass-2 files.** Especially:
   - `authority-links.md` — marked STARTER, 33 URLs need spot-check
   - `assignment-batch-001.md` + `assignment-batch-002.md` — seed paragraphs contain concrete pricing/positioning claims about Moneypenny, AnswerConnect, Smith.ai, VoiceNation, PatLive. Verify these are accurate before the writer agent treats them as ground truth.
   - Approved-index.md — 9 of 14 entries marked UNCONFIRMED for primary keyword

3. **Create `content/ssg/drafter_queue.txt`** with batch order:
   ```
   assignment-batch-001
   assignment-batch-002
   ```

4. **Flip `~/agents/assignment-drafter/config/ssg.yaml` enabled: true**

5. **Wait for `*/30` drafter cron** to fire first batch. First article should produce within 30 min. Auditor scores → GEO scores → verdict persists. Review before approving more.

6. **Flip `~/agents/ship-to-site/config/ssg.yaml` enabled: true** after first approved draft. Deploy throttle kicks in: 20:00 PT preview → 23:00 PT batch deploy.

**Operator suggested doing Pass 3 "real quick" tonight.** Pushed back: it's late Saturday, 23:00 PT deploy window already passed, files unreviewed, ssg-content unbacked-up. Earliest realistic first SSG ship = Sunday night.

---

## Open decisions (live design space)

1. **No new ones from this session.** Lane C completion was mechanical pattern-copying + content authoring from existing specs.
2. **Carrying forward:** the 7 operator-policy questions from cross-agent failure modes audit §7 (still blocking D056 editor production runner).

---

## Mission-bar nuance reminder

Dashboard "API cost" figures are Max-sub burn, not real $. Real-money cost tonight = $0. Pass 1 + Pass 2 across ~5 hours of CC work cost the operator nothing in dollars; only Max-subscription capacity consumed. This whole pipeline activation is value-extracted from a subscription already paid.

---

## Lessons learned

### "Don't we have agents for this?" — listen for it
When operator asked about CC writing articles directly, the right answer was NOT to write 20 article prompts. The right answer was to surface that the pipeline was 95% built and ask whether to finish it instead. Took 1 message of operator pushback to reveal that they wanted quality (audited content), not speed (raw CC output). Lesson: when operator's stated request would skip an audit gate, flag it before executing.

### Pattern-copy safety = verify the source pattern actually works
The asbestos GSC hook had 0 rows since 2026-05-17. Tempting to assume "the hook works, just nothing to populate it." Pre-flight verification proved the safer interpretation: untested-but-structurally-correct. Same outcome (pattern-copy safe), but the evidence chain matters. Future Lane-style migrations should always verify the source pattern was exercised before copying.

### Schema extensions in-file > silent drift
CC extended asbestos exemplar schemas in 3 places (approved-index, authority-links, assignment-batch) for SSG-specific needs. All 3 documented the divergence in the file itself with explicit "Format / extension note" sections. Future readers won't wonder why SSG diverged.

### Multi-site infrastructure was already done at every layer
Recon revealed: ship.sh, deploy_batch.sh, drafter.sh, gsc_submission_queue schema — all multi-site by design. Lane C work was fundamentally just: enable the SSG yaml, fill config fields, add SSG branch to gsc INSERT, author content inputs. Zero new infrastructure. The 2026-05-15 architectural decision to make agents multi-site from day one is paying off.

### 768 lines of operating instructions deserves a remote
ssg-content repo has been local-only since creation (2026-05-16). For 7 days it held only stubs and machinery. Now it holds 768 lines of pipeline-defining content — what the writer agent reads, what the auditor scores against, what determines whether SSG content is editorially defensible. D086 elevation from D066-residual to standalone-critical is right.

---

## Commits this session

| Repo | SHA | What |
|---|---|---|
| ~/brain | e7aa33f | DEFERRED: add D081 (deploy_batch.log canary) + D082 (ship.sh push-before-build) |
| ~/agents | d3025ae | feat(ship-to-site): SSG config wire-up + gsc queue branch (Lane C Pass 1) |
| ~/brain | 5ed5be9 | DEFERRED: add D083 (SSG-fy propose_backlinks + keyword-registry) |
| ~/projects/ssg-content | ef32bc3 | content(ssg): add keyword_research.md from registry (LOCAL ONLY — D086) |
| ~/projects/ssg-content | 3b0ac75 | content(ssg): add approved-index.md (14 live slugs) (LOCAL ONLY) |
| ~/projects/ssg-content | 1a89e11 | content(ssg): add authority-links.md starter (operator-review pending) (LOCAL ONLY) |
| ~/projects/ssg-content | 2d3c2da | content(ssg): add assignment-batch-001.md (moneypenny + answerconnect) (LOCAL ONLY) |
| ~/projects/ssg-content | 8742be1 | content(ssg): add assignment-batch-002.md (smith-ai + voicenation-vs-patlive) (LOCAL ONLY) |
| ~/brain | 748b799 | DEFERRED: add D084 (keyword_research sub-threshold) |
| ~/brain | 923464d | DEFERRED: add D085 + D086 (cross-link backfill + ssg-content remote) |

**Pushed:** ~/agents (Pass 1), ~/brain (all DEFERRED commits).
**NOT pushed:** ~/projects/ssg-content (5 commits local-only — D086 blocker).

---

## Next session start point

Pre-Pass-3 operator checklist (numbered list above, items 1-6). Estimated 30-45 min operator time for items 1-3 (mostly file review), then enabled flip + drafter cron wait, then first-draft review. First SSG slug live earliest Sunday night via 20:00 preview / 23:00 deploy throttle.

If operator wants a fast review path: I recommended opening `authority-links.md` and `assignment-batch-001.md` first — highest-risk content (URL verification + concrete vendor claims). Other 3 files are lower-risk (keyword_research is derived from registry verbatim; approved-index is mechanical; assignment-batch-002 follows -001's pattern).
