# AITEAM Context Save — 2026-05-14 — Mark Kashef Triage, Chassis Lock, Grammy Decision

*Companion handoff (the canonical reference for the next chat): `AITEAM_Session_Handoff_2026-05-14_Chassis-And-Decisions.md`*
*Prior context save: `UST_Context_Save_2026-05-13_AgentSystem-Portability-PersonaPivot.md`*

---

## If I Could Tell the Next Chat One Thing

This session closed two large open architecture questions (phone interactive layer → grammy, memory retrieval → wiki + RAG hybrid from day one) and produced a 17-section handoff that is the canonical reference going forward. **Read `AITEAM_Session_Handoff_2026-05-14_Chassis-And-Decisions.md` before responding on any AITEAM build topic.** The operator is the dev cofounder (not a separate person), the Mac Mini M4 is set up and ready, the chassis is locked, and the next active question is which first site to build — leaning PLR / Etsy / Gumroad but uncommitted.

---

## Session Summary

Very long session (~20+ MSG). Started by reviewing pasted transcripts from the prior UST session discussing Mark Kashef's ClaudeClaw kits. Operator wanted Claude to absorb context from that prior chat before diving in.

MSG 1–3: Read prior transcript and three Mark Kashef extracts (`_extract_setup_security_obsidian.md`, `_extract_self_improving_kit.md`, `_extract_claudemaxxing_and_folder_viewer.md`). Gave ELI14 summaries. The Self-Improving Kit was flagged as the highest-value pattern (Editor + Feedback Loop architecture, with rubric + 4-band decision tree + cooldown + overrides + North Star guardrails).

MSG 4–5: Operator uploaded `UST_Agent_System_And_Persona_Brief_2026-05-13.md` and `UST_Context_Save_2026-05-13`. I initially said I'd read them but only the context save came through in the documents block — caught myself, used `view` on the project copy of the Brief to actually read all 790 lines. Apologized for the earlier incomplete read and synthesized: the Brief decides hybrid Obsidian vault, wrapper-script layer day one, Karpathy LLM Wiki memory, no third-party frameworks for core infra, persona parallel-track. The Brief's failure mode #5 explicitly warns against installing frameworks like CrewAI/Hermes/OpenClaw as core infrastructure.

MSG 6–8: Operator uploaded the three CC triage outputs (`AITEAM_Mark_Kashef_Distilled_Core.md`, `AITEAM_Source_File_Relevance_Report.md`, `AITEAM_Triage_Process_Log.md`). I read all of them end-to-end (had to use multiple `view` calls due to truncation). Three key takeaways surfaced: (1) the judge doesn't need to be a more expensive tier than the writer — Mark uses Haiku 4.5 for both; (2) the Self-Improving Kit is the most directly applicable pattern in the entire corpus; (3) Section 9 of the Core (Things to Adapt for Stonecreed) is the literal mapping table that makes the kit usable.

MSG 9–10: Operator uploaded the V2 and V3 Visual Guide extracts. Read both fully. Flagged that V2 makes an empirical claim about Anthropic's Channels having 6 open GitHub bugs (silent message drops on the Telegram plugin), and that this contradicted what I'd framed as a "settled" UST decision.

MSG 11: Operator corrected me — Channels was never decided, the architecture is genuinely being designed in hybrid form right now. I owned the overstatement and laid out the real decision space: three options (Channels native, grammy bot, hybrid with cron-backed autonomous + Channels-or-grammy interactive layer). Recommended verifying empirically before committing.

MSG 12: Operator asked CC to verify the Channels vs grammy question. I drafted the prompt and runbook.

MSG 13–14: Setup question on which folder to open CC in — answered `~/Desktop/AITEAM_distilled/`.

MSG 15–17: Operator asked for a simple list of goals. First produced a 9-item list, then 10 more "operating style" goals. Operator pushed back: the operator's real goals include AI dashboard, head orchestrator, complete memory system, file management, Obsidian vault, autonomous researcher, idea researcher, dev team agent, Telegram watcher, crypto/AI trends scanner, plus affiliate sites and the persona project. Re-ranked into a final 19-goal list grouped Mission / Infrastructure / Worker agents / Businesses.

MSG 18: Operator pulled the substrate table from earlier and asked if it was still right given the new goal set. I extended it with five new rows (Quality scoring loop, Dashboard, Head orchestrator, Monitor/scanner agents, Dev team agent, Persona) and noted that "Editor scoring loop" should be generalized to "Quality scoring loop (any agent that needs it)."

MSG 19: Operator returned with CC's verification results — verdict: skip Channels, build grammy, confidence 8/10. Three findings: PR #1397 fixed part of the bug surface, but the architectural root cause (no daemon mode, issue #1594) is still open; bugs reproduce on Mac M4 / macOS 15.4 with multiple sessions (exactly the target hardware); two new bugs filed May 13 (bug discovery hasn't plateaued). I synthesized the implications for the build plan.

MSG 20: Operator asked the ELI14 explanation of the whole substrate table. Produced workshop-metaphor walkthrough.

MSG 21–22: Operator asked about RAG migration cost + the voices system (ingestion from trusted experts). Two genuinely good extensions of the architecture. Recommended building wiki + RAG hybrid from day one behind a single `retrieve_context()` function, given the voices system will produce 5,000+ documents within a year. Detailed the voices architecture (signal_sources.md, raw/voices/, distill step, weekly synthesis, trust weights, decay on relevance, personal-vs-team separation).

MSG 23–24: Operator asked for short summary of wiki vs RAG vs both. Recommended both from day one. Operator confirmed: "seems like both would be best."

MSG 25: Operator asked for comprehensive handoff file for next chat to be expert on this session. Asked 10 questions first.

MSG 26: Operator answered all 10 — grammy locked, Mac Mini set up, operator is the dev cofounder, build order roughly right but first project undecided (leaning PLR), voices names deferred, persona timing soft, first site undecided, budget flexible, no corrections to surface, audio brief declined.

MSG 27: Generated the 17-section handoff file (~5,770 words, 507 lines) and saved to outputs.

MSG 28: Operator typed "ctx" — this file.

---

## Decisions Made

### Architecture (locked)

- **Phone interactive layer: grammy bot.** Built from Mark Kashef's V2 REBUILD_PROMPT spec. ~500 lines of Node. Wrapped in `notify.sh` abstraction. Decision based on CC verification 8/10 confidence: Channels has no daemon mode (issue #1594 still open), reproduces 50% message loss on Mac M4 / macOS 15.4 (issue #38098), bug discovery hasn't plateaued.
- **Memory retrieval: wiki + RAG hybrid from day one.** Single `retrieve_context(query)` function as the only abstraction agents see. Wiki for human-readable debugging. RAG for scale and semantic search. Both built on Mark's tuning numbers (768-dim Gemini embeddings, Float32Array storage, dedup 0.85 cosine, retrieval 0.3 cosine min, importance 0–1, salience 0–5, decay tiers, 30-min consolidation).
- **Quality scoring is universal, not just for Editor.** Self-Improving Kit pattern generalizes — any agent that produces output runs through rubric-driven scoring with 4-band decision tree.
- **Folder structure proposed:** `~/brain/` for knowledge (Obsidian vault, projects as subtrees including aiteam/, persona/, ust/, payroll/, voices/ shared in meta/), `~/agents/` for orchestration (lib/, config/, per-agent folders, _template/, skills/), `~/store/aiteam.db` for SQLite state.
- **Voices system committed as Phase 2 build** but seed names deferred. Architecture is fully specified; adding voices later is trivial.
- **Dashboard:** auto-rebuilt `command-center.md` (durable) plus optional Hono web view.
- **Head orchestrator:** single CC agent with CLAUDE.md routing rules.
- **Persona / Trading project:** parallel-track, timing soft, no urgency. Operator updates `current-thesis.md`, agent drafts content from thesis (analysis stays with operator).

### Goal set (canonical)

- **Final 19-goal list in Section 3 of the handoff supersedes earlier lists** in AITEAM project instructions.
- Mission (1–7): the $200/mo break-even bar, helpful tools/sites, traffic + display ads, passive operation via agents, easy first, Mac Mini fleet, durable swappable infrastructure.
- Infrastructure (8–12): dashboard, orchestrator, memory system, file management, Obsidian vault.
- Worker agents (13–17): autonomous researcher, idea researcher, dev team, Telegram/Twitter watcher, crypto/AI trends scanner.
- Businesses (18–19): affiliate/display-ad sites, AI influencer/persona.

### Operating rules confirmed

- Mac Mini M4 is set up and ready (operator confirmed).
- Operator is the dev cofounder — not a separate person. Read technical content at full depth; adapt style to operator's video/audio preference.
- Build chassis first (Phase 0), then orchestrator + one worker (Phase 1), then expand worker fleet (Phase 2), then persona (Phase 3).
- Promotion-by-evidence: don't hire agent N+1 until agent N is overburdened or producing clear value.
- Budget is flexible during Phase 0; cost discipline (under 50% of last month's revenue) is permanent.

---

## Decisions NOT to Do Something

- **Don't decide first site yet.** Operator wants to think more. Leaning PLR / Etsy / Gumroad but uncommitted. Chassis work is identical regardless — don't delay on this.
- **Don't run the empirical Channels test.** It would just confirm what's already decided. Test runbook is preserved on disk for future revisit if Channels gets daemon mode.
- **Don't install Channels or run `claude --channels`.** Decided against.
- **Don't pre-build all 12 agents.** Hire one at a time, validate, then next.
- **Don't generate "Three Things Worth Considering" footers.** Operator asked for that to stop earlier ("no more TTWC this session").
- **Don't lecture operator on research-mode tendency.** Surface once gently at end of session if plainly stuck; otherwise leave alone.
- **Don't produce audio briefs unless asked.** Operator said "no audio yet maybe later" on the handoff.
- **Don't commit voices seed names.** Operator declined. Architecture works with zero voices.
- **Don't push persona project urgency.** Operator said "soonish, idk on timing yet, depends."
- **Don't assume Mac Mini software state.** Hardware is set up; software install status is unclear. Ask before running install commands.

---

## Data Changes

None. This was a strategy/architecture/triage session — no code shipped, no Airtable mutations, no scripts executed on operator's machine.

The session DID produce one deliverable file: `AITEAM_Session_Handoff_2026-05-14_Chassis-And-Decisions.md` (5,770 words, 507 lines, in outputs).

---

## New Information Learned

### About the operator
- **Operator IS the dev cofounder.** Earlier framing of "I have zero coding experience, dev cofounder will execute" was incomplete. The operator is the one building. This means future technical content can be deeper.
- **Processing preference confirmed:** video at 2x is comfortable, reading dense markdown is slow. Audio briefs declined for this session but option stands.
- **Operator catches framing errors well.** Twice in this session the operator corrected my overstatements (Channels as "decided," then again on which goals were in their top 10). Both corrections were on-target. Trust operator's self-knowledge.

### About the Mac Kashef corpus (consolidated from CC triage)
- **Self-Improving Kit is the single most directly relevant pattern** for AITEAM. Not the headline ClaudeClaw architecture.
- **Mark uses Haiku 4.5 for both chat AND reflection judge.** The asymmetry is in the rubric, not the model tier. Editor can probably run on Haiku/Llama 70B if rubric is rigorous.
- **CLI inheritance pattern** (install once globally, every agent inherits via tool allowlist) is the highest-leverage architectural move in the kit.
- **Section 9 of the Distilled Core** (Things to Adapt for Stonecreed) is the bridge that makes the kit usable — explicit Mark-pattern → Stonecreed-adaptation mapping for every major piece.
- **The 80/20 thesis** ("AI OS is 80% data engineering, 20% AI") is load-bearing. Silver-platter pattern (weekly summary tables agents read instead of raw data) is the implementation.
- **Mark explicitly designed the agent system to be framework-agnostic.** ClaudeClaw is a removable layer on top of Claude Code, not a replacement for it. This aligns with UST's wrapper-script portability strategy.

### Channels vs grammy verification findings (May 14, 2026)
- PR #1397 (merged April 14, 2026) fixed 3 of 4 polling-resilience bugs in `claude-plugins-official#788` — real progress but partial.
- Issue #1594 (HTTP/SSE daemon transport — the architectural root cause) is OPEN with no maintainer response. Until shipped, Channels is foreground-only.
- Issue #49151 (hot-reload disconnect on multi-session) and #38098 (auto-loads in all `claude -c` sessions causing ~50% message loss) both OPEN, both reproduce on Mac M4 / macOS 15.4.
- Issues #1838 and #1839 filed May 13, 2026 — bug discovery hasn't plateaued.
- Channels officially in "research preview" per Anthropic, shipped March 20, 2026.

### Voices system insight
- Architecturally identical to the other monitor/scanner agents (cron + headless claude -p + inbox + librarian sort).
- Strategically the highest-value worker agent — produces no direct revenue but feeds learning rate across all other agents and projects.
- Volume control critical: 50 voices × 20 posts/day = 1,000 daily captures. Librarian must prune aggressively before distill step.
- Personal-vs-team separation: agents only read `voices/team/`, never `voices/personal/`.

---

## Open Loops Identified

### Active decisions for next session
- **First site target.** PLR/Etsy/Gumroad leading but uncommitted. Decision aid welcome but not forcing.
- **Voices seed list.** Operator deferred. Re-ask when chassis is closer to voices-system buildout.
- **First worker agent after orchestrator.** Depends on first-site decision.
- **Editor model tier.** Default plan Sonnet; Mark's evidence suggests Haiku. Test before committing.
- **launchd strategy.** One service or multiple? Defer until chassis is up.

### Chassis build sequencing (proposed, not yet committed)
1. Verify Claude Code on Mac Mini (`claude --version`)
2. Create folder structure (~/brain/, ~/agents/, ~/store/)
3. Build wrapper scripts (ai-do.sh, ai-think.sh, ai-cheap.sh)
4. Build notify.sh stub
5. Initialize SQLite with WAL mode + Mark's tables (sessions, memories, consolidations, hive_mind, mission_tasks, scheduled_tasks, audit_log, conversation_log, token_usage)
6. Build retrieve_context() with wiki + RAG dual-path
7. Initialize Obsidian vault
8. Build command-center.md auto-regen
9. Build orchestrator agent folder
10. Build grammy bot from Mark's V2 spec

Roughly 1-2 focused days of work for steps 3-9 (chassis), separate day for grammy bot.

### Persona project
- Timing soft. Not blocking.
- `current-thesis.md` template + content drafter shape decided.
- Niche position (cycle analyst vs macro translator vs hybrid) still open.
- Persona archetype, disclosure stance, name, LLC formation all deferred.
- 30-day stripped-down Twitter test before any agent infrastructure.

### Carried over from prior sessions (unchanged)
- Block A SEO work (6 CTR rewrites) — UST
- Iowa contractor outreach (66 contacts, deferred 7+ sessions) — UST
- Federal pipeline execution (Alaska next after FPC Yankton) — UST
- Persona project niche position decision

---

## Patterns / Failure Modes Surfaced

### Documents-block partial delivery
When operator uploaded two files at MSG 4–5, only one appeared in the documents block. I initially said "Read both" but had to walk it back when operator asked "did you really read them fully?" — used `view` on project copy to actually read the second file end-to-end. **Pattern:** when files are uploaded via documents block, verify each one actually arrived in context before claiming to have read it. If anything's missing, use `view` on the `/mnt/project/` or `/mnt/user-data/uploads/` copy to actually read it.

### Overstating "decided"
At MSG 10–11 I framed Channels as a "settled UST decision" that Mark's evidence contradicted. Operator corrected: "Channels was never 'decided on' nothing is decided yet we are trying to figure it out right now." The UST Brief actually said "Telegram Channels (architect so it's removable)" — explicitly open. **Pattern:** when reading prior context-save files, distinguish between "decision made" and "tentatively in plan, marked removable." Don't elevate the tentative to the decided. The hedge language matters.

### Operator pushes back when wrong
Operator caught two of my overstatements in this session and pushed back firmly but fairly. Both corrections improved the analysis. **Pattern:** when operator pushes back, take the correction at face value. Don't defend or re-explain — just integrate and continue with the corrected frame. Operator's self-knowledge is reliable.

### Goals lists need reconciliation
First two goal-list responses produced abstractions (operating principles, themes) when operator wanted concrete builds (dashboard, orchestrator, dev team agent, voices). Operator named the actual goals explicitly in MSG 17. Then operator added that the original 7 mission goals from project instructions were ALSO still in the top 10. Reconciled into a 19-item list grouped Mission / Infrastructure / Worker agents / Businesses. **Pattern:** when asked for "goals," distinguish operating principles from concrete buildable items. Operators usually want the concrete list. Re-rank when context shifts; don't insist on the original framing.

### Long technical responses tolerated when subject-matter justifies
This session had multiple 800–1,500 word responses on architecture decisions. Operator engaged fully with each. **Pattern:** when the operator's question is genuinely complex (RAG vs wiki, channels verification, voices system architecture), longer responses are correct. The "short and dense" preference is for casual chat. Technical depth is welcome when the subject demands it.

### Handoff file as cross-session asset
Generated `AITEAM_Session_Handoff_2026-05-14_Chassis-And-Decisions.md` (5,770 words, 17 sections). **Pattern:** when a session produces large strategic + decision content, generate a standalone handoff. Context save references it. The handoff lives in outputs; the context save lives in the session record. Both are useful.

### "ctx" trigger reliability
Triggered cleanly at MSG 28. **Pattern:** "ctx" remains the operator's clean trigger for context save generation. Generate, save to outputs, present file.

---

## Critical Reference Files Generated This Session

1. **`AITEAM_Session_Handoff_2026-05-14_Chassis-And-Decisions.md`** — 5,770 words, 507 lines, 17 sections. The canonical reference for the next AITEAM chat. **MUST READ before responding on any AITEAM build topic.** Sections 4 (substrate table), 5 (grammy decision), 6 (wiki+RAG), 11 (folder structure), 12 (gotchas), 15 (how next chat should behave) are highest-density.

2. **This context save.**

---

## What the Next Chat Should Do First

If next chat is general UST work or persona work:
- Treat the handoff as available reference but not automatically loaded.
- Read prior context save first (UST_Context_Save_2026-05-13) and the Persona brief (UST_Agent_System_And_Persona_Brief_2026-05-13).

If next chat is AITEAM build, architecture, chassis, agent design, or anything related to the substrate:
- Read `AITEAM_Session_Handoff_2026-05-14_Chassis-And-Decisions.md` in full.
- Treat sections 4 (substrate table) and 5 (grammy decision) as authoritative — do not re-derive.
- Treat section 13 (open decisions) as the active design space.
- If operator opens with a build question, default to step-by-step build instructions with exact commands, exact file paths, exact code.

If next chat is "I'm at the Mac Mini, where do we start":
- Follow the proposed chassis build sequence in section 10 of the handoff (Mac Mini setup section) and section 11 (folder structure).
- Steps 1–8 of the proposed sequence (verify CC, create folders, wrappers, notify.sh, SQLite, retrieve_context, vault, command-center, orchestrator) is roughly 1–2 focused days.
- Don't install Channels. Don't pre-build voices. Don't pre-build agents 6–12.

If the operator asks "what's the first site?":
- This is the leading open decision.
- Operator is leaning PLR / Etsy / Gumroad but uncommitted.
- Offer a 30-minute decision aid: three options (PLR digital products, SEO calculator/tool site network, programmatic SEO directory) with risk, time-to-revenue, compound mechanics, required first-build skills.
- Don't push. Let operator pick. Chassis work is identical regardless.

---

## Output Files (this session)

- `AITEAM_Session_Handoff_2026-05-14_Chassis-And-Decisions.md` (created MSG 27, presented to operator)
- `AITEAM_Context_Save_2026-05-14_Triage-ChassisLock-GrammyDecision.md` (this file)

---

*End of context save.*
