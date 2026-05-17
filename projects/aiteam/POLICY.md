# AITEAM — OPERATOR POLICY
**Locked:** 2026-05-18
**Source:** Cross-agent failure modes audit §7 (`~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md`)
**Status:** Authoritative. Do not re-derive in conversation.

## Q1 — Auditor false-positive vs false-negative tolerance
**Policy:** False positives (bad content going live) are much worse than false negatives (good content blocked).
**Implication:** Gates lean strict. Accept losing ~5 good articles to prevent 1 bad one shipping. Editor agent FP/FN tuning when built (D056) should bias toward rejection.

## Q2 — Daily cost cap
**Policy:** $25/day soft warning + $50/day hard stop.
**Implementation:**
- At $25 Max-sub-equivalent burn: Telegram warning, drafter cron pauses (queue keeps building, no new pipeline fires).
- At $50: full hard stop. `check_kill_switches.sh` treats day-over-cap sentinel as `SYSTEM_PAUSED=true`. Operator manually clears.
- Both reckoned in estimated-as-if-API dollars (Max-sub value consumed, not real $).
- Tracked as F17/F18 in DEFERRED.md.

## Q3 — Bad-content rollback policy
**Policy:** Manual page-only rollback. No auto-strip of internal links or sitemap entries.
**Trigger to revisit:** After 3 incidents of bad content shipping, automate.
**Today:** When operator catches a bad slug, manually remove page + redeploy. Log incident for pattern-detection.

## Q4 — Burn-in publication count before reducing review
**Policy:** 4-20 clean slugs through the deploy throttle before any review-window reduction is considered.
**Implication:** The 20:00 preview → 23:00 batch deploy stays as-is for the burn-in. After 4-20 clean ships without operator catching problems, evaluate whether to shorten or remove the human review window.

## Q5 — Concurrent run-batch.sh
**Policy:** Serialize. One run-batch.sh at a time.
**Implementation:** Lockfile pattern. `run-batch.sh` at start checks `/tmp/run-batch-asbestos.lock` (or per-site equivalent); if present, exits with skip-reason logged to audit; if absent, creates lockfile, deletes on EXIT trap.
**Rationale:** Drafter cron is */30, queue rarely backs up; collision risk + Max-sub doubling > marginal throughput gain.

## Q6 — Watchdog agent
**Policy:** Build now (not deferred).
**Scope:** Monitors other agents — alive/dead checks, failure alerts, cron-fired-but-no-completion-row, audit-log anomalies. Spec exists in cross-agent audit §6.

## Q7 — Editor agent as pre-ship gate
**Policy:** Editor runs AFTER audit_guide.py, as a second gate. Both must pass before ship.
**Sequence:** mechanical (audit_guide.py, cheap) → editor (Sonnet, judgment, more $). Editor only runs on articles that already pass mechanical.
**Implication for D056:** Editor production runner persists verdict to auditor_verdicts table (composite_score column already reserved). pass_threshold becomes operator-tunable env var; FP-leaning default per Q1.

---

## Open derived work unblocked by these policies
- **D056** — editor verdict persistence (Q1 + Q4 + Q7 answered, now buildable)
- **F17/F18** — cost cap enforcement (Q2 answered, now buildable)
- **Watchdog agent** — spec exists; build (Q6 = build now)
- **Lockfile in run-batch.sh** — Q5 answered, ~10 lines bash
- **Bad-content incident log table** — Q3 answered, lightweight schema for future pattern detection

## Policy evolution
Reopen any of these only with explicit operator decision. Note in DEFERRED.md if a downstream build surfaces evidence a policy needs revision.

---
*End of policy.*
