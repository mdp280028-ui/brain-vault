# AITEAM — SSG Data-Driven Rewrite Plan

**Date:** 2026-05-17 (recovered & reconstructed from 2026-05-16 authoring session, chat `5c03da96-ae1c-4431-851e-41a618741049`, via conversation_search 2026-05-17)
**Original filename referenced as:** `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md` (original lost — not on Mac, not in Claude project)
**Site repo:** `~/projects/smartsourceguide/` (local) → `git@github.com:mdp280028-ui/smartsourceguide.git` (remote)
**Reconstruction note:** Body content is verbatim from original authoring session. Operator answers to the 4 open questions are locked from the 2026-05-16 context save.

---

## Goal

Convert the 14 JSX-inlined articles at `~/projects/smartsourceguide/app/<slug>/page.tsx` into a single dynamic route `app/[slug]/page.tsx` reading from `data/guides/<slug>.json`. Adds `zod` as the only new dependency. Per-article commits for clean revertibility. Site renders identically to the live version (modulo new JSON-LD in `<head>`) at every step.

Unblocks: SSG pipeline output consumption (`ssg-content/` writes JSON in `SSG_OUTPUT_FORMAT.md v1.0` shape → site renders it). `ssg.yaml: enabled: true` is gated on this rewrite completing.

---

## Current state (from 2026-05-16 articles structure recon)

- 14 articles under `app/` at repo root (NOT 15, NOT `src/app/`)
- Bare Next 14 + React 18 + TS, no Tailwind, hand-rolled CSS
- Categories: 5 fleet-tracking, 4 it-support, 5 answering-services
- Universal skeleton on every article:
  1. Breadcrumb (2-level on 10 articles, 3-level on 4)
  2. `<article-header>` — title + intro paragraph
  3. `<takeaway-box>` — single key takeaway block
  4. Body — `<h2>`, `<h3>`, `<p>` only, with inline `<strong>` and `<Link>`. No lists, tables, images, code blocks.
  5. `<faq-section>` — exactly 3 FAQs, every article
  6. `<cta-box>` — single CTA at end
- Known issues:
  - No JSON-LD schema markup (despite structured FAQs)
  - No author / datePublished / dateModified metadata
  - "Last updated April 2026" hardcoded identically on all 14
  - 4 articles have CTA copy / link-target mismatches (flagged in recon, will fix during migration)

---

## Target state

```
~/projects/smartsourceguide/
├── app/
│   ├── [slug]/
│   │   └── page.tsx                ← ONE dynamic route, renders any guide from JSON
│   ├── layout.tsx                  ← unchanged
│   ├── page.tsx                    ← home, unchanged
│   ├── about/page.tsx              ← unchanged
│   └── (noindex shims)             ← unchanged
├── data/
│   └── guides/
│       ├── <slug-1>.json
│       ├── <slug-2>.json
│       └── ... (14 files initially, grows over time)
├── components/
│   ├── Header.tsx                  ← unchanged
│   ├── Footer.tsx                  ← unchanged
│   ├── ArticleHeader.tsx           ← NEW, extracted from skeleton
│   ├── TakeawayBox.tsx             ← NEW
│   ├── ArticleBody.tsx             ← NEW, renders body block array
│   ├── FaqSection.tsx              ← NEW, renders FAQ array + emits JSON-LD
│   ├── CtaBox.tsx                  ← NEW
│   └── Breadcrumb.tsx              ← NEW, handles 2-level + 3-level
├── lib/
│   ├── site-config.ts              ← APPROVED_GUIDE_SLUGS lives here (to match asbestos pattern)
│   └── guides.ts                   ← helper: loadGuide(slug), getAllSlugs(), generateMetadata()
└── ... (config files unchanged)
```

After conversion:
- The 14 original `app/<slug>/page.tsx` files are deleted
- One `app/[slug]/page.tsx` handles all of them dynamically
- Adding a new article = drop a JSON in `data/guides/`, add slug to whitelist, build, ship
- Ship-To-Site SSG config can now be `enabled: true`

---

## JSON schema

One schema covers all 14 existing articles and all future ones. Designed for forward use (Ship-To-Site, pipeline output) and SEO completeness.

### Full schema

```json
{
  "schema_version": "1.0",
  "slug": "string (kebab-case, matches filename)",
  "title": "string (H1 + <title>)",
  "metaTitle": "string (60 chars max, for <title> if different from H1)",
  "metaDescription": "string (155 chars max)",
  "category": "string (e.g., 'fleet-gps', 'answering-services', 'it-msp', 'b2b-services')",
  "breadcrumb": {
    "items": [
      { "label": "string", "href": "string" }
    ]
  },
  "datePublished": "ISO 8601 date (e.g., '2026-04-01')",
  "dateModified": "ISO 8601 date",
  "author": {
    "name": "string",
    "url": "string (optional)"
  },
  "intro": "string (1–2 paragraph lead under H1, plain text with inline markdown allowed)",
  "takeaway": {
    "label": "string (e.g., 'Key Takeaway')",
    "body": "string (1–3 sentence summary)"
  },
  "body": [
    {
      "type": "h2",
      "id": "string (anchor slug, auto-generated if absent)",
      "text": "string"
    },
    {
      "type": "h3",
      "id": "string",
      "text": "string"
    },
    {
      "type": "p",
      "text": "string (markdown: **bold**, [link](url), inline only)"
    }
  ],
  "faq": [
    {
      "question": "string",
      "answer": "string (markdown allowed, inline only)"
    }
  ],
  "cta": {
    "heading": "string",
    "body": "string",
    "buttonText": "string",
    "buttonHref": "string"
  },
  "relatedLinks": [
    {
      "slug": "string (must exist in APPROVED_GUIDE_SLUGS)",
      "title": "string (override, optional — defaults to target's title)"
    }
  ],
  "seo": {
    "noindex": "boolean (default false)",
    "canonical": "string (optional override)",
    "ogImage": "string (optional, path or URL)"
  }
}
```

### Schema notes

- **`body` is an array of typed blocks.** Future expansion (lists, tables, images, code) adds new `type` values without breaking existing JSONs. For now, all 14 articles use only `h2`, `h3`, `p`.
- **Inline markdown in text fields** — body paragraphs, FAQ answers, intro all support `**bold**` and `[text](url)`. Renderer parses these. Keeps JSON authoring readable.
- **Anchors auto-generate** from heading text if `id` absent — kebab-case slug.
- **`relatedLinks` is structured, not free-form.** Each entry must resolve to an existing approved slug. Ship-To-Site filters dead links automatically (already specced).
- **`category` is required.** Drives navigation, internal linking, programmatic SEO later.
- **`schema_version`** so future migrations know what they're reading.

### Schema validation

Add `scripts/validate-guide.ts` that runs in `npm run build` prep:
- Load every JSON in `data/guides/`
- Validate against schema (use `zod` or hand-rolled — zod is one small dep, worth it)
- Fail build if any JSON malformed
- Print clean error: "guides/fleet-gps-comparison.json: missing required field 'datePublished'"

---

## Template (`app/[slug]/page.tsx`)

Single dynamic route. Reads slug from URL, loads JSON, renders.

```tsx
// app/[slug]/page.tsx
import { notFound } from 'next/navigation'
import { loadGuide, getAllSlugs } from '@/lib/guides'
import { APPROVED_GUIDE_SLUGS } from '@/lib/site-config'
import { ArticleHeader } from '@/components/ArticleHeader'
import { TakeawayBox } from '@/components/TakeawayBox'
import { ArticleBody } from '@/components/ArticleBody'
import { FaqSection } from '@/components/FaqSection'
import { CtaBox } from '@/components/CtaBox'
import { Breadcrumb } from '@/components/Breadcrumb'

export async function generateStaticParams() {
  return getAllSlugs().map(slug => ({ slug }))
}

export async function generateMetadata({ params }: { params: { slug: string } }) {
  const guide = loadGuide(params.slug)
  if (!guide) return {}
  return {
    title: guide.metaTitle || guide.title,
    description: guide.metaDescription,
    alternates: { canonical: guide.seo?.canonical },
    robots: guide.seo?.noindex ? { index: false } : undefined,
  }
}

export default function GuidePage({ params }: { params: { slug: string } }) {
  if (!APPROVED_GUIDE_SLUGS.has(params.slug)) notFound()
  const guide = loadGuide(params.slug)
  if (!guide) notFound()

  return (
    <main className="guide">
      <Breadcrumb items={guide.breadcrumb.items} />
      <ArticleHeader
        title={guide.title}
        intro={guide.intro}
        datePublished={guide.datePublished}
        dateModified={guide.dateModified}
        author={guide.author}
      />
      <TakeawayBox label={guide.takeaway.label} body={guide.takeaway.body} />
      <ArticleBody blocks={guide.body} />
      <FaqSection items={guide.faq} />
      <CtaBox {...guide.cta} />
    </main>
  )
}
```

### Component responsibilities

| Component | Renders | Notes |
|---|---|---|
| `Breadcrumb` | 2-level or 3-level breadcrumb | Just iterates `items` array — depth handled by data |
| `ArticleHeader` | H1, intro, datePublished, dateModified, author | Emits Article JSON-LD schema |
| `TakeawayBox` | Single takeaway block | One paragraph |
| `ArticleBody` | h2/h3/p blocks from `body` array | Renders inline markdown (bold, links) |
| `FaqSection` | FAQ Q&A pairs + **emits FAQ JSON-LD schema** | Fixes the missing structured data gap |
| `CtaBox` | Heading, body, button | Single CTA |

### CSS

Keep existing hand-rolled CSS approach (no Tailwind migration). Lift styles from existing page.tsx files into a single `styles/guide.css` or co-located `*.module.css` per component. No visual change to live site.

---

## Helper: `lib/guides.ts`

```ts
import fs from 'fs'
import path from 'path'

const GUIDES_DIR = path.join(process.cwd(), 'data', 'guides')

export function getAllSlugs(): string[] {
  return fs
    .readdirSync(GUIDES_DIR)
    .filter(f => f.endsWith('.json'))
    .map(f => f.replace('.json', ''))
}

export function loadGuide(slug: string) {
  const file = path.join(GUIDES_DIR, `${slug}.json`)
  if (!fs.existsSync(file)) return null
  return JSON.parse(fs.readFileSync(file, 'utf-8'))
}
```

Stays sync — static export, build-time only, no perf concern at 14–500 guides.

---

## Migration approach

### Phase 1 — Build the chassis (no content moved yet)

1. Create `data/guides/` directory (empty)
2. Create all 5 new components in `components/` with CSS extracted from one existing page.tsx
3. Create `app/[slug]/page.tsx` dynamic route
4. Create `lib/guides.ts` + `lib/site-config.ts` with empty `APPROVED_GUIDE_SLUGS = new Set()`
5. Create `scripts/validate-guide.ts` schema validator
6. Add `zod` dependency
7. Run `npm run build` — should succeed with zero guides
8. Commit

**At this point:** Original 14 articles still live at `app/<slug>/page.tsx`. New chassis exists but renders nothing. Site is fully functional unchanged.

### Phase 2 — Convert one article end-to-end (canary)

1. Pick the cleanest of the 14 (not one of the 4 with mismatches) — recommend whichever is most representative
2. CC extracts the page.tsx content into a JSON file matching schema
3. Add slug to `APPROVED_GUIDE_SLUGS`
4. **Delete** original `app/<slug>/page.tsx`
5. Run `npm run build` — confirm the dynamic route renders
6. Run `npm run start` locally, hit the URL, **diff visually** against pre-migration screenshot
7. Commit

**Pass condition:** The canary article renders identically (modulo new JSON-LD in `<head>`) to the original. If it doesn't, fix the template, not the JSON.

### Phase 3 — Convert remaining 13

1. CC converts each remaining page.tsx → JSON, deletes original, adds to whitelist
2. **For the 4 known mismatches:** CC flags each during conversion, applies the fix (CTA copy or link target), notes the fix in conversion log
3. After every 3–4 conversions: `npm run build` + spot-check live render
4. Commit per article (clean revertibility)

### Phase 4 — Validation pass

1. Visual diff every converted article against pre-migration snapshot
2. Run `npm run build` clean
3. Confirm `validate-guide.ts` passes on all 14 JSONs
4. Confirm sitemap.xml still includes all 14 URLs
5. Verify FAQ JSON-LD renders correctly (use Google Rich Results Test on one URL — offline tool, no live site needed for validation)

### Phase 5 — Ship-To-Site enablement

1. Flip `ssg.yaml: enabled: true` (only after Phases 1–4 ✅)
2. Pipeline output (`ssg-content/` writes JSON in `SSG_OUTPUT_FORMAT.md v1.0` shape) now consumed directly by `[slug]/page.tsx` dynamic route
3. D059 (SSG deploy throttle, pattern-copy from asbestos 3ed57dc) triggers
4. D065 (SSG `gsc_submission_queue` INSERT hook, pattern-copy from asbestos c56209f) triggers

---

## Locked answers to the 4 open questions (per 2026-05-16 session)

| # | Question | Locked answer |
|---|---|---|
| 1 | Author name to backfill all 14 | **`"SmartSourceGuide Editorial"`** (single persona, can fragment later if desired) |
| 2 | datePublished backfill strategy | **CC pulls git log first-commit date per file**; fallback `"2026-04-01"` if no git history |
| 3 | Canary article pick | **CC picks** — 2-level breadcrumb, NOT on the 4-mismatch list, ~130 lines representative size |
| 4 | Migration timing | **After Ship-To-Site ✅, before SSG content production resumes** (Ship-To-Site is ✅ as of 2026-05-16 commit 3b43c11) |

All four locked. No further operator input needed before execution.

---

## Estimated CC time

3–5 hours total, broken across phases:
- Phase 1 (chassis): ~60–90 min
- Phase 2 (canary): ~30–45 min
- Phase 3 (13 articles): ~90–120 min (mostly mechanical conversion + fix-during-pass for 4 mismatches)
- Phase 4 (validation): ~30 min
- Phase 5 (enablement): ~10 min (config flips, triggers other deferreds)

Could span 2 sessions if operator prefers — natural break between Phase 2 (canary green-light) and Phase 3.

---

## Pre-execution checklist

Before kicking off Phase 1, confirm on-disk:

- [ ] `~/projects/smartsourceguide/` is clean (git status clean, no uncommitted work)
- [ ] Backup the repo state (`git tag pre-data-driven-rewrite` is the minimum; full local clone is safer)
- [ ] Confirm the live site is operational (curl one URL, expect 200)
- [ ] Pre-migration screenshots taken for 4 canary candidates (so visual diff in Phase 2 has a reference)
- [ ] `~/projects/ssg-content/` is NOT actively running (ssg.yaml: enabled: false confirmed — already true)

---

## Risks

1. **JSON-LD in `<head>` changes the HTML output even on visually identical pages.** Diff tools may flag this. Acceptable — this is an intentional SEO improvement, not a regression.
2. **The 4 CTA/link mismatches will get fixed during conversion.** This means the converted articles will differ from the live ones in those specific spots. Document the fix in the per-article commit message.
3. **`datePublished` backfill may produce surprising values** if git log shows the original commits were spread across many days. Locked answer says use git log; if dates look wrong, swap to the `2026-04-01` fallback uniformly.
4. **Sitemap regeneration must include all 14 dynamic routes.** If `sitemap.ts` enumerates routes from filesystem, it'll pick up the new `[slug]` dynamic route correctly. Verify in Phase 4.
5. **The `relatedLinks` feature may require backfilling links into existing articles.** If the live articles have no inter-article links, the JSONs will have empty `relatedLinks` arrays. This is OK — the feature is forward-facing for new content. Don't invent links retroactively.

---

## Recovery note

This document is a **2026-05-17 reconstruction from 2026-05-16 conversation history (chat 5c03da96-ae1c-4431-851e-41a618741049)**. The original `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md` was authored in that session as a file artifact but neither saved to disk nor mirrored into the Claude project. Recovered via `conversation_search` on 2026-05-17.

Body content is verbatim from the original (JSON schema, template code, helper code, phase structure, component table, CSS approach). The "Locked answers" section uses the answers preserved in the 2026-05-16 context save. The "Estimated CC time", "Pre-execution checklist", and "Risks" sections are additions for execution-readiness in 2026-05-17+.

When ready to execute: hand this document to CC as the spec for Phase 1. Update if scope changes.

---

*End of plan.*
