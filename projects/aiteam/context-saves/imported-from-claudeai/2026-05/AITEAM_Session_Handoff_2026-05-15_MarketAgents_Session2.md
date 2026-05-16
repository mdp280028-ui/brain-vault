# AITEAM Session Handoff — Market Agents Session 2 (Mid-Build)
**Created:** May 15, 2026 (late evening Pacific)
**Operator:** Stonecreed (Bonners Ferry, ID)
**Session topic:** Built Curator agent (Cowen synthesis + Casper outlook-history). Mid-build: re-run in progress to apply threshold recalibration. Briefer + /important + Opus side-by-side still to ship.
**Project:** AITEAM (separate from PayrollInsider/UST/SMART)

---

## ONE-PARAGRAPH SUMMARY

Session 2 of the market-agent pipeline. Built the Curator agent end-to-end with the harder half (Cowen synthesis) shipped successfully through 4 checkpoints. Curator handles both channels asymmetrically: Cowen gets full topic-organized synthesis to `~/brain/expertise/cowen/` across 28 pre-seeded topics in 5 groups, Casper gets mechanical outlook-history appends to 7 asset files in `~/brain/expertise/casper/outlook-history/`. Smoke test against 5/15 briefs passed; calibration data revealed the original 0.50 similarity threshold was too tight (top-1 was correct in 5/5 cases but raw scores cluster at 0.55-0.59 for hits vs 0.38-0.47 for misses). Threshold tuned to 0.35/0.15, re-run kicked off to repopulate two misrouted topics from `_misc/`. Session 2 still owes: briefer agent, `/important` slash command + `model_override`, Opus side-by-side experiment, cron activation. Operator wants morning brief at **8:30 AM Pacific**; the daily cadence (07:30 poll → 08:15 curate → 08:30 brief) was decided this session.

---

## WHY THIS PROJECT EXISTS (refresher for next Claude)

Operator wanted an agent that watches YouTube videos for him and surfaces market timing signals from two specific analysts (Benjamin Cowen + Jayson Casper). NOT an AITEAM revenue agent — uses Max subscription, doesn't move the $200/mo mission bar. Operator was explicit: it's one of the most important agents *to him personally* for market timing. The trade-off vs SSG agent work has been flagged and accepted. Don't re-litigate.

**Asymmetric treatment is the core design decision:**
- **Cowen = wide knowledge harvest.** Personal "Cowen-pedia" across 28 seeded topics. Full synthesis on each new brief.
- **Casper = narrow signal extraction.** Outlook arrows on specific assets, no synthesis, mechanical parse + append only.

---

## ARCHITECTURE — THE 4-AGENT PIPELINE (UPDATED)

```
~/agents/market/
├── scribe/       ✅ BUILT (session 1)  — polls YouTube, fetches transcripts
├── analyst/      ✅ BUILT (session 1)  — generates per-video brief (Sonnet via ai-do.sh)
├── curator/      ✅ BUILT (session 2)  — Cowen synthesis + Casper outlook accretion
└── briefer/      ⏳ NEXT  — morning Telegram roll-up at 8:30 AM Pacific
```

### Daily cadence (DECIDED THIS SESSION)

Operator switched from `*/15 * * * *` polling to daily morning brief:

```
07:30  poll.sh        — detect new videos from last 24h
07:30+ fetch+analyze  — chained per new video (Sonnet)
08:15  curate.sh      — Cowen synthesis + Casper outlook-history append
08:30  briefer.sh     — send Telegram morning brief
```

One trigger, one sequence, one outcome. 45-min headroom for analyze, 15-min for curate. Reason: operator doesn't need 15-min wakeups; wants brief delivered with morning coffee.

Cron lines in `~/agents/market/README.md` are NOT yet activated — D033 still deferred until briefer ships + 1-3 days of manual validation.

---

## WHAT GOT BUILT THIS SESSION (Curator)

### File layout

```
~/agents/market/curator/
├── agent.yaml
├── CLAUDE.md
├── curate.sh                          — orchestrator (calls curate_cowen.sh + curate_casper.sh, writes combined audit row)
├── curate_cowen.sh                    — LLM-heavy Cowen synthesis loop
├── curate_casper.sh                   — mechanical Casper outlook parser
├── similarity.py                      — sentence-transformers cosine sim, CLI w/ embedding cache
├── parse_outlook.py                   — parses Casper outlook table → JSON
├── cowen_topic_prompt_template.md     — per-topic Sonnet prompt with substitution placeholders
├── requirements.txt                   — sentence-transformers, numpy
├── .venv/                             — bootstrapped on first curate.sh run if missing
├── .cache/hf/                         — model cache via SENTENCE_TRANSFORMERS_HOME (~90MB, gitignored)
├── .cache/embeddings.pkl              — embedding cache keyed by (slug, mtime, sha1)
├── workspace/responses/               — LLM responses persisted (added after awk bug to prevent re-call waste)
└── failure_modes.md                   — 3+ documented failure modes
```

### Cowen expertise tree (28 topics, 5 groups + _misc)

```
~/brain/expertise/cowen/
├── cycle-and-risk/
│   ├── btc-cycle-theory/
│   ├── bitcoin-risk-metric/
│   ├── logarithmic-regression-bands/
│   ├── bull-market-support-band/
│   ├── pi-cycle-top-indicator/
│   ├── post-halving-behavior/
│   ├── diminishing-returns-thesis/
│   └── lengthening-cycles-thesis/
├── market-structure/
│   ├── bitcoin-dominance/
│   ├── altcoin-season-conditions/
│   ├── eth-btc-ratio/
│   ├── the-flippening/
│   ├── total3-vs-btc-dominance/
│   └── altcoin-bleed-thesis/
├── macro-liquidity/
│   ├── fed-policy-rates/
│   ├── global-m2-liquidity/
│   ├── dxy-dollar-strength/
│   ├── quantitative-tightening-easing/
│   ├── business-cycle-phase/
│   ├── labor-market-unemployment/
│   ├── inflation-outlook/
│   └── oil-price-outlook/
├── on-chain-and-social/
│   ├── on-chain-metrics/
│   ├── social-engagement-risk/
│   ├── apathy-vs-euphoria-tops/
│   └── institutional-flows-etf/
├── cross-asset/
│   ├── gold-vs-crypto/
│   └── equities-spx-correlation/
└── _misc/                              ← auto-created bucket for unmatched topics
```

Each topic folder contains:
- `synthesis.md` — frontmatter + Current Thesis / Key Evidence / Recent Updates
- `evidence-log.md` — append-only dated entries
- `predictions-log.md` — append-only (only when brief contains predictions)

### Casper outlook-history (7 assets, mechanical-only)

```
~/brain/expertise/casper/outlook-history/
├── btc.md
├── eth.md
├── sol.md
├── xrp.md
├── total.md
├── total2.md
└── total3.md
```

Each file: frontmatter (asset, last_updated, row_count) + append-only markdown table with columns:
`| date | horizon | arrow | conviction | one_line_context | video_id | brief_path |`

Horizon values: `days`, `weeks`, `months` (per Casper's timeframe labels). Zero LLM cost — pure parse + append.

---

## DECISIONS MADE THIS SESSION (KEEP THESE LOCKED)

### Decision 1 — Topic taxonomy seeded with 28 anchors, not 12
Originally proposed 24, expanded to 28 after research surfaced oil as a regular coverage area. Research grounded the seeds in Cowen's actual Q1/Q2 2026 macro risk memos + ITC framework taxonomy + X feed coverage patterns.

### Decision 2 — Seed descriptions are descriptive, not positional
"Cowen covers X using Y framework" embeds better than "Cowen believes X means Y." Six seeds were edited at checkpoint 1 to remove editorial framing — pi-cycle-top-indicator, diminishing-returns-thesis, the-flippening, altcoin-bleed-thesis, business-cycle-phase, inflation-outlook. Pattern logged for any future seed work.

### Decision 3 — Similarity threshold 0.50 → 0.35 (flat) and 0.50 → 0.15 (group mean)
Checkpoint 4 calibration revealed the 0.50 spec threshold was anchored to the wrong reference workload (slug-vs-slug). All-MiniLM-L6-v2 scores cluster at 0.55-0.59 for correct matches vs 0.38-0.47 for misses on conceptually-related-but-lexically-different content. Top-1 was correct in 5/5 non-exact cases. Threshold filtering was rejecting correct answers, not noise. Tuned to 0.35 flat / 0.15 group mean. Audit rows #143 and the upcoming re-run row will capture the decision.

### Decision 4 — Daily 8:30 AM Pacific brief cadence
Operator wants morning brief, not 15-min polling. Polling collapses to one daily run at 07:30 → curate at 08:15 → brief at 08:30. YouTube Data API quota drops from 192/day to 4/day. Operational clarity wins.

### Decision 5 — AGENT_MAX_TURNS=1 override for Cowen topic calls
Inline env-var override in curate_cowen.sh, NOT a separate ai-do-single.sh wrapper. Prevents Sonnet's internal multi-turn from inflating cost on text-in/text-out synthesis calls.

### Decision 6 — Per-cell context on Casper outlook = TL;DR shared across video
For v1, all rows from one video share the brief's TL;DR as the context column. Per-cell richer context deferred (D039). The outlook-history serves the question "what's Casper been saying about ETH days-out over 30 days" — date + arrow + conviction + general context answers that.

### Decision 7 — _misc/ bucket placement
Lives at `~/brain/expertise/cowen/_misc/`, same depth as 5 group folders, underscore-prefixed to sort last in `ls`. Operator `mv`s folders out of _misc/ into real groups when topics prove durable.

---

## CHECKPOINT-BY-CHECKPOINT RESULTS

### Checkpoint 1 — 28 seed descriptions ✅
Pre-created 28 Cowen folders + _misc/ + 7 Casper outlook files. 6 seed descriptions edited for descriptive (not positional) phrasing. Seed stubs anchor similarity index until real thesis content accretes.

### Checkpoint 2 — similarity.py + rank test ✅
Built sentence-transformers all-MiniLM-L6-v2 pipeline with embedding cache. Rank test: query "Powell stepping down as Fed chair, Warsh transition" → fed-policy-rates #1 at 0.38 (1.74× separation from runner-up). CC's preemptive flag: absolute scores naturally low, threshold likely needs recalibration. Logged for checkpoint 4.

### Checkpoint 3 — Casper-only smoke test ✅
parse_outlook.py + curate_casper.sh against the 5/15 Casper brief. All 6 pass conditions met. btc.md got 2 rows (weeks→3, days→2), other 5 outlook files got 0 rows. Audit row #142.

### Checkpoint 4 — Full end-to-end smoke test ✅ (with caveats)
**Awk bug on first run wasted $0.28 on 5 successful LLM calls whose outputs couldn't parse.** Fix: renamed `close` (reserved word) to `startmark`/`endmark`, persisted LLM responses to `workspace/responses/` so future parser bugs don't require re-calls. Second run: clean, $0.1221, 5 topics synthesized, 2 misrouted to _misc/ due to threshold being too tight.

**Pass-condition grading:**
- ✅ Synthesis quality on 5 touched topics is operator-readable (Current Thesis sections distilled, not padded)
- ⚠ 2 topics misrouted to _misc/ (driving the threshold tune in Decision 3)
- ❌ business-cycle-phase not touched — analyst didn't tag it, curator only routes from analyst tags. Real systemic question deferred as D041.

**Audit row #144 payload:**
```json
{
  "cowen_videos_processed": 1,
  "cowen_topics_updated": 5,
  "cowen_new_topics_created": 2,
  "casper_videos_processed": 1,
  "casper_rows_appended": 2,
  "cowen_cost_usd": 0.1221,
  "total_cost_usd": 0.1221
}
```

### Checkpoint 4 RE-RUN — currently in progress at session pause
Operator approved threshold tune. Next CC actions:
1. Delete `~/brain/expertise/cowen/_misc/spx-secondary-correction/` and `~/brain/expertise/cowen/_misc/inflation-reacceleration/`
2. `UPDATE seen_videos SET curated=0 WHERE video_id='FgxAe_NAh5c'` (Casper row untouched)
3. Re-run curate.sh with new thresholds (0.35 / 0.15)
4. Expected: 5 topics resolved, 0 new topics created, content lands in `equities-spx-correlation` and `inflation-outlook` folders

**State at session pause: CC has the re-run prompt but execution may not have completed yet. Next session should confirm re-run audit row first.**

---

## DEFERRED ITEMS — SESSION 2 ADDITIONS

Adding to the running list from session 1. All deferred items now tracked centrally; review at session start.

| ID | Item | Trigger |
|---|---|---|
| **D039 NEW** | **Outlook-history TL;DR repetition** — rows 2+ from same video repeat TL;DR verbatim. Consider per-video header + abbreviated context. | After 7+ days of real Casper data accumulates |
| **D040 NEW** | **Curator idempotency** — re-running curate_casper.sh on an already-processed video will currently append duplicate rows. Add `(date, video_id, horizon)` existence check before append. | Next curator-touch session, or first wild observation |
| **D041 NEW** | **Curator fan-out for untagged topics** — should curator add a Sonnet pass that reads full brief and proposes touched topics beyond analyst's tag list? Pro: catches relevant topics analyst missed (e.g. business-cycle-phase 5/15). Con: cost + hallucination risk. | After 30 days of curator runs reveal under-tag frequency |

Carried forward from session 1 handoff (still open):

| ID | Item | Status |
|---|---|---|
| D025 | Orchestrator permission policy for Telegram-spawned sessions | Still open; doesn't block briefer (briefer runs from cron, not orchestrator routing) |
| D026 | Phase C launchd auto-restart for Telegram bot | Still deferred; Telegram bot still in foreground, dies on terminal close |
| D027 | Add `*.bak` to `~/brain/.gitignore` | Still open, 2-min fix |
| D028 | Whisper fallback in fetch_transcript.sh | Both channels have working auto-captions; defer until one breaks |
| D029 | Prediction-scoring layer with CoinGecko price data | After 30+ days of predictions accumulated |
| D030 | Heuristic auto-routing to Opus | Only if /important flagging reveals a learnable pattern |
| D031 | Casper trading-technique extraction | Toggle exists in watchlist.yaml; flip when operator wants this signal |
| **D032** | **Side-by-side Sonnet vs Opus experiment** | **Scheduled for session 2 (still pending)** — $1.30 one-time |
| **D033** | **Cron activation for market pipeline** | **After briefer ships + 1-3 days of manual validation** |
| D034 | Telegram morning brief readability legend | Decide after operator reads 3-5 morning briefs |
| D035 | Update project instructions to reflect market agent work | Operator's open decision #1 effectively answered: SSG remains revenue-bearing first site, market agents are parallel personal-utility track |

---

## REMAINING SESSION 2 SCOPE

After the re-run completes, three pieces still to build before session 2 closes:

### 1. Briefer agent (next up after re-run confirmation)
- Location: `~/agents/market/briefer/`
- Cron: daily 08:30 AM Pacific
- Reads briefs from last 24h
- Composes morning Telegram message:
  - **Headline section:** thesis shifts since yesterday (highest-signal alert)
  - **Convergence section:** where Cowen and Casper agree/disagree on BTC
  - **Per-channel briefs:** TL;DR + outlook grid (Option A monospace code-block format)
  - **Footer:** disk paths to full briefs
- Outlook format conversion: markdown table → monospace code block with em-dashes for blanks (Telegram doesn't render tables; this is what makes it phone-readable)
- Uses existing Telegram bot's `notify.sh` for delivery
- Cost target: ~$0.05-0.10/day
- Briefer is glorified template rendering — spend less iteration time here than curator

### 2. `/important` slash command + `model_override` column
- `ALTER TABLE seen_videos ADD COLUMN model_override TEXT DEFAULT NULL`
- Telegram slash command: `/important <video_id>` writes 'opus' to that column, kicks off re-analysis
- analyze.sh reads model_override; uses ai-think.sh (Opus) if set, otherwise ai-do.sh (Sonnet)
- ~30-45 min CC

### 3. Opus side-by-side experiment (D032)
- Reuse the Cowen 5/15 smoke-test video
- Run through Opus, store at `<date>_<videoid>_opus.md`
- Operator reads both briefs blind, decides if Opus quality difference justifies cost
- $1.30 one-time
- Result determines whether `/important` becomes frequent-use or rare-exception

### 4. Cron activation
- Only after briefer + curator pass smoke tests AND 1-3 days of manual runs validate behavior
- Install the daily 07:30/08:15/08:30 chain

---

## STATE OF THE FULL AITEAM PROJECT (END OF SESSION 2 BUILD WORK SO FAR)

### What's live and running
- Mac Mini M4 Pro (24GB/512GB) — all peripherals operational
- 3 original agents: orchestrator (Sonnet), librarian (Haiku), editor (Sonnet, r=0.933)
- 3 new agents (sessions 1-2): scribe, analyst, curator
- Dashboard at :3141 (KPI cards, cost chart, diary view, war room with @mentions and discuss mode)
- Brain vault git-tracked, nightly auto-commit at 23:55 to private GitHub
- `/ctx` slash command on CC for context saves
- Telegram bot live with text + voice (Whisper) + slash commands + destructive verb gate
- Token usage logging through all wrappers
- Audit log: ~145+ rows (session 2 added #141 tuning_decision, #142 casper smoke, #143 second tuning_decision, #144 first full curate run + the upcoming re-run row)

### What's still NOT live
- Briefer (next up)
- /important + model_override
- Opus side-by-side (D032)
- Cron activation (D033)
- Phase C launchd (D026 — Telegram bot still dies on terminal close)
- D025 orchestrator perms fix
- Any AITEAM revenue work (SSG agents not yet started)

### Cost accounting (Max-subscription value consumed, NOT real dollars)
- Session 1: ~$0.33 (analyst smoke tests)
- Session 2 to date: ~$0.40 (includes $0.28 awk-bug waste)
- Phase 2 earlier May 15 sessions: ~$1.50
- Cumulative AITEAM Max-sub burn through today: ~$5-6 estimated

---

## IMPORTANT CONTEXT FOR NEXT CLAUDE

### About the operator (unchanged from session 1 — still applies)
- Bonners Ferry, ID — rural northern Idaho
- No coding background. Relies entirely on Claude Code on the Mac Mini for execution
- Reads briefs on phone (Telegram) primarily, Obsidian on Mac secondarily
- Terse, direct shorthand. Defaults to recommendation, not options
- Stays strictly in declared session mode — don't drift
- Asks "what is this?" / "how do I read this?" questions when format/syntax is unfamiliar. Walk through it.
- Operator's biggest execution risk = research-before-building. Push toward shipping when he drifts.

### About sign-off discipline (PROJECT RULE — actively applied this session)
- ✅ requires the pass condition was EXECUTED and produced expected output
- Artifact-exists ≠ ✅
- Honest ⚠ beats premature ✅
- CC modeled this beautifully in checkpoint 4: caught its own awk bug, was transparent about the $0.28 waste, didn't soft-promote the wasted run to ✅. Continue crediting CC's pushback.

### About CC's pattern-pushing this session
CC's pre-build pushback caught two real calibration bugs that would have wasted iterations later:
1. **Cold-start similarity problem** (checkpoint 1) — empty thesis = 0 similarity everywhere → seed-description stubs added
2. **Threshold mis-calibration** (checkpoint 2) — flagged that 0.90 was anchored to wrong reference workload → recalibrated to 0.35 at checkpoint 4

This is sign-off discipline at the design level, not just the build level. Worth logging as a lesson.

### About the market-agent build philosophy
- Cowen = wide knowledge harvest (the goal is a personal Cowen-pedia)
- Casper = narrow signal extraction (the goal is timing the markets)
- Treat them differently in code, templates, and storage
- Knowledge accretion happens at ingestion time (curator rewrites synthesis files) — intentional, not RAG with extra steps
- Topic proliferation is FINE early — operator will trim later. Light dupe guardrail catches obvious dupes.

### Communication style for session continuation
- Format CC prompts in titled code blocks for easy copy/paste
- Time estimates on every CC prompt: ⏱️ CC estimate, bottleneck, your involvement
- Push back honestly when something seems off
- Avoid TTWC footer (permanently disabled per project instructions)
- Default CC launch command: `claude --dangerously-skip-permissions` (preceded by appropriate `cd`)

### Critical state notes
- Telegram bot currently running in foreground. Dies on terminal close. D026 launchd still pending.
- Mac Mini "hardly ever shuts off" per operator preference. Real risk is silent bot death.
- D025 orchestrator perms wall blocks LLM-routed operational queries via Telegram. Doesn't affect cron-driven curator/briefer.

---

## LESSONS LEARNED (SESSION 2)

1. **Embedding similarity thresholds are workload-specific.** all-MiniLM-L6-v2 produces 0.85+ scores on semantically equivalent sentences but 0.3-0.6 on conceptually-related-but-lexically-different content. Threshold tuning must be calibrated against the actual workload, not a generic number. The 0.50 in the original spec was anchored to wrong reference; the calibration loop at checkpoint 4 caught it. Build in calibration checkpoints, don't trust round numbers.

2. **CC's preemptive flagging works — and is now a pattern worth crediting.** Twice this session, CC raised a design-level concern *before* I asked. Both saved iterations downstream. The discipline of "describe the problem before writing code" should remain part of every multi-step build prompt.

3. **Cold-start matters in similarity indexes.** Empty content = zero similarity = every new input creates a new bucket. Seed-description stubs anchor the index until real content accretes. Worth remembering for any future LLM-routed work that relies on accumulated content as the anchor.

4. **Persist LLM responses to disk before parsing.** The awk bug at checkpoint 4 burned $0.28 because the parser failed after the LLM call succeeded. Persisting raw responses to `workspace/responses/` first means future parser bugs don't require re-calls. This is the LLM equivalent of "always log raw HTTP responses." Roll this pattern into every future agent that makes paid LLM calls.

5. **Sign-off discipline scales to the design level, not just the build level.** Original rule was "✅ requires pass condition was executed." This session showed the same discipline catches misaligned design assumptions if you build calibration checkpoints into the spec (which we did — the rank test at checkpoint 2 and the score distribution at checkpoint 4 were both calibration checkpoints, not just build checkpoints).

6. **Asymmetric agent treatments are easier to maintain than uniform ones.** The Cowen-rich / Casper-lean split required two sub-scripts and two storage shapes but kept each path simple. A "one curator does everything" design would have forced compromises in both paths. Worth applying the asymmetric principle elsewhere when use cases genuinely diverge.

---

## FIRST MOVES FOR NEXT SESSION

When operator opens the next chat:

1. **Confirm curator re-run completed cleanly.** Ask for the re-run audit row. Expected payload should show `cowen_new_topics_created=0` and the brief routing to existing `equities-spx-correlation` and `inflation-outlook` folders (not _misc/). If re-run is still mid-execution, wait for it to complete before any other work.

2. **Briefer build is the next sequential step.** Write the briefer build spec. Use the same multi-checkpoint pattern from curator (plan → smoke test → grade). Briefer is structurally simpler than curator — should be ~2 hours CC work, single checkpoint may be sufficient.

3. **Topic seed list for Cowen is locked at 28.** Do not re-derive. Don't re-discuss the asymmetric treatment decision. Operator already approved the architecture.

4. **Don't activate cron until briefer ships + 1-3 days of manual validation.** D033 stays deferred until that gate is cleared.

5. **The Opus side-by-side experiment (D032)** can be done before OR after briefer ships. Probably easier to do it after briefer — it's a one-off experiment and doesn't block anything. Surface it to operator near end of session 2 for a clean close.

### The 80/20 for remaining session 2 scope
- **Briefer quality matters less than curator quality.** Briefer is template rendering on top of curator's synthesis. If curator is good, briefer is easy. Don't over-iterate on briefer's prose.
- **`/important` + Opus side-by-side is genuinely valuable to ship together.** The slash command writes to model_override; the Opus experiment is the first real test of the model_override path. Build them as a pair.

---

## DON'T FORGET

- **Session 2 audit rows so far:** ~#141 through #144+ (re-run row pending). Run-id `curator_run_20260515_210153` was the first full curate.sh run; re-run will have a fresh run-id.
- **Smoke test files** still on disk:
  - `~/brain/channels/cowen/briefs/2026-05-15_FgxAe_NAh5c.md`
  - `~/brain/channels/casper/briefs/2026-05-15_zyHJRBIEVcs.md`
- **Curator workspace dir** contains persisted LLM responses from checkpoint 4 — preserved for debugging if needed: `~/agents/market/curator/workspace/responses/`
- **CC context save** from session 2 will land at `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_HHMM.md` after `/ctx` runs on CC at session end
- **Project instructions v2** is in project knowledge; read first if anything in this handoff feels stale

---

## OPEN DECISIONS (UNCHANGED FROM PRIOR SESSIONS)

1. ~~First site target~~ → Effectively answered: SSG remains revenue-bearing first site, market agents are parallel personal-utility track (D035 captures this)
2. Voices seed list (5+ names) — not blocking
3. First worker agent after orchestrator/librarian/editor — depends on first-site choice
4. Persona project timing — operator: "soonish, depends"
5. launchd service strategy — defer to Phase 2 cleanup
6. Backup verification cadence — decide after first restore
7. HANDOFF.md on Mac Mini was updated by session 1's /ctx; should be current

---

*End of handoff. Operator: drop this file into ~/Downloads/ then run /ctx on CC to trigger auto-harvest. Also drop into Claude.ai project knowledge for the next session.*
