# page.tsx template extraction — 2026-05-16

Extracted from the live AsbestosHQ site for use by the Ship-To-Site agent's
`lib/stage.sh` codemod. Template output:
`~/agents/ship-to-site/templates/page-tsx-template.tsx`.

Read-only on `~/projects/asbestoshq-site/`. No edits, no git ops, no build.

---

## 1. Sample slugs chosen

All 31 per-slug `page.tsx` files under `src/app/guides/` were enumerated;
**every one is exactly 21 lines.** Byte differences (555-645 bytes) are pure
slug-length artifact. There is no true "short / mid / long" by line count, so
samples were chosen for **slug-shape diversity** instead:

| Sample | path | lines | bytes | slug shape |
|---|---|---|---|---|
| 1 | `src/app/guides/chrysotile/page.tsx` | 21 | 555 | single-word, shortest in repo |
| 2 | `src/app/guides/asbestos-shingles-guide/page.tsx` | 21 | 594 | typical multi-word `-guide` suffix |
| 3 | `src/app/guides/how-to-test-popcorn-ceiling-for-asbestos/page.tsx` | 21 | 645 | longest question-form slug in repo |

The full enumeration confirmed the line-count invariant: `wc -l` returned `21`
for all 31 wrappers, no outliers.

---

## 2. Diff summary

### Command + raw output

```
$ cd ~/projects/asbestoshq-site/src/app/guides
$ diff chrysotile/page.tsx asbestos-shingles-guide/page.tsx
3c3
< import data from '@/data/guides/chrysotile.json';
---
> import data from '@/data/guides/asbestos-shingles-guide.json';
10c10
<   alternates: { canonical: '/guides/chrysotile/' },
---
>   alternates: { canonical: '/guides/asbestos-shingles-guide/' },
14c14
<   url: '/guides/chrysotile/',
---
>   url: '/guides/asbestos-shingles-guide/',

$ diff asbestos-shingles-guide/page.tsx how-to-test-popcorn-ceiling-for-asbestos/page.tsx
3c3
< import data from '@/data/guides/asbestos-shingles-guide.json';
---
> import data from '@/data/guides/how-to-test-popcorn-ceiling-for-asbestos.json';
10c10
<   alternates: { canonical: '/guides/asbestos-shingles-guide/' },
---
>   alternates: { canonical: '/guides/how-to-test-popcorn-ceiling-for-asbestos/' },
14c14
<   url: '/guides/asbestos-shingles-guide/',
---
>   url: '/guides/how-to-test-popcorn-ceiling-for-asbestos/',
```

### What's identical across all 3

- Everything except 3 lines.
- All 18 non-slug lines (imports, type cast, metadata struct, openGraph
  shape minus the URL field, default export, Page body) byte-match.
- Trailing newline present in all 3.
- No TODO comments, no `// eslint-disable`, no per-page hardcoded overrides.

### What varies across all 3

- Lines 3, 10, 14 — each carries exactly one occurrence of the slug substring.
- Nothing else.

The wrapper is the cleanest possible: data-driven shell over a single shared
`<GuideArticle>` component, with three slug literals that could (and now
will) be machine-substituted.

---

## 3. Placeholder map

In the source wrappers (21-line bare form, no comment block):

| Template line | Context (3 lines around) |
|---|---|
| **3** | line 2: `import GuideArticle, { type GuideData } from '@/components/GuideArticle';`<br>line 3: `import data from '@/data/guides/<SLUG>.json';`<br>line 4: *(blank)* |
| **10** | line  9: `  description: guide.metaDesc,`<br>line 10: `  alternates: { canonical: '/guides/<SLUG>/' },`<br>line 11: `  openGraph: {` |
| **14** | line 13: `    description: guide.metaDesc,`<br>line 14: `    url: '/guides/<SLUG>/',`<br>line 15: `    type: 'article',` |

In the extracted template file (with the 22-line provenance comment block on
top) the same three slug-bearing lines live at **lines 25, 32, 36**. The
comment block documents this so future-CC reading the template knows where the
placeholder lives in BOTH coordinate systems.

The substitution pattern Ship-To-Site uses is awk `gsub`, which matches by
regex not by line number — so the comment-block prepend does not break
anything.

---

## 4. Data-driven fields (no placeholder needed)

Verified against 3 real `src/data/guides/*.json` files (chrysotile,
asbestos-shingles-guide, how-to-test-popcorn-ceiling-for-asbestos) — all three
have the same 15 top-level keys:

```
['slug', 'metaTitle', 'metaDesc', 'h1', 'h2s', 'paragraphs',
 'category', 'date', 'dateModified', 'authorityLinks', 'relatedLinks',
 'allText', 'author', 'lastReviewed', 'reviewedBy']
```

The wrapper directly reads exactly **2** of these:

| Field | Used at | Purpose |
|---|---|---|
| `metaTitle` | `title` in both `metadata` and `metadata.openGraph` | Next.js page title + OG title |
| `metaDesc` | `description` in both `metadata` and `metadata.openGraph` | Next.js meta description + OG description |

All 13 other fields (slug, h1, h2s, paragraphs, category, date, dateModified,
authorityLinks, relatedLinks, allText, author, lastReviewed, reviewedBy) are
passed straight through to `<GuideArticle data={guide} />`. The renderer
(`src/components/GuideArticle.tsx:42-200`, type `GuideData`) handles them
internally — wrapper is content-agnostic.

This means **Ship-To-Site does not need to look inside the JSON at all** to
generate the wrapper. The 3 slug substitutions are sufficient. (Whether
Ship-To-Site touches the JSON for *other* reasons — e.g. `relatedLinks`
filtering against the whitelist — is its own concern; the wrapper template
side has no JSON dependency.)

---

## 5. Hardcoded-per-page fields

**None found.**

Every wrapper field that varies between pages is either:
- Pulled from the imported `guide` JSON (`metaTitle`, `metaDesc`), or
- Derived from the slug (`canonical` URL, `openGraph.url`, JSON import path).

The hardcoded constants (`type: 'article'`, the GuideArticle import path,
the `as GuideData` cast) genuinely don't vary across the 31 shipped wrappers.
There are no `// TODO`, `// HACK`, or one-off overrides anywhere in the
sample. No Ship-To-Site concerns flagged.

---

## 6. Surprises

- **Zero variance beyond slug.** Across 31 pages I expected to find at least
  one outlier (someone forgot to update a field, or copied from a different
  template generation). None. Every wrapper is mechanically identical to
  every other wrapper modulo slug. Strong signal the existing manual
  copy-paste workflow has been disciplined.
- **The renderer's `GuideArticle` is bring-your-own-data.** It accepts a
  `GuideData` typed prop and handles all 15 fields itself, including
  schema.org JSON-LD injection, link-rewriting against
  `APPROVED_GUIDE_SLUGS`, and inline markdown link parsing. The wrapper is
  truly nothing more than a thin Next.js page shell.
- **No favicon, no Twitter card, no robots directive in the per-page
  metadata.** Those all live in `app/layout.tsx` (root layout) and apply
  globally. Wrapper only sets title/description/openGraph/canonical.
- **`alternates.canonical` and `openGraph.url` are duplicated**, with the
  same slug substring. Two separate substitutions, same value. Looks
  intentional — Next.js doesn't auto-derive one from the other.
- **No trailing slash inconsistency.** Both slug-bearing URL lines (10 and
  14) use trailing slash, consistent with `next.config.ts trailingSlash: true`
  (per `ssg_pipeline_recon_2026-05-16.md §9`).
- **The pre-existing template at `~/agents/ship-to-site/templates/page-tsx-template.tsx`
  was already byte-identical to what an extraction would produce.** This
  extraction's only material change is the added provenance comment block on
  top. The bare wrapper structure was correct in the prior session.

---

## 7. Template usage example (before / after)

### Before — template (slug-bearing lines only; comment block + non-varying lines elided)

```tsx
// ... 22-line provenance comment block ...
import type { Metadata } from 'next';
import GuideArticle, { type GuideData } from '@/components/GuideArticle';
import data from '@/data/guides/<SLUG>.json';                          ← line 25

const guide = data as GuideData;

export const metadata: Metadata = {
  title: guide.metaTitle,
  description: guide.metaDesc,
  alternates: { canonical: '/guides/<SLUG>/' },                         ← line 32
  openGraph: {
    title: guide.metaTitle,
    description: guide.metaDesc,
    url: '/guides/<SLUG>/',                                             ← line 36
    type: 'article',
  },
};

export default function Page() {
  return <GuideArticle data={guide} />;
}
```

### After — Ship-To-Site runs `awk -v slug="asbestos-pipe-insulation" '{ gsub(/<SLUG>/, slug); print }' template > page.tsx`

(Verified in the working tree — output below is the actual awk run, not
hypothetical:)

```tsx
// ... 22-line provenance comment block ...
import type { Metadata } from 'next';
import GuideArticle, { type GuideData } from '@/components/GuideArticle';
import data from '@/data/guides/asbestos-pipe-insulation.json';        ← line 25

const guide = data as GuideData;

export const metadata: Metadata = {
  title: guide.metaTitle,
  description: guide.metaDesc,
  alternates: { canonical: '/guides/asbestos-pipe-insulation/' },       ← line 32
  openGraph: {
    title: guide.metaTitle,
    description: guide.metaDesc,
    url: '/guides/asbestos-pipe-insulation/',                           ← line 36
    type: 'article',
  },
};

export default function Page() {
  return <GuideArticle data={guide} />;
}
```

Post-substitution self-check:
- `grep -c '<SLUG>'` in the output → **0** (no stray placeholders)
- `grep -c 'asbestos-pipe-insulation'` in the output → **3** (exact expected
  substitutions)

The substituted output **also carries the provenance comment block forward**
into the live page.tsx, with the slug interpolated where the comment said it
would appear. Operator-facing footprint: ~22 lines of comment per shipped
page.tsx file. Acceptable trade-off for permanent provenance tagging in the
live tree; if undesirable, the comment block can be stripped post-substitution
with a `tail -n +23` step in `stage.sh`.

---

## 8. Open questions for operator

1. **Should the comment block ship to the live site?** Current behavior:
   yes — awk just substitutes, doesn't strip. Each generated `page.tsx`
   carries the 22-line provenance comment. If the operator prefers leaner
   live pages, add a one-line strip to `lib/stage.sh` (`tail -n +23` after
   the awk pipe). The bare 21-line wrapper is what every existing shipped
   page looks like today — comment block is an opinionated addition.
2. **Is `openGraph` always `type: 'article'`?** All 31 shipped wrappers say
   so. If a future content type (FAQ page, listicle, calculator) would warrant
   `type: 'website'` or `'profile'`, the template needs a second variant — or
   the wrapper should derive `type` from `guide.category` or similar. No
   action needed now; flag if a non-guide page type lands.
3. **Should the `as GuideData` cast become a runtime validation?** Currently
   the cast trusts the imported JSON conforms. If the data layer ever drifts
   (Ship-To-Site copies in a malformed JSON, audit_guide.py misses a missing
   field), the page builds fine but renders broken. A runtime guard would
   catch it at build time. Out of scope for this extraction; flag for a
   future hardening pass.
4. **Are there sites beyond AsbestosHQ that will use this template?**
   Ship-To-Site `config/ssg.yaml` is currently disabled but exists. If SSG
   ever turns on, its wrapper shape may differ (SSG site is hand-built JSX,
   not data-driven per recon §10) — this template would need to be forked
   or made site-aware via the `templates/` directory layout
   (`page-tsx-asbestos-template.tsx`, `page-tsx-ssg-template.tsx`).

---

## Sign-off

- ✅ Template at `~/agents/ship-to-site/templates/page-tsx-template.tsx`
- ✅ Comment block: 3 sampled slugs, date 2026-05-16, placeholder line
  locations (25 / 32 / 36 in the template), plus what doesn't vary
- ✅ Actual `diff` command output pasted (§2)
- ✅ All 8 sections present
- ✅ Placeholder map has line numbers for both source (3/10/14) and template (25/32/36) coordinate systems
- ✅ Hardcoded-per-page fields surfaced — **none found**, explicit
- ✅ Usage example shows real before/after (verified with awk run, not
  hypothetical)
- ✅ All 3 samples are valid TSX (no syntax error to flag — they're verbatim
  copies of files that currently build and ship to Vercel)
