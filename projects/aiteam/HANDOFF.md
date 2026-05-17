# AITEAM Project Handoff
**Last updated:** 2026-05-16 23:15 (touchup; full update by operator at 23:10)
**Last session summary:** Built the verdict-persistence foundation (`auditor_verdicts` table + audit_guide.py persistence — commits `4e2fe0a` in `~/agents/` and `0e8089b` in asbestos-contractors) and the deploy throttle (`preview_ping.sh` at 20:00 PT + `deploy_batch.sh` at 23:00 PT — commit `3ed57dc`). Escalation Triage agent could not be built directly because its assumed upstream (`audit_log.tier='R3'` schema, drafter→auditor R-tier pipeline) doesn't exist; verdict persistence is the foundation that unblocks a partial triage build later. Editor verdict persistence deferred (D056) pending production runner + threshold + Q7 decision. First throttled 23:00 cron fire executed tonight against an empty queue and logged `deploy_batch_empty` as designed.

---

## Current phase

**Phase 1, mid-phase. Chassis complete. Pipeline-wrap agents in active build. Throttled-deploy review window introduced as temporary scaffolding.**

The asbestos pipeline went **fully autonomous on 2026-05-15** but is now under a **23:00 PT batch-deploy throttle** introduced 2026-05-16 — operator edits `drafter_queue.txt` → drafter (`*/30`) produces brief → pipeline writer/auditor produces `approved/guides/<slug>.json` → guide sits in the ready queue until the next 23:00 PT fire. A 20:00 PT preview ping lists what's about to deploy so the operator can skip a slug by deleting its JSON. Throttle is reversible (D057) — designed to be flipped off once draft quality is consistent.

Verdict persistence is half live: `audit_guide.py` writes to `auditor_verdicts` per audit; editor is still stdout-only pending Q7 (D056).

23 cross-agent failure modes documented in `cross_agent_failure_modes_2026-05-16.md`; top-5 priority fixes tracked in `DEFERRED.md`. F2 and D044/F23 both landed this session window (commits `0df8cf6` and `939361d`).

## Agents live

| Agent | Tier | State | Cron | Notes |
|---|---|---|---|---|
| orchestrator | sonnet (do) | live | 23:45 (diary) | War room routing + nightly diary. |
| librarian | haiku (cheap) | live | — | Inbox sort. Not currently in cron. |
| editor | sonnet (do) | idle | — | Quality scorer with 5-axis rubric. Tier-test complete (r=0.933). NOT slotted into production pipeline. Verdict persistence deferred (D056). |
| market/scribe | bash | live | manual+cron-pending (D033) | Cowen + Casper transcript fetch. |
| market/analyst | sonnet (do) | live | manual+cron-pending (D033) | Per-video briefs. |
| market/curator | sonnet (do) | live | manual | 28 topics across 5 Cowen groups; Casper outlook-history. |
| market/briefer | sonnet (do, max-turns=1) | live | manual+cron-pending (D033) | Morning Telegram roll-up. |
| tg-monitor (reader) | python | live | */5 | 5 monitored groups + 1 archive user. |
| tg-monitor (analyzer) | haiku + sonnet | live | 0 7 | Daily 07:00 digest. Cost ~$0.08/run. |
| ship-to-site | bash (no LLM) | live (throttled) | **0 23 daily** (was */15; old line preserved as `# DISABLED 2026-05-17 …` comment) | Pipeline approved → site stage + build + push. Now invoked via `deploy_batch.sh` once per night, not per 15 min. |
| ship-to-site preview ping | bash (no LLM) | live | **0 20 daily** | New this session. Reads `ship.sh --dry-run`, Telegrams the operator with what will deploy at 23:00 + word count + score + audit reasoning. |
| config-synthesizer | sonnet (do) | live | operator-invoked | Synthesizes `keyword-configs/<slug>.json`. |
| assignment-drafter | sonnet (do) | live | */30 | Queue → assignment-batch markdown → fires `run-batch.sh` in background. **Now committed** in this session window via F2 fix (`0df8cf6`). |

Operator-stack out-of-pipeline:
- `telegram/bot.js` — slash command dispatcher. Working tree, uncommitted.
- `dashboard/server.js` — observability + war room API at :3141. Modified, uncommitted.

## Production state

### Asbestos pipeline — fully autonomous, throttled-deploy
- **32 guides live** on `asbestoshq.com` (was 31; `white-asbestos-vs-blue-asbestos` shipped via manual `ship.sh --slug` retry at 2026-05-16 23:11 UTC after the ship-to-site cron PATH fix + `npm install` in the site repo — HTTP 200 verified; audit row 868).
- **33 keyword-configs** in pipeline repo.
- **Autonomous chain (since 2026-05-15, throttled 2026-05-16):** operator appends slug to `drafter_queue.txt` → `assignment-drafter` */30 writes the assignment-batch + fires `run-batch.sh` (with F2 pre-flight config check, `0df8cf6`) → pipeline writer/auditor produces `approved/guides/<slug>.json` → **slug sits in ready queue** → `preview_ping.sh` at 20:00 lists tonight's batch → operator has 3h to skip → `deploy_batch.sh` at 23:00 ships the batch via `ship.sh --slug X` per slug → Vercel auto-deploy.
- **Trade-off:** worst-case time-to-live went from ~30-90 min to up to 23h45m; median +12h. In exchange: 3h human review window per night. Reversal procedure in `deploy_throttle_build_2026-05-17.md`.
- **Verdict persistence:** `audit_guide.py` now writes one row to `auditor_verdicts` per audit (commit `0e8089b`); `audit_log` gets a matching `verdict_recorded` event for cross-join. Editor still stdout-only.
- **First throttled fire:** 2026-05-16 23:00:00, empty queue, `deploy_batch_empty` logged, no Telegram (as designed).

### SSG pipeline — fork state, awaiting re-skin
- `~/projects/ssg-content/` is byte-identical to `asbestos-contractors/` in scripts and content-spec layer.
- No SSG-specific keyword-configs, assignment-batches, or audit-configs yet.
- `config-synthesizer/config/ssg.yaml` and `ship-to-site/config/ssg.yaml` both `enabled: false`.
- To unblock: re-skin `CONTENT_SPEC_GUIDE_SSG.md`, author SSG audit config, populate at least 3 SSG configs.

### Market scribe — manual-validation period (D033)
- Day 2 of 1-3 cron-activation gate (started 2026-05-16). Continue daily manual runs; clean days → flip to cron.

### Brain vault
- `~/brain/` is a git repo. Daily auto-commit + push at 23:55 via `~/agents/scripts/brain-autocommit.sh`. Healthy.

## Open critical items

In rough priority order (full details in `DEFERRED.md`):

0. **Verdict persistence layer is HALF live — `audit_guide.py` persists, editor does not.** Triage agent can be partially built against `audit_guide` verdicts (`auditor_verdicts WHERE scorer='audit_guide' AND triaged_at IS NULL`); full coverage needs editor production model decided. Three pre-reqs to ship editor half: production runner, pass threshold, wiring decision (= Q7). Tracked as D056. Build report: `verdict_persistence_build_2026-05-17.md`.
0a. **Deploy throttle is temporary scaffolding. Reverse when quality is consistent.** Ship-to-site swapped from `*/15` cadence to a single 23:00 PT batch deploy with a 20:00 PT preview ping (commit `3ed57dc`). Original cron line preserved as commented `# DISABLED 2026-05-17 …` for trivial reversal. Reversal procedure: `deploy_throttle_build_2026-05-17.md` §"Reversal procedure". Tracked as D057 (removal trigger); preview-URL upgrade tracked as D058. Trade: time-to-indexed median +12h in exchange for a 3h human review window. **First populated 23:00 fire is the verification event for the deploy_batch populated path** (operator-deferred from test 2 to avoid staging test data into production).
1. **Cost cap reconciliation (F17/F18).** Three different caps in play: orchestrator's `DAILY_API_BUDGET_USD=10` (observability-only), drafter's `daily_pipeline_budget_usd=15.00` (enforced on fires only, fixed $2.50/fire estimate), no system-wide hard stop. Needs unification at `log_token_usage.sh` layer.
2. **F14 — pipeline auditor false positives reach live site.** No pre-ship operator approval gate. Bad content has a clear path to production. The deploy throttle (item 0a) is partial mitigation (3h review window) but no formal approval mechanism — operator must remember to delete bad slugs before 23:00. Editor agent is idle and could slot in as a pre-ship score gate (small build, blocked on D056 + Q7).
3. **7 operator-policy questions in `cross_agent_failure_modes_2026-05-16.md` §7** — Q1: auditor false-positive/negative tolerance; Q2: daily API cap value; Q3: bad-content rollback policy; Q4: burn-in publication count; Q5: concurrent run-batch.sh policy; Q6: watchdog priority; Q7: editor-as-pre-ship-gate (also blocks D056). All require operator decision before further hardening.
4. **`~/agents/` hygiene (D051).** Most of last week's uncommitted tree landed during this session window via intermediate commits (`0df8cf6`, `939361d`, `3ed57dc`, `4e2fe0a`, `0be9d1f`). Spot-check residual: `telegram/bot.js`, `dashboard/server.js`, `market/{analyst,briefer,curator}/`, `scripts/brain-autocommit.sh`, several modifications to `lib/`. Cross-cutting commit needed.

**Resolved this session (no longer in priority list):**
- ~~F2 — drafter pre-flight keyword-config check.~~ Landed in `0df8cf6`.
- ~~D044 / Top-5 #4 — `log_to_audit.sh` SQL escape.~~ Landed in `939361d`. F4 regression test exercised tonight in audit_guide persistence build (apostrophe slugs/reasoning preserved cleanly through both Python sqlite3 + the patched bash escape).
- ~~Ship-to-site cron PATH bug.~~ Landed in `0be9d1f`. Surfaced and fixed within the same session; was blocking every cron-driven ship since the agent landed.

**New deferred items added this session (D052-D055), full details in `DEFERRED.md`:**
- **D052** — drafter.sh exit-2 vs exit-1 routing (UX cleanup for F2's double-row audit logging when a slug skips for missing config)
- **D053** — assignment-drafter `state/` files not gitignored (F2 commit swept `daily_pipeline_spend.txt` in)
- **D054** — audit all cron-invoked scripts for PATH dependencies (systemic version of `build_verify.sh` fix)
- **D055** — site repo `node_modules/` disappeared between manual shipping and autonomous retry; root cause unknown, deps reinstalled this session

## Sign-off discipline

A step is ✅ only when the build plan's pass condition has been *executed* and produced the expected result. Artifact-exists or process-started is not ✅ — that's ⚠ with a TODO. Honest ⚠ beats premature ✅. Verification cuts both ways: don't accept a ✅ claim from the operator without checking audit_log against the bar list. Trust the data, not the claim.

This session: 3 of 4 testable test cases passed end-to-end (verdict persistence: 3/3; deploy throttle: 1/2 — populated deploy_batch deferred to first real 23:00 fire per operator direction). Empty-queue paths for both new scripts verified live: `preview_ping_sent` audit row id=1031, Telegram delivered (http 200, 38 bytes); `deploy_batch_empty` audit rows id=1064 and id=1097.

## Next session opening move

1. Read this file (`HANDOFF.md`).
2. Read `DEFERRED.md` for the live deferred items (D056, D057, D058 are the newest).
3. Read the most recent context save: `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-16_2307.md`.
4. Check whether the 23:00 cron fired a populated batch overnight: `sqlite3 ~/store/aiteam.db "SELECT * FROM audit_log WHERE actor_id='ship-to-site' AND action IN ('deployed','deploy_failed','deploy_batch_complete') ORDER BY ts DESC LIMIT 10;"` — if any rows landed, the populated path of `deploy_batch.sh` was verified by the real fire. If not, queue was still empty and the verification event is still pending.
5. Then choose work based on operator priority:
   - **Operator-policy required (high unblock value):** the 7 questions in `cross_agent_failure_modes_2026-05-16.md` §7, especially Q7 which gates D056 + F14 mitigation.
   - **Cost cap reconciliation (F17/F18):** system-wide hard stop at `log_token_usage.sh`.
   - **Partial-coverage triage build:** can now query `auditor_verdicts WHERE scorer='audit_guide' AND triaged_at IS NULL` for real failing-audit candidates.
   - **Hygiene:** D051 — commit the residual `~/agents/` working tree (smaller pile after this session).

## Files to reference

- **Build reports (new this session):**
  - `~/brain/projects/aiteam/docs/verdict_persistence_build_2026-05-17.md` — auditor_verdicts schema + audit_guide.py persistence; editor half deferred with explicit pre-reqs
  - `~/brain/projects/aiteam/docs/deploy_throttle_build_2026-05-17.md` — 20:00 preview + 23:00 batch deploy; crontab diff + reversal procedure
- **Build reports (prior week):**
  - `~/brain/projects/aiteam/docs/ssg_pipeline_recon_2026-05-16.md` — pipeline map
  - `~/brain/projects/aiteam/docs/ssg_pipeline_automation_recommendations_2026-05-16.md` — stack rank
  - `~/brain/projects/aiteam/docs/ship_to_site_build_2026-05-16.md`
  - `~/brain/projects/aiteam/docs/assignment_drafter_build_2026-05-16.md` + `..._patch_2026-05-16.md`
  - `~/brain/projects/aiteam/docs/config_synthesizer_build_2026-05-16.md`
  - `~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md` — 23 failure modes, §7 has 7 open operator questions
- **Operational docs:**
  - `~/brain/projects/aiteam/PRIME.md` — north-star hard rules
  - `~/brain/projects/aiteam/LESSONS.md` — gotchas, schema quirks, bash 3.2 traps (10 new entries this session)
  - `~/brain/projects/aiteam/DEFERRED.md` — deferred work, by D-number
  - `~/brain/projects/aiteam/docs/restore.md` — disaster recovery
- **New scripts:**
  - `~/agents/ship-to-site/preview_ping.sh` — 20:00 ping; queue from `ship.sh --dry-run`
  - `~/agents/ship-to-site/deploy_batch.sh` — 23:00 fire; wraps `ship.sh --slug X` per slug
  - `~/agents/lib/migrations/2026-05-17_auditor_verdicts.sql` — schema migration (idempotent)
- **Modified files:**
  - `~/projects/asbestos-contractors/scripts/audit_guide.py` — added `persist_verdict()` helper (additive; existing stdout output unchanged)

## Recent commits

| repo | sha | summary |
|---|---|---|
| ~/agents/ | `3ed57dc` | feat(ship-to-site): throttle to 23:00 daily deploy + 20:00 preview ping |
| ~/agents/ | `0be9d1f` | Fix ship-to-site cron npm PATH bug |
| ~/agents/ | `4e2fe0a` | feat(lib/migrations): auditor_verdicts table |
| ~/projects/asbestos-contractors/ | `0e8089b` | feat(audit_guide): persist mechanical-check verdicts + emit audit_log event |
| ~/agents/ | `0df8cf6` | Fix F2: pre-flight keyword-config check in fire_pipeline.sh |
| ~/agents/ | `939361d` | Fix F4/D044: SQL-escape single quotes in log_to_audit.sh |
| ~/agents/ | `3b43c11` | ship-to-site: initial commit + strip template provenance before stage |
| ~/agents/ | `69dd30f` | Add config-synthesizer agent |

## Gotchas for future me

- `ship.sh` always exits 0 even when per-slug deploys fail (`process_site` swallows failures with `|| true`). Wrappers that need per-slug success status must parse stdout for `[ship] <slug>: shipped` patterns. `deploy_batch.sh` documents this in its header.
- `log_to_audit.sh` takes POSITIONAL args (`actor_type actor_id action target payload [corr_id]`), NOT `--flag` style. Specs that show `--flag` syntax are wrong — use positional.
- `audit_guide.py` writes `composite_score=NULL` (mechanical checks, no composite). Display as em-dash, NOT 0 — 0 would imply a real low score.
- Approved-guide JSONs store body in `.paragraphs[]` (array), NOT `.allText` (which exists in schema but is empty string). Word count: `jq -r '.paragraphs | join(" ")' "$f" | wc -w`.
- Vercel, not Cloudflare. Deploy = `git push` to `main`. No `vercel.json`, no `.vercel/`, no preview URLs captured by the pipeline.
- DEFERRED.md D-numbers collide silently — grep before adding (`grep -nE "^### D[0-9]+" DEFERRED.md | tail`).
- Crontab snapshot pre-throttle saved at `/tmp/crontab-snapshot-pre-throttle-1778996892.txt` (will be lost on reboot/tmp wipe; full content also in `deploy_throttle_build_2026-05-17.md`).
