# AITEAM Session Handoff — Chassis Decisions, Voices System, RAG Plan

**Date generated:** May 14, 2026
**Session purpose:** Triage Mark Kashef's ClaudeClaw corpus, resolve open architecture questions, expand the goal set, lock the substrate.
**Prior context save:** `UST_Context_Save_2026-05-13_AgentSystem-Portability-PersonaPivot.md` + companion `UST_Agent_System_And_Persona_Brief_2026-05-13.md`.
**Companion deliverables in `~/Desktop/AITEAM_distilled/`:** `AITEAM_Mark_Kashef_Distilled_Core.md`, `AITEAM_Mark_Kashef_Distilled_Full.md`, `AITEAM_Source_File_Relevance_Report.md`, `AITEAM_Triage_Process_Log.md`, `AITEAM_Channels_vs_Grammy_Verification.md`.

---

## If you only read one paragraph

The operator (Stonecreed) is now the dev cofounder. The Mac Mini M4 is set up and ready. The chassis architecture is locked: markdown brain in Obsidian vault, Karpathy LLM Wiki memory pattern PLUS RAG (hybrid retrieval from day one behind a single `retrieve_context()` function), bash + cron + `ai-do.sh` / `ai-think.sh` / `ai-cheap.sh` wrappers for orchestration, SQLite + WAL for state, Mark Kashef's hive_mind / audit log / kill switch / memory-tuning patterns ported in. Phone interactive layer is grammy bot (Telegram), verified by Claude Code research — Channels is not ready for unattended operation (no daemon mode, foreground-developer feature). First site has not been picked yet; leaning PLR / Etsy / Gumroad digital products, needs more thinking. First build target is still undecided but "easy fast things first" is the operating rule. Budget is flexible. The persona / trading project is parallel-track but soft on timing. Voices system (ingestion from trusted experts) is committed as part of the architecture but seed list of names is not yet committed.

---

## 1 — Operator profile snapshot

**Name and posture:** Stonecreed. Operating both the strategist and the dev cofounder roles. There is no separate developer being briefed — the operator is the one building.

**Coding background:** Project files from earlier sessions framed Stonecreed as "zero coding experience." That framing has now shifted — Stonecreed has admitted to being the dev cofounder. Read the technical content of this handoff at full depth. Don't condescend on technical detail. Adapt only the explanation style to match the operator's stated preference for video and audio over dense markdown reading.

**Processing-mode preference:** Video at 2x is comfortable. Reading dense markdown is slow. When explaining anything dense, prefer: short paragraphs, diagrams in markdown tables or ASCII trees, plain prose over bullet-soup. The operator declined an audio brief of this handoff for now, but the option stands open. If a future session produces a major reference doc, offer to convert it to an 8–12 minute spoken script.

**Existing production:** UST project has live agent infrastructure — headless CC pipelines for guide pages, service pages, programmatic SEO. GitHub Actions for smoke tests and link audits. ~5,800-page contractor directory. This is the proven substrate the AITEAM chassis is being built to share. AITEAM is not building from zero on infrastructure.

**Operating capacity:** Solo at the keyboard. The "dev cofounder" framing in earlier sessions was a placeholder for the operator's own dev role. Tasks that require a second human are not in scope.

**Self-acknowledged pattern (carry forward, do not surface unprompted):** strong research-before-building tendency, flagged across multiple sessions as the biggest threat to execution. Cure is action and external deadlines. Do not lecture; do not repeat the warning every session. If the operator is plainly stuck in research mode at the end of a session, name it once gently and move on.

---

## 2 — The mission (the only goal that has a pass/fail)

Build a team of AI agents that runs a business making enough to at least cover the $200/mo Claude Max subscription. More is better. The bar is break-even on the subscription. Below that, the architecture is wrong no matter how elegant.

Everything in this document serves that goal.

---

## 3 — The 19 goals, ranked and grouped

These were settled in this session by merging the operator's original 7 project-instruction goals with 12 additional goals surfaced mid-session.

### Mission (1–7) — why all of this exists

1. Build a team of AI agents that runs a business making enough to cover the $200/mo Claude Max sub. More is better.
2. Build helpful tools and content sites for real people — calculators, directories, guides. Useful first, monetizable second.
3. Get traffic to those sites and earn through display ads — AdSense first, Mediavine at 50K sessions/month.
4. Hand the running and maintenance over to AI agents so the operation is passive.
5. Start with the easiest, lowest-competition, fastest-to-money idea first. More money is better, but easy comes first.
6. Run the whole thing on a Mac Mini with a fleet of agents — 4–5 to start, scaling only when current ones are profitable.
7. Build durable infrastructure — markdown brain + bash + cron + wrapper scripts — so the AI vendor is swappable and nothing locks the operator in.

### Infrastructure (8–12) — what gets built first to make the mission possible

8. AI dashboard / command center — single auto-updating view of agent activity, pending items, things needing attention.
9. Head AI assistant / orchestrator — the agent the operator talks to; delegates to the team and reports back.
10. Complete memory system every key agent can access — search library, all files, full context on demand.
11. File management / organization system — librarian agent processes the inbox nightly, sorts, flags duplicates, kills stale files.
12. Obsidian vault — the human-readable side of the brain; markdown on disk that both Claude and the operator can read.

### Worker agents (13–17) — the team

13. Autonomous researcher agent — patrols Reddit, Google, niche forums for ideas and opportunities. Note: this also encompasses the "voices system" — ingestion from trusted experts on Twitter, YouTube, Substack, podcasts.
14. Idea / tool researcher agent — validates raw opportunities, turns them into build specs.
15. Dev team agent — builds the actual tools, sites, calculators from specs.
16. Telegram / Twitter watcher agent — monitors specified channels for the operator; the operator stops doom-scrolling.
17. Crypto / AI trends scanner — feeds both the researcher and the persona project with emerging signals.

### The two businesses (18, 19) — what the team operates

18. Affiliate / display-ad sites run mostly by agents — the content factory output. Primary revenue track. First-site target is currently undecided; leaning PLR / Etsy / Gumroad digital products. The operator wants to think this through more before committing.
19. AI influencer / persona run by the operator's edge — trading / financial markets / crypto. Parallel track, timing soft. Operator's 20-year market experience + 6 years crypto is the irreplaceable input.

### Operating rules (apply to all of the above)

- The mission bar (item 1) is the only hard pass/fail.
- Build infrastructure first, hire worker agents one at a time as evidence shows they're overburdened or needed, then operate the businesses.
- Never scale agent count before the current ones are visibly profitable.
- Keep API costs under 50% of last month's revenue. Budget is flexible during phase 0 — the operator confirmed this — but the discipline holds permanently.
- Use Claude for strategy and quality decisions; use cheap models (Llama 70B, Haiku) for bulk content; use code-only (no LLM) where possible.
- Manual validation before automation. Ship one ugly page live before building the agent that ships 100 pages.

---

## 4 — The locked-in chassis

This is the architecture as of this session. Every item below is settled unless explicitly marked otherwise.

### 4.1 The substrate table

| Layer | Substrate | Why |
|---|---|---|
| Brain content (markdown, decisions, knowledge) | Plain markdown files in Obsidian vault | Lasts decades, portable to any future tool |
| Memory — primary | Karpathy LLM Wiki pattern (`raw/` + `wiki/` + `SCHEMA.md`) | Human-readable, debuggable, scales to ~2,000 docs |
| Memory — at scale | RAG vector index built in parallel from day one (768-dim embeddings, SQLite vector store) | Survives the document-volume explosion the voices system will create |
| Retrieval interface | Single `retrieve_context(query)` function — both wiki and RAG sit behind it | Agents never know which retrieval path served them; can swap or rebalance at any time |
| Orchestration | Bash scripts + cron + `ai-do.sh` / `ai-think.sh` / `ai-cheap.sh` wrappers | Survives any LLM vendor change; cost-tiering on same plumbing |
| Autonomous work | Headless `claude -p` calls via wrappers | Already proven in UST pipelines |
| Persistent agent state | SQLite + WAL mode (Mark's pattern) | Standard, portable, survives anything |
| Cross-agent shared activity | `hive_mind` table (Mark's pattern) | Simple, sufficient, ports cleanly |
| Audit log | Append-only SQLite with correlation IDs (Mark's pattern), 90-day retention, pinned rows survive prune | Universal pattern; mandatory once agents touch the internet |
| Kill switches | `.env` flags refreshed on mtime change (Mark's pattern) | Simple, no framework lock-in |
| Memory tuning values | Mark's calibrated numbers: importance 0–1, salience 0–5, dedup 0.85 cosine, retrieval 0.3 cosine min, decay tiers (1%/2%/5% by importance), 30-min consolidation cycle | Saves months of trial-and-error |
| Quality scoring (any agent that produces output) | Self-Improving Kit pattern: 5-criterion rubric with weights, 4-band decision tree (None / Suggestion / Auto-Update / Escalate), 6-hour cooldown, max 3 auto-updates/day, version chain with `previous_version_id`, `locked` flag, North Star guardrails to prevent scope drift | Most directly relevant pattern in the entire Mark corpus |
| Dashboard / command center | Auto-rebuilt `command-center.md` (durable) + optional Hono web view on a port for live updates | Markdown survives anything; web UI when convenient |
| Head orchestrator | Single CC agent in `agents/orchestrator/` with `agent.yaml` + `CLAUDE.md` containing routing rules | Human-readable, swappable, no framework needed |
| Monitor / scanner agents (Reddit, Twitter, Telegram, YouTube, crypto/AI trends, voices) | Cron + `claude -p` + write to `inbox/` + nightly librarian sort | Uniform shape across all monitor types |
| Dev team agent | Agents-as-folders + path-scoped rules + global skill inheritance (Mark's pattern, ported) | Matches the complexity and tool surface of what it does |
| Persona / Trading content | Operator-maintained `current-thesis.md` + agent-maintained content drafter | Keeps the edge with the operator, production with the agents |
| Phone interactive | **grammy bot (Telegram), Node.js, ~500 lines, ported from Mark's V2 spec** | CC verified May 14 2026: Channels not ready for unattended ops, no daemon mode, foreground-developer feature |
| Phone alerts (cron sends summaries) | Reuse the grammy bot from above | Saves maintaining two Telegram integrations |
| All agent payloads | Each agent is its own folder with `agent.yaml` + `CLAUDE.md` + `workspace/`, calls into shared chassis | This is the operator's actual business logic and the compounding asset |

### 4.2 The two non-negotiable abstractions

The chassis works because two pieces are abstracted on day one:

**The model-call abstraction.** Every script in the system calls `ai-do.sh "prompt"` or `ai-think.sh` (expensive) or `ai-cheap.sh` (bulk) — never `claude` directly. When the operator wants to swap to a different model, a local Llama, or a different vendor entirely, one file changes. This was decided in the prior UST session and remains the right call.

**The retrieval abstraction.** Every agent that needs information calls `retrieve_context(query)` — never reads files directly, never queries SQLite directly. Behind this function is the wiki + RAG hybrid. When the balance between wiki and RAG needs to shift, or when a third retrieval mode (knowledge graph, full-text, structured query) gets added, one file changes.

If a future session questions either of these abstractions, the answer is no — these are the leverage points that make the whole "swappable substrate" promise real.

---

## 5 — The phone-control decision (closed)

**Decision:** grammy bot (Telegram), built from Mark Kashef's V2 REBUILD_PROMPT spec.

**Confidence:** 8/10 per Claude Code verification on May 14, 2026.

**Evidence (full report in `AITEAM_Channels_vs_Grammy_Verification.md`):**

1. PR #1397 merged April 14, 2026 fixed 3 of 4 polling-resilience bugs in `claude-plugins-official#788`. Real progress, but only on a slice of the surface area.
2. The architectural root cause is still open. Issue #1594 (request for an HTTP/SSE daemon transport so the bot lives independently of a Claude session) has no maintainer response. Until that ships, Channels is a foreground-developer feature, not unattended infrastructure. This is the kill shot for the use case.
3. Issue #49151 (hot-reload disconnect on multi-session) and #38098 (auto-loads in all `claude -c` sessions causing ~50% message loss) are both open and both reproduce on Mac M4 / macOS 15.4 with multiple sessions — i.e. exactly the target hardware.
4. Two brand-new bugs (#1838, #1839) were filed May 13. Bug discovery hasn't plateaued.

**What this unlocks:**
- The grammy bot is built from Mark's V2 spec — not from scratch. Full implementation in `REBUILD_PROMPT_V2.md` and condensed in the Distilled Core.
- The Telegram watcher agent (goal 16) gets simpler — same bot account watches channels the operator specifies, writes findings to `inbox/`.
- A wrapper script `notify.sh` abstracts the Telegram integration. If Channels ever ships daemon mode, swapping is one file.

**The empirical test runbook** (Section 7 of the verification doc) is preserved on disk in case a future revisit becomes necessary. Don't run it now — it would just confirm what's already decided.

---

## 6 — The wiki + RAG hybrid decision (closed)

**Decision:** Build both from day one, behind a single `retrieve_context()` interface.

**Rationale:**
- Operator's goal set produces document volume fast. Voices system (10 trusted experts × 365 days = 3,650 raw captures in year one, plus distillations and weekly syntheses = 5,000+ docs by year-end).
- Migration cost from wiki-only to wiki+RAG later is roughly equal to building both upfront (~3 extra dev days at start vs. ~3–5 days to migrate later, plus 1–3 months of degraded retrieval during the wiki-only phase).
- Net: build both now, save the operational pain.

**Implementation shape:**
- `raw/` holds the immutable source files. Markdown, scraped pages, transcripts, captures.
- `wiki/` holds clean AI-generated summary pages with cross-references and a `SCHEMA.md` defining the structure. Human-readable. Editable.
- `vectors/` holds the embedding index. A background script chunks every file in `raw/` into ~500-token chunks, generates 768-dim embeddings (Gemini embedding API or local — operator's call), stores in SQLite.
- `retrieve_context(query)` runs both paths in parallel, merges by rank-fusion, returns top N. Agents never know which path served them.

**Tuning starting values (Mark's, port directly):**
- Embedding dim: 768
- Storage format: `Float32Array → Buffer` (3072 bytes per embedding, NOT JSON which is ~10× larger)
- Embedding model: `gemini-embedding-001` (cheap and good)
- Dedup cosine threshold: 0.85 (merge instead of add)
- Retrieval cosine minimum: 0.3 (filter floor)
- Importance scale: 0–1 (0 = trivial, 0.5 = routine, 0.7 = notable, 1.0 = critical decision/preference)
- Salience scale: 0–5
- Decay: pinned never, ≥0.8 importance = 1%/day, ≥0.5 = 2%/day, <0.5 = 5%/day
- Consolidation: every 30 minutes, batch up to 20 unconsolidated memories per chat
- Hard filter: skip messages under 15 characters or starting with `/`

---

## 7 — The voices system (committed, names deferred)

This is the highest-leverage worker agent in the fleet. Operator has not committed a seed list of names yet; the system is designed so adding names later is trivial.

### Concept

Point the AI team at a list of trusted experts (Twitter, YouTube, Substack, podcasts, RSS feeds). Ingest their content daily. Distill each piece into structured notes. Synthesize across sources weekly. Other agents can then query: "what have my trusted voices said about X in the past month?"

### Why it matters

This is potentially the single highest-strategic-value agent even though it produces no direct revenue. Most operators consume expert content reactively and lose 95% within a week. This system runs a structured continuous-intake operation across the entire trusted-source roster with full archive, distillation, cross-reference, and recall.

Six months in, the persona drafter, the dev agent, and the researcher all have access to every relevant thing every trusted voice has said about every topic in the past six months. That's a learning-rate moat — the operation absorbs more per week than competitors who are doom-scrolling.

### Architecture

```
~/brain/voices/
├── signal_sources.md          ← operator-maintained list of trusted people
│                                with platforms, trust weight (1-5), topic tags
├── raw/
│   ├── karpathy/
│   │   ├── 2026-05-14_twitter.md
│   │   └── 2026-05-14_youtube.md
│   ├── [other voices]/
│   └── _inbox/                ← new captures land here, librarian sorts overnight
├── wiki/
│   ├── voices/
│   │   ├── karpathy/
│   │   │   └── 2026-05-14_distilled.md
│   │   └── weekly_synthesis_2026-W20.md
│   └── topics/
│       ├── agent_architectures.md
│       └── crypto_cycles.md
└── vectors/                   ← embedding index over raw/ + wiki/
```

### Three-step pipeline (per voice, per day)

**Capture.** Cron job pulls last 24 hours of content via RSS (free, no API) or platform API where needed. Writes raw markdown into `raw/<voice>/<date>_<platform>.md`. Immutable archive.

**Distill.** Cheap model (Haiku or local Llama) reads each capture and produces a structured note: key claims, supporting reasoning, contrary takes, jargon/concepts introduced, things to verify, relevance to active projects. Writes to `wiki/voices/<voice>/<date>_distilled.md`. This is what other agents read.

**Synthesize.** Weekly job reads all distilled notes from the past 7 days across all voices, produces `wiki/voices/weekly_synthesis_<date>.md` — patterns across multiple voices, contradictions, emerging concepts gaining traction. When three trusted people independently mention the same emerging concept in the same week, that's the highest-signal output the system produces.

### Design choices already settled

- **Trust weights (1–5) per voice.** Operator sets manually based on track record. Revises quarterly. Used by the synthesis agent to weight competing takes on the same topic.
- **Topic tagging at distill time.** Every distilled note gets tagged. Enables "what have my voices said about agent architectures this month?" as a trivial query.
- **Decay on relevance, not capture.** Raw captures live forever. Distilled notes have a relevance score that decays unless re-cited. Mark's salience pattern applied to the voice corpus.
- **Personal-vs-team separation.** `voices/personal/` for entertainment / personal feeds — agents never read. `voices/team/` for trusted experts the agents learn from.
- **Volume control via librarian pruning.** The librarian agent's nightly job includes culling low-value captures (one-line emoji tweets, retweets without commentary, sponsored posts) before they hit the distill step. Otherwise 50 voices × 20 posts/day = 1,000 daily captures and the system drowns.

### Build sequencing for the voices system

- **Months 1–2:** chassis only. Do not start voices.
- **Months 3–4:** add voices ingestion with 5–10 named seed voices. Validate the workflow. Confirm distillation quality.
- **Month 5–6:** scale voice count if working. By this point RAG will be carrying most of the retrieval weight for this corpus.

Do not let "voices" jump the queue. The chassis has to exist first. The voices system is a phase-2 build, not a phase-1 build.

### Seed list

Not committed by operator in this session. The next session should ask: "are you ready to commit 5–10 seed voices?" before scaffolding the system. If still not ready, do not block — the architecture works with zero voices; the operator adds names when they're ready.

---

## 8 — The agent roster (initial build order)

This is the actual ranked list of agents to build, with model tier and trigger.

| # | Agent | Model tier | Build phase | Trigger to build |
|---|---|---|---|---|
| 1 | Chassis (not an agent) | n/a | Phase 0 | Mac Mini setup → wrappers → SQLite → wiki + RAG retrieve → notify.sh → command-center.md regen → grammy bot |
| 2 | Orchestrator | Sonnet (default) | Phase 1 | Immediately after chassis. Agent the operator talks to. |
| 3 | First worker agent — TBD per first-site decision | Sonnet for strategy, Haiku/Llama for bulk | Phase 1 | Once orchestrator delegates correctly |
| 4 | Librarian | Haiku (cheap) | Phase 1 | After enough material lands in inbox to be worth sorting (a few days in) |
| 5 | Editor / quality scorer | Haiku or Sonnet — TBD by rubric test | Phase 1 | When the first agent produces output worth grading |
| 6 | Autonomous researcher (Reddit + Google) | Sonnet | Phase 2 | Once first content site is shipped and earning |
| 7 | Idea / tool researcher | Sonnet | Phase 2 | Once researcher is producing leads faster than they can be evaluated |
| 8 | Dev team agent | Sonnet or Opus | Phase 2 | Once specs from idea researcher exceed manual build capacity |
| 9 | Telegram / Twitter watcher | Haiku | Phase 2 | Once a defined source list exists |
| 10 | Crypto / AI trends scanner | Haiku | Phase 2–3 | Tied to persona project pacing |
| 11 | Voices ingestion (Karpathy-style intake) | Haiku for ingest, Sonnet for synthesis | Phase 2–3 | Once at least 5 seed voices are committed |
| 12 | Persona / Trading drafter | Sonnet | Phase 3 | Once persona project is officially launched; timing soft |

**Promotion-by-evidence rule:** Don't hire agent N+1 until agent N is visibly overburdened or producing clear value. The list above is the *order*, not the *schedule*. Building all 12 at once is the failure mode — building #2, validating, then #3, validating, then #4 is the success mode.

**Mark Kashef's insight, ported:** The judge doesn't need to be a more expensive tier than the writer. Mark uses Haiku 4.5 for both the chat agent and the reflection judge in his Self-Improving Kit. The asymmetry that makes the system work is in the rubric-driven evaluator prompt, not the model tier. For AITEAM, this means the Editor can probably run on Haiku or Llama 70B if the rubric is rigorous enough. Test before committing to Sonnet for the Editor — could save 80–90% on QA costs.

---

## 9 — First-site decision (open)

**Current status:** undecided. Leaning toward PLR (Private Label Rights) related — Etsy or Gumroad digital products. Operator wants to think this through more before committing.

**Operating rule:** start with the easiest, lowest-competition, fastest-to-money idea. Easy beats lucrative at the starting line.

**Candidate categories the operator has surfaced across sessions:**

- **PLR digital products on Etsy / Gumroad.** Currently leading. Low overhead, low traffic dependency, fast cash possible if the product fits. Risk: marketplace saturation, race-to-bottom pricing.
- **SEO calculator / tool site network.** Ranked S-tier in AITEAM Master Rankings v4. Build-once tools, AdSense + affiliate monetization, real moat through data accumulation over time. Mulch calculator was floated as a first build earlier but is not actively prioritized.
- **Programmatic SEO directory in finance/SaaS vertical.** UST has already proven this pattern works at scale. Risk: longer time to revenue (SEO bake time).

**Things the next session should help the operator decide:**

- Is the goal "first revenue at any size" or "first revenue at a scale that signals product-market fit"? These point at different first projects.
- PLR is fast to revenue but has weak compound mechanics — every dollar earned next month requires roughly the same effort as this month. Calculator/tool sites compound (one good tool earns for years). The right first build depends on what the operator wants to learn, not just what earns first.
- The chassis investment is the same regardless. First-site choice doesn't affect the architecture above this section — it affects what payload the first worker agent runs.

**Recommended next-session move:** offer a 30-minute decision aid — three options laid out with risk, time-to-revenue, compound mechanics, and required first-build skills. Then let the operator pick. Do not delay chassis work waiting on the first-site decision; chassis is identical regardless.

---

## 10 — The Mac Mini setup (done)

**Status (confirmed in this session):** Mac Mini M4 is set up and ready for the build to begin.

**Hardware completed:**
- Mac Mini M4 Pro, 24GB unified memory, 512GB SSD
- 4TB external SSD (Crucial X9 or similar)
- WD Elements Desktop HDD for archival
- UPS for power protection
- Ethernet cable (wired, not WiFi)

**Software state:** unclear from this session whether tools are installed. The next chat should ask before assuming. Probable need-list:
- Homebrew, Git, Python 3.12+, Node.js v20+, sqlite3 (preinstalled), Obsidian, Claude Code

**Software to install for this stack specifically:**
- `pandoc`, `pdftotext`, `xlsx2csv`, `jq` (for the file-conversion `SessionStart` hook pattern from Mark — drop messy files in `raw_dropzone/`, get markdown out)
- Wrangler CLI (Cloudflare deploy) once the dev/publisher agent is built
- gh CLI (GitHub) for UST-style smoke tests
- Anthropic and Gemini API keys for the model + embedding pipeline

**The "first thing built on the Mac Mini" question** is currently open. Recommended sequence the next session should propose:

1. Verify Claude Code is installed and works (`claude --version`, run a one-shot prompt).
2. Create the folder structure (next section).
3. Build the three wrapper scripts (`ai-do.sh`, `ai-think.sh`, `ai-cheap.sh`).
4. Build `notify.sh` stub (it can be empty for now — just an abstraction).
5. Initialize SQLite databases with WAL mode.
6. Initialize the Obsidian vault.
7. Stand up `command-center.md` auto-regen.
8. Build the orchestrator agent folder.

Steps 3–8 are roughly one focused day of work. The grammy bot can come after the orchestrator is talking to itself successfully.

---

## 11 — Folder structure (proposed, not yet built)

This is the proposed layout on the Mac Mini, harmonized with the existing UST setup so AITEAM lives as a sibling project, not a separate vault.

```
~/brain/                       ← single Obsidian vault, shared across projects
├── meta/                      ← cross-project knowledge
│   ├── SCHEMA.md
│   ├── voices/                ← see Section 7
│   └── decisions/             ← architecture-level decisions like this one
├── operator/                  ← personal notes, not seen by agents
└── projects/
    ├── ust/                   ← existing UST work
    ├── payroll/               ← existing PayrollInsider work
    ├── aiteam/                ← THIS PROJECT
    │   ├── raw/
    │   ├── wiki/
    │   ├── inbox/             ← agents drop findings here, librarian sorts
    │   ├── outputs/
    │   │   ├── post_drafts/
    │   │   ├── publishable/
    │   │   └── morning_briefs/
    │   ├── silver_platters/   ← weekly summary tables (Mark's 80% pattern)
    │   ├── command-center.md  ← auto-rebuilt dashboard
    │   ├── PRIME.md           ← identity + critical context, <200 tokens
    │   ├── HANDOFF.md         ← cross-session resumption state
    │   └── SCHEMA.md          ← memory structure rules
    └── persona/               ← parallel-track persona project (timing soft)

~/agents/                      ← orchestration, not knowledge
├── lib/
│   ├── ai-do.sh
│   ├── ai-think.sh
│   ├── ai-cheap.sh
│   ├── notify.sh
│   ├── retrieve_context.sh    ← single retrieval entry point (wiki + RAG)
│   └── log_to_audit.sh
├── config/
│   └── .env                   ← kill switches, API keys
├── orchestrator/
│   ├── agent.yaml
│   ├── CLAUDE.md
│   └── workspace/
├── librarian/
│   ├── agent.yaml
│   ├── CLAUDE.md
│   └── workspace/
├── editor/
│   ├── agent.yaml
│   ├── CLAUDE.md
│   ├── rubric.md              ← Self-Improving Kit pattern
│   └── workspace/
├── _template/                 ← copy this when adding a new agent
│   ├── agent.yaml
│   └── CLAUDE.md
└── skills/                    ← global skills (Mark's CLI inheritance pattern)
    ├── publish-to-cloudflare/
    │   └── SKILL.md
    ├── score-content-quality/
    │   └── SKILL.md
    └── [more as needed]

~/.claudeclaw-backups/         ← SQLite backups (Mark's pattern)
~/store/
└── aiteam.db                  ← single SQLite, WAL mode, all tables
```

**Notes on the structure:**

- One vault, not per-project vaults. Decided in the UST session.
- `meta/voices/` is shared because trusted-expert content compounds across all projects. AITEAM, UST, and persona all benefit from the same voices corpus.
- `~/agents/` is orchestration (scripts, agent configs, skills). `~/brain/` is knowledge (markdown, raw captures, wiki, decisions). Two separate trees, two different lifecycles. Don't merge them.
- The `_template/` folder is the agent-creation shortcut from Mark's V3. Copy → rename → edit two files → register → live.

---

## 12 — Key gotchas to surface to the operator on day one

Pulled from the Distilled Core. These are real bugs that will bite during the build if not surfaced upfront.

1. **Spaces in paths break `import.meta.url.pathname`.** Use `fileURLToPath(import.meta.url)` everywhere `__dirname` is needed in Node. Single most common cause of "Missing script: build" errors during setup. The operator's path is `~/Desktop/` so this matters.
2. **Telegram voice notes arrive as `.oga`. Groq Whisper rejects `.oga`.** Rename to `.ogg` before upload — same format, different extension.
3. **`bypassPermissions: 'bypassPermissions'` is mandatory for the grammy bot's unattended ops.** Without it, the Claude subprocess pauses waiting for terminal approval on every tool call.
4. **launchd `KeepAlive` needs `ThrottleInterval ≥ 5 seconds`** to prevent crash-restart loops.
5. **Gemini embedding model returns 768-dim vectors.** Store as `Float32Array → Buffer`. Not JSON — 10× larger.
6. **PIN hash timing attacks.** Use `crypto.timingSafeEqual()`, never `===`.
7. **FTS5 triggers must restrict to content columns only.** `AFTER UPDATE` trigger must specify `UPDATE OF summary, raw_text` — otherwise every salience-decay sweep triggers FTS rewrites and write amplification destroys performance.
8. **`AGENT_MAX_TURNS=30` default.** Without it, runaway tool-use loops burn the entire context window and token budget. Pass to the SDK on every call.
9. **Memory v2 ingestion is fire-and-forget.** Never block the user-facing response. If you `await ingestConversationTurn`, the UX breaks.
10. **Hard filter before extraction:** skip messages under 15 characters or starting with `/`. Prevents trivia from cluttering memory.
11. **Exfiltration guard false positives in code blocks.** Long hex strings (git hashes, file checksums) trigger the hex-key pattern. Skip content inside fenced code blocks.
12. **Database backup pattern:** `sqlite3 store/aiteam.db ".backup '/dest/aiteam_$(date +%s).db'"` every 6 hours via cron. Restore is `cp` back — but stop the orchestrator first; never restore over a live DB.
13. **Cron jobs that fire every minute can rack up costs quickly.** Default to longer intervals (hourly, daily) and audit-log every scheduled-task fire so spend is traceable.

Full list of 60 in the Distilled Core. Surface these 13 on day one because they're the ones likely to bite during chassis construction.

---

## 13 — Open decisions the next session should help close

In rough priority order:

1. **First-site target.** Leading toward PLR / Etsy / Gumroad digital products but uncommitted. See Section 9 for proposed decision aid.
2. **Voices system seed list.** Operator declined to commit names in this session. Next session should ask again; if still uncommitted, do not block — architecture works with zero voices.
3. **Editor model tier.** Default plan is Sonnet; Mark's evidence suggests Haiku may be sufficient. Test before committing.
4. **Persona project timing.** Operator said "soonish, idk on timing yet, depends." Not blocking AITEAM. Next session should leave it alone unless the operator raises it.
5. **First worker agent after orchestrator.** Depends on first-site choice. If PLR Etsy/Gumroad, the first agent is probably a product-research agent. If a tool site, the first agent is the dev/builder. Decision flows from #1.
6. **launchd service strategy.** Mark uses 5 separate launchd units. For AITEAM, the question is: one service or multiple? Probably multiple (orchestrator + librarian + scheduler) but defer until the chassis is up.
7. **Backup strategy beyond SQLite.** The Obsidian vault needs backup too. Time Machine to the 4TB external SSD plus a periodic clone to the WD HDD is the simplest answer. Probably no decision needed; just confirm operator is doing this.

---

## 14 — Companion files and where they live

All in `~/Desktop/AITEAM_distilled/` unless noted:

| File | Purpose |
|---|---|
| `AITEAM_Mark_Kashef_Distilled_Core.md` | The working reference. ~11,150 words. Full schemas, gotchas, regex, tuning values, adaptation table. **Upload this to AITEAM project knowledge in Claude.ai.** |
| `AITEAM_Mark_Kashef_Distilled_Full.md` | Safety net. ~14,800 words. Corpus-indexed compression. Open only if Core ever feels incomplete. |
| `AITEAM_Source_File_Relevance_Report.md` | Map of every Mark Kashef source file with HIGH/MEDIUM/LOW/SKIP rating. ~4,800 words. **Also worth uploading to project knowledge.** |
| `AITEAM_Triage_Process_Log.md` | QA report of the triage pass. 14 surprises section is high-signal. |
| `AITEAM_Channels_vs_Grammy_Verification.md` | Phone-control decision record. ~373 lines. Includes test runbook (Section 7) in case of future revisit. |
| `_extract_*.md` (5 files) | Working artifacts from the triage. Not required reading. Can delete after Core is reviewed. |

In project knowledge already (from prior UST session):
- `UST_Agent_System_And_Persona_Brief_2026-05-13.md` — the 10-section architecture brief. Authoritative on chassis decisions.
- `UST_Context_Save_2026-05-13_AgentSystem-Portability-PersonaPivot.md` — session-level handoff.

---

## 15 — How the next chat should behave

**Style:**
- Match the operator's processing-mode preference. Short paragraphs, visual structure (tables, ASCII trees), prose over bullet-soup.
- Do not condescend on technical detail. The operator is the dev cofounder. Speak at the level the substrate demands.
- Adapt to dev-cofounder framing: when explaining new architecture, include enough implementation detail for the operator to actually build, not just understand.

**What the next chat should do first if the conversation is on AITEAM build topics:**
- Read this handoff in full (sections 4, 5, 6, 11, 12 are highest-density).
- Treat sections 4 (substrate table) and 5 (grammy decision) as authoritative — do not re-derive.
- Treat section 13 (open decisions) as the active design space.
- If the operator opens with a build question, default to step-by-step build instructions with exact commands, exact file paths, exact code.

**What the next chat should NOT do:**
- Do not re-derive decisions in section 4 or 5. They're closed.
- Do not push the persona project unless the operator raises it.
- Do not lecture the operator about research-mode tendency unless they're plainly stuck in it at end of session.
- Do not assume the Mac Mini setup state. Ask before installing things.
- Do not produce audio briefs unless asked.
- Do not generate "Three Things Worth Considering" footers — the operator asked for that to stop in the prior session.

**When in doubt:**
- Check `AITEAM_Mark_Kashef_Distilled_Core.md` for implementation details.
- Check this handoff for decisions and rationale.
- Check `UST_Agent_System_And_Persona_Brief_2026-05-13.md` for cross-project context.
- Ask the operator before guessing on ambiguity.

---

## 16 — Carry-forward from this chat

Verbatim items worth preserving for the next session, in case they come up:

- The operator asked for triage outputs of Mark Kashef's full kit, and CC delivered ~32,000 words of distillation across four files. The Core is the working reference; everything else is fallback or map.
- CC verified Channels vs grammy on May 14, 2026. Verdict: skip Channels, build grammy, confidence 8/10. Three findings: (1) PR #1397 fixed only part of the bug surface, (2) the architectural root cause — no daemon mode — is still open as issue #1594, (3) bug discovery hasn't plateaued (two new issues filed May 13).
- The operator initially overstated my "ClaudeClaw vs UST" framing, then corrected me that they want a hybrid, then we built the hybrid substrate table. The current table in section 4.1 reflects that final state.
- Wiki + RAG hybrid was added as an upgrade to the original UST substrate table during this session. Rationale: the voices system will produce 5,000+ documents within a year, and the migration cost from wiki-only to hybrid later is roughly equal to building both upfront. Plus 1–3 months of degraded retrieval during the wiki-only phase. Net positive to build both now.
- The voices system was surfaced this session as a top-tier strategic asset. Highest-leverage worker agent in the fleet even though it produces no direct revenue.
- The 19-goal list in section 3 is the canonical goal set as of this session, supersedes earlier lists in project instructions.
- The first-site decision (PLR Etsy/Gumroad vs calculator vs directory) is open and the operator wants to think more.

---

## 17 — One paragraph for the operator to read on a phone screen

You're at the Mac Mini, everything is set up, the chassis is decided. Build order: wrappers, SQLite, retrieve_context, notify.sh, command-center, then the orchestrator agent folder, then the grammy bot. Mark Kashef's kit has been triaged into one reference doc (Distilled Core) plus a map (Relevance Report) — upload Core to project knowledge in the next chat. Channels is dead for your use case; grammy is the call, ~500 lines of Node ported from Mark's V2 spec. Wiki + RAG hybrid from day one behind one `retrieve_context()` function. First site is undecided, leaning PLR Etsy/Gumroad but think it through. The voices system (ingest trusted experts daily, distill, synthesize weekly) is committed but seed names deferred — phase 2 build. Persona project is parallel-track, timing soft, no urgency. Don't hire agent N+1 until agent N is overburdened. The bar is the $200/mo Claude Max sub; if the architecture doesn't break even, the architecture is wrong.

---

*End of handoff.*
