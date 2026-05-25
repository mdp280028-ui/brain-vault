# AITEAM Context Save — SSG Lane B Complete (Phase 1 + Phase 2 Shipped)

**Date:** 2026-05-23 (single focused session, ~4-5 hours elapsed)
**Chat role:** Primary chat driving SSG site data-driven rewrite end-to-end
**Session character:** Plan recovered from disk → executed Phase 1 (chassis) + canary + Phase 2 (13 articles) → live on production. Zero premature ✅. Three CC pre-flight HALTs caught real issues. Four operator-approved CTA mismatch fixes shipped. Two parser/schema deviations honestly flagged and approved.

---

## TL;DR

**SSG Lane B is closed.** 14 articles at smartsourceguide.com migrated from JSX-inlined → data-driven (JSON + dynamic route). Site renders identically to visitors. Pipeline (`ssg-content/`) can now ship JSON that the site actually consumes.

**16 commits live in production:**
- zod dep + chassis (zod schema, dynamic `[category]/[slug]` route, 6 components, page-level Article + FAQPage JSON-LD)
- Canary article (best-bilingual-answering-service) verified the chassis end-to-end
- 13 article conversions, one commit each, all clean

**4 mismatch fixes shipped:**
- `best-answering-service-law-firm` — cross-category IT pivot → in-category comparison link
- `best-fleet-gps-tracking` — 2 cross-category links removed, heading upgraded to "Building out your fleet stack?"
- `managed-it-vs-break-fix` — self-link CTA removed (was pointing to itself)
- `best-it-support-law-firm` — over-promise CTA retargeted to deliver what it promised

**Backup tag `pre-data-driven-rewrite` at c05c1f6** on remote. Full rollback path available.

**Push SHA cf78061. Audit row C6F443AE-D145-4BE3-9DB4-482E4E361B79.**

**Mission bar:** $0 / $200/mo unchanged. SSG can now technically receive AI-generated pipeline output once Lane C lands. Revenue still gated on (a) Lane C (ssg.yaml enable + throttle + GSC hook) (b) operator keyword research → drafter_queue.txt (c) AdSense enrollment.

---

## SESSION ARC

1. Operator asked for 1-3 SSG pages ASAP to avoid "stale content." Reframed: SSG isn't stale (14 articles <60 days old), the real problem is 0 indexed in Google's main index. Path A (hand-author JSX) creates debt. Path B (Lane B data-driven rewrite) was the right call.

2. **Lane B recon prompt** — confirmed site repo clean, plan doc on disk, 14 articles match plan, 2 noindex shims separate from migration scope, TS version skew flagged (unrelated to Lane B).

3. **Pre-Phase-1 prep** — caught backup tag missing (set it), confirmed the 2 top-level pages are genuinely noindex shims (excluded from scope), confirmed 14-article scope.

4. **Route shape decision** — plan specified `app/[slug]/page.tsx` but URLs are 2-level. Locked `app/[category]/[slug]/page.tsx` (two-segment dynamic route, category as first-class JSON field).

5. **Phase 1 pre-flight (CC)** — caught 5 real ambiguities. Operator answered each:
   - CTA rendering: KEEP embedded-link pattern (zero visual diff goal)
   - Intro shape: ARRAY of strings (reality > spec, canary had 3 paragraphs not 1-2)
   - relatedLinks: schema yes, component no (forward-compat for Internal Link Agent)
   - Mismatch identification: defer to Phase 2 (now Phase 2 since canary folded in)
   - JSON-LD: emit at page level, not component level
   - Inline markdown: lightweight parser (~30 lines), no react-markdown
   - Date display: `<time dateTime>` not hardcoded "Last updated April 2026"

6. **Phase 1 HALT at step 13** — CC caught real Next 14 + `output: 'export'` constraint: dynamic routes MUST have non-empty `generateStaticParams()`. Plan author didn't account for it. Three options surfaced. Chose Option 3: fold canary into Phase 1. Renamed downstream: original Phase 2 = canary, now Phase 2 = bulk migrate 13 remaining.

7. **Phase 1 + canary close-out** — 3 commits (zod 4811858, chassis 5506534, canary c7db2f2). Two CC self-caught regressions BEFORE sign-off: robots tag leaking undefined (would've nuked indexability), unique OG copy lost (added `seo.ogTitle`/`seo.ogDescription` schema fields). One framework deviation: async `generateStaticParams`/`generateMetadata` (Next 14 requires it). Visual diff clean — only allowed changes (new JSON-LD, `<time>` tag).

8. **Phase 2 pre-flight (CC)** — corpus scan found:
   - 4 articles with 3-level breadcrumbs (3 of 13)
   - 0 articles with `<h3>` in body (h3 only in FAQ)
   - 8 articles with significant inline `<strong>` (up to 15 instances each)
   - 5 CTAs with multiple inline links
   - Proposed 4 mismatch fixes (3 cross-category, 1 self-link)

9. **Operator approved 4 mismatch fixes** — accepted 3 as-proposed, adjusted #2 heading from "More fleet guides" → "Building out your fleet stack?" (specific framing converts better). Declined to fix 2 other "best outsourced IT support services" link-text awkwardness (scope creep, sed fixes later if it bugs).

10. **Category 1 (it-support, 4 articles)** — 4 commits, including mismatch fixes #3 and #4. CC honestly flagged a schema deviation: `intro: z.array(z.string()).min(1).max(5)` was overconstrained because 9/13 articles have no intro paragraphs. Operator approved relax to `max(5)`. Locked as new baseline.

11. **Category 2 (answering-services, 4 articles)** — 4 commits, including mismatch fix #1. Markdown parser validated at scale (8 `<strong>` instances on `answering-service-pricing` worked clean). 3-level breadcrumb with linked middle item validated.

12. **Category 3 (fleet-tracking, 5 articles)** — 5 commits, including mismatch fix #2. Real parser regression caught at category-3 article 1: nested `<Link>` inside `<strong>` rendered as literal markdown. CC fixed by recursing `parseInline` into bold content, bundled fix into the dash-cam commit (d60fed2) per commit-message rule.

13. **Pre-push verification** — clean tree, 16 commits, backup tag anchored, clean build (21 routes), all 14 JSONs validated by zod (`valid: 14 fail: 0`), JSON-LD spot-check on 3 random articles all confirmed.

14. **Push** — `c05c1f6..cf78061  main -> main` + `pre-data-driven-rewrite` tag pushed. Vercel auto-deployed.

15. **Live verification** — 3 URLs HTTP 200 across all 3 categories. JSON-LD Article count = 1 on canary. Done.

16. **Audit log** — row C6F443AE-D145-4BE3-9DB4-482E4E361B79 with full payload.

---

## WHAT GOT SHIPPED

### Commits (16 total this session, all in `~/projects/smartsourceguide/`)

| # | SHA | Description |
|---|---|---|
| 1 | `4811858` | feat(ssg): add zod dependency for guide JSON validation |
| 2 | `5506534` | feat(ssg): data-driven chassis (zod schema, dynamic [category]/[slug] route, 6 components) |
| 3 | `c7db2f2` | feat(ssg): canary article best-bilingual-answering-service migrated to JSON (chassis verified) |
| 4 | `17eab75` | feat(ssg): migrate best-it-support-law-firm to JSON + CTA fix |
| 5 | `f1c40fa` | feat(ssg): migrate managed-it-services-pricing to JSON + schema relax (intro min dropped) |
| 6 | `7f2296b` | feat(ssg): migrate managed-it-vs-break-fix to JSON + CTA fix (self-link removed) |
| 7 | `0ae595e` | feat(ssg): migrate outsource-help-desk-guide to JSON |
| 8 | `9917f75` | feat(ssg): migrate answering-service-pricing to JSON |
| 9 | `c804a6d` | feat(ssg): migrate best-ai-answering-service to JSON |
| 10 | `8f062e8` | feat(ssg): migrate best-answering-service-law-firm to JSON + CTA fix |
| 11 | `f1b2799` | feat(ssg): migrate best-answering-services to JSON |
| 12 | `d60fed2` | feat(ssg): migrate best-fleet-dash-cam to JSON + parser fix (bold-with-nested-link) |
| 13 | `daedf53` | feat(ssg): migrate best-fleet-fuel-cards to JSON |
| 14 | `1ee6c70` | feat(ssg): migrate best-fleet-gps-tracking to JSON + CTA fix |
| 15 | `3f2cdcd` | feat(ssg): migrate best-fleet-maintenance-software to JSON |
| 16 | `cf78061` | feat(ssg): migrate fleet-tracking-contracts to JSON |

Backup tag: `pre-data-driven-rewrite` at `c05c1f6` (pushed to remote).
Push: `c05c1f6..cf78061  main -> main`.
Audit row: `C6F443AE-D145-4BE3-9DB4-482E4E361B79`.

### Final file structure

```
~/projects/smartsourceguide/
├── app/
│   ├── [category]/[slug]/page.tsx        ← NEW: serves all 14 guides dynamically
│   ├── about/page.tsx                    ← unchanged
│   ├── best-answering-services/page.tsx  ← noindex shim, unchanged
│   ├── outsourced-it-support-cost/page.tsx ← noindex shim, unchanged
│   ├── page.tsx                          ← home, unchanged
│   └── layout.tsx                        ← unchanged
├── components/
│   ├── Header.tsx, Footer.tsx            ← unchanged
│   ├── ArticleHeader.tsx                 ← NEW
│   ├── TakeawayBox.tsx                   ← NEW
│   ├── ArticleBody.tsx                   ← NEW (+ inline markdown parser, recurses into bold)
│   ├── FaqSection.tsx                    ← NEW
│   ├── CtaBox.tsx                        ← NEW
│   └── Breadcrumb.tsx                    ← NEW (handles 2-level + 3-level)
├── data/guides/                          ← NEW
│   ├── answering-services/  (5 JSON)
│   ├── fleet-tracking/      (5 JSON)
│   └── it-support/          (4 JSON)
└── lib/
    ├── site-config.ts                    ← NEW: APPROVED_GUIDE_SLUGS (length 14)
    └── guides.ts                         ← NEW: GuideSchema + loadGuide + getAllGuides + generateMetadata
```

Zero remaining article `page.tsx` files. Migration goal achieved.

### Schema deviations from plan (all approved, locked as new baseline)

1. **`intro: z.array(z.string()).max(5)`** — dropped `min(1)`. 10/14 articles have no intro paragraphs. Pipeline writer/auditor spec at `~/projects/ssg-content/CONTENT_SPEC_GUIDE_SSG.md` should reflect this (intro is optional). Not blocking.
2. **Parser recurses into `**bold**`** — required for nested-link rendering on `best-fleet-dash-cam`. Caught by visual diff before commit.
3. **`seo.ogTitle` / `seo.ogDescription` optional fields added** — required because 6/14 articles have unique OG copy that the original schema would have lost. Forward-compat for AI-generated articles.

### Mismatch fixes (4 shipped)

| # | Article | Old | New |
|---|---|---|---|
| 1 | best-answering-service-law-firm | "Also outsourcing IT?" → `/it-support/...` | "Comparing answering services?" → `/answering-services/best-answering-services/` |
| 2 | best-fleet-gps-tracking | "More fleet guides" → 3 links incl. 2 cross-category | "Building out your fleet stack?" → fleet-maintenance-software + fleet-tracking-contracts |
| 3 | managed-it-vs-break-fix | "Ready to compare providers?" → SELF-LINK + pricing | "Ready to compare providers?" → managed-it-services-pricing + best-it-support-law-firm |
| 4 | best-it-support-law-firm | Over-promise ("pricing and contract details") → vs-break-fix (delivers neither) | Same heading → managed-it-services-pricing + managed-it-vs-break-fix (delivers both) |

---

## CC PRE-FLIGHT HALTS (3 real catches)

1. **HALT at Phase 1 step 13** — Next 14 `output: 'export'` requires non-empty `generateStaticParams()`. Plan author missed this. Forced re-scoping: canary folded into Phase 1.
2. **HALT pre-canary-commit** — robots tag conditional spread bug + OG copy loss. Both fixed before commit, called out in commit body.
3. **HALT pre-dash-cam-commit** — parser regression on nested `<Link>` inside `<strong>`. Visual diff caught it. Parser fix bundled.

Each HALT prevented a real production regression. Pre-flight discipline working as designed.

---

## LESSONS LEARNED

### N=1 calibration is brittle (recurring pattern)

The `intro: min(1).max(5)` schema was anchored to the canary alone. Full corpus scan revealed 10/14 articles have no intro paragraphs. Same arc as the editor agent's threshold burn-in (3.8 → 3.6 across 4 slugs). Anytime you build a spec from one reference example, expect the corpus to break the assumption. Plan for the relax.

### Page-level vs component-level JSON-LD

Decision in this session: emit Article + FAQPage JSON-LD at page level in `app/[category]/[slug]/page.tsx`, not from inside components. Reason: cleaner separation (components render visible content, page emits schema), avoids double-emission if a component is reused. Applies to any future structured-data work.

### Pre-deletion HTML snapshots are the right diff baseline

CC's approach: capture `out/<cat>/<slug>/index.html` BEFORE deleting the JSX, run conversion, capture AFTER, diff. Cleaner than dev-mode rendering. Use for any JSX→JSON or HTML restructure work.

### Per-article commits paid off

13 separate commits + 3 chassis commits = 16 reversal points. If anything had gone wrong on article 9, articles 1-8 stayed shipped. Per-bundle commits would have forced a full rollback. Worth the marginal commit overhead.

### The plan doc was right about scope but wrong about route shape

Plan said `app/[slug]/page.tsx`. Reality is 2-level URLs. Operator locked `app/[category]/[slug]/page.tsx` before Phase 1 fired — caught by recon, fixed in spec, no rework. Pattern: plan docs need a recon pass before execution. The plan was a starting point, not a contract.

---

## DEFERRED ITEMS

### New this session

**D-SSG-LB-01 — pipeline writer spec needs `intro` optional + `seo.og*` fields**
- Title: SSG pipeline writer spec should reflect Lane B schema reality
- Description: `~/projects/ssg-content/CONTENT_SPEC_GUIDE_SSG.md` v1.0 likely mandates intro paragraphs and doesn't mention `seo.ogTitle`/`seo.ogDescription`. AI-generated articles will need to know intro is optional and unique OG copy is supported. Quick spec update.
- Trigger: before Lane C activates SSG pipeline (before `ssg.yaml: enabled: true`)
- Estimate: 15-30 min CC

**D-SSG-LB-02 — leftover "best outsourced IT support services" link-text awkwardness (2 articles)**
- Title: managed-it-services-pricing + outsource-help-desk-guide have over-promise link text
- Description: Both CTAs use the phrase "best outsourced IT support services" pointing to destinations that aren't exactly that. Destinations roughly match the promise (per Phase 2 pre-flight) so left alone. If conversion data shows poor CTR on these CTAs later, fix.
- Trigger: post-AdSense + post-30-days-traffic on these 2 articles, OR opportunistically
- Fix: one-line sed edits per article
- Estimate: 10 min CC

### Carried forward from prior sessions (no change)

D025, D026, D028, D033, D039, D040, D041, D042, D043, D052, D053, D055, D057, D058, D060, D063, D065, D066, D069, D071, D072, D073, D074, D-SSG-01, D-SSG-02, D-SSG-03, D-SSG-04, D-SSG-06, D-SSG-07, D-SSG-08, D-SSG-09.

### Closed this session

- **Lane B execution** — wasn't tracked as a D-item but was the largest single-session execution gap in the project. Now closed.

### Affected by Lane B completion (status changes)

- **D059 (SSG deploy throttle)** — now genuinely actionable. Trigger fires when `ssg.yaml: enabled: true` flips. Pattern-copy from asbestos `3ed57dc`. Estimate 30-45 min CC.
- **D065 (SSG GSC submission queue hook)** — same activation trigger. Pattern-copy from asbestos `c56209f`. Estimate 15-20 min CC.

---

## HANDOFF — NEXT CHAT

### Read in this order

1. **This context save** (you're reading it)
2. `~/brain/projects/aiteam/PRIME.md` — points at POLICY/HANDOFF/DEFERRED
3. `~/brain/projects/aiteam/POLICY.md` — 7 locked operator policies
4. `~/brain/projects/aiteam/HANDOFF.md` — current state
5. `~/brain/projects/aiteam/DEFERRED.md` — D-SSG-LB-01 + LB-02 new; D059 + D065 newly actionable

### Current state at session close

| Surface | State |
|---|---|
| `~/projects/smartsourceguide/` | LIVE on production. cf78061 main. Backup tag pre-data-driven-rewrite at c05c1f6 on remote. |
| `~/projects/ssg-content/` (pipeline) | Cold. Ready to fire if operator authors `drafter_queue.txt` + `assignment-batch-*.md`. `ssg.yaml: enabled: false`. |
| Asbestos site | LIVE, 32 guides. Pipeline live. Awaiting operator slugs in drafter_queue.txt. |
| Mission bar | $0 / $200/mo. Lane B complete unlocks SSG agentic output. Revenue still gated on (Lane C OR asbestos slugs) + AdSense. |

### Opening moves for next chat

**1. Lane C (~60-90 min CC).** Three sub-builds, all pattern-copy from asbestos:
   - Flip `ssg.yaml: enabled: true` (10 min, includes regression test that asbestos pipeline still fires)
   - SSG deploy throttle (D059) — pattern-copy from asbestos `3ed57dc`, 30-45 min
   - SSG GSC submission queue hook (D065) — pattern-copy from asbestos `c56209f`, 15-20 min
   - End state: SSG pipeline can ship JSON → site renders it → throttle previews at 20:00 PT, deploys at 23:00 PT → GSC queue tracks the URL for submission

**2. D-SSG-LB-01** (~15-30 min CC) — update SSG pipeline spec to reflect schema reality. Can land before, during, or after Lane C; lightweight. Recommend bundling with Lane C step 1 (the ssg.yaml flip).

**3. Test fire on SSG.** After Lane C lands, operator drops one keyword-config-ready slug into `~/projects/ssg-content/content/ssg/drafter_queue.txt`. Drafter cron fires, writer/auditor run, GEO + editor gates fire, ship-to-site stages, 20:00 preview pings to Telegram, 23:00 deploys live. End-to-end SSG loop closed.

### Operator-only revenue track (unchanged, gates everything)

- **Asbestos slugs into `drafter_queue.txt`** — Ahrefs keyword research blocking. Loop is starving.
- **AdSense enrollment for asbestoshq.com** — 32 articles live, zero monetization. Start the days-to-weeks clock.
- **AdSense enrollment for smartsourceguide.com** — same.

### Critical context for next Claude

- **SSG live-site bytes have changed.** The 14 articles now render via dynamic route with new JSON-LD, `<time>` tags, and 4 fixed CTAs. If anything regresses, `git revert` per-article-commit or `git reset --hard pre-data-driven-rewrite` for full rollback. Backup tag is the safety net.
- **Schema baseline is the post-Phase-2 state**, not the original plan. Three deviations are locked: `intro: max(5)` (no min), parser recurses into bold, `seo.ogTitle/ogDescription` are optional schema fields.
- **Pipeline-to-site contract is now real.** AI agents in `~/projects/ssg-content/` produce JSON; site at `~/projects/smartsourceguide/` consumes it. Both must agree on schema. D-SSG-LB-01 closes the writer-spec gap.
- **CC's `static-output diff` approach** (compare `out/<cat>/<slug>/index.html` BEFORE vs AFTER) is the right pattern for any future HTML-impacting changes. Use it.
- **Operator's research-before-building bias was absent this session.** Direct execution. Sustain it.

### Do NOT

- Push article-level changes to SSG without re-running visual diff against the locked baseline
- Forget that pipeline writer spec (`CONTENT_SPEC_GUIDE_SSG.md`) may now disagree with the live site schema until D-SSG-LB-01 lands
- Build Lane C and then immediately fire content through it without operator authoring at least one assignment-batch file
- Touch `ssg.yaml: enabled: true` before Lane C deploy throttle (D059) is in place — would risk content shipping without the 3-hour review window

---

## SIGN-OFF NOTE

16 commits, 14 articles migrated, 4 mismatch fixes shipped, 2 schema deviations honestly flagged and approved, 3 CC pre-flight HALTs caught real production regressions, zero premature ✅, zero misleading commit messages.

Foundation is materially harder than session start. The SSG pipeline is no longer building toward an imaginary destination — there's a real site that consumes its output. Lane C is mechanical pattern-copy from asbestos. After Lane C + one operator keyword session, SSG closes its loop the same way asbestos did on 2026-05-16.

Mission bar: $0 / $200 mo. Two of the three gates (Lane B, pipeline) are now closed for SSG. Lane C is the third. After that, revenue is gated only on keyword research + AdSense enrollment.

---

*End of context save. Resume next session by reading this file first, then PRIME.md. Lane C is the natural opener — `~60-90 min CC, three sub-builds, pattern-copy from asbestos.*
