# AITEAM — Context Save 2026-05-24 — Domain Hunter + Idea Agent Fix + Issues Queue

**Session date:** 2026-05-24
**Duration:** ~full-day session
**Parallel chats:** Single chat, no D-number collision sync needed.

---

## Headline

Three major builds shipped, all autonomous, all on cron:

1. **domain-hunter v0.1.5** — daily expired-domain scout with topical-fit gating against ust/asbestos/ssg, Wayback sanity check, Ahrefs deep links in Telegram digest
2. **idea-agent synth fix + daily cadence** — robust JSON parser, retry-once, hardened prompt, switched weekly→daily at 07:10
3. **Issues Queue dashboard view** — `/issues` route, capture daemon, "Generate CC Prompt" button, fresh-start anchor

All three cron-installed and live. $0 real money. Combined Max-sub burn for new agents: ~$25-28/mo projected.

---

## Decisions Made

### Domain Hunter v0.1.5

- **Scope correction:** v0.1 shipped without the topical scoring, backlink data, or auction data the original spec called for. v0.1.5 adds topical-fit scoring against 3 named sites. Backlink/auction data deferred (would require $10/mo ExpiredDomains.net Pro).
- **3 target sites locked:**
  - `ust` — ustcontractors.com — Underground storage tank contractors (CONFIRMED: operator's most important site)
  - `asbestos` — asbestoshq.com — Asbestos abatement and contractors
  - `ssg` — smartsourceguide.com — B2B services comparison (answering services, fleet tracking, IT support)
- **Killed the weak_301 track** — non-topical backlinks aren't worth surfacing (transfer 5-15% equity at best, can carry penalty signals from prior pharma/foreign/spam usage)
- **Topical-fit threshold:** 0.30 minimum to surface
- **Sort:** topical_fit_max DESC (no auction data, so no time-remaining sort)
- **Wayback gate:** clean | unknown only (not_archived currently dropped — flagged for potential v0.2 loosening if too many "0 candidates" days)
- **Cadence:** Daily 06:30 PT — analysis showed expiring auctions run 5-10 days, no benefit to hourly polling
- **Manual Ahrefs check workflow:** digest provides Ahrefs deep link per candidate; operator spends 2-3 min per domain in Ahrefs Site Explorer

### Idea-Agent Fix

- **Root cause of failure:** Sonnet rambled 34K tokens with prose interstitials and fragmented JSON fences. Parser was brittle, choked, surfaced cryptic JSON error
- **Three-part fix:**
  1. Hardened prompt with anti-narration guardrails ("first char must be `{`, last must be `}`")
  2. 3-tier robust parser (strict → fence-strip → bracket-extract → exit 4)
  3. Retry-once on exit 4 with corrective prompt prefix
- **Budget cap removed** — initial $0.20 cap aborted healthy runs ($0.35-0.45 actual); the hardened prompt is the real runaway guard
- **Cadence switched weekly → daily, 07:10 PT** (5min after market scribe to avoid stacking)
- **AI_DO_MAX_BUDGET_USD passthrough kept in ai-do.sh** as feature for future agents (idea-agent intentionally doesn't use it)

### Issues Queue

- **Scope decisions:**
  - Capture every audit_log error/failure (broad filter, tune later)
  - 4 status states: open / in_progress / resolved / wontfix (simpler beats richer for solo operator)
  - View + status + "Generate CC Prompt" button (manual version of eventual Telegram→CC fix loop)
  - **No backfill** — fresh start, anchor at id 2991
- **Routing:** separate `/issues` page (not folded into `/` SPA), token-gated data/mutations, ungated page load
- **Home counter:** "Issues: N open" link added to existing `/` page
- **Local tail feed:** `~/agents/issues-capture/log/new_issues.log` for headless monitoring without dashboard
- **Telegram piping deferred to v1.1** — need real noise-rate data first

### Three-Step Plan (operator-defined)

1. ✅ Fix idea-agent
2. ✅ Build Issues Queue
3. ⏳ Telegram → CC auto-draft fix loop — **deferred 1-2 weeks** until we have real failure-pattern data

---

## Files Created/Modified

### Domain Hunter (v0.1 + v0.1.5)
- `~/agents/domain-hunter/` — full agent (8 commits across two phases)
- `~/agents/domain-hunter/config/sites.yaml` — 3 site definitions with niches + related topics
- `~/agents/domain-hunter/score/topical_fit.py` — Haiku-based site relevance scoring
- `~/agents/domain-hunter/score/backfill_topical.py` — one-off backfill of existing rows
- `~/store/aiteam.db` — added: `domain_candidates` table + topical columns (topical_fit_asbestos, topical_fit_ust, topical_fit_ssg, best_site_match, topical_fit_max)
- `~/agents/lib/cron.txt` — domain-hunter daily 06:30 (INSTALLED)

### Idea Agent
- `~/agents/idea-agent/synthesize.sh` — 3-tier parser + retry-once + tightened invocation
- `~/agents/idea-agent/prompts/synthesize.md` — hardened JSON-only constraints
- `~/agents/lib/ai-do.sh` — added AI_DO_MAX_BUDGET_USD passthrough (feature, default off)
- `~/agents/idea-agent/CLAUDE.md` — documented daily cadence, real cost ~$23/mo, no synth cap
- Crontab — weekly→daily migration, 07:10 (INSTALLED)

### Issues Queue
- `~/store/aiteam.db` — added: `agent_issues` table + 3 indexes (status, created_at DESC, agent_id)
- `~/agents/issues-capture/capture.sh` — daemon with start_anchor_id state (anchored 2991, fresh start)
- `~/agents/issues-capture/log/new_issues.log` — local tail feed
- `~/agents/dashboard/server.js` — new `/issues` route + data feed + 3 POST routes (status, notes, generate-prompt) + home counter on `/`
- `~/agents/lib/cron.txt` — issues-capture */5 (INSTALLED)

---

## Cost Data Points

| Source | Burn (Max-sub value) | Real $ |
|---|---|---|
| domain-hunter v0.1 build | $0.38 | $0 |
| domain-hunter v0.1.5 build | $0.24 | $0 |
| Idea-agent fix verification runs | ~$1-1.50 | $0 |
| Issues Queue build | $0 (no LLM calls) | $0 |
| **Projected ongoing daily** | | |
| domain-hunter | $0.05-0.10/day = ~$3/mo | $0 |
| idea-agent | ~$0.35-0.45 synth + ~$0.35 scan/search = ~$23/mo | $0 |
| issues-capture | $0 (no LLM) | $0 |

**Key insight:** idea-agent at daily cadence is the single biggest Max-sub line item among new agents. ~11% of $200/mo mission-bar-equivalent in Max-sub burn. Worth watching whether daily output justifies the burn vs weekly.

---

## Lessons Learned

### Operator-Communication Lessons

- **CC's pre-flight discipline caught a real spec error on v0.2.** v0.1 had shipped without the backlink/auction/topical infrastructure I'd assumed when writing v0.2. Pre-flight surfaced the gap before any code was touched. This is exactly the pattern that's saved the project multiple times in May — never skip pre-flight to "save time."
- **Composite-scoring with weak_301 was wrong design.** Operator clarified twice ("sites useful to OUR sites") before I fully internalized it. When operator pushes back, take the correction at face value, don't half-integrate it.
- **Operator's mental model of value:** UST is the priority site, not asbestos (despite asbestos being the live/active one in build state). This shifts where to expect digest signal — UST is also the narrowest niche, so lowest natural inventory match rate.

### Technical Lessons

- **claude CLI has no --max-tokens flag.** It does have `--max-budget-usd`. Don't assume CLI flags exist; verify before specifying in build prompts.
- **--json-schema is a trap for ai-do.sh.** Routes output through internal tool-use turns, leaves `.result` empty. Rearchitecting around it touches shared infra used by every agent. Stay away unless dedicating a session to it.
- **Sonnet rambles on ambiguous "return JSON" prompts when context is large.** Hardened prompt with "first char must be `{`, last must be `}`" + retry-once is the working pattern. Document this for any future agent that needs structured JSON output.
- **audit_log column naming:** `actor_id` not `agent_id`, `payload_json` not `result`. Standard project schema, easy to forget across agents.
- **The bash 3.2 gotcha for cron PATH was preempted** — operator's crontab already has a PATH header line. D054 (systemic PATH audit) implicitly resolved for new agents via this header pattern. Keep using it.
- **ExpiredDomains.net free tier is thin.** No TF/CF/RDs, no auction prices, no direct auction URLs. Useful as a name-only feed, useless for backlink-driven domain hunting. Pro membership ($10/mo) is the inflection point for that.

### Process Lessons

- **The "research before building" pattern was avoided this session.** Three substantial builds shipped same-day. Operator showed bias-toward-action throughout. No backslide.
- **Three-step plan stayed intact across complex builds.** Operator clearly stated the order at the start; each step closed cleanly before the next began. Sequencing discipline.
- **CC making honest "this was wrong, here's the round-trip" commit traces is the right behavior.** Two commits (budget cap added then removed) is more honest than rebasing to hide the learning.

---

## DEFERRED ITEMS

(Numbers TBD — sync to ~/brain/projects/aiteam/DEFERRED.md at next CC session via grep -oE "\bD[0-9]+\b" command from project instructions.)

### From this session

- **D-new — Domain hunter: ExpiredDomains.net Pro evaluation** — $10/mo for real backlink + auction data + direct auction URLs. Revisit after 2-4 weeks of v0.1.5 digests + if operator acquires 1+ domain via current flow.
- **D-new — Domain hunter: loosen Wayback gate to include not_archived** — current gate is clean|unknown only, dropping a lot of legitimately-obscure-but-clean domains. Tune after 1 week of digest data if "0 candidates" days are too frequent.
- **D-new — Domain hunter: UST-specific threshold tuning** — UST is narrow enough that 0.30 topical-fit threshold may be too strict. Consider 0.20 specifically for UST after 2 weeks of data.
- **D-new — Domain hunter: GoDaddy direct fetcher reactivation** — currently edge-blocked (Akamai 403). Options: DropCatch/NameJet swap, or GoDaddy Aftermarket API (paid, requires reseller). Defer until ExpiredDomains-only feels insufficient.
- **D-new — Idea-agent: calibration semantics under daily cadence** — `week_of` field accumulates 7x faster, calibrate.sh runs 7x/week against same week's data. Calibration gate logic was designed against weekly assumptions. Revisit after 2 weeks of daily data.
- **D-new — Idea-agent: cost-cadence reconsideration** — ~$23/mo Max-sub burn is highest of new agents. If 2-week observation shows daily output ≠ 7x weekly value, drop to every-other-day or weekday-only.
- **D-new — Issues Queue: nested-agent log path resolution** — current best-guess logic resolves top-level agents (domain-hunter/log/...) but fails on nested (market/scribe/...). Operator must edit generated CC prompts manually. Easy v1.1 fix.
- **D-new — Issues Queue: Telegram piping for new_issues.log** — defer until noise rate is known after 1 week of capture.
- **D-new — Issues Queue: capture filter tuning** — broad filter on first ship. After 1 week, tighten/widen based on real signal-to-noise.
- **D-new — Step 3: Telegram → CC auto-draft fix loop** — deferred 1-2 weeks for failure-pattern data collection. Design depends on what real failures actually look like.

### From prior sessions (still open, no new info)

- D045 — rewire run-batch.sh from `claude -p` (Opus) to `ai-do.sh` (Sonnet): ~75% cost reduction
- D056 — editor production runner (gated on operator-policy answers)
- D059 — SSG deploy throttle (when SSG actually ships first batch)
- The 7 operator-policy questions from cross-agent failure modes audit §7 still open

---

## Where We Left Off

**All three agents live on cron, fully autonomous:**

| Agent | Schedule | First autonomous fire | Expected behavior |
|---|---|---|---|
| domain-hunter | Daily 06:30 PT | Tomorrow 06:30 | Likely "0 candidates" most days |
| idea-agent | Daily 07:10 PT | Tomorrow 07:10 | 3 ideas + ~8 rejections, ~$0.40 burn |
| issues-capture | */5 min | Already firing | 0 issues so far, captures any *_failed |

**Next planned session focus:** Step 3 (Telegram → CC auto-draft fix) is deferred 1-2 weeks. Suggested mid-week check-in: review /issues data, idea-agent daily output quality, and any domain-hunter candidates that surfaced.

**Open immediate tasks for operator (not requiring Claude/CC):**
- Tap-test clipboard copy on `/issues` page (when first real issue surfaces)
- Spot-check idea-agent's daily output quality vs prior weekly output (signal/noise)
- Optional: tail `~/agents/issues-capture/log/new_issues.log` for live error feed

---

## Operator Corrections This Session

1. **"sites useful to OUR sites, not just sites with good backlinks"** — corrected composite-scoring design twice. Locked in v0.1.5 with topical-only gating.
2. **"ustcontractors.com is my main most important site"** — corrected the assumption that UST might be a placeholder. UST is priority #1, asbestos is the live build artifact.
3. **"I would like the idea agent to run daily not weekly"** — added scope mid-fix, properly handled with cadence-switch commit.

---

## Commits Landed This Session

**Domain Hunter v0.1 (6 commits):**
- ee9e34c, cf99967, 75aa423, 5a31df0, 8fa1f66, 0a4adad

**Domain Hunter v0.1.5 (6 commits):**
- 8c4ec29, dd52c53, 4f554d9, c2d5a49, 73ad067, 044d619

**Idea-Agent fix (8 commits):**
- ca21007, 344e34b, 5329a09, 8722089, 6192499, 77f11a8, 353d1d3, ce46867

**Issues Queue (6 commits):**
- bdc1909, eded032, d4df59d, f133cac, 40d7f6c, 0d099ab

**Total: 26 commits across 4 builds, all clean accurate messages, no narrow-message-on-wide-commit violations.**
