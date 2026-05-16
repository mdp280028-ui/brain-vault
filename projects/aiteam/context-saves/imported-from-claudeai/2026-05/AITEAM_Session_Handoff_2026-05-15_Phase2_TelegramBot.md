# AITEAM Session Handoff — Phase 2 Telegram Bot Build (In Progress)

**Created:** May 15, 2026 (~4:35 PM Pacific)
**Operator:** Stonecreed (Bonners Ferry, ID)
**Project:** AITEAM (separate from PayrollInsider/UST/SMART)
**Previous handoffs in chronological order:**
- `AITEAM_Session_Handoff_2026-05-14_Step14-Orchestrator.md`
- `AITEAM_Context_Save_2026-05-14_StepsThroughEditor.md`
- `AITEAM_Context_Save_2026-05-14_Phase1_Complete.md`
- `AITEAM_Context_Save_2026-05-15_SitePortfolio-SSGAgentSpec.md`
- `AITEAM_Context_Save_2026-05-15_CtxSystem-BrainGitInit.md`
- `AITEAM_Context_Save_2026-05-15_FriendBot-Recovery.md`

---

## ONE-PARAGRAPH SUMMARY

Phase 1 chassis complete (orchestrator + librarian + editor agents, dashboard, cost tracking, war room with discuss mode, failure modes documented, ctx system, brain git-init). Phase 2 Telegram bot is mid-build. Phase A (text mode + slash commands + allowlist) signed off ✅ — bot responds to operator's Telegram messages, rejects burner account silently, slash commands (/morning_brief, /weekly_review, /ctx) all working with cost attribution to telegram_bot_<label>. Phase B (Whisper voice transcription with confirmation flow) in progress — model downloading at session end. Phases C (launchd) and D (destructive verb gate) remaining. After Phase 2 closes, operator wants to build Telegram group monitoring agent next (separate from this bot, monitors burner account's groups via userbot API).

---

## WHERE PHASE 2 IS RIGHT NOW

### Phase 2 progress: ~30%

| Phase | Status | Weight | Notes |
|---|---|---|---|
| A — Text bot + slash commands + allowlist | ✅ DONE | 25% | All 8 sign-off bars verified |
| B — Whisper voice transcription + confirmation flow | IN PROGRESS | 45% | Model downloading, wrapper built |
| C — launchd auto-restart service | PENDING | 15% | ~1-1.5 hrs CC time |
| D — Destructive verb safety gate | PENDING | 15% | ~30-60 min CC time |

**Total Phase 2 remaining at session end: ~4-7 hrs CC time + model download wildcard.**

### What's currently happening when this handoff was written

CC is running Phase B. Whisper medium model (1.5GB) downloading at ~80 MB/min over rural ADSL. Wrapper script (`transcribe.sh`) built. Voice handler + confirmation flow code written. Model lives at `~/whisper/models/medium/` (internal SSD, 388 GB free) instead of `/Volumes/ActiveSSD/` due to TCC permission block on CC's writes to external volume (full diagnosis in deferred items section below).

**Next operator action expected:** CC pings when model download completes and smoke test runs. Operator then sends 3 test voice notes to verify Phase B end-to-end.

---

## PHASE A — WHAT GOT BUILT

### Files created/modified on Mac Mini

| File | Purpose |
|---|---|
| `~/agents/lib/ai-do.sh` | Patched — respects `AGENT_ID_OVERRIDE` env var for cost attribution |
| `~/agents/lib/log_to_conversation.sh` | New helper, self-tested |
| `~/agents/telegram/package.json` | grammy ^1.30 + dotenv ^16.4 |
| `~/agents/telegram/.env.example` | Template (committed to git) |
| `~/agents/telegram/.env` | Populated with BOT_TOKEN + ALLOWLIST_USER_IDS (chmod 600, gitignored) |
| `~/agents/telegram/bot.js` | Full Phase A bot, syntax-checked |
| `~/agents/telegram/transcribe.sh` | Phase B stub (exits 99 if accidentally invoked during Phase A) |
| `~/agents/telegram/logs/` | Empty, ready for launchd in Phase C |
| `~/agents/telegram/node_modules/` | 11 packages, clean install on node 26 |
| `.gitignore` | Added telegram/.env, telegram/logs/, telegram/package-lock.json |

### Phase A capabilities (live now)

- grammy long polling (no webhooks — fits rural internet)
- Allowlist middleware (first in chain, silent drop on rejection, logs to audit_log as `rejected_unauthorized`)
- Fails closed if BOT_TOKEN or ALLOWLIST_USER_IDS empty
- Per-chat serial queue (rapid messages process in order)
- Slash commands: `/start`, `/help`, `/morning_brief`, `/weekly_review`, `/ctx`
- Text router: free-text → orchestrator (Sonnet) via `run_agent.sh`, agent_id tagged `telegram_bot_orchestrator`
- Voice handler stub: replies "Phase B, not yet enabled" until WHISPER_MODEL_PATH populated
- Photo/doc/sticker catch-all: "only text and voice are supported"
- Telegram 4096-char cap handled via `sendChunked()`
- Graceful SIGINT/SIGTERM shutdown with shutdown audit row
- Every inbound + outbound message logged to `conversation_log` with shared `source_turn_id`
- `bot.catch()` global error trap

### Phase A sign-off verification (all ✅)

| Bar | Evidence |
|---|---|
| Text response from operator's ID | conv #37→#38, audit #57→#58 |
| Rejection of burner account | audit #60, #61 — silent drop verified |
| /morning_brief from Telegram | audit #59, #64, #69 |
| audit_log row for first message | #56–#61, six rows |
| token_usage row for first response | #52, #53, then #56–#61 |
| Fix #1 slash-command attribution | #56, #57, #60, #61 tagged `telegram_bot_morning_brief` |
| /ctx end-to-end with narrative | audit #65→#66, 26.7s, $0.0472 (under $0.15 budget) |
| Bullet spacing for phone readability | operator-confirmed |

### Strategic decision made mid-build: /ctx implementation

Original spec was option (c) — Telegram `/ctx` runs `ctx.sh` skeleton, replies "run CC locally to fill narrative." **Operator overrode to option (b)** — Telegram `/ctx` fires `ctx.sh` AND pipes through orchestrator (Sonnet) to fill narrative sections, replies with summary + cost.

Reasoning: the whole point of Phase 2 is phone-first access. A `/ctx` requiring desk access defeats the purpose. Saved as memory `project-phase2-phone-first`.

Cost per /ctx call: ~$0.05 average, budget cap $0.15. Acceptable.

### Audit log emergent insight

After Phase A close, the most recent `/ctx` from Telegram produced `AITEAM_Context_Save_2026-05-15_1614.md` — orchestrator picked up the rejected burner user pattern from audit_log and surfaced "log repeated unauthorized users for pattern review" as a forward-looking action. Exactly the kind of emergent observation the pipeline was designed to produce.

---

## CURRENT PROJECT STATE (END OF SESSION)

### Phase progress overall

| Phase | Status | Description |
|---|---|---|
| Phase 0 — Chassis (13 steps) | ✅ COMPLETE | Substrate locked May 14 |
| Phase 1 — Agent foundation (13 steps) | ✅ COMPLETE | Closed May 14, 11/13 with 2 deferred (10b, 17a) |
| Phase 2 — Telegram bot + voice + launchd + destructive gate | 30% | Phase A ✅, B in progress |
| Phase 3 — First worker agents → first site (SSG) | NOT STARTED | 7-8 agents specced, none built |
| Phase 4 — Revenue + scaling | $0 | No sites earning |

### Active agents (3)

| Agent | Tier | Role |
|---|---|---|
| orchestrator | Sonnet | Routes tasks, war-room standups, slash commands, diary, now Telegram entry point |
| librarian | Haiku | Nightly inbox sort |
| editor | Sonnet (locked at r=0.933) | 5-axis content scoring |

### Infrastructure live

- Dashboard at :3141 (cost chart + war room + audit + diary + agents)
- Token usage logging through all wrappers
- Brain auto-commit cron scheduled (23:55 nightly, private GitHub repo `mdp280028-ui/brain-vault`)
- ctx system (`/ctx` slash command for CC, also `/ctx` over Telegram now)
- HANDOFF.md, LESSONS.md, context-saves all written to `~/brain/projects/aiteam/`
- Telegram bot running in foreground (PID 40747 as of session end — will need launchd in Phase C)

### Cost accumulator

- Phase 1 total spend: ~$1.19
- Phase A spend: ~$0.10 across testing
- Phase B: pending
- All numbers are Claude Max subscription value consumed, NOT real API dollars

### Database state (`~/store/aiteam.db`)

- 10 tables, populated except `mission_tasks` (0 rows) and `memories`/`fts_memory` (RAG stubbed)
- `audit_log`: 69+ rows
- `token_usage`: real attribution including `telegram_bot_*` agent_ids now
- `warroom_transcript`: standup + slash + discuss rows with `round_number` column
- `conversation_log`: includes Telegram message pairs with `source_turn_id` for join

---

## STRATEGIC ROADMAP FROM HERE

### Path from here to first revenue

1. **Finish Phase 2 (this session and next)** — ~4-7 hrs CC time remaining
2. **Build Telegram group monitoring agent** — operator's stated next priority (12-18 hrs CC time)
3. **Build SSG agent fleet** — 7-8 agents, ~25-35 hrs CC time (Keyword Screener deferred until article #20+)
4. **Ship articles to SSG, let indexing clock run** — 2-4 months for meaningful ranking
5. **Sign up for affiliates as traffic appears** — operator's task, low priority until clicks start
6. **First dollar earned** — month 3-6 realistic estimate

### Why this order

Operator explicitly chose this sequence:
- "I'm going to need to talk with these agents without being at the computer all the time" → Phase 2 first
- After Phase 2: "make the agent that monitors my tg groups so it can start working for me"
- After both Telegram pieces: SSG fleet build (affiliate research happens in parallel, signup deferred)

### Site portfolio context

- 12 candidate categories → top 7 active focus → SSG locked as first AITEAM workload
- SSG: ~17 pages live, 0 indexed yet, $0 revenue, 0 affiliates enrolled
- Same 8-agent chassis serves SSG AND the asbestos site (separate Stonecreed property, also in build)
- Asbestos needs YMYL caution layer + stricter citation rules but same agents with different rubric configs

### SSG agent build order (when ready, post Phase 2 + Telegram monitoring)

| # | Agent | Tier | Status |
|---|---|---|---|
| 1 | Writer | Sonnet | Specced, not built |
| 2 | Editor (extend existing with SSG rubric) | Sonnet | Existing agent, needs SSG rubric |
| 3 | Mechanical Auditor | Haiku | Specced, not built |
| 4 | Publisher (with dashboard approval gate) | Haiku | Specced, not built |
| 5 | Research Agent | Sonnet | Specced, not built |
| 6 | Internal Link Manager | Haiku | Specced, not built |
| 7 | GSC Monitor | Haiku | Specced, not built |
| 8 | Keyword Screener | Haiku | Deferred until article #20+ exhausts roadmap |

---

## NEXT MAJOR BUILD: TELEGRAM GROUP MONITORING AGENT

After Phase 2 closes, this is operator's next priority. Specced loosely; will need a dedicated build session.

### What it does

Operator has a burner Telegram account in ~10 groups. Wants an agent to monitor those groups for signal/intel — keywords, mentions, opportunities, patterns. Read-only; no posting from burner account.

### Tech stack (separate from Phase 2 bot)

- **NOT** the bot API. Bots can't read groups they're not added to as admins.
- **MUST** use Telegram's userbot API (MTProto protocol) via `Telethon` (Python) or `gramjs` (Node.js).
- Auth: phone number + SMS verification one time (operator's burner phone needed during setup).
- Session file stored after first auth.
- Read messages from groups the burner account is already a member of.

### Key risks to flag

- Telegram has been cracking down on userbots used for scraping. Account ban risk exists.
- Mitigations: read-only (don't react/seen-mark aggressively), rate-limit reads, don't post via the agent.
- Burner is expendable by definition, but worth knowing the risk.

### Build estimate

- ~12-18 hrs CC time, 2-3 weeks calendar time
- Components: Telethon/gramjs install + auth, group reader loop, SQLite schema for messages, keyword/signal extraction, LLM analysis layer (Haiku triage → Sonnet for high-signal items), alert routing
- Operator-side time: deciding which groups to monitor and what signal patterns to extract

### Important separation

- Phase 2 Telegram bot = operator's PERSONAL account talking TO agents (this build)
- Group monitoring agent = operator's BURNER account being MONITORED BY an agent (next build)
- Different APIs (Bot API vs User API/MTProto)
- Different auth models (token vs phone+SMS)
- Different code locations (`~/agents/telegram/` vs `~/agents/tg-monitor/`)
- Different launchd services

---

## DEFERRED ITEMS — UPDATED LIST

Combining all prior deferred items plus new ones from this session.

| ID | Item | Why deferred | Trigger |
|---|---|---|---|
| D001 | Editor cascade pattern (Haiku triage → Sonnet final) | Production volume not high enough | ≥100 articles/week |
| D002 | Kanban view (step 10b) | mission_tasks empty | When agent emits mission_tasks rows |
| D003 | Editor tier re-validation with operator-typed manual scores | Quarterly check or Sonnet version bump | Quarterly OR new Sonnet release |
| D004 | HANDOFF.md on Mac Mini stale wording | Project file artifact | Next CC session — likely auto-resolved by ongoing /ctx runs |
| D005 | Affiliate enrollment for SSG (4-6 programs) | Defer until traffic appears | When SSG GSC shows organic clicks |
| D006 | Voices seed list (5+ names) | Not blocking | When voices ingestion next on plan |
| D007 | Persona project | Operator: "soonish, depends" | When operator raises |
| D008 | launchd strategy beyond Telegram bot | Address per-agent | When agent becomes long-running |
| D009 | Backup verification cadence | First restore hasn't happened | First real recovery scenario or 90 days |
| D010 | Step 17a silver platters | No data sources to summarize | First site live + 30 days GA4 |
| D011 | UST tier intent re-labeling | Operator hasn't spot-checked | Optional, low priority |
| D012 | gwern proto_essays.md corpus swap | Was index page not essay | If running another corpus test |
| D013 | Budget enforcement hook (DAILY_API_BUDGET_USD) | Theatrical on Max sub | When project moves to direct API |
| D014 | Detection cron infrastructure | Phase 2 dependency | After launchd established |
| D015 | Per-agent switch bypasses | Defense-in-depth | Phase 2 polish |
| D016 | PUBLISHER_DEPLOY_ENABLED enforcement | No publisher built yet | When publisher agent ships |
| D017 | rubric.md change-detection / version pinning | Silent edits recoverable only from backup | When editor goes into production |
| D018 | Project Instructions v2 reflect Max-sub vs API spend | Current instructions imply real dollars | Next instructions update |
| D019 | Phase 2 vs Phase 3 ordering | Now resolved — Phase 2 first | RESOLVED |
| **D020** | **TCC Removable Volumes permission for claude binary** | CC writes to /Volumes/ActiveSSD/ blocked by EPERM. Terminal has FDA but claude binary is the responsible process for TCC purposes. Three fixes documented (Files+Folders permission, tccutil reset, or FDA for claude binary). Not blocking — internal SSD has 388GB free. | If an agent's workflow specifically requires CC writing to /Volumes/ — likely never given current architecture |
| **D021** | **Project Instructions stale on /Volumes/AgentSSD/** | Actual mount is /Volumes/ActiveSSD/ per live `df` | Next project instructions update |
| **D022** | **Telegram group monitoring agent (burner account)** | Operator's next priority after Phase 2 | After Phase 2 closes (Phases B, C, D complete) |
| **D023** | **Friend-bot launchd integration** | Friend-bot at `~/projects/friend-bot/` died May 15 when working dir moved. Phase 2 Telegram bot's launchd config will demonstrate the pattern. Friend-bot could be added to same launchd setup at minimal effort. | After Phase 2 Telegram bot launchd is verified working |
| **D024** | **Project Instructions update with Phase 2 in progress + Telegram bot live** | Current instructions don't reflect Phase 2 work | Next project instructions update |

---

## WHAT THE NEXT CHAT SHOULD DO FIRST

### If next session is "continue Phase 2 build"

1. **Verify Phase B status.** CC should have surfaced Whisper smoke test results. Was it accurate? Was latency under 20s per 30-sec note?
2. **Run 3 voice notes if not already done.** Pre-planned test set:
   - Short command: "what's my total spend today"
   - Longer mixed-content: "look at the audit log from the last hour and tell me if there are any rejected unauthorized rows from telegram"
   - Stress test: "scout agent for SmartSourceGuide, find keywords with cost-per-click above twenty dollars and search volume between three hundred and three thousand monthly, prioritize answering services niche"
3. **Sign off Phase B.** Bars: voice transcribes correctly, confirmation flow works, no leftover .ogg files, latency under 20s.
4. **Move to Phase C** (launchd auto-restart). ~1-1.5 hrs CC time. Operator-verified by killing bot process and rebooting Mac Mini.
5. **Move to Phase D** (destructive verb gate). ~30-60 min.
6. **Phase 2 sign-off.** Audit row `phase_2_telegram_complete` with payload listing all 4 component statuses.

### If next session is "start group monitoring agent build"

Operator stated this is next priority after Phase 2. See "NEXT MAJOR BUILD: TELEGRAM GROUP MONITORING AGENT" section above for the spec.

First steps:
1. Confirm operator's burner phone is accessible during setup
2. Decide Telethon (Python) vs gramjs (Node.js) — Telethon has more mature features, but mixing Python with the existing Node-based dashboard adds complexity. Recommendation: Telethon — better library, easier monitoring patterns, separate from main bot stack anyway.
3. Get list of 10 groups operator wants to monitor (operator-provided)
4. Decide signal extraction patterns (keywords? sentiment? mentions of specific terms?)
5. Decide alert routing (alerts go to which channel — main Telegram bot? Email? Dashboard?)

### If next session is general strategy

- Don't drift back into research mode. Operator's research-before-building risk is locked in project instructions.
- Phase 2 is in flight. The momentum is real. Closing Phase 2 is the immediate goal.
- Operator's strategic clarity is strong; execution is what matters.

---

## CRITICAL CONTEXT FOR NEXT CLAUDE

### About the operator

- **Stonecreed** — solo, Bonners Ferry ID, rural northern Idaho
- No coding background — relies on CC to write all code
- Multitasks across many windows; keep responses dense and visible
- "g" at end of message = ultra-short reply requested (6-sentence ceiling, no preamble)
- Trail-off characters at message end ("g", "fg") are not meaningful — ignore
- Operator pushes back hard ("I don't fucking care") — take corrections at face value, integrate, move on
- "Explain like I'm 14" requests when an explanation isn't landing
- Operator's research-before-building pattern: locked as primary execution risk in project instructions. Watch for it.

### About this project

- Mission bar: $200/mo revenue to break even on Claude Max subscription
- $0 current revenue; first dollar realistic at month 3-6 from SSG
- 100% on Claude Max subscription — dashboard "spend" numbers are value consumed, not real dollars
- Editor tier locked at Sonnet 4.6 (r=0.933 vs operator/Opus baseline)
- Build plan is in `AITEAM_Build_Plan_2026-05-14_FINAL.md` (operator uploaded this session)

### About sign-off discipline

- ✅ requires the spec being met end-to-end, not "process started" or "file exists"
- For agents: ✅ requires the agent ran end-to-end at least once and produced human-readable output
- Honest ⚠ beats premature ✅
- This rule has saved the project from multiple silent failures (Opus-not-Sonnet bug, premature 10a sign-off, slash command attribution miss)

### About CC corrections

CC's corrections have been right multiple times in recent sessions:
- May 14: run_agent.sh fix syntax (`$2` not `export`)
- May 14: failure_modes file didn't actually exist
- May 15: SSD path is `/Volumes/ActiveSSD/`, not `/Volumes/AgentSSD/`
- May 15: AGENT_ID_OVERRIDE injection point for slash commands

**Trust CC's environment knowledge over Claude.ai's spec when they conflict.** CC sees the actual filesystem.

### About the chassis being shared

The agent chassis is built once. SSG and asbestos site both use the same 7-8 agents with different rubric/voice/config files. This is the "passive income holding company" architecture made real — build once, deploy many sites.

---

## OPEN STRATEGIC QUESTIONS

1. **Approval gate UI for Publisher** — terminal, file flag, dashboard button, or Telegram message? Recommendation in project files is dashboard button. Operator hasn't locked it. Decide before building Agent 4 (Publisher) in SSG fleet.
2. **Build location for SSG agents** — `~/agents/ssg/` subfolder vs separate `~/agents-ssg/` tree? Not decided.
3. **Friend-bot at `~/projects/friend-bot/`** — should it migrate to `~/agents/friend-bot/` for consistency? Or stay where it is? D023 covers integration into launchd; doesn't address path consolidation.
4. **Niche decision for non-SSG AITEAM work** (HR vs Real Estate vs Accountants) — locked by SSG having its own niches. Reopens only if AITEAM agents work beyond SSG.

---

## FILES UPDATED THIS SESSION

| File | Status |
|---|---|
| `~/agents/lib/ai-do.sh` | Patched (AGENT_ID_OVERRIDE support) |
| `~/agents/lib/log_to_conversation.sh` | New |
| `~/agents/telegram/` (full directory) | New, Phase A built and Phase B in progress |
| `.gitignore` | Updated |
| `~/whisper/models/medium/` | Downloading (1.5GB) |
| Memory: `feedback-telegram-bullet-spacing` | New |
| Memory: `project-phase2-phone-first` | New |
| Multiple audit_log rows | #56 through #69+ |
| `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_1614.md` | New (from /ctx test) |

---

## SESSION-SPECIFIC FLAGS

### Watch for: Stale "first site is open" framing

Several project files still reference "first site target — open decision" or "leaning PLR/Etsy/Gumroad." **SSG is locked as the first AITEAM workload** per May 15 portfolio session. Don't re-derive the niche decision; the operator owns SSG and the 8-agent build is for SSG.

### Watch for: Mac-sub vs API cost confusion

Dashboard numbers are Claude Max subscription value consumed. NOT real API dollars. Some project files imply real spend ("$X/month API costs"). Re-frame as "subscription burn rate" when relevant.

### Watch for: Friend-bot path

Friend-bot lives at `~/projects/friend-bot/`, NOT `~/agents/friend-bot/`. Earlier handoffs had the path wrong; May 15 friend-bot recovery session corrected it. Trust the most recent save.

### Watch for: Telegram bot is in foreground

When this handoff was written, the Phase 2 Telegram bot was running in foreground (PID 40747). It will die if the terminal closes. Phase C (launchd) is what makes it survive crashes and reboots. Until Phase C ships, operator should not close that terminal window.

---

*End of session handoff. Operator should drop this file into project knowledge before next session starts.*
