# AITEAM Context Save — 2026-05-15_1813
**Generated:** 2026-05-15T18:13:23-0700
**Since last save:** 2026-05-15 16:14:29
**Session topic:** Phase 2 Telegram stack — text bot, voice transcription, /ctx pipeline, and destructive-verb gate built and signed off in one sitting; launchd auto-restart deferred. `phase_2_telegram_complete` audit row #121 fired at 18:00:22.

---

## Mechanical record

### Git activity since last save
```
(no commits since last save)
```

### Files changed
```
(no file changes)
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
| id  |         ts          | actor_id |             action             |     target      |
|-----|---------------------|----------|--------------------------------|-----------------|
| 125 | 2026-05-15 18:12:13 | bot      | text_responded                 | chat:7572496076 |
| 124 | 2026-05-15 18:11:35 | bot      | text_received                  | chat:7572496076 |
| 123 | 2026-05-15 18:00:32 | bot      | startup                        | bot             |
| 122 | 2026-05-15 18:00:31 | bot      | shutdown                       | bot             |
| 121 | 2026-05-15 18:00:22 | claude   | phase_2_telegram_complete      | aiteam_phase_2  |
| 120 | 2026-05-15 17:59:36 | bot      | text_responded                 | chat:7572496076 |
| 119 | 2026-05-15 17:59:28 | bot      | text_received                  | chat:7572496076 |
| 118 | 2026-05-15 17:58:09 | bot      | text_responded                 | chat:7572496076 |
| 117 | 2026-05-15 17:57:35 | bot      | destructive_verb_confirmed     | chat:7572496076 |
| 116 | 2026-05-15 17:57:15 | bot      | destructive_verb_detected      | chat:7572496076 |
| 115 | 2026-05-15 17:57:15 | bot      | text_received                  | chat:7572496076 |
| 114 | 2026-05-15 17:47:13 | bot      | destructive_verb_stray_confirm | chat:7572496076 |
| 113 | 2026-05-15 17:47:02 | bot      | destructive_verb_stray_confirm | chat:7572496076 |
| 112 | 2026-05-15 17:45:56 | bot      | startup                        | bot             |
| 111 | 2026-05-15 17:45:55 | bot      | shutdown                       | bot             |
| 110 | 2026-05-15 17:45:21 | bot      | stray_confirm                  | chat:7572496076 |
| 109 | 2026-05-15 17:42:06 | bot      | text_responded                 | chat:7572496076 |
| 108 | 2026-05-15 17:41:54 | bot      | text_received                  | chat:7572496076 |
| 107 | 2026-05-15 17:41:22 | bot      | destructive_verb_cancelled     | chat:7572496076 |
| 106 | 2026-05-15 17:40:54 | bot      | destructive_verb_detected      | chat:7572496076 |
| 105 | 2026-05-15 17:40:54 | bot      | text_received                  | chat:7572496076 |
| 104 | 2026-05-15 17:40:33 | bot      | destructive_verb_timeout       | chat:7572496076 |
| 103 | 2026-05-15 17:40:03 | bot      | destructive_verb_detected      | chat:7572496076 |
| 102 | 2026-05-15 17:40:02 | bot      | text_received                  | chat:7572496076 |
| 101 | 2026-05-15 17:32:54 | bot      | destructive_verb_timeout       | chat:7572496076 |
| 100 | 2026-05-15 17:32:24 | bot      | destructive_verb_detected      | chat:7572496076 |
| 99  | 2026-05-15 17:32:23 | bot      | voice_corrected                | chat:7572496076 |
| 98  | 2026-05-15 17:31:23 | bot      | voice_transcribed              | chat:7572496076 |
| 97  | 2026-05-15 17:31:15 | bot      | voice_received                 | chat:7572496076 |
| 96  | 2026-05-15 17:31:03 | bot      | voice_routed                   | chat:7572496076 |
| 95  | 2026-05-15 17:30:49 | bot      | voice_confirmed                | chat:7572496076 |
| 94  | 2026-05-15 17:30:45 | bot      | voice_transcribed              | chat:7572496076 |
| 93  | 2026-05-15 17:30:36 | bot      | voice_received                 | chat:7572496076 |
| 92  | 2026-05-15 17:24:45 | bot      | startup                        | bot             |
| 91  | 2026-05-15 17:24:44 | bot      | shutdown                       | bot             |
| 90  | 2026-05-15 17:22:40 | bot      | startup                        | bot             |
| 89  | 2026-05-15 17:22:39 | bot      | shutdown                       | bot             |
| 88  | 2026-05-15 17:10:42 | bot      | voice_timeout                  | chat:7572496076 |
| 87  | 2026-05-15 17:00:41 | bot      | voice_transcribed              | chat:7572496076 |
| 86  | 2026-05-15 17:00:25 | bot      | voice_received                 | chat:7572496076 |
| 85  | 2026-05-15 16:59:52 | bot      | voice_routed                   | chat:7572496076 |
| 84  | 2026-05-15 16:58:40 | bot      | voice_corrected                | chat:7572496076 |
| 83  | 2026-05-15 16:57:40 | bot      | voice_transcribed              | chat:7572496076 |
| 82  | 2026-05-15 16:57:29 | bot      | voice_received                 | chat:7572496076 |
| 81  | 2026-05-15 16:55:03 | bot      | voice_routed                   | chat:7572496076 |
| 80  | 2026-05-15 16:54:31 | bot      | voice_confirmed                | chat:7572496076 |
| 79  | 2026-05-15 16:54:08 | bot      | voice_transcribed              | chat:7572496076 |
| 78  | 2026-05-15 16:54:00 | bot      | voice_received                 | chat:7572496076 |
| 77  | 2026-05-15 16:53:33 | bot      | startup                        | bot             |
| 76  | 2026-05-15 16:53:31 | bot      | shutdown                       | bot             |

### Token usage
|           model           | in_tok | out_tok | cost_usd |
|---------------------------|--------|---------|----------|
| claude-sonnet-4-6         | 63     | 6162    | 0.6373   |
| claude-haiku-4-5-20251001 | 4480   | 69      | 0.0048   |

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

- **Phase 2 split as A/B/D shipped, C deferred.** Telegram text bot (A), Whisper voice with confirm-flow (B), destructive-verb gate (D) all complete. launchd auto-restart (C) explicitly deferred as D026 — operator's reasoning: "computer hardly ever shuts off." Real risk (silent bot death undetected) named in DEFERRED.md with un-defer triggers.
- **/ctx promoted from desk-only to phone-complete (option b, not c).** Telegram /ctx now runs `ctx.sh` mechanical skeleton AND pipes through orchestrator (Sonnet, $0.15 budget cap, 120s timeout) for narrative fill, splices into save file, replies with summary + cost. Alternative (c) — "skeleton written, run CC locally to finish" — was rejected mid-build: "the entire point of Phase 2 is reaching the system from the phone."
- **Models on internal SSD permanently (`~/whisper/`), external SSD reserved for rsync archives.** Pivot forced initially by macOS TCC denial on `/Volumes/ActiveSSD/` for the `claude` CLI process; made permanent after diagnosis showed external SSD's real role is bulk content archives that don't transit CC's process context. 388 GB free on internal is plenty.
- **Destructive-gate detection is pure verb scan, no resource cross-check.** 14-verb list (`delete remove kill stop pause shutdown drop truncate force override disable wipe revoke reset`) in hot-reloaded `destructive_verbs.txt`. False positives accepted explicitly as the cost of certainty. Rejected verbs (`cancel abort clear terminate`) on alert-fatigue grounds.
- **Single-chokepoint architecture for the destructive gate.** `routeAndLog(chatId, ctx, text, turnId, source)` is the only path that talks to the orchestrator from message handlers. Both text and voice-confirmed paths flow through it via `maybeOpenDestructiveGate()`. One place to maintain, one place to audit, no parallel implementations of the same check.
- **Strict `CONFIRM` for destructive ACCEPT, case-insensitive for stray-CONFIRM intercept.** The all-caps friction on accept is a deliberate safety signal — autocorrect-mangled "Confirm" safely cancels a real gate. The stray intercept can relax because the consequence of mis-classification is just suppressing a reply, not auto-routing a destructive command.
- **`AGENT_ID_OVERRIDE` env var in `ai-do.sh` for Telegram-originated cost attribution.** 3-line patch; non-Telegram callers unaffected. Token rows now land as `telegram_bot_<purpose>` so dashboard cost rollups separate Telegram spend from agent-tier spend.
- **Bullet spacing on every Telegram outbound message.** `spaceBullets()` helper wraps `sendChunked()` and inserts blank lines between consecutive bullets outside code fences. Phone readability — operator-stated, durable feedback memory.

## Architecture decisions (concise table form, supplementing the prose above)

| Decision | Reasoning |
|---|---|
| grammy + long polling (not webhooks) | Rural ADSL; no port forwarding |
| faster-whisper medium model, CPU int8 | Apple Silicon performance plenty for 20s/30s bar (22s audio → 11.7s transcribe); ctranslate2 has no Metal/ANE on macOS but doesn't need it |
| `@grammyjs/files` plugin (`hydrateFiles`) | grammy ≥1.21 moved the rich File helper out of core; bare `ctx.getFile()` returns a plain object without `.download()` |
| Per-chat serialization queue | Sonnet calls take 10–60s; preserve in-order replies |
| Pending state in-process (no DB persistence) | Restart wipe is acceptable — operator can resend |
| Soft budget cap on /ctx, not hard pre-cap | Pre-estimating LLM cost is unreliable; audit post-call, alert on over-budget |

## Lessons learned

- **grammy ≥1.21 moved file-download helpers to `@grammyjs/files`.** `ctx.getFile()` in core returns a bare `File` (file_id, file_path, file_size) — no `.download()` method. Install `@grammyjs/files`, then `bot.api.config.use(hydrateFiles(bot.token))` hydrates the response. Without that, voice-download dies with `file.download is not a function`.
- **Apple Silicon + faster-whisper does NOT use Metal/Neural Engine.** ctranslate2 runs CPU-only on macOS via int8 AVX/NEON kernels. Still ~1.8x realtime for medium model on M4 Pro (22s audio → 11.7s transcribe). If true Metal acceleration is the goal, that's whisper.cpp or mlx-whisper, not faster-whisper. Don't trust prose claims about "neural engine acceleration is mature" — verify with a real timing test.
- **macOS TCC denial on external volumes presents as EPERM, not EACCES.** Unix perms can be correct (drwxrwxr-x mmm2 staff) and the volume mounted RW, and a `touch` still fails. The discriminator is errno: EPERM = something above filesystem (TCC/sandbox/SIP/flags), EACCES = Unix perms. macOS 13+ has a separate **Removable Volumes** TCC permission that's NOT covered by granting Terminal "Full Disk Access" — the responsible process for a Claude Code child is the `claude` CLI binary, not Terminal.
- **Non-interactive `claude -p` sessions inherit default permission policy, which denies most Reads on script files.** Phase A's text test ("what's our SSD mount path?") dodged this because `df` is allowed by default; Phase B's voice queries ("what's our token spend today?") hit it immediately. The orchestrator surfaces denial as "I need read permission for…" in chat — that's an LLM honestly reporting being blocked, not a bot bug. Long-term fix is either an allowlist `~/agents/.claude/settings.json` (fragile) or moving operational meta-queries to deterministic slash commands like `/spend` and `/audit_recent` (cleaner — captured as D025).
- **iOS autocorrect capitalizes the first letter of a sentence.** A user typing `confirm` will see Telegram deliver `Confirm`. Safety-critical accept paths should remain strict (autocorrect → cancel is safe-fail). Stray-detection paths can relax to case-insensitive on the lone word — false-positive cost is just suppressing a reply, not auto-routing a destructive command.
- **Verify the audit_log against the bar list before requesting more test inputs.** Phase B sign-off: operator sent 3 voice notes that hit every bar; I asked for 3 more, wasted their time. The data was already there. Same rule applies to sign-off itself: operator claimed all 6 Phase D bars passed; audit had no `destructive_verb_confirmed` row. Pushed back, got the clean retry. Trust the data, not the claim — in both directions.
- **EPERM vs EACCES, and "responsible process" model, generalize beyond TCC.** Whenever a permission-style error appears with valid-looking Unix metadata, ask: is this the Unix permission system, or something above it? Above-Unix denials are usually app-bound, not user-bound, and won't transfer from a parent process that has the grant.
- **Telegram voice notes are Opus-in-Ogg (`.oga` extension, not `.ogg`).** faster-whisper handles via ffmpeg internally, so a wrapper that accepts both is one line. ffmpeg is a hard runtime dep for any non-WAV input to faster-whisper — `brew install ffmpeg` is part of the install recipe.
- **Soft budget caps with cost-after audit beat hard pre-caps for LLM-using pipelines.** /ctx pipeline runs ai-do.sh, then queries `token_usage` by correlation_id for cost, audits `ctx_over_budget` if exceeded, but always completes. Pre-estimating token cost is unreliable; complete-and-flag is more useful than refuse-with-guess.
- **`.bak` files written by tooling will get auto-committed to git if not in gitignore.** /ctx writes a pre-narrative skeleton backup before splicing; the nightly brain-autocommit swept it in. Not harmful (just pre-narrative noise) but bloats history. D027 logged.
- **Single-chokepoint security gates beat parallel implementations.** Phase D's destructive scan sits in `routeAndLog()`, called from text and voice-confirmed paths both. One scan, one audit, no risk of a future path forgetting to gate.
- **macOS bash 3.2 still bites.** No `${var,,}`, no associative arrays, no `mapfile`. Workarounds: `tr '[:upper:]' '[:lower:]'` for case, `case` for routing, plain arrays for collections. The user's existing patterns in `~/agents/lib/` follow this already; new shell code should match.

## Operator corrections

- **"Going with option (b), not (c)" on /ctx.** Mid-Phase-A, operator promoted the /ctx Telegram flow from "skeleton-only with desk handoff" to "full narrative pipeline including orchestrator narrative-fill." Verbatim reasoning: *"the entire point of Phase 2 is reaching the system from the phone. A /ctx that requires desk access defeats that."* This rule generalizes — captured as project memory `project-phase2-phone-first`.
- **"Can you have it make a space between bulletin points."** Phone-readability ask after seeing the first cramped Telegram reply. Applied via `spaceBullets()` helper. Captured as feedback memory `feedback-telegram-bullet-spacing`.
- **"We already did 3 tests, why are we doing 3 more?"** Direct pushback on my over-asking during Phase B sign-off. The existing audit rows had already cleared every bar including the 10-min timeout. Captured as feedback memory `feedback-verify-before-requesting-more`.
- **"This isn't a CC permission prompt — it's the bot itself returning this string to Telegram."** Phase B mid-session, when the orchestrator's reply ("I need read permission for the lib directory") looked like a bot error. Operator's diagnostic was sharp: lib perms not the issue; real cause was upstream (non-interactive `claude -p` permission policy). I had presumed a filesystem perms problem; operator's framing led to the actual D025 finding.
- **"Models on internal SSD permanently."** After my pivot from `/Volumes/ActiveSSD/` (TCC-blocked) to `~/whisper/` (working), operator made it the permanent choice rather than fixing TCC for the external SSD. Reasoning: external SSD's actual purpose is bulk rsync archives, not models. Captured as project memory `project-models-on-internal-ssd`.
- **"Skipping Phase C (launchd) for now."** Operator-stated reason: *"computer hardly ever shuts off."* Real risk acknowledged in D026 with explicit un-defer triggers.
- **My TCC-naming sloppiness.** I called it "Full Disk Access blocks me" when the actual permission is **Removable Volumes** (separate category as of macOS 13+). Operator caught this by verifying their Terminal had FDA: *"your earlier diagnosis that TCC blocked the /Volumes/ActiveSSD/ write was wrong, or there's something else going on."* The shape of the diagnosis was right; the name was wrong.
- **Stale path in project CLAUDE.md.** Build spec referenced `/Volumes/AgentSSD/`; actual mount is `/Volumes/ActiveSSD`. Operator noted: *"Project instructions are stale on the mount name — I'll note that for the next instructions update."* (Side note: this is exactly the kind of stale-state drift that justifies the verification-before-recommendation rule.)

## What's next

- **Phase 2 closed; Phase 3 awaits its triggers.** Phase 3 gate (per existing HANDOFF.md) remains "first AITEAM site live (silver platters) AND `mission_tasks` has rows (kanban)." No internal blockers — just external readiness.
- **D025 most likely to be next un-deferred.** Orchestrator permission policy for Telegram-spawned `claude -p` sessions. Recommended fix is path (b): deterministic slash commands (`/spend`, `/audit_recent`, `/tasks_open`, `/kill_switches`) that run sqlite queries directly, never burning LLM tokens or hitting the permission wall. Will fire first time operator hits the "I need read permission for..." reply on a real voice query.
- **D026 (launchd) un-defer triggers:** (1) first time bot dies silently and operator notices, (2) before any externally-publishing agent goes live (Twitter, Pinterest, etc. — those *must not* miss a beat), (3) operator wants to fully close Phase 2 with all four sub-phases ✅.
- **D027 (`*.bak` to `~/brain/.gitignore`) — 2-min fix.** Fold in next time `~/brain/.gitignore` is being edited, or whenever `.bak` files become noticeable noise in `git log` diffs.
- **No active bugs.** Bot is healthy at PID 45291, voice=true, allowlist=1, 300s destructive timeout restored, all sign-off audit rows landed.
- **No in-progress build threads.** This session's work is fully committed to code; deferred items are documented; nothing half-finished.
- **First Telegram-Originated operational query** (e.g. *"what's our token spend today?"*) will hit D025. When that happens, decide whether to ship a single `/spend` slash command quickly (~30 min) or build the broader deterministic-command set as a Phase 2.1 sub-phase.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_1614.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-15 13:18 **Last session summary:** Git-initialized the `~/brain/` Obsidian vault. SSH key generated, GitHub remote wired (`git@github.com:mdp280028-ui/brain-vault.git`), initial push landed, nightly autocommit cron scheduled for 23:55. First offsite backup chain live.  --- 
