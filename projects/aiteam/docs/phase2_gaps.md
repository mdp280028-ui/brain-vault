# Phase 2 gaps

Failure mode coverage gaps surfaced during Step 19 that are tracked but deferred to Phase 2. None of these block Phase 1 sign-off.

Index: [failure_modes_index.md](./failure_modes_index.md). Source step: audit row 27, `step_19_completed`.

---

## Gap 1: Detection cron infrastructure

**Problem:** Every documented detection signal in `failure_modes.md` is manual or eyeball-via-dashboard today. There is no scheduled job that watches for these conditions and alerts.

**Specific cron jobs that should exist:**
- **Editor calibration timer** — daily check that runs `SELECT (strftime('%s','now') - MAX(ts))/86400 FROM audit_log WHERE action LIKE 'tier_test_completed_v%';`. If >90 days, alert. Also fires on Sonnet model version bumps (separate trigger).
- **Diary watchdog** — runs at 23:00 each day, checks that yesterday's `~/brain/projects/aiteam/diary/<YYYY-MM-DD>.md` exists. If missing, alerts.
- **Stuck inbox watchdog** — runs nightly after librarian, checks `find ~/brain/projects/aiteam/inbox/ -type f -mtime +1` for files that survived a librarian pass. Each such file is a candidate failure case.

**Why Phase 2:** Phase 1 is about the fleet running cleanly when actively used. Cron-based monitoring is a maintenance-mode concern — sensible once the system is running unattended.

**Sketch of implementation:** likely a `~/agents/lib/watchdogs/` directory with one script per check, all called from a single crontab entry. Each script writes its findings to `audit_log` and optionally fires `~/agents/lib/notify.sh` (currently disabled via `TELEGRAM_OUTBOUND_ENABLED=false`).

---

## Gap 2: Budget enforcement hook

**Problem:** `DAILY_API_BUDGET_USD=10` and `MONTHLY_API_BUDGET_USD=200` are in `.env` but no code consults them. Spend is observable on the dashboard cost chart but not enforced.

**Why deferred:** Project currently runs on Claude Max subscription — spend is observability-only because the subscription absorbs cost up to its cap. Enforcement becomes load-bearing when the project moves to direct API billing.

**Sketch of implementation:** a check in the three wrappers (`ai-cheap.sh`, `ai-do.sh`, `ai-think.sh`) that queries `token_usage` for today's total before firing, and refuses if total + estimated-call-cost would exceed `DAILY_API_BUDGET_USD`. The estimate part is tricky — Claude's per-call cost is only known after the call. A simpler version: refuse if today's total already exceeds the budget (post-hoc, not pre-hoc).

---

## Gap 3: Per-agent switch bypasses via wrappers / slash commands

**Problem:** `run_agent.sh` honors per-agent kill switches (`<AGENT_ID>_ENABLED=false`). But direct invocations of the wrappers — `bash ai-do.sh "prompt"` — and slash commands that invoke wrappers (e.g. `/morning_brief` calling `ai-do.sh` from inside its bash script) bypass the per-agent check entirely. Only `SYSTEM_PAUSED` covers these paths.

**Why deferred:** practical impact today is low — most agent invocations go through `run_agent.sh`, and the global kill switch covers the rest. But "defense in depth" matters once the fleet has more autonomous routines.

**Sketch of implementation:** wrappers could accept an `AGENT_ID` env var (they already do, since Step 10c) and consult `<AGENT_ID>_ENABLED` before firing. Slash commands and direct invocations would need to set `AGENT_ID` explicitly for the check to bite — which they already do for token_usage logging, so the plumbing is half-built.

---

## Gap 4: PUBLISHER_DEPLOY_ENABLED enforcement

**Problem:** `PUBLISHER_DEPLOY_ENABLED=false` is set in `.env` but no script reads it. No enforcement site exists.

**Why deferred:** This is a real Phase 2 concern — the switch only matters once there's a publisher agent actually capable of pushing content to the public internet. None of the current Phase 1 agents do that.

**Sketch of implementation:** when the writer/publisher agent ships, every deploy command (e.g. `git push` to a public-facing repo, `s3 cp` to a CDN, Netlify/Vercel deploy hook) must be gated by a check on this flag. Best embedded in a `~/agents/lib/deploy.sh` wrapper that becomes the only sanctioned path to public content.

---

## Status legend for future-you

When picking these up, the order I'd suggest:

1. **Gap 4** first — it's the cheapest to wire in (single switch check in a deploy wrapper) and it's the highest-stakes blast radius (public-facing content).
2. **Gap 1** next — once monitoring exists, you can detect the other gaps in production.
3. **Gap 2** when moving off Max sub.
4. **Gap 3** last — defense-in-depth, lowest urgency.
