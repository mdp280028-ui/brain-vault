# AITEAM Context Save — May 15, 2026
## Topic: Site Portfolio Analysis + SmartSourceGuide Agent Spec

**Session date:** May 15, 2026
**Filename:** AITEAM_Context_Save_2026-05-15_SitePortfolio-SSGAgentSpec.md
**Companion deliverable:** AITEAM_Site_Portfolio_And_SSG_Agent_Spec_2026-05-15.md (full session work)

---

## SESSION SUMMARY

Strategic session. Operator asked for the list of sites AITEAM agents could run and walked them through three ranking lenses (failure rate → top 7 lock → speed-to-first-dollar combined). Mid-session pivoted when operator pasted the SmartSourceGuide briefing — established SSG as a real existing property that the AITEAM fleet could be applied to. Closed with full 8-agent build spec for taking SSG from 0% to 80% agent-run.

No code shipped. No agents built. This was a portfolio-direction + agent-spec session.

---

## DECISIONS MADE

### Portfolio Decisions

- **12 candidate site categories narrowed to 7 active focus.** Killed: PoD, KDP, Pinterest affiliate, crypto persona, AI dating/social platforms (failure rates 75-95%).
- **Top 7 locked for active consideration:** PLR Creation, Calculator sites, Programmatic SEO directory, Newsletter, Etsy/Gumroad PLR rebranding, Affiliate reviews, Faceless YouTube.
- **Three of the seven need a niche-lock first** (PLR Creation, Programmatic SEO, Newsletter) — share the HR/Real Estate/Accountants decision.
- **Only 3 of 7 require a website day one:** Calculators, Programmatic SEO, Affiliate reviews. Others can launch on platforms.

### SSG Strategic Decision

- **SmartSourceGuide adopted as first real AITEAM agent workload.** Recommendation accepted (implicitly via "spec out the agents"). Reasons:
  - 17 pages already attempting to index
  - Niches pre-locked (fleet GPS, IT/MSP, answering services)
  - Faceless model = agent content works without E-E-A-T problems
  - Working deploy pipeline (Next.js + Vercel + GSC) already exists
  - Skips 2-4 months of niche selection + site setup that PLR Creation or Calculators would need

### Agent Build Decisions

- **8 agents specced for SSG**, in this build order:
  1. Keyword Screener (Haiku)
  2. Research Agent (Sonnet)
  3. Writer (Sonnet)
  4. Editor — extend existing AITEAM agent with SSG rubric (Sonnet)
  5. Mechanical Auditor (Haiku)
  6. Internal Link Manager (Haiku)
  7. Publisher (Haiku, with approval gate)
  8. GSC Monitor (Haiku)

- **Build sequence: 5-6 weeks**, ~30-40 hours CC time, ~$20-50 API testing cost.
- **Parallel track recommended:** affiliate enrollment runs in parallel with agent build. Agents and affiliates land together at week 4-5.

### Explicit Non-Builds

- No Affiliate Enroller agent (human-only task)
- No standalone Affiliate Link Inserter until 3+ affiliates enrolled
- No Niche/Strategy Agent (operator-only)
- No Revenue Tracker until revenue exists

---

## FILES CREATED OR MODIFIED

| File | Path | Status |
|---|---|---|
| AITEAM_Site_Portfolio_And_SSG_Agent_Spec_2026-05-15.md | /mnt/user-data/outputs/ | Created and presented |
| AITEAM_Context_Save_2026-05-15_SitePortfolio-SSGAgentSpec.md | /mnt/user-data/outputs/ | This file |

No Mac Mini files touched. No agents built. No git commits.

---

## DATA POINTS LEARNED

### From the Operator (Corrections to Project Files)

- **SSG briefing claims affiliates enrolled. They are not.** ReceptionHQ, Smith.ai, Ruby, AnswerConnect, ManageEngine, LoneStar — none enrolled per operator. The SSG briefing is stale on this critical fact.
- **No active AITEAM-managed sites exist.** AITEAM project files describe business model categories, not specific sites.

### From This Session's Analysis

- SSG could be ~80% agent-run with the current AITEAM chassis + 8 specced agents
- The 20% that stays human: affiliate enrollment, final review, strategic decisions, Ahrefs UI work
- Critical path bottleneck: affiliate enrollment, not agent build
- The SSG agent stack doubles as a chassis for other niche-locked / niche-agnostic SEO plays in the top 7

---

## NEW LEARNINGS

### About the Project

- **SSG is structurally a much better first AITEAM workload than building from scratch.** Site exists, niches are locked, pipeline works. The work is encoding what the operator already does manually, not designing from blank.
- **Two-chat writer/auditor workflow from SSG maps directly onto AITEAM Writer + Editor (Sonnet-locked).** The architecture decision from Step 17 was effectively pre-validated by SSG's manual workflow.
- **Mechanical Auditor pattern from Mark Kashef distilled core has a real-world test case in SSG** — the 4/10 stub-overwrite disaster is exactly what mechanical auditing prevents.

### About SSG's Real State

- Briefing claims 17 pages live + 4 drafted; deployment status of those 4 unclear as of 4/21
- GSC: 22 URLs known, 5 fleet-tracking in "Redirect error" validation, 15 "Discovered – currently not indexed", 0 indexed
- Revenue: $0
- Affiliate programs: 0 enrolled (contra briefing)

---

## OPERATOR CORRECTIONS APPLIED

1. **"I haven't done this yet"** re: affiliate programs — agent spec corrected to treat affiliate enrollment as 0%, and flagged data integrity gap in SSG briefing.

---

## DEFERRED ITEMS

| Item | Trigger to revisit |
|---|---|
| Niche decision for non-SSG AITEAM work (HR vs Real Estate vs Accountants) | When AITEAM moves beyond SSG to PLR Creation, Programmatic SEO, or Newsletter |
| Build location decision: `~/agents/ssg/` vs separate `~/agents-ssg/` tree | Before building Agent 1 |
| Approval gate UI for Publisher: terminal prompt vs file flag vs dashboard vs Telegram | Before building Agent 7 |
| Order for week 1: Agent 1 first or affiliate apps first | Next session — current recommendation is both in parallel |
| Update SSG briefing to reflect zero affiliates enrolled | Next SMART project session |
| Revenue Tracker agent | When SSG has revenue |
| Affiliate Link Inserter agent | When 3+ affiliates enrolled |
| Apply SSG agent chassis to other top-7 sites (PLR Creation, Calculators) | After SSG 80% agent-run |
| AITEAM Phase 1 step 10c (cost chart) | Still queued per project instructions; not addressed this session |
| AITEAM Phase 1 steps 17a (silver platters), 17b (slash commands), 18 (war room discuss mode), 19 (failure modes) | Still queued; not addressed this session |

---

## OPEN LOOPS FROM THIS SESSION

1. **SSG ↔ AITEAM project boundary** — SSG lives in a separate Claude project (SMART prefix). AITEAM agents working on SSG creates a cross-project dependency. Worth clarifying: do SSG-related context saves go in AITEAM (because agents) or SMART (because content)? Recommendation: AITEAM owns agent build, SMART owns content output, cross-reference both.
2. **First agent CC prompt** — operator's likely next ask. Agent 1 (Keyword Screener) is the recommended starting point.
3. **AITEAM Phase 1 progress not advanced this session** — step 10c (cost chart) still next per project instructions. Did not touch it. Worth flagging if next session opens with "what's next on the build plan."

---

## PATTERNS / FAILURE MODES SURFACED

### The Stale Briefing Pattern
The SSG briefing claimed affiliates were enrolled. Operator correction caught it. **Pattern:** when a project file makes specific factual claims (X is signed up, Y is configured, Z is deployed), verify with operator before building on top of those facts. The 4/21 sitemap lesson from SSG's own learnings applies here: "Always verify the live file before recommending a fix to it." Extended version: always verify the live *state* before recommending action that depends on it.

### The Cross-Project Boundary Pattern
This session operated in AITEAM but consumed an SSG briefing and produced an agent spec that bridges both projects. Worth being explicit: agent build context (AITEAM) and content production context (SSG) are different sessions in different projects with different file prefixes. Future sessions should declare which project's context they're in.

### The Research-Before-Building Risk (per project instructions)
This was a strategy session — no agents built, no code shipped. Per the locked rule in project instructions, this is exactly the pattern flagged as the operator's biggest execution risk. The session output is action-oriented (concrete agent spec, build sequence, parallel-track affiliate enrollment recommendation), but execution hasn't started yet. **Watch for this pattern repeating in the next session.** If operator opens with "let me think about which agent first" — push back toward shipping Agent 1.

---

## WHAT THE NEXT CHAT SHOULD DO FIRST

If next session is AITEAM agent build:
- Read this context save
- Confirm operator's first build target (recommendation: Agent 1 Keyword Screener + start 4-6 affiliate applications in parallel)
- Write the actual CC build prompt for Agent 1
- Do not drift back into portfolio re-analysis — portfolio decision is locked

If next session is general AITEAM strategy:
- Read this save plus AITEAM_Build_Plan_2026-05-14_FINAL.md
- AITEAM Phase 1 step 10c (cost chart) is the documented next step on the build plan — not addressed this session
- Watch for the cross-project boundary between AITEAM and SSG

If next session is SSG content work (SMART project):
- The affiliate enrollment data correction needs to land in SMART project files
- The AITEAM agent spec is reference material but not yet built — SSG continues manual until agents come online

---

## OUTPUT FILES THIS SESSION

- AITEAM_Site_Portfolio_And_SSG_Agent_Spec_2026-05-15.md (comprehensive session deliverable)
- AITEAM_Context_Save_2026-05-15_SitePortfolio-SSGAgentSpec.md (this file)

---

*End of context save.*
