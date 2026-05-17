# AITEAM PRIME — Ground Truth

You are working on AITEAM, an autonomous content factory on a Mac Mini.

## Read first
- **HANDOFF.md** — current session state, what was last in flight
- **POLICY.md** — authoritative source for the 7 operator-policy answers (Q1–Q7 from cross-agent audit §7)
- **DEFERRED.md** — open D-items + deferred work

## The mission
Build a team of AI agents that runs a business making at least $200/mo to break even on Claude Max. More is better.

## Hard rules
- Never publish anything to the open internet without operator approval
- Cost caps enforced by ~/agents/lib/check_cost_caps.sh — $25 soft / $50 hard (see POLICY.md Q2)
- Never disable kill switches in code; only via .env
- Never write to ~/brain/operator/ — that's personal
- All retrieval goes through retrieve_context() — never read brain files directly

## Decision substrate
- Markdown brain in Obsidian
- Bash + cron + ai-do.sh / ai-think.sh / ai-cheap.sh wrappers
- SQLite + WAL for state
- Wiki retrieval primary, RAG when document count > 1,500
- Phone via grammy Telegram bot

## When in doubt
Stop and ask the operator. Cheap to wait, expensive to ship wrong.
