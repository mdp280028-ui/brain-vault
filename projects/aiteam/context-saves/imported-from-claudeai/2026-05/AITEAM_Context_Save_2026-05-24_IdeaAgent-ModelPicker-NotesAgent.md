# AITEAM Context Save — Idea Agent + /model Picker + Notes Agent

**Date:** 2026-05-24
**Session focus:** Three sequential agent/feature builds shipped end-to-end
**Status:** All three live, all smoke-tested at static-integration level, awaiting live phone smoke

---

## What shipped this session

### 1. Idea Agent (system-improvement weekly digest)

**Purpose:** Sunday 09:00 cron weekly. Surfaces 1-5 ranked improvement ideas to operator phone + saves full report to `~/brain/projects/aiteam/ideas/weekly_<date>.md`.

**Build:** Phase 1 (core) + Google search addendum (Serper) + three small fixes (Track A/B rubric, self-scoring trend, zombie deferred surfacing with 3x kill).

**Architecture (7-step chain):**
diagnose → zombie_check → scan → google_search → synthesize → publish → calibrate

**Pipeline:**
- Diagnose pulls 14 SQL aggregations from `~/store/aiteam.db` (audit_log patterns, cost trends, dead agents, orphaned starts, drafter_queue depth, verdict pass rates, repo freshness, etc.)
- Zombie check surfaces DEFERRED.md items 60+ days old, 3x-resurface kill rule, D-SSG namespace exception when SSG not yet enabled, first_seen_at grace for no-date items
- Scan: 4 RSS sources (Anthropic news, HN front page, r/LocalLLaMA hot, Ahrefs blog) — agent-local venv with trafilatura for full-fidelity body extraction
- Google search: Serper API, 8-12 targeted queries grounded in diagnose signals + 1 rotating wildcard, Haiku filters, $0.50/week hard cap
- Synthesize: Sonnet via ai-do.sh applies 5 hard tests + Track A (productivity, $/hrs) or Track B (structural/insurance with failure mode + frequency + impact) rubric, anti-duplication query against `idea_proposals`, operator-profile filter, forbidden phrases list. Zero ideas is a valid output.
- Publish: markdown report + Telegram digest of top 3 only
- Calibrate: weekly trend computed against operator_action outcomes; synthesize.sh never reads this data (air-gap with explicit code comment)

**Tables added:**
- `idea_proposals` (PK id, with operator_action enum pending/built/deferred/rejected/ignored)
- `idea_calibration` (weekly aggregates, NULL build_rate on zero-data weeks)
- `zombie_resurface_log` (PK on d_number, UPSERT with ON CONFLICT)

**First run output (2026-05-23 smoke):** 5 ideas all Track B (defensive infrastructure — appropriate for current operation state, no revenue signal yet to feed Track A). Sonnet self-rejected 7 candidates including a same-problem-same-mechanism duplicate (Test 5 working), a content-strategy idea routed to research-opportunity's lane (Test 1), and a D044 fix-queue item out of agent scope. Quality gate verified working.

**Cost:** Smoke ~$1.10 across two attempts. Inside POLICY Q2 caps.

**Commits:**
- 37b2451 — three migrations
- ad6df7a — agent code

**Tracking:** D087 (had D-number collision with sister chat that claimed D084-D086; we re-checked, used D087)

**Deferred Phase 2 (post-v1 observation):**
- #1 prediction + 30-day check-in
- #2 competitor failure mining
- #7 AITEAM-chat-only Telegram mining with 3+ mention threshold

**Deferred Phase 3 (conditional):**
- #4 revenue plumbing (when revenue exists)
- #5 case study research (with cargo-cult caveats)
- #8 $100/mo gap analysis monthly (reframed from $10M)
- #10 commitment audit (with pattern-not-individual fix)

---

### 2. /model Picker (per-agent Sonnet/Opus toggle from Telegram)

**Purpose:** Operator can toggle which model an agent uses via slash commands. Persistent (writes to DB), per-agent, default off (uses agent's hardcoded default).

**Build path:** Originally specced as inline keyboard with callback handlers. Operator chose slash-command grid instead — faster build (1.5h vs 4h), no new bot infrastructure needed.

**5 picker-eligible agents identified in pre-flight:**
- orchestrator_morning_brief (commands/morning_brief.sh)
- orchestrator_weekly_review (commands/weekly_review.sh)
- editor (editor/score.sh)
- briefer (market/briefer/compose_brief.sh)
- idea-agent-synth (idea-agent/synthesize.sh)

**6th added mid-session:** orchestrator-chat path — the free-text question routing in bot.js that was missing from initial picker. Wired with same resolve_model.sh pattern.

**Architecture:**
- New SQLite table `model_overrides` (PK agent_id, CHECK model IN sonnet/opus, enabled flag)
- `~/agents/lib/resolve_model.sh` — bash 3.2 compatible, APOS pattern, default-safe fallback
- `ai-do.sh` got new `AI_DO_MODEL_OVERRIDE` env hook (line 36, matches existing override patterns)
- 15+ slash command handlers in bot.js: `/model_<agent>_sonnet|opus|off`
- `/model` (no args) renders monospace grid showing current state per agent
- Opus selections prefix cost warning ("Opus uses ~5× tokens vs Sonnet")

**Slug-to-agent_id mapping:**
- omb → orchestrator_morning_brief
- owr → orchestrator_weekly_review
- editor → editor
- briefer → briefer
- ideas → idea-agent-synth
- (chat → orchestrator-chat added in 6th-agent addition)

**Commit:** 2927a71 — 9 files, 183 insertions

**Tracking:** D088 (umbrella for: more agents wired when scope expands, /model_*_once per-task override, Haiku addition to picker, future inline keyboard upgrade, /help discoverability, bot restart on settings change)

**Bot restart needed after build** — CC restarted bot from terminal (PID 88684, then 99463, then 7796 across the session). bot-launch.sh path bug found: dotenv reads from CWD not script dir. Operator deferred 1-line fix (cd "$(dirname "$0")" at top of launch script).

**`/restart` command:** Not yet built. CC restarted manually during this session.

---

### 3. Notes Agent (capture + organize → Obsidian brain)

**Purpose:** Operator captures notes/ideas/screenshots/forwards from phone, agent categorizes + files them into `~/brain/projects/aiteam/notes/`. Weekly sort consolidates inbox files into category files.

**Three capture modes built:**

**Mode 1 — Text notes (Phase 2):**
- Triggers: `/note <text>`, `note <text>`, `note: <text>`, `n: <text>` (case-insensitive regex `/^(/note|note[:\s]|n:)\s+/i`)
- Inserted as PRIORITY 2.5 in existing `message:text` handler (fires before orchestrator routing)
- Haiku categorizes against 7 seeds (ideas, crypto, websites, business, reminders, tasks, misc) or improvises new category
- Lands in `~/brain/projects/aiteam/notes/inbox_<YYYY-MM-DD>.md` with `## <HH:MM> <category>` heading
- Telegram reply: "📝 noted in <category>"

**Mode 2 — Image capture (Phase 2.5):**
- `bot.on("message:photo", ...)` — net-new handler
- Caption rule: empty caption OR caption starts with note-trigger → capture; non-trigger caption → routed to orchestrator-chat as vision question with `@<image-path>` prefix
- Images stored at `~/brain/projects/aiteam/notes/images/<YYYY-MM-DD>/<HH-MM-SS>_<hash>.jpg`
- Gitignored (`brain/.gitignore`: `notes/images/*/` with `.gitkeep` exception)
- Haiku vision call extracts: category, predefined bool, one-line description, key_data (numbers/tickers/prices/usernames/dates/transcribed text)
- max-turns=2 conservative ceiling for @-path attachment behavior (may inline natively at turns=1; v1.1 will test)
- Album handling: each photo separate update with shared `media_group_id`; caption attaches to first photo only

**Mode 3 — Forward capture with confirmation (Phase 2.6):**
- `handleForwardedMessage` detects via Bot API 7+ `forward_origin` OR legacy `forward_from`/`forward_from_chat`
- Replies with preview + sender metadata + confirmation prompt
- Pending forwards stored in `pending_forwards` table (5-min expiry, ORDER BY id DESC LIMIT 1 for most-recent-resolves)
- `/yes` → Haiku categorize and capture
- `/no` → discard
- `/yes_<category>` → manual category override (e.g. `/yes_crypto`)
- Auto-expiry sweep via `setInterval` (5 min) marks expired pending rows
- Forward entries in inbox: `## <HH:MM> <category> [forward]` + `**From:** <sender>` + `**Original time:** <ts>`

**Sort flow (Phase 3):**
- `sort.sh` cron at `0 10 * * 0` (Sunday 10:00 PT, staggered 1h after idea-agent)
- Reads all `inbox_*.md` files in notes/
- Single Sonnet call consolidates + merges conceptual duplicates
- Appends entries to category files (`notes/ideas.md`, `notes/crypto.md`, etc.)
- Image links adjust from inbox-relative `images/...` to category-relative `../images/...`
- Sorted inbox files move to `notes/archive/`
- First Sunday will be no-op (no inbox files older than today)

**Commits:**
- 73a2ecb — 6 files, 538 insertions (Phase 1-2.5 + Phase 3)
- 2011c1a — 3 files, 366 insertions (Phase 2.6 forward handler)

**Tracking:** D089 (umbrella for: per-note retrieval `/find`, voice note → note, note-to-task promotion to idea_proposals, date-range retrieval `/notes 7d`, deletion/edit commands, album caption replication, max-turns=1 retest, /help discoverability, message_id threading, pending_forwards hard cap at 1000)

**v1 known limits documented:**
- Caption only attaches to first photo of album
- max-turns=2 for image captures (conservative)
- Bot commands not yet in `/help`
- No reply threading
- `pending_forwards` rows accumulate forever (cap deferred until >50)

**Synthetic test entries** in inbox_2026-05-24.md from CC's standalone smoke (crypto from "alice", reminders from "@founder_user") — will be archived Sunday or manually removable.

---

## Cross-cutting patterns this session

### Pre-flight discipline held
Every build started with pre-flight that returned blocking findings:
- Idea agent: caught DEFERRED.md format heterogeneity, surfaced 12 unparseable items, escalated approach for operator approval
- /model picker: caught absence of Grammy callback infrastructure, escalated cost estimate from 2.5h to 4h before committing
- Notes agent: caught vision-attachment uncertainty (Haiku @-path vs file_id API), mitigated with max-turns=2

CC never built on unverified assumptions. All three builds had at least one "stop and report" gate that surfaced real spec questions.

### D-number collision pattern (parallel chats)
D084 was claimed by sister chat (SSG Lane C) between this chat's pre-flight check and commit time. CC re-checked highest D-number, used D087 instead. Pattern is the recurring parallel-chat collision flagged in project instructions. The fix `grep -oE "\bD[0-9]+\b" ~/brain/projects/aiteam/DEFERRED.md | sort -u -V` worked.

### Operator override of recommendations
- Operator overrode "build idea-agent after GEO Phase 3 ships" → built it now anyway
- Operator overrode "drop items 4/5/7/8/10 for v1" → asked for fixes to all 10, then accepted phased Option A
- Operator overrode inline keyboards in favor of slash-command grid (60% time savings, accepted UX tradeoff)
- Both Project Instructions warnings about research-before-building were raised. Operator chose to proceed each time. Documented for pattern visibility.

### Quality-gate-as-feature
Operator's instinct on filtering ideas led to material design improvements:
- 5 hard tests in synthesize prompt
- Forbidden phrases list (no hedging vocabulary)
- "Zero is a valid answer" explicit instruction
- First-run cap at 5 ideas to calibrate trust
- Track A/B split to keep defensive builds buildable

These changes made the first run actually produce 5 disciplined Track B ideas instead of 10 plausible-sounding listicle items.

### Honest negative-effect reassessment
After operator asked "do any have negative effects" — Claude reassessed and found 6 of 10 items had medium-high negatives, including #7 (Telegram mining) and #8 ($10M experiment) which Claude had initially recommended. Items downgraded transparently. Operator then asked how to reduce the negatives, leading to fix-each-item pass where most negatives dropped to low. The honest reassessment pattern produced better final design than the initial enthusiastic recommendation would have.

---

## DEFERRED ITEMS (added this session)

### D085 — Hygiene: remove test agent_ids
Remove `step16_sonnet_v2` and `test_do_v2` from fleet inventory (or filter from audit_log queries). Surfaced during idea-agent diagnose Q(d). Trigger: next hygiene session.

### D087 — Idea agent v1.1 / v2 follow-ups
Umbrella tracking:
- Monthly Opus self-critique pass after 4+ weekly runs
- Operator-action feedback mechanism (slash command or web UI to mark idea_proposals.operator_action; currently SQLite UPDATE direct)
- Expanding scan.sh sources after 4-week burn-in
- 5th feed source choice (placeholder)
- Phase 2 features (prediction + check-in, failure mining, AITEAM-chat mining) after first weekly run output
- Phase 3 features (revenue plumbing, case studies, $100/mo gap, commitment audit) conditional on triggers

### D088 — /model picker v1.1
- Wire remaining agents to resolve_model.sh as scope expands
- `/model_*_once` per-task override
- Haiku addition to picker
- Inline keyboard upgrade when bot.js gets callback handlers
- /help discoverability for /model commands
- Bot restart automation on settings change

### D089 — Notes agent v1.1
- Per-note retrieval `/find <text>`
- Voice note → note (Whisper integration with /note context)
- Note-to-task promotion (move notes/tasks.md entries into idea_proposals as Track A)
- Date-range retrieval `/notes 7d`, `/notes month`
- Note deletion / edit commands
- Album caption replication to all photos in album
- max-turns=1 retest for image @path attachment
- /help discoverability for note commands
- Message_id threading for reply chains
- pending_forwards hard cap (target >1000 unresolved as trigger)

### Implicit deferreds (not numbered)
- bot-launch.sh self-contained `cd "$(dirname "$0")"` patch (1 line, operator deferred during session)
- `/restart` Telegram command (proposed mid-session, operator deferred)

---

## Open loops / next session triggers

**Live phone smoke tests pending** for all three builds:
1. Idea agent: first Sunday cron fires 2026-05-24 (today/tomorrow depending on time of save) 09:00 PT. Will run unsupervised.
2. /model picker: 9-step smoke (operator from phone — render grid, toggle states, verify token_usage rows show new model after override)
3. Notes agent: 7-step text smoke + 9-step forward smoke from phone

**Mark idea_proposals operator_action**: 5 ideas from 2026-05-23 sit in `idea_proposals` with operator_action='pending'. Until /idea_mark Telegram handler ships (D087), must manually SQLite UPDATE. File at `~/brain/projects/aiteam/ideas/weekly_2026-05-23.md`.

**Sunday cron stack** now includes:
- 0 9 * * 0 — idea-agent run_weekly.sh
- 0 10 * * 0 — notes-agent sort.sh (staggered 1h)

---

## What did NOT happen this session
- No GEO Phase 3 work (operator chose idea-agent build first, despite Claude's recommendation)
- No SSG content production
- No AdSense / affiliate enrollment
- No revenue work — mission bar still $0
- No first organic asbestos slug indexing check
- No editor production runner (D056 still gated on policy)

Total session was infrastructure/agent build, not revenue-adjacent work. Pattern flagged in project instructions held — operator continued building systems first.

---

## Cost rollup (estimated, Max-sub burn not real $)

- Idea agent smoke: ~$1.10 across two end-to-end attempts
- /model picker: minimal (static integration test, no live LLM call)
- Notes agent: minimal (standalone capture.sh smoke, two synthetic entries)

Estimated total session burn: ~$2-5 in Max-equivalent value. Real-money cost: $0.

---

## Files added/changed this session (high-level inventory)

**New agent directories:**
- `~/agents/idea-agent/` (14 files including agent.yaml, CLAUDE.md, rubric.md, prompts/synthesize.md, 7 phase scripts + parse_deferred.py + scan_poll.py + venv)
- `~/agents/notes-agent/` (4 files: agent.yaml, CLAUDE.md, capture.sh, sort.sh)

**New library helpers:**
- `~/agents/lib/resolve_model.sh`
- `~/agents/lib/ai-do.sh` modified (AI_DO_MODEL_OVERRIDE hook)

**Migrations (all dated 2026-05-23 or 24):**
- idea_proposals
- idea_calibration
- zombie_resurface_log
- model_overrides
- pending_forwards

**Bot.js:**
- 18 → 30+ slash commands (estimated)
- New handlers: photo, forward, /note, /model_*, /yes, /no, /yes_<category>, /notes, /notes_today, /model_help, /forward_help

**Brain side:**
- `~/brain/projects/aiteam/notes/` directory created (README, archive/, images/, .gitkeep)
- `~/brain/projects/aiteam/ideas/weekly_2026-05-23.md` (first idea agent output)
- DEFERRED.md entries for D085, D087, D088, D089
- `.gitignore` updates for image paths

**Cron additions:**
- `0 9 * * 0` — idea-agent run_weekly.sh
- `0 10 * * 0` — notes-agent sort.sh

---

## Next session likely scope

If operator returns to revenue-adjacent work:
- Mark idea_proposals from 2026-05-23 batch (built / deferred / rejected / ignored)
- Review first cron output if Sunday has passed
- Test all three builds from phone

If operator continues infrastructure:
- GEO Phase 3 (writer-revision loop on GEO-failed pages) — still open from prior session
- D045 rewire run-batch.sh from claude -p (Opus) to ai-do.sh (Sonnet) — ~75% cost reduction
- D056 editor production runner — gated on POLICY answers (Q1, Q4, Q7)

If operator wants more capture/organization features:
- Voice notes → notes integration (Whisper already wired for Phase 2 voice handling)
- /find search across all notes/category files
- Note-to-idea_proposals promotion

---

## Quotes worth keeping

Operator: "stop talking so much also. I don't understand half the stuff you say anyways"
→ Triggered tighter responses, less hedging, less pre-flight narration. Pattern held for rest of session.

Operator: "I won't remember to add google search later lets just add it now."
→ Capture-now-tune-later pattern. Self-aware about future-self behavior. Used to justify adding features in current session vs deferring.

Operator: "I just wanna get this thing built"
→ Bias-to-action signal. Used to skip a cross-reference check on research-opportunity-triage that wasn't blocking.

Operator: "do any of the 10 have negative effects?"
→ Forced honest reassessment that materially improved final design.

---

End of context save.
