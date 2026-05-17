# AITEAM Context Save — Market Agents Session 2 Close-Out
**Date:** May 15, 2026 (~10:45 PM Pacific)
**Session topic:** Market-agent pipeline session 2 close-out — Curator threshold tune + Briefer agent build + notify.sh outbound extension + /important slash command + D032 Opus side-by-side verdict + D043 ETH/SOL convergence groundwork
**Project:** AITEAM (separate from PayrollInsider/UST/SMART)
**Companion on disk:** `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_2242.md` (CC-side mechanical save)

---

## TL;DR

Session 2 of the market-agent pipeline is functionally complete. Five major builds shipped in sequence, all signed off with executed pass conditions:

1. **Curator threshold tune (0.50 → 0.35/0.15)** — re-run flipped `cowen_new_topics_created` from 2→0, brief routes to existing seed folders
2. **Briefer agent** — daily morning Telegram brief with thesis shifts, convergence section, per-channel TL;DRs, monospace 7×3 outlook grid
3. **notify.sh extension** — stub → live Telegram Bot API delivery with audit logging + HTTP error handling
4. **/important slash command + model_override column** — operator-triggered Opus re-analysis from phone, brief filename suffix preserves both versions on disk
5. **D043 ETH/SOL convergence groundwork** — `mentioned_assets` frontmatter contract added, briefer loops convergence over intersection (1-asset case exercised, multi-asset waits on organic data)

**D032 verdict: Sonnet stays as default analyst model.** Opus distills harder but generates non-canonical topic slugs (`fed-chair-transition` vs `fed-policy-transition`, `spx-seasonal-correction` vs `spx-secondary-correction`) that would force curator threshold re-tuning. Sonnet's deeper key-point reasoning chains preserve causal context. Quality delta does not justify 5x cost. Opus reserved for operator-triggered spot-checks via `/important`.

**Pipeline is complete.** Only D033 (cron activation) remains, gated on 1-3 days of manual validation runs starting tomorrow morning.

---

## DECISIONS MADE THIS SESSION

### Decision 1 — Threshold tune: 0.50 → 0.35 flat, 0.50 → 0.15 group mean

Checkpoint 4 calibration revealed the original 0.50 was anchored to wrong reference workload. All-MiniLM-L6-v2 scores cluster at 0.55-0.59 for correct matches vs 0.38-0.47 for misses on conceptually-related-but-lexically-different content. Top-1 was correct in 5/5 non-exact cases. Threshold was rejecting correct answers, not noise.

Applied option 1 (flat 0.35 + group 0.15) over margin-based rule (option 2) for simplicity. Re-evaluate after 7-10 days of real briefs accumulate.

**Audit rows: #145-146.**

### Decision 2 — D032 verdict: Sonnet remains default analyst model

Blind comparison on FgxAe_NAh5c. Sonnet 6116B / Opus 4420B. Both briefs end clean (no truncation, frontmatter complete). Operator read both:

| Axis | Result |
|---|---|
| Signal extraction | Roughly equal |
| TL;DR sharpness | Opus tighter, Sonnet more contextual |
| Evidence with timestamps | Both 6 quotes, different selections |
| Operator-readability | Sonnet's deeper reasoning chains preferred |

**Decisive factor: topic slug compatibility.** Opus generated novel slugs that don't map to curator's 28 seeded topics. Adopting Opus would force a curator threshold re-tune or a slug-normalization step. Sonnet's slugs map cleanly (e.g. `fed-policy-transition` → `fed-policy-rates` at score 0.59).

**Policy:** Opus only via operator-triggered `/important` slash command. Expected use "occasional spot-check on videos that feel especially important." 5x cost ($0.20 vs $0.04 per brief) not justified for default routing.

**Audit row: #164.**

### Decision 3 — Briefer message spacing: visible blank lines between bullets

Original briefer output read as walls of text on phone. Operator flagged after first live send. Applied 5 spacing rules:
- Blank line before AND after every section header
- Blank line between bullets within thesis-shifts section
- Blank line between TL;DR / Topics / Full path lines within each channel block
- Blank line above and below the monospace grid block (grid internally tight)
- Blank line before closing separator

Implemented via post-processing in brief.sh (awk insertion), not Sonnet prompt — keeps spacing logic deterministic and token cost unchanged. Telegram's blank-line collapse behavior didn't require `\n\n\n` escalation; single blank line per break was sufficient.

### Decision 4 — Opus briefs saved with _opus.md suffix, NOT canonical

`/important` writes Opus brief to `<date>_<video_id>_opus.md` alongside the original `<date>_<video_id>.md`. Both files coexist. Curator only reads canonical (no-suffix) Sonnet brief — Opus brief is human-comparison only.

This means **the curator stays one-brief-per-video** regardless of model overrides. If Opus ever becomes default, the canonical suffix convention changes; until then, model-override briefs are operator artifacts.

### Decision 5 — Briefer canonical brief discovery must exclude model-override artifacts

Caught mid-D043 validation: Opus spot-check brief at `..._opus.md` made the briefer double-count the Cowen video in stub run #1. Fixed with `grep -Ev '_(opus|haiku)\.md$'` filter on brief discovery globs for both Cowen and Casper paths.

**Lesson:** model-override artifacts are operator spot-check outputs, not canonical pipeline inputs; discovery globs must exclude them explicitly. Applies anywhere downstream agents discover-by-glob.

**Audit row: #172.**

### Decision 6 — `mentioned_assets` frontmatter is briefer-only data

Added to analyst's brief frontmatter contract. Briefer computes intersection across last-24h Cowen + Casper briefs and loops convergence section over each shared asset. Curator does NOT read this field — it's purely for downstream briefer composition.

For Cowen: include any asset with substantive discussion (1+ minute coverage or directional call). Passing mentions don't count.
For Casper: derived mechanically from outlook table — any asset with non-blank arrow.
Allowed values: btc, eth, sol, xrp, total, total2, total3 (matches Casper outlook-history asset list).

### Decision 7 — Don't fake multi-asset test data for D043

D043 build only exercised 1-asset case (BTC) tonight. Multi-asset code path is correct by code review but unexercised against real data. Operator + CC agreed: don't synthesize fake briefs to "exercise the path" — creates noise in audit log and brief content. First real multi-asset case will fire organically when Cowen next mentions ETH or SOL substantively.

**Watch when it fires:** are asset blocks visually distinct? Does Sonnet emit coherent per-asset "Read:" lines, or lazy-paste the same template?

---

## FILES CREATED OR MODIFIED ON MAC MINI

### New agent: briefer
```
~/agents/market/briefer/
├── agent.yaml
├── CLAUDE.md
├── brief.sh                          ← orchestrator + spacing fix + Opus-filter
├── compose_brief.sh                  ← Sonnet call, AGENT_MAX_TURNS=1 inline
├── compose_prompt_template.md        ← per-asset convergence loop
├── render_outlook_grid.py            ← markdown table → monospace 7×3 grid
├── failure_modes.md                  ← 3 documented modes
└── workspace/responses/              ← Sonnet outputs persisted (curator lesson applied)
```

### Modified
| File | Change |
|---|---|
| `~/agents/lib/notify.sh` | Stub → live Telegram Bot API delivery + audit + HTTP errors |
| `~/agents/market/curator/curate_cowen.sh` | `SIM_THRESHOLD="0.35"`, `GROUP_THRESHOLD="0.15"` |
| `~/agents/market/analyst/CLAUDE.md` | `mentioned_assets` frontmatter contract + slug-naming discipline + Opus policy |
| `~/agents/market/analyst/prompt_template.md` | Instruction #11 (mentioned_assets) |
| `~/agents/market/analyst/brief_cowen.template.md` | frontmatter field |
| `~/agents/market/analyst/brief_casper.template.md` | frontmatter field |
| `~/agents/market/analyst/analyze.sh` | Reads `model_override`, routes to ai-think.sh if 'opus' |
| `~/agents/telegram/bot.py` (or equivalent) | `/important <video_id>` handler |
| Schema | `ALTER TABLE seen_videos ADD COLUMN model_override TEXT DEFAULT NULL` |
| `~/brain/channels/cowen/briefs/2026-05-15_FgxAe_NAh5c.md` | retro patch: `mentioned_assets: [btc]` |
| `~/brain/channels/casper/briefs/2026-05-15_zyHJRBIEVcs.md` | retro patch: `mentioned_assets: [btc]` |

### Generated artifacts
| Path | Purpose |
|---|---|
| `~/brain/channels/cowen/briefs/2026-05-15_FgxAe_NAh5c_opus.md` | Opus side-by-side brief (D032), 4420B |
| `~/brain/expertise/cowen/cross-asset/equities-spx-correlation/synthesis.md` | re-run output (correctly routed) |
| `~/brain/expertise/cowen/macro-liquidity/inflation-outlook/synthesis.md` | re-run output (correctly routed) |
| `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_2242.md` | CC-side mechanical save |
| `~/brain/projects/aiteam/LESSONS.md` | session 2 subsection appended (15,505B total) |
| `~/brain/projects/aiteam/HANDOFF.md` | rewritten (17,207B) |

---

## AUDIT ROW REFERENCE — SESSION 2

| ID | Action | Target |
|---|---|---|
| 141 | tuning_decision | curator_similarity_threshold (0.90 → 0.50) |
| 142 | curator_casper_run | first Casper smoke test |
| 143–144 | curator_run | failed run (awk bug, $0.28 waste) + successful run (checkpoint 4) |
| 145–146 | tuning_decision | similarity → 0.35, group → 0.15 |
| 147 | deferred_items_logged | D039/D040/D041 |
| 148 | curator_run | re-run with new thresholds (cowen_new_topics_created: 0) |
| 149–156 | notify_stub/sent + briefer_run | briefer build smoke tests |
| 161–163 | slash_important_invoked / brief_generated / slash_important_complete | D032 Opus run |
| 164 | tuning_decision | D032 verdict — Sonnet stays default |
| 165 | config_change | TELEGRAM_OUTBOUND_ENABLED flipped to true |
| 167–171 | briefer_run + notify | D043 mentioned_assets validation runs |
| 172 | regression_fixed | briefer Opus-brief filter |

Total session cost: ~$0.78 (Max-sub value consumed, not real dollars). Cumulative AITEAM Max-sub burn through today: ~$6-7 estimated.

---

## LESSONS LEARNED

1. **Embedding similarity thresholds are workload-specific.** all-MiniLM-L6-v2 produces 0.85+ scores on semantically equivalent sentences but 0.3-0.6 on conceptually-related-but-lexically-different content. Don't trust round numbers; build calibration checkpoints into every spec that depends on a similarity floor.

2. **Persist LLM responses to disk BEFORE parsing them.** The awk bug at checkpoint 4 burned $0.28 because the parser failed after the LLM call succeeded. Persisting raw responses to `workspace/responses/` first means future parser bugs don't require re-calls. Rolled into briefer build automatically.

3. **Model-override artifacts must be excluded from canonical-discovery globs.** Caught mid-D043 validation: Opus spot-check brief was double-counted by briefer. Single-line `grep -Ev` filter fixes it. Applies anywhere downstream agents discover-by-glob — any future spot-check pattern (haiku-cheap, dev-test, etc.) needs the same exclusion.

4. **Sonnet's slug discipline is a real advantage over Opus for routed pipelines.** D032 verdict turned on this. Opus generated novel slugs that wouldn't map to curator seeds. Quality lift on the content itself didn't matter once integration cost was factored in. **Pattern:** when a higher-tier model exists, the question isn't just "is it better content," it's "does its output integrate cleanly with downstream agents that depend on specific naming conventions."

5. **Manual validation gates exist for a reason — and the bugs they catch are exactly the bugs cron-time would hide.** The Opus-brief double-counting bug would have silently corrupted every future morning brief once cron activated. Manual runs caught it. **D033 (cron activation) stays deferred** until 1-3 days of clean manual runs confirm no other latent issues.

6. **Spacing on phone-delivered output is not cosmetic.** Operator flagged wall-of-text on first live brief. Fix took 15 min but materially changes whether the morning brief is scannable in 30 seconds or skipped. Treat phone-render visual hierarchy as a first-class pass condition, not a polish item.

7. **Asymmetric agent treatments scale to asymmetric data treatments.** Cowen briefs treat assets as discussion topics (with frontmatter `mentioned_assets`); Casper briefs treat assets as outlook rows (with arrows). The same `mentioned_assets` field is derived differently per channel — extracted from analyst judgment for Cowen, derived mechanically from outlook table for Casper. Single contract, two implementations, clean downstream consumption.

8. **CC's preemptive flagging is a reliable pattern now.** Three times this session, CC raised a design-level concern before being asked: (a) ID collision check on D043/D044; (b) Opus brief filter regression caught mid-validation; (c) seed_taxonomy.sh Casper-overwrite slip flagged honestly with restoration. Credit this discipline; it's the operator's preferred mode.

---

## OPERATOR CORRECTIONS APPLIED

1. **"both channel cover eth too and somewhat sol"** — corrected my initial assumption that Cowen was BTC-only for the convergence section. Drove the D043 scope to handle multi-asset intersection rather than hardcoding BTC. Result: extensible convergence loop that will fire organically when Cowen next discusses ETH/SOL.

2. **"on the Briefer I want more spacing between bulletins and paragraphs"** — surfaced the spacing-as-functional-requirement read. Led to the 5-rule spacing fix. Phone readability is a first-class pass condition.

3. **"Sonnet seems good enough for now"** — D032 verdict in 4 words. Operator validated the slug-compatibility argument and made the routing call cleanly. Decisive operator reads are themselves a signal — when operator commits in one line, the question was already well-framed.

---

## DEFERRED ITEMS

### Added this session
| ID | Item | Trigger |
|---|---|---|
| D039 | Outlook-history TL;DR repetition (rows 2+ from same video repeat TL;DR verbatim) | After 7+ days of real Casper data accumulates |
| D040 | Curator idempotency — duplicate-row guard on Casper re-runs (add `(date, video_id, horizon)` existence check) | Next curator-touch session, or first wild observation |
| D041 | Curator fan-out for analyst-untagged topics — should curator add a Sonnet pass that proposes touched topics beyond analyst's tag list? | After 30 days of curator runs reveal under-tag frequency |
| D042 | `seed_taxonomy.sh` should not overwrite live Casper outlook-history files (skip if row_count > 0) | Next curator-touch session, ~5 min fix |
| ~~D043~~ | ~~ETH/SOL convergence~~ | ✅ SHIPPED — remove from deferred |
| D044 | `log_to_audit.sh` SQL-escape on apostrophes (caught during D032 verdict log; renumbered from collision) | Next audit-touch session or first apostrophe-eating bug |
| D045 (proposed) | Opus prompt-template constraint to use existing seed slugs if Opus usage frequency rises | When `/important` invocation frequency exceeds ~2-3/week |

### Carried forward from prior sessions (still open)
| ID | Item | Status |
|---|---|---|
| D025 | Orchestrator permission policy for Telegram-spawned sessions | Still open; doesn't block briefer (briefer runs from cron, not orchestrator routing) |
| D026 | Phase C launchd auto-restart for Telegram bot | Still deferred; bot still in foreground, dies on terminal close |
| D027 | Add `*.bak` to `~/brain/.gitignore` | Still open, 2-min fix |
| D028 | Whisper fallback in fetch_transcript.sh | Both channels have working auto-captions; defer until one breaks |
| D029 | Prediction-scoring layer with CoinGecko price data | After 30+ days of predictions accumulated |
| D030 | Heuristic auto-routing to Opus | Only if /important flagging reveals a learnable pattern |
| D031 | Casper trading-technique extraction | Toggle exists in watchlist.yaml; flip when operator wants this signal |
| **D033** | **Cron activation for market pipeline** | **After 1-3 days of manual validation runs (starting tomorrow morning)** |
| D034 | Telegram morning brief readability legend | Decide after operator reads 3-5 morning briefs |
| D035 | Update project instructions to reflect market agent work | Effectively answered: SSG remains revenue-bearing first site, market agents are parallel personal-utility track |

### Carried forward from AITEAM Phase 1 (still open)
| ID | Item | Status |
|---|---|---|
| D001 | Editor cascade pattern (Haiku triage → Sonnet final gate) | Production volume not high enough |
| D002 | Kanban view (step 10b) | mission_tasks table empty |
| D003 | Editor tier test re-validation with operator-typed manual scores | Quarterly OR Sonnet version bump |
| D004 | HANDOFF.md on Mac Mini stale wording | ✅ now current (rewritten this session) |
| D005 | First site target decision (PLR/Etsy/Gumroad/calculator) | When operator wants to start revenue work — **SSG was the answer** per May 15 earlier session |
| D006 | Voices seed list | Not blocking |
| D007 | Persona project | When operator raises it |
| D008 | launchd service strategy | Phase 2 cleanup |
| D009 | Backup verification cadence | After first successful test restore |
| D010 | Step 17a (silver platters) | First site live + 30+ days GA4 data |
| D013 | Budget enforcement hook | When project moves to direct API |
| D014 | Detection cron infrastructure | Phase 2 (after launchd) |
| D015 | Per-agent switch bypasses via wrappers | Phase 2 polish |
| D016 | PUBLISHER_DEPLOY_ENABLED enforcement | When publisher agent built |
| D017 | rubric.md change-detection | When editor goes into production |
| D019 | Strategic Phase 2 vs Phase 3 sequencing decision | **Effectively decided:** market agents = parallel personal-utility track; SSG = revenue track |

---

## STATE OF AITEAM AT SESSION CLOSE

### What's live and running
- Mac Mini M4 Pro (24GB/512GB) — all peripherals operational
- **3 original agents:** orchestrator (Sonnet), librarian (Haiku), editor (Sonnet, r=0.933)
- **4 market agents:** scribe, analyst, curator, briefer — full pipeline
- Dashboard at :3141 (KPI cards, cost chart, diary view, war room with @mentions and discuss mode)
- Brain vault git-tracked, nightly auto-commit at 23:55 to private GitHub
- `/ctx` slash command on CC for context saves
- `/important <video_id>` slash command on Telegram for operator-triggered Opus
- Telegram bot live with text + voice (Whisper) + slash commands + destructive verb gate + outbound delivery via notify.sh
- Token usage logging through all wrappers
- Audit log: ~172 rows

### What's still NOT live
- **D033 cron activation** — gated on 1-3 days manual validation (starts tomorrow morning)
- D026 Phase C launchd (Telegram bot still dies on terminal close)
- Any AITEAM revenue work (SSG agents not yet started — Agent 1 Keyword Screener is the documented starting point)

### The mission bar
- **$200/mo to break even on Claude Max:** $0 revenue, $0 toward bar
- **Market agents do NOT move the bar** — they consume Max-sub value, don't generate revenue
- **SSG is the revenue track** — 8-agent stack specced, build sequence 5-6 weeks, Agent 1 (Keyword Screener) is the documented first build

---

## OPEN DECISIONS

1. ~~First site target~~ → **ANSWERED:** SSG (SmartSourceGuide) per May 15 earlier session
2. Voices seed list (5+ names) — not blocking
3. ~~First worker agent~~ → **ANSWERED:** SSG Keyword Screener (Agent 1 of 8-agent SSG stack)
4. Persona project timing — operator: "soonish, depends"
5. launchd service strategy — defer to Phase 2 cleanup
6. Backup verification cadence — decide after first restore
7. HANDOFF.md currency — ✅ now current (rewritten this session)

---

## FIRST MOVES FOR NEXT SESSION

### If next session is market-agent finalization
1. **Manual validation run tomorrow morning.** Run `bash ~/agents/market/scribe/poll.sh` → wait for analyst → run `bash ~/agents/market/curator/curate.sh` → run `bash ~/agents/market/briefer/brief.sh`. Verify clean end-to-end. Log to audit.
2. **If 1-3 days of manual runs are clean:** activate cron (D033). Daily 07:30/08:15/08:30 chain. Don't activate cron tonight.
3. **Watch for first multi-asset convergence** in any future brief. Validates D043's untouched code path.

### If next session is revenue work (RECOMMENDED — per project mission bar)
1. **Start SSG Agent 1 — Keyword Screener (Haiku).** Per `AITEAM_Site_Portfolio_And_SSG_Agent_Spec_2026-05-15.md`. ~3-4 hours CC. First of the 8-agent SSG stack.
2. **In parallel:** operator applies to 4-6 affiliate programs (ReceptionHQ, Smith.ai, Ruby, AnswerConnect, ManageEngine, LoneStar). Operator-only task. Approval timelines are 1-3 weeks — start the clock now.
3. **Open SSG build location decision:** `~/agents/ssg/` vs separate `~/agents-ssg/` tree. Decide before building Agent 1.

### If next session is general AITEAM strategy
1. Read this save + `AITEAM_Context_Save_2026-05-15_SitePortfolio-SSGAgentSpec.md` + `AITEAM_Site_Portfolio_And_SSG_Agent_Spec_2026-05-15.md`
2. Surface the question: market-agent finalization vs SSG Agent 1 — operator should pick explicitly
3. **Watch the research-before-building pattern.** The market-agent pipeline is functionally complete. Spending another session polishing it before touching revenue work is the same anti-pattern dressed as "let me make this perfect first."

---

## IMPORTANT CONTEXT FOR NEXT CLAUDE

### About the operator (unchanged)
- Bonners Ferry, ID — rural northern Idaho
- No coding background. Relies entirely on Claude Code on the Mac Mini for execution
- Reads briefs on phone (Telegram) primarily, Obsidian on Mac secondarily
- Terse, direct shorthand. Defaults to recommendation, not options
- Stays strictly in declared session mode — don't drift
- Operator's biggest execution risk = research-before-building. Push toward shipping when he drifts.

### About sign-off discipline (PROJECT RULE — actively applied this session)
- ✅ requires the pass condition was EXECUTED and produced expected output
- Artifact-exists ≠ ✅
- Honest ⚠ beats premature ✅
- CC modeled this beautifully this session: caught its own awk bug ($0.28 waste, transparent), caught the Opus-brief filter regression mid-validation, flagged the seed_taxonomy.sh Casper-overwrite slip honestly. Continue crediting CC's pushback.

### About the market-agent build philosophy
- Cowen = wide knowledge harvest (the goal is a personal Cowen-pedia)
- Casper = narrow signal extraction (the goal is timing the markets)
- Treat them differently in code, templates, and storage
- Knowledge accretion happens at ingestion time (curator rewrites synthesis files) — intentional, not RAG with extra steps
- The market agents consume Max-sub value and do NOT move the $200/mo mission bar. They are operator personal utility, parallel to the revenue track.

### About the revenue track (NOT yet started)
- **SSG = first AITEAM revenue workload.** Existing site (17 pages attempting to index), niches pre-locked (fleet GPS, IT/MSP, answering services), pipeline works (Next.js + Vercel + GSC).
- 8 agents specced in build order: Keyword Screener → Research → Writer → Editor (extend existing) → Mechanical Auditor → Internal Link Manager → Publisher → GSC Monitor
- Affiliate enrollment is operator-only and runs in parallel — 1-3 week approval timeline starts when operator applies
- This is where the $200/mo mission bar actually moves.

### Communication style
- Format CC prompts in titled code blocks for easy copy/paste
- Time estimates on every CC prompt: ⏱️ CC estimate, bottleneck, your involvement
- Push back honestly when something seems off
- TTWC footer permanently disabled per project instructions
- Default CC launch command: `claude --dangerously-skip-permissions` (preceded by appropriate `cd`)

### Critical state notes
- Telegram bot currently running in foreground. Dies on terminal close. D026 launchd still pending.
- D033 cron NOT activated yet — manual validation gate runs tomorrow morning + 1-3 days
- `mentioned_assets` multi-asset code path is correct by code review but unexercised against real data; first organic occurrence is the test
- Opus brief filter regression has been fixed and audit-logged — protects against silent corruption at cron time

---

## DON'T FORGET

- **D033 cron activation is the only outstanding session-2 item.** Don't activate tonight. Manual validation run tomorrow morning is day 1 of the gate.
- **Session 2 audit rows:** 141–172. Run-ids `curator_run_20260515_210153` (initial) and `curator_run_20260515_210925` (re-run with new thresholds).
- **CC-side context save** lives at `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_2242.md`. This Claude.ai context save is the project-knowledge twin.
- **LESSONS.md** grew this session — 8 new entries in the session 2 subsection.
- **HANDOFF.md** fully rewritten this session — currency note in operator's open decisions list is now ✅.
- **Cumulative AITEAM Max-sub burn:** ~$6-7 estimated. Real dollars: $0 (operator on Max subscription, dashboard numbers reflect value consumed, not API billing).

---

*End of context save. Operator: drop this file into Claude.ai project knowledge to make it available for the next session.*
