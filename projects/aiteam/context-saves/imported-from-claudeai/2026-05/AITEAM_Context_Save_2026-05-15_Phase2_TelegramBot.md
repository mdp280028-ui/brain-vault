# AITEAM Context Save — Phase 2 Telegram Bot Mid-Build

**Date:** May 15, 2026 (~4:40 PM Pacific)
**Session duration:** ~4 hours
**Operator:** Stonecreed (Bonners Ferry, ID)
**Companion file:** `AITEAM_Session_Handoff_2026-05-15_Phase2_TelegramBot.md` (comprehensive next-session briefing)

---

## ONE-LINE SUMMARY

Launched Phase 2 Telegram bot build. Phase A (text mode + slash commands + allowlist + /ctx pipeline) signed off ✅. Phase B (Whisper voice transcription) in progress at session end with model downloading.

---

## SESSION ARC

1. **Opened with project file review** — Claude read all project files + recent context saves to get current. Phase 1 closed yesterday; mid-strategic-pivot point.
2. **Phase 2 vs Phase 3 strategic discussion** — operator decided Phase 2 first ("I'm going to need to talk with these agents without being at the computer all the time").
3. **Scope-down on Phase 2** — Claude pushed to defer launchd and approval gates from original Phase 2 spec, build Telegram only first. Operator agreed.
4. **Voice transcription discussion** — operator asked about voice. Three paths presented (voice in/text out, voice in/voice out, real-time). Operator picked Path 1 (voice in, text out) for now.
5. **Three pre-build decisions locked**: Whisper medium model, confirm-first transcription routing, grammy framework.
6. **Two Telegram account use cases clarified**: this build is operator's PERSONAL account talking TO agents. Burner account being MONITORED BY an agent is a SEPARATE build deferred to next.
7. **CC prompt sent for Phase 2 build** — 4-phase structure (A: text bot, B: voice + Whisper, C: launchd, D: destructive verb gate).
8. **Phase A built and signed off** — all 8 sign-off bars verified including /ctx narrative pass test ($0.0472, 26.7s).
9. **Mid-build /ctx scope decision** — operator overrode Claude's recommendation (option c) to option b: Telegram /ctx fires full narrative pass through orchestrator, not just skeleton. Reasoning: phone-first principle defeats "run CC locally" workarounds.
10. **Phase B started** — Whisper medium model download in progress, wrapper code built.
11. **TCC permission issue surfaced + diagnosed** — CC's writes to /Volumes/ActiveSSD/ blocked by EPERM. CC pivoted to ~/whisper/models/ on internal SSD. Root cause: claude binary is the "responsible process" for TCC, Terminal's FDA grant doesn't propagate. Documented as D020, fix deferred.
12. **Comprehensive handoff file produced** for next session.

---

## KEY DECISIONS MADE

### 1. Phase 2 before Phase 3 (LOCKED)

Operator chose phone access (Phase 2) before first-site/revenue work (Phase 3). Reasoning: "I'm going to need to talk with these agents without being at the computer all the time." Mission bar ($200/mo) doesn't change based on this order; the project files frame Phase 3 as where revenue starts, but operator's actual workflow constraint makes Phase 2 the right priority.

### 2. Scope reduction from original Phase 2 spec

Original Phase 2 = Telegram + launchd + approval gates. Reduced this session to **Telegram + launchd only.** Approval gates moved into Publisher agent (SSG Agent 4) where they have a concrete job. Reasoning: building approval-gate UI without a Publisher to gate is theatrical.

### 3. Voice path: voice in, text out (Path 1)

Operator explicitly: "I talk into it then I get text back." Path 2 (voice in/voice out via TTS) is optional polish for later. Path 3 (real-time voice API) killed — not available on Claude stack.

### 4. Whisper medium model

1.5GB, near-human accuracy on English, ~10-15 sec per 30-sec note on M4 Pro. Sweet spot for English at operator's scale. Larger models (large-v3) marginal gains, smaller models lose nuance.

### 5. Confirmation-first transcription flow

Bot replies "I heard: <text>. Reply 'g' to send" before routing to agent. Operator chose this over direct-routing-with-safety-net. Reasoning: trust hasn't been built; better to verify Whisper accuracy before letting it execute commands. Can promote to direct routing later.

### 6. grammy framework (Node.js)

Stays consistent with existing AITEAM dashboard. Friend-bot uses python-telegram-bot but mixing language stacks adds maintenance burden. AITEAM is Node, this bot is Node.

### 7. /ctx implementation — option (b), full narrative pass (LOCKED)

Operator overrode Claude's option (c) recommendation. The phone-first principle: tools that defeat their own purpose by requiring desk access shouldn't ship. $0.05 per /ctx call is the cost; budget cap set at $0.15. Saved as memory `project-phase2-phone-first`.

### 8. Burner account monitoring is a separate build

Phase 2 Telegram bot = personal account → bot.
Burner monitoring agent = userbot API (Telethon/gramjs) reading burner's groups.
Different APIs, different auth (token vs phone+SMS), different code locations, different launchd services. Build sequencing: Phase 2 first → burner monitoring agent next.

### 9. Whisper models on internal SSD

After TCC block on /Volumes/ActiveSSD/, internal SSD (~/whisper/models/) becomes permanent home. 388GB free is more than enough. External SSD reserved for content archives via direct rsync (which doesn't go through CC's TCC context).

---

## FILES CREATED OR MODIFIED THIS SESSION

### On Mac Mini (CC's work)

| File | Status |
|---|---|
| `~/agents/lib/ai-do.sh` | Patched — AGENT_ID_OVERRIDE support |
| `~/agents/lib/log_to_conversation.sh` | New helper |
| `~/agents/telegram/package.json` | grammy + dotenv |
| `~/agents/telegram/.env.example` | Template, committed |
| `~/agents/telegram/.env` | Populated, chmod 600, gitignored |
| `~/agents/telegram/bot.js` | Full Phase A bot |
| `~/agents/telegram/transcribe.sh` | Phase B stub initially, now Phase B wrapper built |
| `~/agents/telegram/logs/` | Empty, ready for Phase C |
| `~/agents/telegram/node_modules/` | 11 packages |
| `.gitignore` | Telegram artifacts excluded |
| `~/whisper/models/medium/` | Downloading at session end |
| `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_1614.md` | New (auto-generated from /ctx test) |

### In project knowledge (this session's artifacts)

| File | Path |
|---|---|
| `AITEAM_Session_Handoff_2026-05-15_Phase2_TelegramBot.md` | /mnt/user-data/outputs/ — comprehensive next-session briefing |
| `AITEAM_Context_Save_2026-05-15_Phase2_TelegramBot.md` | This file |

### Memories saved by CC

| Memory | Purpose |
|---|---|
| `feedback-telegram-bullet-spacing` | Bot output must space bullets for phone readability |
| `project-phase2-phone-first` | Telegram-facing features must work end-to-end from the phone |

---

## NEW LEARNINGS

### About the project

1. **TCC's "responsible process" model can silently block CC subprocesses.** Granting Terminal Full Disk Access does NOT grant claude binary the same access. The claude binary is the responsible process for TCC purposes. Symptoms: writes return EPERM (errno 1) instead of EACCES (errno 13). Diagnosis pattern: if Unix permissions look correct AND the volume isn't read-only AND it's not SIP/sandbox, it's TCC.

2. **/ctx from Telegram via narrative pass is genuinely useful.** First production test ($0.0472, 26.7s) produced an emergent insight — orchestrator noticed the rejected burner user pattern and surfaced "log repeated unauthorized users for pattern review" without being asked. This validates the pipeline design.

3. **Phase 2 vs Phase 3 ordering was correctly contested.** Operator's pivot to Phase 2 first wasn't research-before-building; it was reading actual workflow constraints. Sometimes the operator knows their context better than the project plan.

4. **The 8-agent SSG chassis is shared with asbestos.** Same agents, different rubric/voice/config per site. This is the "AI holding company" architecture made concrete.

### About operator interaction patterns

1. **Operator delegates design decisions confidently** when given a clear recommendation with reasoning. The pattern: Claude presents 2-3 options with a clear pick + reasoning → operator picks the recommendation OR overrides crisply with their own reasoning.

2. **"explain like I'm 14" requests when an explanation isn't landing.** Used twice this session (Phase 2 explanation, what does GSC Monitor do). Always works to reset.

3. **Operator pushes back when Claude's recommendation conflicts with workflow reality.** /ctx option (c) was Claude's recommendation; operator overrode to (b) because phone-first matters more than cost. Take corrections at face value.

4. **CC corrections are reliable when they conflict with Claude.ai's spec.** SSD path correction, AGENT_ID_OVERRIDE injection point, slash command attribution miss — all CC catches that Claude.ai got wrong because CC sees actual environment.

### About the build process

1. **Sign-off discipline catches real issues.** Phase A's first sign-off attempt would have shipped with slash command attribution missing (item 7 of spec). CC flagged honestly instead of declaring ✅. The fix was 2 lines.

2. **Honest ⚠ beats premature ✅.** CC's pattern of flagging gaps before declaring sign-off has caught multiple real issues across recent sessions. This rule is the project's most valuable discipline.

3. **Background downloads with parallel work is the right pattern for slow dependencies.** Whisper model download (~15-30 min) ran in background while CC scaffolded the wrapper. Zero idle time. Worth replicating for future long-running setup tasks.

---

## DEFERRED ITEMS — NEW THIS SESSION

| ID | Item | Why deferred | Trigger |
|---|---|---|---|
| D020 | TCC Removable Volumes permission for claude binary | Three fixes documented; internal SSD has 388GB free | If external SSD writes from CC become genuinely needed |
| D021 | Project Instructions stale on /Volumes/AgentSSD/ | Actual mount is /Volumes/ActiveSSD/ | Next instructions update |
| D022 | Telegram group monitoring agent (burner account) | Operator's next priority after Phase 2 | After Phase 2 closes |
| D023 | Friend-bot launchd integration | Wait until Phase 2 launchd pattern proven | After Phase 2 launchd verified |
| D024 | Project Instructions update reflecting Phase 2 work | Instructions don't reflect current state | Next instructions update |

Combined with D001-D019 from prior sessions, deferred items now total 24.

---

## WHAT'S LEFT MID-TASK

**Phase 2 Telegram bot — Phase B in progress.**

- Whisper medium model downloading (~halfway through at session end, ~80 MB/min over rural ADSL)
- Wrapper script (transcribe.sh) built
- Voice handler + confirmation flow code written in bot.js
- Smoke test pending (CC will run synthetic test once download completes)
- Operator will then send 3 live voice notes to verify Phase B end-to-end

**Three voice test set pre-planned for next session:**
1. Short command: "what's my total spend today"
2. Longer mixed-content: "look at the audit log from the last hour and tell me if there are any rejected unauthorized rows from telegram"
3. Stress test (technical terms + numbers): "scout agent for SmartSourceGuide, find keywords with cost-per-click above twenty dollars and search volume between three hundred and three thousand monthly, prioritize answering services niche"

**Phase 2 sign-off bars remaining:**
- Phase B: voice transcribes correctly, confirmation flow works, no leftover .ogg files, latency <20s
- Phase C: launchd auto-restart on kill, survives Mac reboot, log files functional
- Phase D: destructive verbs trigger confirmation, audit trail captures the events
- Final: audit row `phase_2_telegram_complete`

**Current run state:**
- Telegram bot running in foreground (PID 40747)
- Terminal window must stay open until Phase C launchd ships
- Operator can close laptop but Mac Mini terminal stays running

---

## STRATEGIC POSITION

### Where we are vs the mission

- **Mission bar:** $200/mo revenue
- **Current revenue:** $0
- **Path to first dollar:**
  - Phase 2 closes (4-7 hrs CC time remaining + model download)
  - Telegram group monitoring agent builds (~12-18 hrs CC time)
  - SSG agent fleet builds (~25-35 hrs CC time)
  - SSG content production starts
  - 2-4 month indexing/ranking clock
  - Affiliate signups happen as traffic appears
  - First dollar: month 3-6 realistic

### Next session priorities (in order)

1. Finish Phase 2 (Phases B, C, D)
2. Build Telegram group monitoring agent (operator's explicit next priority)
3. Start SSG agent fleet (Writer first)

### Open strategic questions still

1. Approval gate UI for Publisher (terminal/file/dashboard/Telegram) — decide before SSG Agent 4
2. Build location for SSG agents (`~/agents/ssg/` vs separate tree)
3. Friend-bot path consolidation (`~/projects/friend-bot/` vs `~/agents/friend-bot/`)

---

## OPERATOR CORRECTIONS APPLIED THIS SESSION

1. **Affiliate enrollment is NOT the critical path.** Claude framed affiliate enrollment as the bottleneck blocking SSG agent work. Operator pushed back: SSG has no traffic yet, signing up for affiliates now doesn't change month-3-6 revenue. Defer affiliate work; agents should produce content. Correction applied: revised build plan to skip Agent 7 (Publisher with approval gate) in original SSG sequence, then operator reversed again — keep Publisher in the sequence because articles need to ship live to start indexing clock. Both reversals were operator integrating reality faster than Claude.

2. **The Telegram bot we're building isn't the burner-monitoring agent.** Operator caught this distinction proactively. Claude clarified: bot API (Phase 2) vs userbot API (next build). Different stacks, different auth.

3. **Token storage doesn't need a password manager.** Claude suggested keychain/1Password. Operator pointed out: BotFather keeps the token accessible anytime via `/mybots` → API Token. No separate storage needed. Correction applied.

4. **/ctx must work from phone fully.** Claude recommended option (c) — half-baked /ctx from phone, full /ctx from desk. Operator overrode: phone-first means phone-complete. Saved as memory.

5. **Terminal HAS Full Disk Access, contrary to CC's assumption.** Operator caught Claude's diagnosis of TCC block — sent screenshot showing Terminal toggled on for FDA. CC re-diagnosed correctly: the claude binary is the TCC responsible process, not Terminal. Apology + correct diagnosis followed.

---

## WHAT THE NEXT CHAT SHOULD DO FIRST

1. **Read this context save + the comprehensive handoff file.**
2. **Check Phase B status.** Did the model download complete? Did smoke test pass? Are 3 voice notes tested?
3. **If Phase B pending:** continue with smoke test + 3 voice notes + sign-off.
4. **If Phase B done:** proceed to Phase C (launchd) then Phase D (destructive gate).
5. **After Phase 2 closes:** start Telegram group monitoring agent build per D022 and the spec in the handoff file.

---

## SESSION-END STATE

- **Phase 2 progress:** ~30% (Phase A ✅, B mid-build)
- **Phase 2 remaining:** ~4-7 hrs CC time + model download wildcard
- **Active build threads:** 1 (Phase B Whisper voice transcription)
- **Cost this session:** ~$0.10 across Phase A testing
- **Cumulative Phase 1+2 spend:** ~$1.29 (Max-sub value, not API dollars)
- **Telegram bot live:** YES (foreground, PID 40747)
- **Burner account verified blocked:** YES (audit #60, #61)
- **/ctx from Telegram producing real narrative saves:** YES (test ran at $0.0472)

---

*End of context save. Drop in project knowledge before next session.*
