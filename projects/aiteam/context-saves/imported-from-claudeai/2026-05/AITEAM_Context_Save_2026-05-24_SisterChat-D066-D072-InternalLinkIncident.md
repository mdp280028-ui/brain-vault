# AITEAM Context Save — Sister Chat: D066 + D072 + Internal-Link Incident

**Date:** 2026-05-24
**Session topic:** Sister-chat `~/agents/` lane work — D066 audit, D072 truncation fix, internal-link false-positive incident, lane closeout
**Companion chat:** Sister chat to `~/projects/asbestos-contractors/content/run-batch.sh` lane (GEO Phase 3 + D061 + D052 + dead-code cleanup). No file collisions this session.

---

## TL;DR

Sister-chat lane partial closeout. **3 of 4 items done**, 1 partial:

- ✅ **D066 ~/agents/ half** — re-audit confirms clean tree, no untracked code, no modifications. Auto-commit 3631871 from 2026-05-23 absorbed prior drift.
- ✅ **D072 editor score.sh truncation** — raised cap from 8KB → 60KB (env-overridable via `EDITOR_MAX_BODY_BYTES`). TRUNCATED audit flag stays as canary. Smoke test on 11,688B body confirmed `truncated=false`.
- ✅ **Internal-link fix lane** (UNPLANNED — scope grew from "audit" to "incident + fix"): 99 path-prefix "fix" was a false positive that broke 99 working body links; reverted; 44 dead-target links repointed correctly bare→bare; D091 logged for v1 audit_links.sh blind spots.
- 🟡 **D074 editor + GEO gate live verify** — still passive, triggers when next new slug ships through asbestos pipeline.
- ⏳ **Internal Link Agent SSG-fy + D091 patch** — deferred to fresh session. Scope grew from 2-3h to 3-4h after this session's findings.

Net: site is in better state than at session start (44 previously-dead-target links now resolve via render-time rewrite), zero regression from the mid-session mistake, lessons captured.

---

## Session arc

1. **Sister-chat split locked.** Operator handed off the 4-item `~/agents/` lane: D066 → D074 (passive) → D072 → Internal Link Agent. Sequence chosen to audit tree before adding new code to it.
2. **D066 audit ran.** Clean: 0 untracked, 0 modified, 0 staged. The 23:55 auto-commit cron is doing its job.
3. **D072 fix.** Picked Strategy A (raise cap, not chunk-and-aggregate). Single-line literal change + env-override. Smoke test confirmed the prior 8KB cap was creating false negatives — same guide's hook score went 2→3 with full body visible. Capped at 60KB. Threshold (3.6) deliberately left unchanged.
4. **Internal Link Agent recon.** Headline finding: v1 agent already exists at `~/agents/internal-link/` (committed 928671a 2026-05-17). Not greenfield — SSG retrofit task. Recon also surfaced 44 "potentially dead" asbestos link targets.
5. **Dead-link triage triggered.** Live HTTP probes showed every internal `/<slug>/` link in the asbestos guide corpus returns 404 (~124 link occurrences). Scope ballooned from 44 to 143 links across 32 guides.
6. **The mistake.** I diagnosed all ~99 path-prefix occurrences as "broken" based on raw URL HTTP probes. Did not read the site's render-time code path. Rewrote bare `/<slug>/` to `/guides/<slug>/` in JSON across 32 files. Committed and deployed.
7. **CC caught it.** When verifying deploy, CC read `parseInlineLinks` in `src/components/GuideArticle.tsx` and discovered the component takes BARE `/<slug>/` and rewrites at render time to `/guides/<slug>/` if the slug is in `APPROVED_GUIDE_SLUGS`. My "fix" prepended `/guides/` so the runtime check failed → 99 working links stripped to plain text.
8. **Recovery.** Two reverts (newest first), then a corrected dead-target pass that rewrites bare→bare (no `/guides/` prefix), letting the existing render-time logic handle it.
9. **Closeout.** LESSONS.md updated. D091 logged (covers v1 audit_links.sh blindness to body paragraphs + render-path mismatch). Lane closed.

---

## What got built / committed

### ~/agents (agents-vault)

| SHA | What |
|---|---|
| `deea54d` | fix(editor): raise score.sh body cap 8KB → 60KB (D072) |

### ~/projects/asbestoshq-site

| SHA | What |
|---|---|
| `36cb356` | **REVERTED** — fix(internal-links): rewrite bare /slug/ → /guides/slug/ across 32 guides (the mistake) |
| `cd03847` | **REVERTED** — fix(internal-links): repoint 44 dead-target links to live guides (the mistake's second step) |
| `711748b` | Revert cd03847 |
| `81ab725` | Revert 36cb356 — restored 99 working body links |
| `c0b6471` | fix(internal-links): repoint 44 dead-target links to live approved slugs (CORRECTED, bare→bare form) |

Final state on asbestos: original bare-link JSON form restored + 44 previously-dead-target references now resolve to live guides via render-time rewrite.

### ~/brain (brain-vault)

| SHA | What |
|---|---|
| `e081598` | chore(deferred): close D072 (editor score.sh truncation cap raised) |
| `80469d4` | lessons: verify render path not raw URLs (2026-05-24 internal-link incident) |
| `c9a69b0` | deferred: add D091 (audit_links.sh body-paragraph + render-path blindness) |

D066 close-line was appended to existing entry (likely landed in brain auto-commit later, not its own commit this session).

---

## Decisions made (and why, briefly)

### D072 Strategy A (raise cap) over Strategy C (chunk-and-aggregate)
Sonnet's 200K context handles full guide bodies in one call. Building chunking machinery for hypothetical 20k-word future guides is the research-before-building pattern. TRUNCATED flag stays as canary; if a future guide exceeds 60KB we revisit. Env-overridable.

### Dead-target mapping (b) "repoint to closest live guide" over (a) un-sanitize Phase-2 backup or (c) drop link wrapper
- (a) is a 1.5MB service-page resurrection — separate Phase-2 project.
- (c) loses SEO value the link was placed for.
- (b) gives readers a real working link with related content.

### Final mapping (after the double-link issue surfaced):
| Dead target | Repoint to |
|---|---|
| `/asbestos-abatement/` | `/asbestos-abatement-near-me/` |
| `/asbestos-inspection/` | `/asbestos-inspection-cost/` |
| `/asbestos-removal/` | `/asbestos-remediation-cost/` |
| `/asbestos-testing/` | `/how-to-test-popcorn-ceiling-for-asbestos/` |

The testing→how-to-test path triggers the self-link drop rule on one file: `how-to-test-popcorn-ceiling-for-asbestos.json` had `[asbestos testing](/asbestos-testing/)` which collapsed to plain prose "asbestos testing" rather than self-linking.

### Bare→bare rewrite (no /guides/ prefix in JSON)
The component does the rewrite at render time. JSON stays bare to match the existing convention.

---

## The mistake — what to remember

**Root cause:** Read paragraphs[] markdown, saw `[anchor](/slug/)`, ran `curl https://www.asbestoshq.com/<slug>/` → 404 → concluded "broken." Did not read `parseInlineLinks` to see what the component does with that markdown at render time.

**The component (`src/components/GuideArticle.tsx:71-120`):**
- `pathToSlug("/asbestos-siding-guide/")` → `asbestos-siding-guide`
- If slug in `APPROVED_GUIDE_SLUGS` → renders `<Link href="/guides/asbestos-siding-guide/">` ✓
- If not in whitelist → strips wrapper to plain text

**Why it stayed undetected for hours after deploy:**
- My post-deploy "verification" counted `/guides/` hrefs in rendered HTML. The page DID have those — but they were coming from the `relatedLinks[]` "Related Reading" section (independent code path), not from `parseInlineLinks` rendering body paragraphs.
- I never distinguished body links (class `guide-inline-link`) from related-reading links (inline style `font-weight:500`).

**Lesson captured in LESSONS.md (2026-05-24 entry):** Before fixing link formatting on a React/Next.js site, grep the JSON field's render function. If bare-URL form is wrong-looking but rendered output is correct, the renderer is doing work. Verify against `curl <page> | grep guide-inline-link` (or equivalent body-link class for the site), not against raw URLs.

**Corollary added:** HTML attribute order is not stable. Grepping `class="x"[^>]*href="y"` and `href="y"[^>]*class="x"` are different patterns. CC's first verify-pass returned 0 matches due to this; second pass corrected.

---

## New DEFERRED items captured this session

| ID | Topic | Trigger |
|---|---|---|
| **D091** | v1 `audit_links.sh` blind to body-paragraph links + render-path mismatch | During the SSG-fy v1 pass — extend audit to handle both site renderers correctly while in the file |

D091 was originally planned as D087 but D087 was already taken (idea-agent v1 follow-ups). Highest existing D-number is D090; took D091. The parallel-chat D-number-collision pattern is recurring — periodic `grep -oE "\bD[0-9]+\b" projects/aiteam/DEFERRED.md | sort -u -V` continues to be the right sync.

---

## What was left mid-task

### Internal Link Agent SSG-fy + D091 patch — deferred to fresh session

**Scope (revised, 3-4h not 2-3h):**

1. **SSG-fy v1** — multi-site schema:
   - `audit_link_failures` add `site` column
   - `backlink_proposals` add `site` column (current `UNIQUE(new_slug, source_slug)` will collide once SSG and asbestos share a slug literal)
   - `internal_link_inventory.site` already has DEFAULT 'asbestos' — SSG rows need explicit `site='ssg'` on insert
2. **propose_backlinks.sh asbestos filter** — currently `WHERE slug != '$NEW_SLUG' AND site = 'asbestos'`. Parameterize for site.
3. **CLAUDE.md prompts** — current opener: "You maintain a live SQLite inventory of every approved asbestos guide…". Needs neutralizing.
4. **JSON schema differences** — asbestos has flat `paragraphs[]`, SSG has `body: [{type:"h2"|"p", text:"..."}, ...]`. Build_inventory.sh and audit_links.sh both need to handle both shapes.
5. **Path handling** — asbestos slug = bare; SSG slug = `category/slug`. Decision was: bare slug + separate `category` column in DB.
6. **D091 audit upgrade** — audit_links.sh should:
   - Parse markdown links out of body paragraphs (asbestos `paragraphs[]`, SSG `body[].text`)
   - Mirror the site's render-time path logic per-site (asbestos: bare slug → `/guides/<slug>/` if approved; SSG: `/<category>/<slug>/`)
   - "Broken" = link's resolved slug not in APPROVED_GUIDE_SLUGS, OR is self-link, OR markdown malformed
7. **Re-ship path for SSG** — asbestos v1 writes to `~/agents/ship-to-site/state/updated_slugs_queue.txt`. SSG deploy path differs (D065 gsc hook deferred until first SSG batch). Pattern of "edit JSON → enqueue → cron re-ships" needs an SSG equivalent.

### D074 — still passive
Editor + GEO gates live verification fires when next new approved slug ships through asbestos pipeline. Cheapest item in the lane; no dedicated session needed.

---

## Open decisions (live design space)

No new ones from this session. Carrying forward:
- The 7 operator-policy questions from cross-agent failure modes audit §7 (still blocking D056 editor production runner)

---

## Coordination notes with the other chat

This chat owned: `~/agents/editor/score.sh` (D072), `~/agents/` audit recon (D066), `~/projects/asbestoshq-site/src/data/guides/*.json` (internal-link fix lane).

Sister chat (the run-batch.sh lane): owned `~/projects/asbestos-contractors/content/run-batch.sh` + `~/agents/assignment-drafter/drafter.sh`.

**No collisions this session.** Different files entirely. Status of the other chat's items unknown at time of save — if they completed GEO Phase 3 (touches `run-batch.sh` to add a writer-revision loop), the editor cap raise here is unaffected; if they completed D061 (rewires `claude -p` → `ai-do.sh` in `run-batch.sh`), that's also unaffected by editor changes.

---

## Mission-bar nuance reminder

Dashboard "API cost" figures are Max-sub burn, not real $. Real-money cost tonight = $0. ~3 hours of CC work cost nothing in dollars; only Max-subscription capacity consumed.

---

## Lessons learned

### Verify render path, not raw URLs
Captured verbatim in LESSONS.md (2026-05-24 entry). The single most expensive lesson of the session.

### Read the component before "fixing" data
Adjacent to the render-path lesson but distinct. The asbestos site has THREE rendering surfaces for internal links: `parseInlineLinks` (body paragraphs, runtime rewrite), `relatedLinks[]` (Related Reading section, direct render), and sitemap (build-time, separate logic). Diagnosing link health requires knowing which surface a given link lives in.

### Pre-flight assumption-validation discipline catches real bugs
CC's "read parseInlineLinks first" instinct caught my mistake before I could pile on. The recon → dry-run → apply → verify → commit pattern with explicit "stop and report" gates is what made the recovery cheap (2 reverts, no data loss). Compressing the pattern to "just fix it" would have left the site silently degraded.

### Sample-of-one rescores aren't recalibration
D072 smoke test showed a previously-failed guide passing 3.40→3.60 with the larger context window. Tempting to read this as "threshold should change." It's not — it's evidence the truncation was creating false negatives, not evidence the rubric or threshold needs adjustment. Threshold (3.6) left unchanged; revisit only with full burn-in distribution.

### D-number collisions across parallel chats are real and recurring
D087 was taken before I picked it; CC caught the collision via grep against existing DEFERRED.md. The documented `grep -oE "\bD[0-9]+\b" ~/brain/projects/aiteam/DEFERRED.md | sort -u -V` sync remains the right fix.

---

## Commits this session — consolidated

| Repo | SHA | What |
|---|---|---|
| ~/agents | `deea54d` | fix(editor): D072 truncation cap 8KB → 60KB |
| ~/projects/asbestoshq-site | `36cb356` | (REVERTED) fix(internal-links): 99 path-prefix |
| ~/projects/asbestoshq-site | `cd03847` | (REVERTED) fix(internal-links): 44 dead-target /guides/ form |
| ~/projects/asbestoshq-site | `711748b` | revert cd03847 |
| ~/projects/asbestoshq-site | `81ab725` | revert 36cb356 |
| ~/projects/asbestoshq-site | `c0b6471` | fix(internal-links): 44 dead-target bare→bare (corrected) |
| ~/brain | `e081598` | chore(deferred): close D072 |
| ~/brain | `80469d4` | lessons: verify render path not raw URLs |
| ~/brain | `c9a69b0` | deferred: add D091 |

All pushed to respective remotes.

---

## Next session start point

**For the SSG-fy work:** open with the 7-item scope above. Start with a recon of v1's current state (confirm nothing has shifted since today's recon), then DB schema migration, then prompt/script SSG-awareness, then D091 audit upgrade. Estimated 3-4h.

**For D074:** no dedicated session. Surfaces when first new approved slug ships through asbestos pipeline.

**Cheapest path:** SSG-fy in a fresh chat. The D091 audit upgrade is in scope of that work, so don't try to peel it off.
