# AITEAM failure modes — index

One file per agent under `~/agents/<id>/failure_modes.md`. This index links to them and documents the global kill-switch surface.

## Per-agent failure mode docs

| agent | tier | file | mode count |
|---|---|---|---|
| orchestrator | sonnet (do) | [~/agents/orchestrator/failure_modes.md](../../../../agents/orchestrator/failure_modes.md) | 6 |
| librarian | haiku (cheap) | [~/agents/librarian/failure_modes.md](../../../../agents/librarian/failure_modes.md) | 5 |
| editor | sonnet (do) | [~/agents/editor/failure_modes.md](../../../../agents/editor/failure_modes.md) | 5 |

The orchestrator file also covers `write_diary.sh` (no separate diary-writer agent — diary is part of orchestrator) and budget-overrun observability.

---

## Global kill switches (verified existing)

All switches live in `~/agents/config/.env`. The file is re-sourced by `~/agents/lib/check_kill_switches.sh` on every long-running script invocation (mtime-based refresh, ~2s liveness). To flip a switch, edit `.env` and any running script picks it up within ~2 seconds.

| switch | default | what it stops | enforced in |
|---|---|---|---|
| `SYSTEM_PAUSED=true` | `false` | **all LLM wrappers** (`ai-cheap.sh`, `ai-do.sh`, `ai-think.sh`) exit 1. War room endpoint returns 503. | `ai-*.sh`, `/api/warroom/standup` |
| `LLM_SPAWN_ENABLED=false` | `true` | identical effect to `SYSTEM_PAUSED=true` | same enforcement sites |
| `PUBLISHER_DEPLOY_ENABLED=false` | `false` | (referenced but no enforcement site found yet — Phase 2 gate, not active in Phase 1) | (none active) |
| `WRITER_ENABLED=false` | `true` | writer agent's `run_agent.sh` invocation exits 1 (when writer is registered) | `run_agent.sh` |
| `EDITOR_ENABLED=false` | `true` | editor's `run_agent.sh` invocation exits 1 | `run_agent.sh` |
| `LIBRARIAN_ENABLED=false` | `true` | librarian's `run_agent.sh` invocation exits 1 | `run_agent.sh` |
| `TELEGRAM_OUTBOUND_ENABLED=false` | `false` | `notify.sh` exits 0 without sending | `notify.sh` |

## Per-agent kill switch convention

`run_agent.sh` computes the switch variable name dynamically: `<AGENT_ID_uppercase>_ENABLED`. So adding a new agent `foobar` automatically gets a `FOOBAR_ENABLED` switch (default true if unset).

```bash
# from run_agent.sh:
SWITCH_VAR="$(printf '%s' "$AGENT_ID" | tr '[:lower:]' '[:upper:]')_ENABLED"
SWITCH_VAL=$(eval "echo \${$SWITCH_VAR:-true}")
if [ "$SWITCH_VAL" = "false" ]; then exit 1; fi
```

**Important caveat:** per-agent switches only fire when the agent is invoked via `run_agent.sh`. Direct invocations of the wrappers (`ai-cheap.sh "prompt"`) or slash commands that invoke wrappers directly **bypass** per-agent switches. To halt those, use `SYSTEM_PAUSED=true` or `LLM_SPAWN_ENABLED=false`.

## Order of escalation (when stopping the bleeding)

1. **Single misbehaving agent:** flip its per-agent switch (`<AGENT>_ENABLED=false`).
2. **All LLM spend at risk:** flip `SYSTEM_PAUSED=true`. Within ~2s, every active wrapper refuses new calls. Existing API requests already in flight will complete.
3. **Hard kill (instance level):** kill the running script process (`kill <pid>`) or restart the dashboard server. The dashboard `/api/warroom/standup` watchdog at 300s will also terminate fan-outs that exceed it.

## Universal recovery checklist

After flipping a switch and diagnosing the issue:

1. Read the agent's `failure_modes.md` for the specific mode.
2. Apply the recovery steps from that mode.
3. Test in isolation: `bash ~/agents/lib/run_agent.sh <agent_id> "ping"`.
4. Flip the switch back to its enabled value.
5. Add an `audit_log` row noting the incident, mode, and resolution (use `~/agents/lib/log_to_audit.sh`).
6. If the mode wasn't already documented or the recovery path was wrong, update the agent's `failure_modes.md`.

## Coverage gaps across the fleet

- **No active enforcement of `DAILY_API_BUDGET_USD`** — observability-only via the dashboard cost chart. Will be revisited when the project moves from Claude Max sub to direct API.
- **No automated detection cron** for missing diary, stale editor calibration, or stuck inbox files. All such checks are manual today.
- **Per-agent switches bypassed** by direct wrapper or slash invocations — only `SYSTEM_PAUSED` covers those paths.
- **`PUBLISHER_DEPLOY_ENABLED`** is referenced in env but has no enforcement site in code yet — Phase 2 gate.
