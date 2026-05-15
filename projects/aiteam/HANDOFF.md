# AITEAM Project Handoff
**Last updated:** 2026-05-15 00:42
**Last session summary:** Phase 1 closed; built and verified context-save system (ctx.sh + harvest + slash command + handoff template).

---

## Current phase
**Phase 1 closed. Awaiting Phase 2 triggers.**

## Completed (cumulative)
- [x] Phase 0 chassis (steps 1–16): folders, .env, wrappers, DB schema, kill switches, dashboard scaffold, agents (orchestrator, librarian, editor), diary writer, smoketests
- [x] Step 10c — dashboard cost chart with KPI row + Chart.js stacked bar + breakdown table
- [x] Step 17 — editor tier test (v2 after retraction); Sonnet 4.6 confirmed as production judge at r=0.933 composite vs operator/Opus baseline
- [x] Step 17b — war room slash commands (`/morning_brief`, `/weekly_review`) with auto-discovery
- [x] Step 18 — war room discuss mode (multi-round with cross-agent engagement requirement)
- [x] Step 19 — failure modes per agent + 4 Phase 2 gaps documented at `docs/phase2_gaps.md`
- [x] Step 19 closure — `write_diary.sh` accepts date arg for backfill; `run_agent.sh` propagates `$AGENT_ID` to wrappers for cost attribution
- [x] Context-save system — `ctx.sh`, `auto-harvest-to-library.sh`, `/ctx` slash command, `handoff-template.md`

## In progress
- [ ] none — Phase 1 closed, no active build thread

## Blocked
- Phase 2 work blocked on: first AITEAM site going live (triggers 17a silver platters) and `mission_tasks` having rows (triggers 10b kanban)

## Known bugs
| Bug | Severity | File(s) | Notes |
|---|---|---|---|
| Per-agent kill switches bypassed by direct wrapper or slash command invocations | low | `~/agents/lib/ai-*.sh`, slash commands | only `SYSTEM_PAUSED` covers those paths; defense-in-depth gap, deferred to Phase 2 |
| `PUBLISHER_DEPLOY_ENABLED` has no enforcement site | low | `~/agents/config/.env` | Phase 2 gate; no publisher agent exists yet |
| `DAILY_API_BUDGET_USD` observability-only, no enforcement | low | wrappers | revisit when off Claude Max subscription |
| 16 historical audit_log rows have trailing `}` from old `${5:-{}}` heredoc bug | cosmetic | `audit_log` rows 1–17 | fixed at write time in `log_to_audit.sh`; historical rows left as-is per append-only convention |

## Architecture decisions (recent)
| Decision | Reasoning | Date |
|---|---|---|
| Sonnet 4.6 production judge for editor | r=0.933 vs operator baseline; 2.7× cheaper than Opus; composite MAE 0.363 won't flip publish decisions | 2026-05-14 |
| Append-only audit log; bugs written as corrective rows, not UPDATEs | historical truth preserved; future investigators see the actual chain | 2026-05-14 |
| Slash commands auto-discover from `~/agents/orchestrator/commands/*.sh` | drop a `.sh`, it's live without server restart | 2026-05-14 |
| Skip JSONL token-usage backfill | one day of historical data not worth the project-path-filter complexity; log-forward-only | 2026-05-14 |
| Defer silver platters until first site has GA4 data | building summary jobs for empty data sources is premature | 2026-05-14 |

## Deferred items
- **10b kanban** — trigger: `mission_tasks` table has rows
- **17a silver platters** — trigger: first AITEAM site live + 30 days of GA4 data
- **Phase 2 gaps** (4 items in `~/brain/projects/aiteam/docs/phase2_gaps.md`): detection cron infra, budget enforcement, per-agent switch bypass closure, publisher deploy enforcement

## Next session should
1. **If first site is live with GA4 data**: revisit 17a (silver platters).
2. **If mission_tasks has rows**: build 10b (kanban view on dashboard).
3. **Otherwise**: maintenance mode — keep dashboard running, watch the cost chart, let diary cron accumulate.

## Files to reference
- `~/agents/scripts/ctx.sh` — context save script (run via `/ctx`)
- `~/agents/scripts/auto-harvest-to-library.sh` — pulls Claude.ai context saves from `~/Downloads/`
- `~/agents/.claude/commands/ctx.md` — slash command definition
- `~/agents/CLAUDE.md` — orienting note for CC; describes ctx routine
- `~/brain/projects/aiteam/docs/handoff-template.md` — template this file follows
- `~/brain/projects/aiteam/docs/failure_modes_index.md` — fleet failure modes + global kill switches
- `~/brain/projects/aiteam/docs/phase2_gaps.md` — 4 deferred gaps + triggers
- `~/brain/projects/aiteam/LESSONS.md` — accumulated lessons across sessions
- `~/brain/projects/aiteam/context-saves/` — append-only save library
- `~/agents/dashboard/server.js` — dashboard backend (token-guarded endpoints + war room)
- `~/store/aiteam.db` — SQLite: `audit_log`, `token_usage`, `warroom_transcript`, `mission_tasks`

## Gotchas for future me
- **`ai-do.sh` MUST stay pinned to `--model sonnet`**. Without it, `claude -p` inherits Opus from the parent Claude Code session config and silently triples spend.
- **macOS bash is 3.2.57** — no `${var,,}`, no associative arrays, no `mapfile`/`readarray`. Scripts must declare with this constraint.
- **SQLite columns**: `ts` (unix epoch INTEGER), `estimated_cost_usd`, `description` — not `timestamp`/`cost_usd`/`title`. Don't trust spec docs blindly.
- **Audit log is append-only**. Found a bug? Write a corrective row pointing back. Never UPDATE/DELETE history.
- **`claude -p --output-format json`** is the authoritative cost source. Don't build a pricing map.
- **Discuss mode round-2+ prompts** must explicitly require cross-agent engagement (cite a prior claim). Without that, multi-round drifts into parallel monologues.

---

## Pre-existing context
- friend-bot/ moved from `~/agents/` to `~/projects/friend-bot/` at chassis init — operator's separate project, not part of AITEAM. Venv will need rebuild at new path.

## Sign-off discipline
A step is ✅ only when the build plan's pass condition has been *executed* and produced the expected result. Artifact-exists or process-started is not ✅ — that's ⚠ with a TODO. Honest ⚠ beats premature ✅.
