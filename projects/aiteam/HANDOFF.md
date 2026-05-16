# AITEAM Project Handoff
**Last updated:** 2026-05-15 22:42
**Last session summary:** Market Scribe session 2 — built and signed off the Curator (Cowen knowledge accretion + Casper outlook-history) and the Briefer (morning Telegram roll-up), wired `notify.sh` to live Telegram Bot API, shipped `/important` slash command + `model_override` schema, ran D032 Sonnet-vs-Opus blind comparison (Sonnet stays default), and shipped D043 mentioned_assets-driven multi-asset convergence (1-asset case exercised on today's real briefs). Audit rows #141–#172; total session cost ~$1.20.

---

## Current phase
**Market Scribe session 2 closed (curator + briefer + /important + D032 + D043 ✅). Awaiting morning-of-2026-05-16 manual validation run as day 1 of D033 cron-activation gate (1-3 days clean before cron flips on).**

## Completed (cumulative)
- [x] Phase 0 chassis (steps 1–16): folders, .env, wrappers, DB schema, kill switches, dashboard scaffold, agents (orchestrator, librarian, editor), diary writer, smoketests
- [x] Step 10c — dashboard cost chart with KPI row + Chart.js stacked bar + breakdown table
- [x] Step 17 — editor tier test; Sonnet 4.6 confirmed as production judge at r=0.933
- [x] Step 17b — war room slash commands (`/morning_brief`, `/weekly_review`)
- [x] Step 18 — war room discuss mode
- [x] Step 19 — failure modes per agent + Phase 2 gaps doc
- [x] Context-save system — `ctx.sh`, harvester, `/ctx` slash command, handoff template
- [x] Brain vault git-init — `~/brain/` is now a git repo with private GitHub remote, daily auto-commit at 23:55
- [x] **Phase 2A** — Telegram text bot (grammy long-polling, allowlist, queue, slash commands, AGENT_ID_OVERRIDE pattern)
- [x] **Phase 2B** — Whisper voice transcription (faster-whisper medium, confirm flow)
- [x] **Phase 2 /ctx pipeline** — Telegram `/ctx` runs `scripts/ctx.sh` then pipes through orchestrator for narrative fill
- [x] **Phase 2D** — Destructive-verb safety gate (single-chokepoint `routeAndLog()`)
- [x] **Market Scribe session 1** — Scribe (poll + transcript) + Analyst (per-video brief) for Cowen + Casper. 10 steps shipped. Audit row #140.
- [x] **Market Scribe session 2 — Curator + Briefer + /important + D032 + D043** (this session):
  - Curator agent built: `curate_cowen.sh` (LLM-heavy synthesis, 5 LLM calls / Cowen brief), `curate_casper.sh` (mechanical outlook-history append), `curate.sh` orchestrator, sentence-transformers MiniLM similarity with calibrated threshold 0.35/0.15
  - 28 Cowen seed topics across 5 groups + `_misc/` pre-created with seed-description embeddings
  - Briefer agent built: `brief.sh` orchestrator, `compose_brief.sh` (one Sonnet max-turns=1 call), `render_outlook_grid.py` (monospace 7×3 grid), three named failure modes
  - `notify.sh` extended inline: direct Telegram Bot API curl, HTTP error handling, audit logging (notify_sent / notify_failed / notify_stub), retains `TELEGRAM_OUTBOUND_ENABLED` kill switch
  - `/important <video_id>` slash command added to bot.js: validates id, writes `model_override='opus'`, spawns analyze.sh, sends both file paths + cost in follow-up reply
  - `model_override` column added to seen_videos; analyst routes to `ai-think.sh` when 'opus', writes parallel `<date>_<vid>_opus.md` brief, clears override on success
  - `AGENT_MAX_TURNS_OVERRIDE` pattern added to `ai-do.sh` (mirrors AGENT_ID_OVERRIDE)
  - **D032 verdict (audit row #164):** Sonnet stays analyst default. Opus only via /important, occasional spot-check. Reasoning: Opus generates novel slugs that break curator routing; quality delta doesn't justify the downstream cost.
  - **D043 mentioned_assets contract (audit row #172 regression catch):** Analyst frontmatter field `mentioned_assets: [<list>]` drives per-asset convergence loop in briefer. 1-asset case exercised tonight; multi-asset waits on real-world data.
  - Operator-confirmed phone receipt of morning brief with monospace grid rendering correctly via `<pre>` + parse_mode=HTML.

## In progress
- [ ] **Day 1 of D033 cron-activation gate** — operator runs briefer + scribe + curator manually tomorrow morning to validate the full chain on fresh data. 1-3 days clean → cron activation.

## Blocked
- Phase 3 (silver platters, kanban) gated on: first AITEAM site live + GA4 data, `mission_tasks` having rows. No internal blockers.
- D043 multi-asset convergence code path: built but only 1-asset branch exercised tonight. Multi-asset waits on Cowen or Casper actually discussing ETH/SOL/etc in a new video. No action needed.

## Known bugs
| Bug | Severity | File(s) | Notes |
|---|---|---|---|
| Per-agent kill switches bypassed by direct wrapper or slash command invocations | low | `~/agents/lib/ai-*.sh`, slash commands | only `SYSTEM_PAUSED` covers; defense-in-depth gap, deferred to Phase 3 |
| `PUBLISHER_DEPLOY_ENABLED` has no enforcement site | low | `~/agents/config/.env` | gate; no publisher agent exists yet |
| `DAILY_API_BUDGET_USD` observability-only, no enforcement | low | wrappers | revisit when off Claude Max |
| 16 historical audit_log rows have trailing `}` from old `${5:-{}}` heredoc bug | cosmetic | `audit_log` rows 1–17 | fixed at write time; historical rows append-only |
| Pre-existing skeleton `.bak` files committed to brain repo | cosmetic | `~/brain/projects/aiteam/context-saves/*.bak` | tracked as D027 |
| Phase 2 telegram-stack work uncommitted in `~/agents/` | low | scribe/curator/briefer/lib/telegram | surfaced 4× across sessions now — retro commit before D033 cron activation |
| `log_to_audit.sh` breaks on `'` in payload | low | `~/agents/lib/log_to_audit.sh` | tracked as D044; workaround: `sed "s/'/''/g"` before passing |
| `seed_taxonomy.sh` clobbers accreted Casper outlook-history rows | low | `~/agents/market/curator/seed_taxonomy.sh` | tracked as D042; fix: skip writes when row_count > 0 |

## Architecture decisions (recent)
| Decision | Reasoning | Date |
|---|---|---|
| Sonnet 4.6 production judge for editor | r=0.933 vs operator baseline; 2.7× cheaper than Opus | 2026-05-14 |
| Append-only audit log; bugs as corrective rows | historical truth preserved | 2026-05-14 |
| Phone-first principle for all Telegram-facing features | "the entire point of Phase 2 is reaching the system from the phone" | 2026-05-15 |
| ML models on internal SSD (`~/whisper/`, `~/brain/channels/`) permanently | external SSD is for bulk rsync archives | 2026-05-15 |
| Single-chokepoint destructive gate via `routeAndLog()` | one gate, one place to maintain | 2026-05-15 |
| Per-pipeline `<pipeline>_<purpose>` agent_id naming | dashboard rollups become trivial; generalises AGENT_ID_OVERRIDE | 2026-05-15 |
| YouTube channel resolution via `search.list` + verify, NOT `forHandle` | handles drift; `forHandle` returns empty `items[]` silently | 2026-05-15 |
| Analyst prompt in external `prompt_template.md` with `${VAR//A/B}` substitution | dodges bash 3.2 heredoc-with-apostrophe bug | 2026-05-15 |
| **Curator similarity threshold = 0.35; group threshold = 0.15** | calibrated against checkpoint 4 data; top-1 correct in 5/5 cases with scores clustering 0.38-0.59 | 2026-05-15 |
| **Sentence-transformers MiniLM model, project-co-located cache** at `~/agents/market/curator/.cache/hf/` | matches whisper precedent; per-agent ML state on internal SSD | 2026-05-15 |
| **Seed-stub anchored embedding** (slug-as-words + seed_description + current_thesis) | discriminates on cold start before any briefs accrete; current_thesis dominates after | 2026-05-15 |
| **Curator persists LLM responses to `workspace/responses/<corr_id>.md` before parsing** | a parse-only bug becomes free to retry; awk close-reserved-word bug burned $0.28 we couldn't recover | 2026-05-15 |
| **Briefer = deterministic stitching + one Sonnet max-turns=1 narrative call** | header/per-video/grid bash-built; LLM composes only THESIS_SHIFTS + CONVERGENCE | 2026-05-15 |
| **notify.sh direct Bot API curl** (BOT_TOKEN + ALLOWLIST_USER_IDS from telegram/.env) | grammy's ctx.reply is inbound-only; cron outbound needs its own path; HTML parse_mode for monospace grids | 2026-05-15 |
| **Stub-first → live-flip discipline** for outbound-side infrastructure work | validate message body before committing to phone delivery; pattern set by notify.sh build | 2026-05-15 |
| **AGENT_MAX_TURNS_OVERRIDE env-var pattern** in ai-do.sh / ai-think.sh | non-agentic text-in/text-out callers pin max-turns=1 without global .env edits | 2026-05-15 |
| **D032: Sonnet stays analyst default; Opus is /important-only** | Opus generates non-canonical slugs that don't map to curator seeds; quality delta doesn't justify slug-compat burden | 2026-05-15 |
| **model_override consumed-on-success** (cleared to NULL after analyze.sh) | /important is one-shot, not sticky; re-trigger if needed | 2026-05-15 |
| **Model-override briefs go to parallel `<date>_<vid>_opus.md`** (canonical stays at unsuffixed path) | Sonnet brief remains pipeline canonical; Opus is operator spot-check | 2026-05-15 |
| **Briefer filters `_(opus\|haiku)\.md$` from canonical discovery globs** | model-override = operator artifact, not pipeline input | 2026-05-15 |
| **mentioned_assets frontmatter contract** (analyst → briefer) | drives per-asset convergence; Cowen rule = substantive discussion, Casper rule = mechanical from outlook arrows | 2026-05-15 |

## Deferred items
- **D025** — Orchestrator permission policy for Telegram-spawned `claude -p` sessions
- **D026** — Phase C launchd auto-restart for Telegram bot (bot currently `nohup`'d, dies on Mac reboot)
- **D027** — Add `*.bak` to `~/brain/.gitignore`
- **D033** — Cron activation for poll.sh + briefer. Gate: 1-3 days clean manual validation starting 2026-05-16.
- **D039** — outlook-history TL;DR repetition on rows 2+ from same video. Re-evaluate after 7+ days of real data.
- **D040** — curator idempotency duplicate-row guard. Add `(date, video_id, horizon)` existence check before append. Trigger: next curator touch or first wild observation.
- **D041** — curator topic fan-out for analyst-untagged topics. Decide after 30 days of runs show under-tag frequency.
- **D042** — `seed_taxonomy.sh` Casper preserve guard. Skip writes to outlook-history files when `row_count > 0`. 5-min fix at next curator touch.
- **D044** — `log_to_audit.sh` SQL-escape on apostrophes. Add `sed "s/'/''/g"` before SQL embed. 3-line fix at next chassis touch.
- 4 Phase 2 gaps from `~/brain/projects/aiteam/docs/phase2_gaps.md`

## Next session should
1. **If manual morning run is clean tomorrow:** start the D033 cron-activation clock (day 1 of 1-3). After day 3, flip the cron lines on.
2. **If anything in the chain misfires tomorrow:** debug, then either re-run or roll the failure into a fix-and-resume.
3. **If operator wants to fix infrastructure debt:** address D042 (Casper preserve guard) and D044 (SQL-escape) — both are 5-minute, high-signal fixes with no architectural risk.
4. **If a Telegram operational query dead-ends on permissions:** address D025 (orchestrator permission policy for `claude -p` spawns).
5. **If bot dies silently after a reboot:** address D026 (launchd plist).
6. **If `mission_tasks` accumulates rows:** build 10b (kanban view on dashboard).

## Files to reference
- `~/agents/market/README.md` — Market Scribe overview
- `~/agents/market/scribe/{watchlist.yaml, poll.sh, fetch_transcript.sh}` — Scribe layer
- `~/agents/market/analyst/{analyze.sh, prompt_template.md, brief_cowen.template.md, brief_casper.template.md, CLAUDE.md}` — Analyst layer (CLAUDE.md added this session)
- `~/agents/market/curator/{curate.sh, curate_cowen.sh, curate_casper.sh, similarity.py, parse_outlook.py, seed_taxonomy.sh, cowen_topic_prompt_template.md, requirements.txt, agent.yaml, CLAUDE.md, failure_modes.md}` — Curator layer (built this session)
- `~/agents/market/briefer/{brief.sh, compose_brief.sh, render_outlook_grid.py, compose_prompt_template.md, agent.yaml, CLAUDE.md, failure_modes.md}` — Briefer layer (built this session)
- `~/agents/lib/{ai-do.sh, ai-think.sh, notify.sh, log_to_audit.sh}` — wrappers + infra (notify.sh + ai-do.sh extended this session)
- `~/agents/telegram/bot.js` — `/important` handler added this session
- `~/brain/channels/<slug>/{transcripts,briefs}/` — Scribe + Analyst outputs
- `~/brain/expertise/cowen/<group>/<slug>/{synthesis.md, evidence-log.md, predictions-log.md}` — Curator outputs (28 topics across 5 groups + `_misc/`)
- `~/brain/expertise/casper/outlook-history/<asset>.md` — Casper per-asset outlook history (7 files)
- `~/agents/market/shared/seen_videos.db` — SQLite of processed videos (now has `curated`, `model_override` columns)
- `~/store/aiteam.db` — audit_log, token_usage, conversation_log

## Gotchas for future me
- **`ai-do.sh` MUST stay pinned to `--model sonnet`.** Without it, `claude -p` inherits Opus from parent CC session.
- **macOS bash is 3.2.57** — no `${var,,}`, no associative arrays, no `mapfile`/`readarray`.
- **macOS bash 3.2 heredoc bug**: unquoted heredoc + variable expansion + apostrophe → cascading phantom quote errors. Use external file + `${VAR//A/B}` substitution.
- **macOS `sed -i`** requires an argument: `sed -i '' -e '...'`. GNU pattern fails.
- **SQLite columns**: `ts` (unix epoch INTEGER), `estimated_cost_usd`, `description`.
- **Audit log is append-only**. Bugs → corrective rows.
- **`log_to_audit.sh` does NOT SQL-escape its payload.** Any `'` in the payload breaks the INSERT. Pre-escape via `sed "s/'/''/g"` until D044 is fixed.
- **`claude -p --output-format json`** is the authoritative cost source.
- **Brain git identity is `mdp280028-ui` LOCALLY** in `~/brain/`. Global is `AITEAM Builder`.
- **`ssh -T git@github.com`** exits 1 on success — look for "successfully authenticated" string.
- **Do not force-push to `~/brain/` origin/main**.
- **grammy ≥1.21 requires `@grammyjs/files`** for `.download()`.
- **faster-whisper on macOS is CPU-only**. Don't claim ANE/Metal acceleration.
- **Non-interactive `claude -p` sessions** inherit default permission policy and deny most script-file reads.
- **Telegram voice notes are `.oga`** (Opus-in-Ogg).
- **iOS autocorrect capitalizes the first letter** — strict CONFIRM for safety-critical accept paths, case-insensitive for stray-intercepts.
- **Bot pending state is in-process Map** — bot restart wipes it.
- **YouTube channel handles drift.** Resolve via `search.list?q=...&type=channel` + verify subs/customUrl, not `forHandle`. `forHandle` returns HTTP 200 with empty `items[]` on rename.
- **YouTube descriptions contain raw control characters** (U+0000–U+001F). Pre-clean with `tr -d '\000-\037'` before jq strict mode.
- **YouTube auto-VTT rolling captions** need overlap-based dedup, not exact-match.
- **`yt-dlp` output filename is `<base>.<lang>.vtt`** — use a glob, not a hardcoded name.
- **Sonnet via `claude -p` does internal multi-turn** even on text-in/text-out tasks. Pin via `AGENT_MAX_TURNS_OVERRIDE=1` for non-agentic uses.
- **Opus on `claude -p` does NOT do that.** Opus wrote 1.7K tokens and stopped; Sonnet writes 12-13K of internal reasoning. Don't assume "Opus is always more expensive."
- **bash `${VAR//pattern/replacement}` handles multi-line values cleanly** — prefer over sed for template substitution with newlines.
- **Analyst attributes quotes to start-of-spoken-thought timestamps** (concatenated VTT cues to sentence boundaries). Operator-confirmed desired.
- **Telegram doesn't render markdown tables.** Storage = markdown table; presentation (briefer) = HTML `<pre>` monospace grid with em-dash (—) for blanks.
- **all-MiniLM-L6-v2 scores cluster low** (0.3-0.6) for short-query-vs-long-anchor matches. 0.50 threshold is too tight. Working values: 0.35 sim, 0.15 group.
- **`close` is reserved in awk** — never use as `-v` variable name; produces misleading syntax errors at wrong line numbers.
- **`paste -sd ", " -` on BSD/macOS** uses each char as cycling delimiter, not multi-char separator. Use `tr | sed` instead.
- **Persist LLM responses to `workspace/responses/` before parsing** for any agent with non-trivial per-call cost. A parse-only bug becomes free to retry.
- **Model-override briefs (`_opus.md`, `_haiku.md`)** are operator spot-check artifacts. Filter `grep -Ev '_(opus|haiku)\.md$'` from canonical pipeline discovery globs.
- **MEMORY-stored operator preferences need active recall.** Operator wants comparison/grading artifacts in `~/Downloads/`. Copy proactively without being asked.
- **Telegram outbound from cron jobs** needs direct Bot API curl. grammy `ctx.reply` is inbound-only.

---

## Pre-existing context
- friend-bot/ moved from `~/agents/` to `~/projects/friend-bot/` at chassis init — operator's separate project, not part of AITEAM.

## Sign-off discipline
A step is ✅ only when the build plan's pass condition has been *executed* and produced the expected result. Artifact-exists or process-started is not ✅ — that's ⚠ with a TODO. Honest ⚠ beats premature ✅. Verification cuts both ways: don't accept a ✅ claim from the operator without checking audit_log against the bar list. Trust the data, not the claim.
