# AITEAM Project Handoff
**Last updated:** 2026-05-16 19:20
**Last session summary:** Config Synthesizer agent built and committed (`69dd30f`) with `--validate-config-only` flag added to `audit_guide.py` (`e504793` in asbestos-contractors). 4 test configs synthesized + triaged + deleted. page.tsx template extracted from 3 real shipped wrappers; ship-to-site committed for the first time (`3b43c11`) including the provenance-strip tweak to `lib/stage.sh`. Cross-agent failure-modes audit landed at `~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md` documenting 23 failure modes across the now-autonomous asbestos chain.

---

## Current phase

**Phase 1, mid-phase. Chassis complete. Pipeline-wrap agents in active build.**

The asbestos pipeline went **fully autonomous on 2026-05-15** (assignment-drafter patch 2): operator edits `drafter_queue.txt` → ~30-90 minutes later a guide is live on `asbestoshq.com`. No human review gate in the loop. The Config Synthesizer (built 2026-05-16) closes the upstream input gap (keyword-configs) but is operator-invoked, not autonomous. The SSG fork remains in scaffold state — pipeline scripts byte-identical to asbestos, no SSG-specific inputs authored yet.

This session also surfaced **23 cross-agent failure modes** in a system-level audit; top-5 priority fixes are tracked in `DEFERRED.md`.

## Agents live

| Agent | Tier | State | Cron | Notes |
|---|---|---|---|---|
| orchestrator | sonnet (do) | live | 23:45 (diary) | War room routing + nightly diary. Tracked in `~/agents/` since `38815f6`. |
| librarian | haiku (cheap) | live | — | Inbox sort. Not currently in cron (`run_agent.sh` invocation only). Failure modes doc untracked. |
| editor | sonnet (do) | idle | — | Quality scorer with 5-axis rubric. Tier-test complete (r=0.933) but not slotted into production pipeline. Failure modes doc untracked. |
| market/scribe | bash | live | manual+cron-pending (D033) | Cowen + Casper transcript fetch. |
| market/analyst | sonnet (do) | live | manual+cron-pending (D033) | Per-video briefs. |
| market/curator | sonnet (do) | live | manual | 28 topics across 5 Cowen groups; Casper outlook-history. |
| market/briefer | sonnet (do, max-turns=1) | live | manual+cron-pending (D033) | Morning Telegram roll-up. Body-spacing live-verified on phone. |
| tg-monitor (reader) | python | live | */5 | 5 monitored groups + 1 archive user. `messages.db` + `archive.db`. |
| tg-monitor (analyzer) | haiku + sonnet | live | 0 7 | Daily 07:00 digest. Cost ~$0.08/run. |
| ship-to-site | bash (no LLM) | live | */15 | Pipeline approved → site stage + build + push. Committed for the first time this session in `3b43c11`. |
| config-synthesizer | sonnet (do) | live | operator-invoked | Synthesizes `keyword-configs/<slug>.json`. Committed in `69dd30f`. |
| assignment-drafter | sonnet (do) | live | */30 | Queue → assignment-batch markdown → fires `run-batch.sh` in background. Built per `assignment_drafter_build_2026-05-16.md` + `assignment_drafter_patch_2026-05-16.md`. Files in working tree of `~/agents/`, **NOT yet committed.** |

Operator-stack out-of-pipeline:
- `telegram/bot.js` — slash command dispatcher (`/halt_shipping`, `/halt_drafting`, etc.). Working tree, uncommitted.
- `dashboard/server.js` — observability + war room API at :3141. Modified, uncommitted.

## Production state

### Asbestos pipeline — fully autonomous
- **31 guides live** on `asbestoshq.com` (`APPROVED_GUIDE_SLUGS` count).
- **33 keyword-configs** in pipeline repo (one per shipped slug + 2 templates).
- **Autonomous chain (since 2026-05-15):** operator appends slug to `~/projects/asbestos-contractors/content/asbestos/drafter_queue.txt` → `assignment-drafter` */30 cron writes the assignment-batch + fires `run-batch.sh` in background (subject to `daily_pipeline_budget_usd: 15.00` cap, fixed estimate $2.50/fire) → pipeline writer/auditor/AuditKit produces `approved/guides/<slug>.json` → `ship-to-site` */15 cron stages + builds + pushes → GitHub Action `schema-check.yml` + Vercel auto-deploy → live within 30-90 min total.
- **Operator's only ongoing role:** daily digest review (07:00 ship-to-site + 07:00 tg-monitor). Plus operator must run `config-synthesizer` for any new slug *before* enqueuing in `drafter_queue.txt` — see DEFERRED F2 (highest-priority unfilled gap; no automatic pre-flight check yet).
- **Most-recent E2E:** `chrysotile-attic-insulation-removal` and `white-asbestos-vs-blue-asbestos` drafted via the autonomous chain (audit rows referenced in `assignment_drafter_patch_2026-05-16.md`).

### SSG pipeline — fork state, awaiting re-skin
- `~/projects/ssg-content/` is byte-identical to `asbestos-contractors/` in scripts and content-spec layer.
- No SSG `keyword-configs/`, no SSG `assignment-batch-*.md`, no SSG `audit-configs/guide.json` tuned to SSG vocabulary.
- `config-synthesizer/config/ssg.yaml` is `enabled: false`.
- `ship-to-site/config/ssg.yaml` is `enabled: false`.
- To unblock: re-skin `CONTENT_SPEC_GUIDE_SSG.md`, author SSG audit config, populate at least 3 SSG configs (so config-synthesizer in-context examples have fodder).

### Market scribe — manual-validation period (D033)
- Day 1 of 1-3 cron-activation gate began 2026-05-16. Daily manual run validates the full chain on fresh data; 1-3 clean days → flip to cron schedule.

### Brain vault
- `~/brain/` is a git repo (private `mdp280028-ui/brain-vault`). Daily auto-commit + push at 23:55 via `~/agents/scripts/brain-autocommit.sh`. Healthy.

## Open critical items

In rough priority order (full details in `DEFERRED.md`):

1. **F2 — drafter fires `run-batch.sh` for slugs with no keyword-config.** Highest-dollar cross-agent failure (~$1-5 wasted in writer/auditor per missing-config slug). One-line bash test in `assignment-drafter/lib/fire_pipeline.sh` blocks the fire. Not yet implemented.
2. **D044 (= cross-agent audit Top-5 #4) — `log_to_audit.sh` single-quote injection.** Silent failure: callers wrap with `|| true` so audit rows quietly disappear. Hit in config-synth build, defensively patched at caller; underlying lib still vulnerable.
3. **Cost cap reconciliation (F17/F18).** Three different caps in play: orchestrator's `DAILY_API_BUDGET_USD=10` (observability-only), drafter's `daily_pipeline_budget_usd=15.00` (enforced on fires only, fixed $2.50/fire estimate), no system-wide hard stop. Needs unification at `log_token_usage.sh` layer.
4. **F14 — pipeline auditor false positives reach live site.** No pre-ship operator review gate. Bad content has a clear path to production. Editor agent is idle and could slot in as a pre-ship score gate (small build).
5. **7 operator-policy questions in `cross_agent_failure_modes_2026-05-16.md` §7** — auditor false-positive/negative tolerance, daily API cap value, bad-content rollback policy, burn-in publication count, concurrent run-batch.sh policy, watchdog priority, editor-as-pre-ship-gate. All require operator decision before further hardening.
6. **`~/agents/` hygiene (D051).** Substantial work from prior sessions is uncommitted in the working tree: `assignment-drafter/` (full agent), `editor/failure_modes.md`, `librarian/failure_modes.md`, `orchestrator/{commands/,failure_modes.md}`, `telegram/`, `tg-monitor/`, modifications to `lib/{ai-*.sh, notify.sh, log_to_audit.sh, run_agent.sh}`, `dashboard/server.js`, `market/{analyst,briefer,curator}/`, `scripts/brain-autocommit.sh`. Cross-cutting commit, not a single-feature one.

## Sign-off discipline

A step is ✅ only when the build plan's pass condition has been *executed* and produced the expected result. Artifact-exists or process-started is not ✅ — that's ⚠ with a TODO. Honest ⚠ beats premature ✅. Verification cuts both ways: don't accept a ✅ claim from the operator without checking audit_log against the bar list. Trust the data, not the claim.

## Next session opening move

1. Read this file (`HANDOFF.md`).
2. Read `DEFERRED.md` for the live deferred items in priority order.
3. Read the most recent context save under `~/brain/projects/aiteam/context-saves/` for tactical state.
4. Then choose work based on operator's stated priority:
   - **Cheap and high-leverage:** F2 pre-flight check (one-line bash, blocks the worst cross-agent failure)
   - **Quick infra fix:** D044 / Top-5 #4 SQL escape in `log_to_audit.sh` (eliminates a silent-failure class)
   - **Operator-policy required:** the 7 questions in `cross_agent_failure_modes_2026-05-16.md` §7
   - **Hygiene:** D051 — commit `~/agents/` working tree
   - **Next agent in the stack:** Escalation Triage (small) or Failure Pattern Reporter (medium) per `ssg_pipeline_automation_recommendations_2026-05-16.md`

## Files to reference

- **Build reports (this week):**
  - `~/brain/projects/aiteam/docs/ssg_pipeline_recon_2026-05-16.md` — pipeline map
  - `~/brain/projects/aiteam/docs/ssg_pipeline_automation_recommendations_2026-05-16.md` — stack rank
  - `~/brain/projects/aiteam/docs/ship_to_site_build_2026-05-16.md`
  - `~/brain/projects/aiteam/docs/assignment_drafter_build_2026-05-16.md`
  - `~/brain/projects/aiteam/docs/assignment_drafter_patch_2026-05-16.md` — autonomous-fire patch
  - `~/brain/projects/aiteam/docs/config_synthesizer_build_2026-05-16.md`
  - `~/brain/projects/aiteam/docs/page_tsx_template_extraction_2026-05-16.md`
  - `~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md` — system-level audit, 23 modes, §7 has 7 open operator questions
  - `~/brain/projects/aiteam/docs/synth_configs_triage_2026-05-16.md` — synthesized-config decision template
- **Operational docs:**
  - `~/brain/projects/aiteam/PRIME.md` — north-star hard rules
  - `~/brain/projects/aiteam/LESSONS.md` — gotchas, schema quirks, bash 3.2 traps
  - `~/brain/projects/aiteam/docs/restore.md` — disaster recovery
  - `~/brain/projects/aiteam/docs/phase2_gaps.md` — Phase 2 outstanding items
- **Agents:**
  - `~/agents/config-synthesizer/` — built + committed (`69dd30f`)
  - `~/agents/ship-to-site/` — committed (`3b43c11`) with provenance-strip tweak
  - `~/agents/assignment-drafter/` — in working tree, not committed
  - `~/agents/{orchestrator,librarian,editor,market/*,tg-monitor,telegram,dashboard}/` — see git log in `~/agents/`

## Recent commits

| repo | sha | summary |
|---|---|---|
| ~/agents/ | `3b43c11` | ship-to-site: initial commit + strip template provenance before stage |
| ~/agents/ | `69dd30f` | Add config-synthesizer agent |
| ~/projects/asbestos-contractors/ | `e504793` | Add --validate-config-only flag to audit_guide.py |
| ~/agents/ | `8560861` | diary: step 19 closure + cron schedule fix |
| ~/agents/ | `df19fa3` | Market scribe session 1 steps 8-9 |
