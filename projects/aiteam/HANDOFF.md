# AITEAM Project Handoff
**Last updated:** 2026-05-17 00:30 (POLICY.md lock + brain bookkeeping; SSG-scaffolding + D044 cleanup summaries preserved below)
**Last session summary:** Brain-only bookkeeping session. Locked 7 operator-policy answers from cross-agent failure modes audit §7 into new file `~/brain/projects/aiteam/POLICY.md` (commit `6e7fb52`) — Q1 false-positives worse than false-negatives, Q2 $25/$50 daily cap, Q3 manual page-only rollback, Q4 4–20 burn-in ships, Q5 serialize run-batch.sh via lockfile, Q6 watchdog now, Q7 editor after audit_guide.py. POLICY.md unblocks D056, F17/F18, watchdog build. Closed D045 as obsolete after pre-flight showed writer/auditor already on Sonnet (`run-batch.sh:107-108`); opened successors D061 (route through ai-do.sh for kill-switch + token attribution — blocker is missing `--dangerously-skip-permissions` pass-through) and D062 (vestigial flag cleanup) in commit `9818e20`. Synced 10 D-items from sister chats into DEFERRED.md (D059 deploy throttle + D-SSG-01..09) in commit `b2a6fb0`. Added correction box to `cross_agent_failure_modes_2026-05-16.md` noting R1/R2/R3 tiering is planning-doc only with zero presence in code (commits `bcc16d5` + `38f77d0`). Doc-vs-reality audit Gaps 2 + 3 verified as zero-edit (active docs already correct). **Prior session summary (preserved):** Pipeline discovery + SSG content-pipeline scaffolding. Cloned 5 production repos from GitHub into `~/projects/`; built `~/projects/ssg-content/` as local-only fork of asbestos-contractors. Moved 101 asbestos content artifacts to `~/projects/_asbestos-reference/`. **D044 cleanup (preserved):** removed redundant `tr "'" '_'` scrub from `config-synthesizer/synth.sh#handle_failure()` after end-to-end verification confirmed shared-lib SQL-escape fix (commit 939361d) handles apostrophes. Single source of truth restored (commit c1aa9c0). Closed D045, D-SSG-05, D054.

---

## Current phase

**Phase 1, mid-phase. Chassis complete. Pipeline-wrap agents in active build. Throttled-deploy review window introduced as temporary scaffolding.**

The asbestos pipeline went **fully autonomous on 2026-05-15** but is now under a **23:00 PT batch-deploy throttle** introduced 2026-05-16 — operator edits `drafter_queue.txt` → drafter (`*/30`) produces brief → pipeline writer/auditor produces `approved/guides/<slug>.json` → guide sits in the ready queue until the next 23:00 PT fire. A 20:00 PT preview ping lists what's about to deploy so the operator can skip a slug by deleting its JSON. Throttle is reversible (D057) — designed to be flipped off once draft quality is consistent.

Verdict persistence is half live: `audit_guide.py` writes to `auditor_verdicts` per audit; editor is still stdout-only pending Q7 (D056).

23 cross-agent failure modes documented in `cross_agent_failure_modes_2026-05-16.md`; top-5 priority fixes tracked in `DEFERRED.md`. F2, D044/F23, and the D044 caller-layer cleanup all landed this session window (commits `0df8cf6`, `939361d`, `c1aa9c0`).

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
| ship-to-site preview ping | bash (no LLM) | live | **0 20 daily** | Reads `ship.sh --dry-run`, Telegrams the operator with what will deploy at 23:00 + word count + score + audit reasoning. |
| config-synthesizer | sonnet (do) | live | operator-invoked | Synthesizes `keyword-configs/<slug>.json`. Caller-layer SQL-escape scrub removed this window — shared-lib fix (939361d) verified end-to-end. |
| assignment-drafter | sonnet (do) | live | */30 | Queue → assignment-batch markdown → fires `run-batch.sh` in background. F2 pre-flight in place (`0df8cf6`). |

Operator-stack out-of-pipeline:
- `telegram/bot.js` — slash command dispatcher. Working tree, uncommitted.
- `dashboard/server.js` — observability + war room API at :3141. Modified, uncommitted.

## Production state

### Asbestos pipeline — fully autonomous, throttled-deploy
- **32 guides live** on `asbestoshq.com`.
- **33 keyword-configs** in pipeline repo.
- **Autonomous chain (since 2026-05-15, throttled 2026-05-16):** operator appends slug to `drafter_queue.txt` → `assignment-drafter` */30 writes the assignment-batch + fires `run-batch.sh` (with F2 pre-flight config check, `0df8cf6`) → pipeline writer/auditor produces `approved/guides/<slug>.json` → **slug sits in ready queue** → `preview_ping.sh` at 20:00 lists tonight's batch → operator has 3h to skip → `deploy_batch.sh` at 23:00 ships the batch via `ship.sh --slug X` per slug → Vercel auto-deploy.
- **Trade-off:** worst-case time-to-live went from ~30-90 min to up to 23h45m; median +12h. In exchange: 3h human review window per night. Reversal procedure in `deploy_throttle_build_2026-05-17.md`.
- **Verdict persistence:** `audit_guide.py` now writes one row to `auditor_verdicts` per audit (commit `0e8089b`); `audit_log` gets a matching `verdict_recorded` event for cross-join. Editor still stdout-only.
- **First throttled fire:** 2026-05-16 23:00:00, empty queue, `deploy_batch_empty` logged, no Telegram (as designed). Second empty fire logged tonight (audit row 1097). **Populated path still unverified — first non-empty 23:00 fire is the verification event.**

### SSG pipeline — scaffolded local-only, awaiting voice rewrite
- `~/projects/ssg-content/` exists as a local-only fork of asbestos-contractors. **No GitHub remote.** HEAD: `e12b122`.
- 3 commits this session: `aaabcaa` (initial cp -R + git init), `37c20f6` (content/asbestos→content/ssg rename + 97-file asbestos artifact cleanup), `e12b122` (4 material-named asbestos stragglers moved).
- Pipeline mechanics unchanged from asbestos source: `run-batch.sh` v10.13, `audit_guide.py` v3.4, `AUDIT_SPEC.md` v1.6.4, `AUDIT_APPENDICES.md`.
- 3 CONTENT_SPEC renames: `CONTENT_SPEC_GUIDE_SSG.md` (v2.7, **asbestos-flavored, needs rewrite** — AsbestosHQ.com persona + popcorn-ceiling/transite-pipe/black-mastic/vermiculite examples), `CONTENT_SPEC_SERVICE_SSG.md` (stub), `CONTENT_SPEC_SSG.md` (stub).
- 8 `claude -p` invocations in `run-batch.sh` all hardcode "AsbestosHQ.com" persona — lines 410 (skeleton), 444 (service), 474 (guide), 569 (default), 595 (revision), 847/881/972 (auditor passes 1–3). All 8 need SSG-target rewrite before pipeline can run for SSG.
- `~/projects/ssg-content/CLAUDE.md` root still titled "Asbestos Project Context" — needs SSG rewrite.
- `cc-context/05-REFERENCE/ported-from-ust-adapted/` retains 6 `ASBESTOS_*.md` strategy docs (Strategic_Prompts, Fake_Rotation_Strategy, Data_Processing_Playbook, Directory_Business_Context, State_Database_Research_Skill, Guide_Creation_Workflow). Operator hasn't decided if these are reusable templates or noise.
- `config-synthesizer/config/ssg.yaml` and `ship-to-site/config/ssg.yaml` both `enabled: false` (unchanged from prior state).
- **Decision to GitHub-publish ssg-content is deferred** until smartsourceguide deploy pipeline is understood.

### `_asbestos-reference/` (new this session)
- `~/projects/_asbestos-reference/` — 101 files moved out of ssg-content as preservation, not deletion.
- Contents: `approved-guides/` (34 JSONs), `keyword-configs/` (34 per-slug configs), 32 `assignment-batch-*.md` files (asbestos-prefix + material-named: black-mastic, chrysotile, transite-pipe, vermiculite), `Asbestos_Keyword_Research_Report_April2026.md`, 5 `ASBESTOS_*.md` reference files (Writer Reference, Templates, Output Format, Authority Links, Approved Index), 2 site-level files (SITE_MAP, TEMPLATES_ASBESTOS).
- Untracked working folder (no git). Queryable reference if SSG content adopts similar patterns.

### Site repos cloned this session (`~/projects/`)
- `UST-Contractors-/` — full pipeline + Next.js site, last commit 2026-05-14 ("decisions: log AITEAM audit fixes"). Pipeline versions: run-batch v10.11, audit_guide v3.2, AUDIT_SPEC **v1.6.5** (newest, includes SPC-16 UMBRELLA-KEYWORD-GUARD), CONTENT_SPEC_GUIDE v2.3, CONTENT_SPEC_SERVICE v1.7, spc-audit.py present.
- `asbestos-contractors/` — pipeline-only, last commit 2026-04-27. Pipeline versions: run-batch v10.13 (newest), audit_guide v3.4 (newest, includes G1 SEMANTIC-VARIETY), AUDIT_SPEC v1.6.4, CONTENT_SPEC_GUIDE v2.7 (newest), service/index stubs, no spc-audit.py.
- `asbestoshq-site/` — Next.js site target, last commit 2026-04-27.
- `payrolldetective/` — Next.js site only; pipeline lives elsewhere (synced AUDIT_SPEC.md header points to `~/Desktop/payroll-detective/`). Last commit 2026-04-26.
- `smartsourceguide/` — **bare Next.js site shell. 0 .md files, no CLAUDE.md, no specs, no keyword research.** Last commit 2026-04-10. 6 app routes across 3 categories: answering-services (5 subpages), fleet-tracking (5 subpages), it-support (4 subpages), plus about/best-answering-services/outsourced-it-support-cost orphans.
- **`payroll-detective` (hyphenated content repo)** — `git@github.com:mdp280028-ui/payroll-detective.git` → `Repository not found`. **Does not exist on GitHub.** Only at `~/Desktop/payroll-detective/` (TCC-blocked).

**UST↔asbestos pipeline fork state:** Neither is downstream of the other. UST ahead on AUDIT_SPEC (v1.6.5 with SPC-16) + service spec (v1.7) + spc-audit.py. Asbestos ahead on run-batch (v10.13) + audit_guide (v3.4 with G1 SEMANTIC-VARIETY) + guide spec (v2.7). Future "merge forward" needs cherry-picks per file.

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
3. ~~**7 operator-policy questions in `cross_agent_failure_modes_2026-05-16.md` §7**~~ — **LOCKED in `POLICY.md` (commit `6e7fb52`, 2026-05-17 00:30).** All 7 answered. Authoritative file: `~/brain/projects/aiteam/POLICY.md`. Do not re-derive in conversation. Downstream work now unblocked: D056 (editor verdict persistence — Q1+Q4+Q7), F17/F18 (cost cap — Q2), watchdog agent (Q6 = build now), run-batch.sh lockfile (Q5), bad-content incident log table (Q3).
4. **D061 / D062** — route `run-batch.sh` writer/auditor through `ai-do.sh` for kill-switch enforcement + per-call token attribution (D061), and drop the vestigial `--sonnet`/`--sonnet-audit` flags and `WRITER_MODEL`/`AUDITOR_MODEL` indirection (D062). Pair naturally — same callsites, ~30-45 min CC. D061 depends on F17/F18 OR per-call token attribution urgency.
5. **`~/agents/` hygiene (D051).** Most of last week's uncommitted tree landed during the prior session window via intermediate commits (`0df8cf6`, `939361d`, `3ed57dc`, `4e2fe0a`, `0be9d1f`, `c1aa9c0`). Spot-check residual: `telegram/bot.js`, `dashboard/server.js`, `market/{analyst,briefer,curator}/`, `scripts/brain-autocommit.sh`, several modifications to `lib/`. Cross-cutting commit needed.

**New SSG-track items this session (to be assigned D-numbers when promoted):**
- **SSG-1** — Decide SSG target domain + persona for the 8 `claude -p` prompts in ssg-content/content/run-batch.sh. Blocking everything else on the SSG track.
- **SSG-2** — Rewrite `CONTENT_SPEC_GUIDE_SSG.md` (367 lines, v2.7, asbestos-flavored) for SSG voice.
- **SSG-3** — Fill in stub specs `CONTENT_SPEC_SSG.md` + `CONTENT_SPEC_SERVICE_SSG.md`.
- **SSG-4** — Rewrite ssg-content root `CLAUDE.md` from asbestos to SSG context.
- **SSG-6** — Decide GitHub destination for ssg-content (or whether it stays local).
- **SSG-7** — Re-adapt or discard the 6 `ASBESTOS_*.md` strategy docs in `cc-context/05-REFERENCE/ported-from-ust-adapted/`.
- **SSG-8** — `~/Desktop/` TCC unblock decision: grant Full Disk Access to Terminal/Claude Code, or move all referenced folders out via Finder. Blocks any plan to consume Mandy's payroll pipeline as a comparison template.
- **SSG-9** — Decide if smartsourceguide repo (currently bare site shell, 0 docs/specs/keyword research) is the deploy target for ssg-content, or if a new site repo is planned.

**Resolved this session (no longer in priority list):**
- ~~D044 caller-layer redundancy in config-synthesizer/synth.sh.~~ Landed in `c1aa9c0`. Shared-lib fix (939361d) verified end-to-end with apostrophe-bearing payload — round-trip preserved verbatim through `jq --arg reason` and `SYNTH_LOG` echo. Single source of truth restored.
- ~~D045~~ — model swap defaults already applied; `action=deferred_closed` audit row 1098 records `path: B, defaults_changed: [WRITER_MODEL, AUDITOR_MODEL]`. DEFERRED.md row also annotated CLOSED-OBSOLETE this brain bookkeeping session (commit `9818e20`). NEW D061 supersedes the broader scope D045 was originally created for (routing run-batch.sh through ai-do.sh — blocker is missing `--dangerously-skip-permissions` pass-through). D062 covers vestigial `--sonnet`/`--sonnet-audit` flag cleanup.
- ~~D-SSG-05~~ — markdown-table URL regex in `ssg-content/content/run-batch.sh:~269`; audit row 1100.
- ~~D054~~ — cron PATH dependencies; `path: crontab-PATH-header`, `preserved: build_verify.sh per-script guard, untracked_scripts_separate: true`; audit row 1099. Belt-and-suspenders: crontab header sets PATH once for everything that goes through it; per-script guard in build_verify.sh stays because untracked scripts won't see the header decision.
- ~~7 operator-policy questions (cross-agent audit §7)~~ — all locked in `POLICY.md` (commit `6e7fb52`). See updated priority item #3 above.

## Sign-off discipline

A step is ✅ only when the build plan's pass condition has been *executed* and produced the expected result. Artifact-exists or process-started is not ✅ — that's ⚠ with a TODO. Honest ⚠ beats premature ✅. Verification cuts both ways: don't accept a ✅ claim from the operator without checking audit_log against the bar list. Trust the data, not the claim.

This session window: D044 caller-layer cleanup verified end-to-end with audit row 1101 (apostrophe-bearing reason `primary 'foo' must not appear in body` preserved verbatim through the patched lib). Three deferred items closed with payload-encoded decisions.

## Next session opening move

1. Read this file (`HANDOFF.md`).
2. Read `POLICY.md` — **authoritative for Q1–Q7**. Do not re-derive in conversation.
3. Read `DEFERRED.md` for the live deferred items (D056, D057, D058, D059, D061, D062 + D-SSG-01..09 are the newest).
4. Read the most recent context save: `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-17_0030.md`.
5. Check whether the 23:00 cron fired a populated batch overnight: `sqlite3 ~/store/aiteam.db "SELECT * FROM audit_log WHERE actor_id='ship-to-site' AND action IN ('deployed','deploy_failed','deploy_batch_complete') ORDER BY ts DESC LIMIT 10;"` — if any rows landed, the populated path of `deploy_batch.sh` was verified by the real fire. If not, queue was still empty and the verification event is still pending.
6. Then choose work based on operator priority (all now unblocked by POLICY.md):
   - **Watchdog agent (Q6 = build now):** spec in `cross_agent_failure_modes_2026-05-16.md` §6. No upstream blockers.
   - **F17/F18 cost cap (Q2 answered = $25 soft / $50 hard):** system-wide hard stop at `log_token_usage.sh`, sentinel for `check_kill_switches.sh`.
   - **D056 editor verdict persistence (Q1 + Q4 + Q7 answered):** editor runs AFTER `audit_guide.py` as second gate; FP-leaning default per Q1; persist to `auditor_verdicts` (composite_score column already reserved); `pass_threshold` env var.
   - **`run-batch.sh` lockfile (Q5 = serialize):** ~10 lines bash. Check `/tmp/run-batch-asbestos.lock` at start; trap-cleanup on EXIT; log skip to audit if locked.
   - **D061/D062 paired:** route writer/auditor through ai-do.sh + drop vestigial flags. Trigger: after F17/F18 ships OR per-call token attribution becomes urgent. Requires `ai-do.sh` to accept `--dangerously-skip-permissions` pass-through first (else cron pipeline hangs).
   - **Partial-coverage triage build:** can query `auditor_verdicts WHERE scorer='audit_guide' AND triaged_at IS NULL` for real failing-audit candidates.
   - **Hygiene:** D051 — commit residual `~/agents/` working tree.

## Files to reference

- **Build reports (most recent):**
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
  - `~/brain/projects/aiteam/POLICY.md` — **authoritative Q1–Q7 operator policy answers (locked 2026-05-18 in commit `6e7fb52`)**
  - `~/brain/projects/aiteam/LESSONS.md` — gotchas, schema quirks, bash 3.2 traps (7 new entries this session window, on POLICY.md lock + brain bookkeeping)
  - `~/brain/projects/aiteam/DEFERRED.md` — deferred work, by D-number (D045 now closed-obsolete; D059, D061, D062, D-SSG-01..09 newest)
  - `~/brain/projects/aiteam/docs/restore.md` — disaster recovery
- **New scripts (prior session window):**
  - `~/agents/ship-to-site/preview_ping.sh` — 20:00 ping; queue from `ship.sh --dry-run`
  - `~/agents/ship-to-site/deploy_batch.sh` — 23:00 fire; wraps `ship.sh --slug X` per slug
  - `~/agents/lib/migrations/2026-05-17_auditor_verdicts.sql` — schema migration (idempotent)

## Recent commits

| repo | sha | summary |
|---|---|---|
| ~/brain/ | `6e7fb52` | POLICY.md: lock 7 operator-policy answers from cross-agent audit §7 (Q1 FP-strict, Q2 $25/$50, Q3 manual rollback, Q4 4–20 burn-in, Q5 serialize, Q6 watchdog now, Q7 editor after audit) |
| ~/brain/ | `9818e20` | DEFERRED: close D045 obsolete; open D061 (ai-do.sh rewire for kill-switch + attribution) + D062 (vestigial flag cleanup) |
| ~/brain/ | `38f77d0` | doc-vs-reality: correction box on cross_agent_failure_modes_2026-05-16.md (R1/R2/R3 tiering is planning-doc only) |
| ~/brain/ | `bcc16d5` | add cross_agent_failure_modes_2026-05-16.md (was untracked) |
| ~/brain/ | `b2a6fb0` | DEFERRED: sync D059 + D-SSG-01..09 from sister chats |
| ~/projects/ssg-content/ | `e12b122` | cleanup: move 4 asbestos assignment stragglers (black-mastic, chrysotile, transite-pipe, vermiculite) to reference folder |
| ~/projects/ssg-content/ | `37c20f6` | cleanup: moved asbestos content artifacts to ~/projects/_asbestos-reference/ (incl. content/asbestos→content/ssg rename + 3 CONTENT_SPEC_*_ASBESTOS→*_SSG renames) |
| ~/projects/ssg-content/ | `aaabcaa` | Initial: cloned from asbestos-contractors as SSG pipeline template |
| ~/agents/ | `c1aa9c0` | D044 cleanup: remove redundant caller-layer escape in config-synthesizer/synth.sh (F4 fixed at shared lib, commit 939361d) |
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
- D044 fix is now SHARED-LIB ONLY (commit 939361d). Caller-side scrubs have been removed (commit c1aa9c0 for config-synthesizer). If you find a `tr "'" '_'` or `sed "s/'/''/g"` defensive wrapper at any other caller, it's also redundant — strip it after end-to-end verification with an apostrophe-bearing test payload.
- **POLICY.md is authoritative for Q1–Q7.** Do not re-derive in conversation. Reopening any of them requires an explicit operator decision noted in DEFERRED.md. Locked 2026-05-18 (commit `6e7fb52`).
- **`ai-do.sh` is NOT a drop-in `claude -p` replacement for non-interactive (cron-fired) callers.** Three blocker differences: no `--dangerously-skip-permissions` pass-through (cron pipeline hangs on first tool-use permission prompt), hardcoded `--max-turns ${AGENT_MAX_TURNS}` (currently 30; was unlimited at callsite), forced `--output-format json` + Python extraction of `.result` (changes stdout shape). Any rewire needs a per-callsite flag audit. See D061 notes.
- **`ctx.sh` mechanical record only covers `~/agents/`, not `~/brain/`.** Brain-only sessions produce blank mechanical records ("no commits since last save"). For brain-only sessions, the narrative section is the entire signal — empty mechanical block ≠ "nothing happened".
- **DEFERRED.md D-number collisions are silent.** Pre-grep `grep -nE "^### D[0-9]+|^\| \*\*D[0-9]+" DEFERRED.md` (covers detail sections AND Priority view table) before assigning any new D-number. The file already has duplicate D045, D051, D027 as evidence of past collisions.
- **Untracked files in `~/brain` trap commit messages.** Before committing a "small fix" to a file, run `git ls-files --error-unmatch <path>` — if untracked, a one-line edit becomes a multi-hundred-line `git add`. Either split into add-file + apply-fix commits, or update the commit message to honestly describe the addition.
- **macOS Desktop is TCC-blocked for terminal apps.** Any CLAUDE.md that references `~/Desktop/<project>/` (UST → `~/Desktop/CC UST FILES/`, asbestos → `~/Desktop/CC ASBESTOS FILES/`, payrolldetective → `~/Desktop/payroll-detective/`, AITEAM → `~/Desktop/AITEAM FILES/`) is pointing to a path CC cannot read without Full Disk Access. `mdfind` (Spotlight) is the TCC-bypass for existence checks but cannot read content. Solutions: grant FDA, or move folders out of Desktop via Finder.
- **The hyphenated `payroll-detective` content repo does not exist on GitHub** despite being referenced as "Source of truth" in `payrolldetective/CLAUDE.md`. Lives only at `~/Desktop/payroll-detective/` (TCC-blocked).
- **UST and asbestos pipelines have forked.** Neither is downstream of the other. Cherry-pick if you want best-of-both; do not rebase.
- **`smartsourceguide` is a bare site shell.** 0 .md files, no CLAUDE.md, no specs, no keyword research, last touched 2026-04-10. SSG content production starts from zero domain reference material.
- **`gh` (GitHub CLI) is not installed.** `brew install gh && gh auth login` to unblock repo enumeration/clone workflows.
- **`~/agents/_template/` is empty** (just `.gitkeep`). Not a usable scaffold. New agents currently hand-cloned from `editor/` or `librarian/`.
- **`~/projects/ssg-content/` has no git remote** (deliberate). Do not `git push` blindly — there is no remote to push to.
