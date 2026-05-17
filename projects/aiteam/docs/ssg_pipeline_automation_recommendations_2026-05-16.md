# SSG Pipeline Automation Recommendations — 2026-05-16

Opinionated, stack-ranked recommendations for which manual steps an AITEAM agent should automate. Companion to `ssg_pipeline_recon_2026-05-16.md` — read that first.

Definitions:
- **Tier** is the smallest model that can plausibly handle the work (Haiku 4.5 / Sonnet 4.6 / Opus 4.7).
- **Build complexity** is rough engineering effort, not LLM cost: small = a day; medium = a week; large = multi-week.

---

## The map of manual steps (recon §12 order)

Each step below: what it is, why it's manual today, can an AITEAM agent automate it, and if yes/partial, the proposed shape.

### Step 1 — Author the assignment-batch markdown for each slug

What: 30-90 min of operator work per slug — keyword research, SERP scan, seed prose, writer notes. The 39 assignment-batch files in `_asbestos-reference/` show stable structure but the content of each is bespoke (seed paragraph, audience, regulatory entities, writer notes).

Why manual: the brief depends on judgment about which slug to target, what SERP looks like, what angle differentiates the guide. There is no upstream queue.

Automatable: **partial**.
- Stable structural fields (slug → expected H2 archetype, primary keyword from a research file, related-guide cross-link suggestions from existing approved-index) can be filled by a small agent reading the keyword-research report + approved-index + asbestos-authority-links file.
- Judgment fields — audience framing, seed paragraph that pre-decides hook angle, hand-picked manufacturer names ("Eternit, Johns-Manville"), the choice of which regulatory entities to require — should stay operator-authored or be drafted-for-review.

Proposed shape: **Assignment Drafter agent**. Tier = Sonnet. Inputs: target slug, existing keyword-config (if started), `Asbestos_Keyword_Research_Report_April2026.md`, `approved-index-asbestos.md`, `asbestos-authority-links.md`. Output: a markdown draft of the assignment-batch following the 11-field schema. Operator edits the seed paragraph and writer-notes bullets in-place, ships. Build complexity: **medium** — needs the agent to read the keyword research file (~62KB), pick a primary kw + secondaries with realistic ranges, scan SERP via web tool, and propose 4-5 required entities. Best-leverage if it can fill 70% of the markdown and the operator only edits seed + notes.

### Step 2 — Author the per-slug keyword-config JSON

What: 10-20 min/slug. Primary keyword, secondary ranges, ngram_ignore stopword set, intent_concepts (verbs), required_entities (regulatory citations), secondary_nouns (per-material vocabulary — note spec v2.5 explicitly says popcorn ceiling ≠ transite pipe ≠ black mastic vocabulary).

Why manual: ngram_ignore + secondary_nouns are tightly material-specific and tuned to what `audit_guide.py` SAT1 + G1 will actually accept. Each new slug invites a calibration round.

Automatable: **yes** (this is the highest-confidence win).
- All five fields can be derived from the slug + a single material-class label.
- `secondary_nouns` can be back-filled by scanning the body of an existing approved guide on the same material class (e.g., new floor-tile guide reuses much of `asbestos-floor-tile-removal.json`'s noun list).
- Primary/secondary ranges are formulaic (range [4,10] / [1,3]).

Proposed shape: **Config Synthesizer agent**. Tier = Haiku — this is essentially structured data assembly. Inputs: slug + primary keyword (1 line from operator). Output: a fully-populated `keyword-configs/<slug>.json`. Validates against `_TEMPLATE.json`. Build complexity: **small** — a couple hundred lines of agent + a small library of material-class → secondary_nouns mappings.

### Step 3 — Manual review of `needs-review/` escalations (3 failed rounds)

What: variable. Pipeline gives up after R3; the operator either rewrites by hand or kicks `--force` after editing the brief/config.

Why manual: a triage decision (root-cause the failure — was the brief wrong? config wrong? primary keyword too saturated? section topic doesn't map to a sub-query?).

Automatable: **partial**.
- The triage step (read the R3 feedback file + the failing draft + the assignment + the keyword config, classify the failure into {brief-wrong, config-wrong, kw-mistargeted, audit-threshold-too-tight, prompt-gap}) is well-suited to an LLM.
- Acting on the triage (re-running, editing the brief, retuning thresholds) should stay operator-confirmed.

Proposed shape: **Escalation Triage agent**. Tier = Sonnet. Inputs: the escalated `<slug>.json` + last `feedback/<slug>-r3.md` + assignment + keyword-config. Output: a short ranked diagnosis ("80% confidence: secondary_nouns list doesn't include 'chrysotile', G1 can never pass") + the 1-3 specific edits to try. Operator approves the edit + reruns. Build complexity: **small** if it ships as a one-shot post-batch report, **medium** if it should propose-and-apply patches.

### Step 4 — Copy approved JSON to site `src/data/guides/<slug>.json`

What: 1-2 min/slug. `cp ~/projects/asbestos-contractors/content/asbestos/approved/guides/<slug>.json ~/projects/asbestoshq-site/src/data/guides/<slug>.json`. The hash diff between the two trees shows the operator sometimes edits during the copy (e.g., adjusting `relatedLinks` slugs based on what's actually approved on the site).

Why manual: copy is trivial; the in-flight edit needs judgment.

Automatable: **yes** — the copy is trivial; the in-flight edit can be done by an agent that knows the site's current `APPROVED_GUIDE_SLUGS` and filters/corrects `relatedLinks` accordingly.

Proposed shape: **part of a larger Ship-To-Site agent** (see step 5). Tier = Haiku for the copy + Sonnet only if the relatedLinks rewrite needs reasoning. Build complexity: **small**.

### Step 5 — Create `src/app/guides/<slug>/page.tsx`

What: 2-3 min/slug. 21-line wrapper. Hand-copy from a template.

Why manual: nobody has written the generator.

Automatable: **yes** (this is a clear win).

Proposed shape: combined with step 4 + step 6 into a **Ship-To-Site agent** (or just a script — the LLM tier is overkill). Tier = Haiku or pure script. Inputs: the slug + the approved JSON. Outputs: copies JSON, generates page.tsx from template, appends slug to `APPROVED_GUIDE_SLUGS` Set, runs `npm run build` to verify, commits, pushes. Build complexity: **small** — could be a 100-line Node script. The LLM-agent layer is only needed if the agent should also clean up `relatedLinks` references that don't exist yet.

### Step 6 — Append slug to `APPROVED_GUIDE_SLUGS`

What: < 1 min. One line added to a TypeScript Set literal.

Why manual: nobody has written the codemod.

Automatable: **yes**. Part of the Ship-To-Site agent.

### Step 7 — `git push` and monitor `schema-check.yml`

What: 5-10 min including watching the CI land.

Why manual: there's nothing wrong with the operator pressing Enter; the cost is the 5-minute wait.

Automatable: **partial**. The push is fine to automate when bundled into Ship-To-Site. The CI monitoring is an attention drain — wrap it in a Watcher that pings the operator when schema-check fails.

Proposed shape: bundled. Build complexity: **small**.

### Step 8 — GSC submission

What: 2-5 min/slug, off-repo.

Why manual: operator-driven and outside both repos.

Automatable: **no, in practice**. Search Console doesn't expose URL-inspect / request-indexing in a reliable API contract; the indexing API exists but is for narrow use cases and risks abuse flags. Recommend leaving this manual.

### Step 9 — Pipeline orchestration choices on each run

What: picking the slug, `--force`, model flags, watching spinner.

Why manual: it's the operator's control surface.

Automatable: **partial**. The "pick the slug" question can be queue-driven — an Assignment Queue agent that reads the keyword research file + existing approved index + a priority list and proposes the next 3 slugs. The model-flag and `--force` decisions should stay operator-confirmed.

Build complexity: **small** (the queue agent is trivial; the value is mostly in lowering the activation energy to start a new batch).

### Step 10 — Maintaining the spec / pipeline itself

What: tweaking V5/V7/V8/V4c/G1 thresholds when failure patterns shift. v10.13 followed v10.10 by 24 hours.

Why manual: requires reading a corpus of recent failures and judging whether a threshold is too tight or whether the writer prompt needs a new constraint.

Automatable: **partial** — pattern detection is automatable; threshold changes should stay operator-decided.

Proposed shape: **Failure Pattern Reporter agent**. Tier = Sonnet. Reads the last N batch log files + the feedback markdowns; produces a weekly digest of "checks that fail most often" + "checks that never fail (overspec'd?)" + "drafts that fail R1 then pass on R2 with the same content" (calibration gap). Operator decides what to change. Build complexity: **medium**.

---

## Stack-ranked: if I were the operator, here's what I'd automate in order and why

You have one pipeline that works (asbestos guides are approved and on a live site) and a fork that needs everything from inputs to renderer. The leverage question is: where does the operator's hour-by-hour time go, and which automation eliminates the most of it without removing judgment from the steps that need it?

The current pipeline is already 80% automated *inside* a single slug — writer, post-processor, auditor, mechanical gate, FAQ schema, escalation routing, deploy verification. The operator's remaining time goes to (a) authoring the per-slug inputs (assignment + config), (b) shipping approved JSON to the site, and (c) deciding what to write next. So that's where automation pays back fastest.

**1. Ship-To-Site agent (steps 4+5+6+7).** Build this first. It's small, it eliminates 8-15 minutes of pure-mechanical operator work per approved slug, the contract is sharp ("given an approved JSON, get it on the live site"), and the failure mode is benign (operator catches it in `git status` or schema-check.yml). At 31 already-shipped slugs + however many in the SSG fork once it's live, this saves real hours fast. Tier = Haiku or no LLM. **Build complexity: small.** Highest-leverage wrap — start here.

**2. Config Synthesizer agent (step 2).** Second-fastest payback. The keyword-config schema is small, the inputs are scoped (slug + primary keyword), and the failure mode is caught immediately by the next pipeline run (MISSING_CONFIG check at line 1121 of run-batch.sh, plus G1/SEO5/TOPIC1 will fail mechanically). Tier = Haiku. **Build complexity: small.** This is the most "pure value" win — eliminates 10-20 min/slug with near-zero risk because the mechanical gate catches mistakes.

**3. Assignment Drafter agent (step 1).** Bigger build, bigger return. The 30-90 min/slug operator-authoring step is the largest single time sink in the loop, and the LLM is good at the structural-fill 70% even if the seed paragraph + writer notes stay operator-authored. Tier = Sonnet (this needs reasoning over the keyword research file + approved-index). **Build complexity: medium.** Build this third because it depends on having the keyword-research file in a parseable shape and needs more iteration to get the seed/audience drafting right.

**4. Escalation Triage agent (step 3).** Lower-frequency event, but high pain per occurrence. When a guide hits R3 escalation, the operator has to switch context to debug. A triage agent that consumes the feedback file + draft + brief + config and proposes a root-cause diagnosis is high-signal because the input is already structured. Tier = Sonnet. **Build complexity: small** if shipped as a one-shot post-batch report, **medium** if it should propose patches. Build the report version first.

**5. Failure Pattern Reporter (step 10).** The slowest-ROI item but the one that compounds. A weekly digest of "most-common failures across the last N batches" → operator decides whether to retune V8 (sentence-word cap), drop a banned phrase, or widen SEO5 range. Pays off most when batch volume grows. Tier = Sonnet. **Build complexity: medium.**

Items I would explicitly not automate:
- **GSC submission (step 8).** API is finicky; risk-of-flag is real; the operator already does this fast.
- **Pipeline orchestration choices (step 9).** The agent should suggest the next slug, never run the batch on its own. Cost-of-error is high and the model-flag decisions are operator preferences.
- **The writer + auditor + mechanical gate themselves.** They're already automated and working. Tuning them is what step 10 supports.

A pragmatic build order: **Ship-To-Site first (week 1) → Config Synthesizer (week 1-2) → Assignment Drafter (weeks 2-4) → Escalation Triage report (week 5) → Failure Pattern Reporter (weeks 6-8).** That sequence frontloads the highest-leverage, lowest-risk wins, defers the higher-judgment automations until the operator has used the low-stakes ones long enough to know exactly what they want, and never touches the parts of the loop where the current pipeline is already doing the work.

The SSG fork doesn't change this ranking. It does mean Ship-To-Site needs a site-config parameter (which site repo, which APPROVED slug whitelist file, which renderer component) so it works against both AsbestosHQ and SmartSourceGuide once SSG has actual approved JSONs to ship — which it won't until at least Config Synthesizer + Assignment Drafter are in place and the SSG specs/reference files have been re-skinned from their currently-byte-identical-to-asbestos state.
