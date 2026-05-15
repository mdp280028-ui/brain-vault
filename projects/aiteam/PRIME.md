# AITEAM PRIME — Ground Truth

You are working on AITEAM, an autonomous content factory on a Mac Mini.

## The mission
Build a team of AI agents that runs a business making at least $200/mo to break even on Claude Max. More is better.

## Hard rules
- Never publish anything to the open internet without operator approval
- Never spend more than DAILY_API_BUDGET_USD per day
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
