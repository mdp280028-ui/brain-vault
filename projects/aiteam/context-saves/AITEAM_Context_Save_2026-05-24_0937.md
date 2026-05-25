# AITEAM Context Save — 2026-05-24_0937
**Generated:** 2026-05-24T09:37:59-0700
**Since last save:** 2026-05-24 09:37:55
**Session topic:** idea-agent v1 + /model picker + notes-agent v1 (text + image + forwards) — three new agents + Telegram surface

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
(no audit rows since last save)

### Token usage
(no token usage since last save)

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

**Three new agents + extensions, seven commits in this chat:**

1. **idea-agent v1 — weekly self-improvement loop.** 7-phase chain (diagnose → zombie_check → scan → google_search → synthesize → publish → calibrate) wired as one cron at Sunday 09:00 local. Three new tables (`idea_proposals`, `idea_calibration`, `zombie_resurface_log`). Sonnet does synthesis (one call/week, $0.40); Haiku does scan + search triage (~$1.25/week). Strict 5-test gate (signal cite, <4h first action, downside, number Track A/B, not a re-suggestion) plus a first-run cap of 5. Smoke produced 5 Track-B ideas at composite 3.8-4.2 — all instrumentation gaps (`drafter-tick-completion-trap`, `telegram-message-chunker`, `editor-rejection-reason-tagger`, `ship-to-site-skip-dedup`, `drafter-sonnet-retry`). Commits `37b2451` (migrations) + `ad6df7a` (agent). DEFERRED **D087**.

2. **/model slash-command picker — REPLACED the inline-keyboard build.** Operator pivoted after I flagged that grammy bot.js had zero callback infrastructure (would have been ~1.5h of net-new wiring before the picker logic could land). Final shape: `/model` renders a 6-row grid (one per picker-eligible agent), each row has three setter commands (`/model_<slug>_sonnet`, `_opus`, `_off`). Cost warning prefixed on `_opus` selections per POLICY Q2. 6 agents wired: `orchestrator_morning_brief`, `orchestrator_weekly_review`, `editor`, `briefer`, `idea-agent-synth`, `telegram_bot_orchestrator` (free-text chat). New `lib/resolve_model.sh` helper + `AI_DO_MODEL_OVERRIDE` env hook in `ai-do.sh` (matches existing `AI_DO_SKIP_PERMISSIONS` / `AGENT_MAX_TURNS_OVERRIDE` pattern). Commits `2927a71` (initial 5 agents) + `05fc5b6` (chat entry added). DEFERRED **D088**.

3. **notes-agent v1 — Telegram capture + Haiku categorize + weekly Sonnet sort.** Triggers: `/note <text>`, `note <text>` / `note: <text>` / `n: <text>` prefix in message:text, any photo. Photo caption rule: empty or note-trigger → capture; non-trigger caption → vision-chat through orchestrator with `@<image-path>` prefix (Sonnet vision). 7 seed categories with Haiku improvise. Output: `~/brain/projects/aiteam/notes/inbox_<date>.md` (capture) → weekly Sonnet sort moves entries to `notes/<category>.md` (Sunday 10:00, staggered after idea-agent). Commit `73a2ecb`. DEFERRED **D089**.

4. **notes-agent Phase 2.6 — forwarded message capture with confirmation.** Forwards (from any chat) trigger `📩 Forward from X (source): <preview> /yes /no /yes_<cat>` prompt. 5-min auto-expiry via `setInterval`. Manual category override skips Haiku. Bot API 7+ `forward_origin` + legacy `forward_from`/`forward_from_chat` both handled. Photo forwards download to `notes/images/`; other media types (video/document/sticker/etc.) log `forward_media_unsupported` and capture text/caption only. New `pending_forwards` table. Commit `2011c1a`. Added to D089.

5. **One-line `bot-launch.sh` fix.** Manual bot restart from `~/agents/` (not `~/agents/telegram/`) crashed bot because `dotenv` reads `.env` from `process.cwd()`. Patched script to `cd "$(dirname "$0")"` near top. Commit `45a0851`. Sister chat then closed D026 (launchd KeepAlive) at `07925f8` — restart dance is now mostly auto.

**Sister-chat commits interleaved (NOT mine, but landed in shared repos):**
- `07925f8` D026 closed — grammy bot under launchd `KeepAlive`. The "pkill + nohup" restart pattern I documented mid-session is now obsolete (launchd restarts on crash).
- `b11a439` watchdog `launchctl_label` probe added — grammy-bot entry now monitored.
- `23ccd52` D033 closed — market pipeline cron activated.
- `da88d09` cron CLAUDE_CODE_OAUTH_TOKEN env-handling fixes.
- `923464d` / `748b799` / `5ed5be9` etc. — D083 through D086 added by Lane C work. This caused my D-number race (see Lessons).

## Lessons learned

- **D-number pre-flight is racy across concurrent chats.** Sister-chat SSG Lane C added D084, D085, D086 to DEFERRED.md between my pre-flight (read at session start) and my idea-agent commit time. I had reserved D084 in my pre-flight report; the operator's "cleared" message came hours later by which time D084 was taken. **Pattern**: pre-flight reports the proposed D-number, then RE-CHECK `grep -oE '^### D[0-9]+' DEFERRED.md | sort -uV | tail -1` immediately before writing the new section. Don't trust pre-flight numbers across long sessions.

- **`dotenv` reads `.env` from `process.cwd()`, not the script's directory.** `bot-launch.sh` did `exec /opt/homebrew/bin/node /Users/mmm2/agents/telegram/bot.js` with absolute path — but Node's `dotenv/config` looks in CWD. Manual restart launched from `~/agents/` crashed the bot with `FATAL: BOT_TOKEN missing in .env`. **Pattern**: any script that exec's a dotenv-using process must `cd "$(dirname "$0")"` near the top. Worth a fleet-wide audit.

- **Heredoc Python helper must explicitly write to `sys.argv[2]`, not `print()`.** First idea-agent end-to-end smoke produced JSON-correct Sonnet output but `synthesize.sh` got `IDEAS_COUNT=0` because the parser `print()`-ed to stdout instead of writing to the output file path. Cost: one wasted Sonnet call (~$0.40). **Pattern**: when using `python3 - INPUT OUTPUT <<PYEOF`, the script MUST `with open(sys.argv[2], "w") as f: ...` — never rely on stdout capture for the secondary "parsed" output.

- **jq `\"\"` inside bash double-quoted interpolation is a parse hazard.** `google_search.sh` used `jq -r '... \(.value.snippet // \"\")...'`. jq saw INVALID_CHARACTER and rejected all 6 scoring batches, defaulting to score=3. **Pattern**: use `+` concatenation outside the interpolation: `... + (.value.snippet // "")` rather than escaped empty strings inside jq's `\(...)`.

- **grammy `bot.on(filter, ...)` is TERMINAL by default.** Once `bot.on("message:text", ...)` matches, the catchall `bot.on("message", ...)` does NOT fire. This means forward-detection logic can live INSIDE the message:text/message:photo handlers via `if (isForward(...)) return handleForward(ctx)` — no need for a separate `bot.on(":forward_origin", ...)` and no worry about double-firing. Simpler than I initially planned.

- **grammy `bot.command()` accepts strings or string arrays but NOT regex.** For dynamic patterns like `/yes_<category>` (where category is unknowable in advance), intercept in `bot.on("message:text")` BEFORE the unknown-command reject: `text.match(/^\/yes_([a-z][a-z0-9-]{0,29})\b/i)`. The 7 pre-known categories could each be a bot.command, but new ones operator improvises would miss — message:text intercept catches them all.

- **`AI_DO_MODEL_OVERRIDE` env hook in `ai-do.sh` is the right scope for per-agent overrides.** Backward-compatible (default sonnet preserved), opt-in per call (caller sets env before invoking ai-do.sh), invalid values fall back to sonnet silently. Matches the existing `AI_DO_SKIP_PERMISSIONS` / `AI_DO_AGENT_ID_OVERRIDE` / `AGENT_MAX_TURNS_OVERRIDE` patterns — fleet convention is opt-in env hooks, not config files.

- **First-run cap pattern for synthesis agents.** When `PRIOR_IDEAS` is empty, cap output at 5 regardless of pass count. The first run is calibration, not delivery. If 8 ideas pass the gate but it's run 1, keep top-5 by composite, move the rest to `rejected[]` with `failed_test: "first_run_cap"`. Prevents flooding the operator's first review queue while preserving the rejections for spot-check.

- **`claude -p @<image-path>` for vision — `max-turns=2` is the safe ceiling, `max-turns=1` is the optimization to test.** Built notes-agent image capture with `AGENT_MAX_TURNS_OVERRIDE=2` since I couldn't confirm whether `@path` inlines as native multimodal (no tool call) or requires a Read tool turn. Smoke didn't disambiguate. v1.1 task: capture 5 images, query `token_usage` for the `notes-capture` rows to see if any used 2 turns. If all used 1, drop to `max-turns=1`.

- **bot.js→sqlite3 via runShell — JS-side APOS pattern transfers cleanly.** `s.replace(/'/g, "''")` mirrors the bash APOS helper. Used in `/model` setters, `pending_forwards` inserts, `model_overrides` UPSERT. Defensive — relies on caller controlling input shape; for user-controlled text use `--separator '|'` + structured parse.

- **Forward metadata: handle Bot API 7+ `forward_origin` AND legacy `forward_from`/`forward_from_chat`.** Grammy 1.30 forwards both. `forward_origin.type` covers `user` / `chat` / `channel` / `hidden_user`. Default fallback to "unknown" / "unknown" — never crash on unfamiliar forward shapes.

- **When operator says "just build" after sign-off, drop the cross-reference reports.** Operator preference from this session: terse status updates only between STOP gates, no extended sign-off tables, no rationale recaps. Save the structured reports for natural STOP points (pre-flight, smoke, build complete). This saved tokens AND matched operator's pace.

- **Synthetic smoke entries pollute operator's brain.** `capture.sh forward` smoke wrote two test entries to today's real inbox with fake senders ("alice", "@founder_user"). Operator now has to delete them during Sunday review. **Pattern**: smoke writes that touch operator-visible storage should go to a `_test_` prefixed file path or include explicit cleanup at the end of the smoke. Or: use a smoke-only inbox path.

- **Two end-to-end smoke runs cost ~$1.10 total.** First run had two parser bugs that wasted ~$0.70 in Sonnet+Haiku; re-smoke after fixes was $0.40. Worth it — the bugs would have shipped to first Sunday cron run otherwise. Budget rule: budget for one re-smoke at full cost. Save raw responses to disk BEFORE parsing (already a fleet pattern per session-2 lesson) so re-parses don't burn new calls.

- **Don't pre-register every possible /yes_<category> command.** Telegram allows arbitrary single-word categories (Haiku may improvise during capture). Pre-registering 7 seed categories misses improvised ones. Intercept once in message:text via regex, dispatch dynamically.

- **Telegram forward of a forward (nested): use outermost `forward_origin` only.** Don't try to unpack chains — the chain depth isn't reliably preserved by Telegram. Outermost source is the operator-visible attribution; that's what they want to file under.

## Operator corrections

Verbatim where possible — these are the highest-signal items for future-CC:

- **"skip the cross-reference. Just build. Stop and show me the synthesize prompt when it's ready."** (Round 4 of idea-agent build, after I'd produced too-long sign-off summaries). Future pattern: between explicit STOP gates, output 1-sentence status updates, not full tables.

- **"REPLACES the previous inline-keyboard build. Pre-flight findings from prior report still apply. Build is now simpler — no callback infrastructure needed."** Operator pivoted from inline-keyboard to slash-command grid after my escalation. Lesson: when escalation flags substantial extra scope, the operator MAY simplify rather than absorb. Always flag scope, don't quietly absorb.

- **"Synthesize prompt approved with three changes"** — operator added: (a) Track B "unknown frequency — no prior incidents on record" permission (do NOT fabricate numbers); (b) Test 5 "conceptually similar" → "same problem + same mechanism" with bias-toward-propose-and-cite when in doubt; (c) first-run cap of 5 when PRIOR_IDEAS is empty. Operator's gate-design intuition: be honest about uncertainty, narrow the duplicate net, ramp up calibration before flooding.

- **"two not three"** — picker agent_ids for notes-agent. I had proposed three (capture-text, capture-image, sort); operator collapsed capture into one. Lesson: cost-attribution granularity per-action-type in audit_log payload is enough; don't multiply agent_ids beyond what the operator reviews.

- **"Photo caption rule — yes, your proposed rule is right (empty/note-trigger → capture, otherwise → vision chat)"** — confirmed the explicit dual-path: photos can be captures OR vision-chat queries based on caption shape. Lesson: when proposing a fuzzy classifier rule, give the operator the concrete decision rule to ratify.

## What's next

**Immediate (operator-side):**
- Live smoke notes-agent forward capture from phone: forward a message, /yes, /no, /yes_<cat>, photo forward, 5-min expiry. Bot is up as PID 7796 with all handlers loaded.
- Review the 5 idea-agent proposals in `~/brain/projects/aiteam/ideas/weekly_2026-05-23.md` and mark `operator_action` for each (until D089 v1.1 ships a `/idea_mark` handler, this is manual SQLite UPDATE).
- Two synthetic test forward entries in `inbox_2026-05-24.md` from my smoke ("alice" / "@founder_user") — delete before Sunday sort if you don't want them in the archive.

**First scheduled fires:**
- Sunday 2026-05-24 09:00 local — idea-agent weekly run (first auto-execution).
- Sunday 2026-05-24 10:00 local — notes-agent sort (first auto-execution — no inbox files older than today yet, so first real sort is the Sunday after).

**Deferred items active:**
- **D087** idea-agent follow-ups: monthly Opus self-critique, /idea_mark operator-action handler, scan-source expansion, 5th source choice.
- **D088** /model picker follow-ups: wire remaining 12 ai-do.sh callers, per-task `_once` variants, Haiku in picker, inline-keyboard upgrade (now possible since D026 closed in sister chat).
- **D089** notes-agent follow-ups: max-turns=1 test for image captures, album media_group_id grouping, vision-chat prompt engineering, sort.sh re-categorization audit trail, pending_forwards hard cap if >1000 unresolved.

**Closed in sister chat during this session (relevant to my work):**
- **D026** — bot now under launchd `KeepAlive` (commit `07925f8`). Manual restart dance still works but is no longer load-bearing.
- **D033** — market pipeline cron activated.

**Blocked:** none.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-24_0937.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-17 22:00 (Keyword Registry live + brain-tracked pain log + digest archive · 4 production hooks · nightly rebuild cron firing) **Last session summary:** Shipped Keyword Registry agent (`~/agents/keyword-registry/` + `~/brain/projects/aiteam/keyword_registry/`). 162 canonical keywords tracked across 32 live asbestoshq.com slugs, alias-collapsed (chrysotile/white-asbestos/amosite/crocidolite etc — 14 seed rules). Three lib scripts: rebuild_registry.py (full regen, 0.5s), update_registry.py (write-through helper, verb dispatch), backfill_registry.sh (cron wrapper). Four production hooks wired best-effort: deploy_batch.sh new+refresh branches, internal-link/apply_approval.sh, research-opportunity/extract.sh pre-Sonnet shortcut (multi-word title match, 31 of 162 keywords qualify). Nightly cron at 0 3 * * * installed. Brain commit `6c15f3f`, agents commit `6047fe0`. Earlier this wall-clock window: brain-tracked pain_points_log + digest archive (commit 3f7145e) + Gap-1 fix for digest_log.posts_scanned + D078/D079/D080 deferred. Sister chat shipped Backlink Prospector v0.1 (b873996) in parallel — not inspected here. **Prior session summary (preserved):** Session 2 of Research/Opportunity Agent live (SE + YouTube + weekly digest + cron + master switch ON, commit bc7504b). Session 1 (Reddit vertical slice) at commit 2413a94. D075 Opportunity Scout deferred at brain 83cb3ed. Prior: D068 watchdog hygiene + D067 SSG plan recovery + D056 editor production runner + GEO Optimizer Phases 1 & 2 + Internal Link Agent v1.  --- 
