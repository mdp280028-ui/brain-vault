# AITEAM — Deferred items

Engineering decisions we've consciously kicked down the road. Each item names a trigger ("when to revisit") so they don't rot quietly.

**Last refresh:** 2026-05-16. Consolidates D-items previously scattered across HANDOFF.md and per-session context-saves; adds new items from this session; cross-references the 23 cross-agent failure modes documented in `~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md` (F-prefix IDs).

---

## Priority view

Items in roughly the order they should be picked up next.

| ID | Item | Trigger | Notes |
|---|---|---|---|
| **F2** | Drafter pre-flight: refuse to fire `run-batch.sh` for slugs with no `keyword-configs/<slug>.json` | Before the next time operator enqueues a slug without first running config-synthesizer | One-line bash test in `~/agents/assignment-drafter/lib/fire_pipeline.sh`. Highest-dollar cross-agent failure (~$1-5 wasted per missing-config slug). Cross-agent audit Top-5 #1. |
| **D044** | `log_to_audit.sh` SQL-escape on apostrophes | Next infra-touching session | **UPGRADED to live bug per cross-agent audit (Top-5 #4 = F23).** Already hit in config-synth, defensively patched at caller with `tr "'" '_'`. Underlying lib still vulnerable; every agent's audit_log writes silently drop when payload contains `'`. Fix: parameterize via `python3 -c` or use sqlite3 `.parameter set`. |
| **F17/F18** | Daily API spend hard cap enforced at `log_token_usage.sh` | After operator answers question §7.2 in cross-agent audit (what's the right number?) | Three caps currently in play: orchestrator's $10 (observability-only), drafter's $15 (pipeline-fire only, fixed $2.50 estimate), no system-wide. Fix: after every `INSERT` into `token_usage`, query today's sum; if > cap, touch a sentinel that `check_kill_switches.sh` treats as `SYSTEM_PAUSED=true`. Cross-agent audit Top-5 #2. |
| **F14** | Pre-ship operator approval gate (first N slugs after any config change) | Before next net-new SSG slug ships OR after a bad page lands on live | Auditor false positives have a clear path to live site. Editor agent (sonnet, idle) could slot in as the pre-ship score gate. Operator policy decision needed first (cross-agent audit §7.1 and §7.4). |
| **D051** | `~/agents/` working-tree hygiene | When operator has bandwidth for a cross-cutting cleanup commit | Substantial work uncommitted: full `assignment-drafter/` agent, `editor/failure_modes.md`, `librarian/failure_modes.md`, `orchestrator/{commands/,failure_modes.md}`, `telegram/`, `tg-monitor/`, modifications to `lib/{ai-*.sh, notify.sh, log_to_audit.sh, run_agent.sh}`, `dashboard/server.js`, `market/{analyst,briefer,curator}/`, `scripts/brain-autocommit.sh`. NOT a single-feature commit — needs grouping into multiple coherent commits. |
| **D061** ✅ CLOSED 2026-05-17 | Route `run-batch.sh` writer/auditor through `ai-do.sh` for kill-switch enforcement + per-call token attribution | ✅ — ai-do.sh `5d68876` (AI_DO_SKIP_PERMISSIONS env hook), asbestos `4ea5242`, ssg `1864971`. Audit row id=1154 (`0381734F-2D7F-4267-A72F-4ADBD146C4B1`). | Sub-decision A: option (a) — added `AI_DO_SKIP_PERMISSIONS=1` env-hook in ai-do.sh (opt-in --dangerously-skip-permissions pass-through). Sub-decision B: no override needed — default `AGENT_MAX_TURNS=30` covers expected 10-20-turn writer/auditor calls; escape hatch via `AGENT_MAX_TURNS_OVERRIDE` already present in ai-do.sh. 16 callsites replaced (8 per repo). Smoke-tested: SYSTEM_PAUSED env-flip causes ai-do.sh to exit 1 before claude invocation, no token_usage row landed. |
| **D062** ✅ CLOSED 2026-05-17 | Remove vestigial `--sonnet` / `--sonnet-audit` CLI flags + `WRITER_MODEL`/`AUDITOR_MODEL` indirection in `run-batch.sh` | ✅ — asbestos `7166c7b`, ssg `c71a54c`. Audit row id=1155 (`0F97332C-5E93-4821-B61E-006C3160CEF1`). Note: CLI flag parsing already removed by cluster #9 (`082e957` asbestos, `56a86e0` ssg); this commit removed only the leftover variable indirection. | Lines 105-106 (var assignments) replaced with doc comment. Lines 383 + 1330 (asbestos) / 386 + 1347 (ssg) banner echoes hardcoded to `"Writer: Sonnet (via ai-do.sh) \| Auditor: Sonnet (via ai-do.sh)"`. |

---

## Live bugs (fix order matters)

### F2 — Drafter fires `run-batch.sh` without a keyword-config

**Status:** Open (Phase 1, cross-agent audit 2026-05-16)

**Problem:** `assignment-drafter`'s `maybe_fire_pipeline` doesn't check whether `<pipeline_repo>/content/asbestos/keyword-configs/<slug>.json` exists before launching `nohup run-batch.sh`. If config is missing, pipeline either pre-flights MISSING_CONFIG (cheap fail) or — worse — runs writer/auditor against a wrong/stale config and burns ~$1-5 in Opus across 3 rounds before landing in `needs-review/` with no actionable diagnosis. Drafter has already exited (`nohup` decouples), so the failure is invisible to drafter logs.

**Fix:** In `~/agents/assignment-drafter/lib/fire_pipeline.sh`, before `nohup run-batch.sh`:
```
[ -f "${pipeline_repo}/content/asbestos/keyword-configs/${slug}.json" ] || {
  echo "FATAL: no keyword-config for ${slug}" >&2
  exit 1
}
```
Drafter already buckets fire failures into `FIRE_FAILED_SLUGS` and surfaces them in Telegram; this just teaches the gate to fire earlier.

**Trigger to revisit:** Before the next time operator enqueues a net-new slug. Operator must currently remember to run config-synth before enqueueing.

**Reference:** `cross_agent_failure_modes_2026-05-16.md` F2 + Top-5 #1.

### D044 — `log_to_audit.sh` breaks on apostrophes in payload

**Status:** Open, **UPGRADED to live bug** (Phase 1, 2026-05-15 → 2026-05-16)

**Problem:** `~/agents/lib/log_to_audit.sh` builds SQL via shell heredoc with `${PAYLOAD}` expanded inline. A single `'` anywhere in the payload terminates the SQL string and the INSERT aborts. Most callers wrap the call with `|| true` so the failure is silent — the audit row simply never appears.

**Discovered:** Config-synthesizer build (2026-05-16). Python validate.py's `f"{cfg['primary']!r}"` produced messages like `primary 'foo' must not appear` which broke the audit_log write. Defensive scrub `tr "'" '_'` added at config-synth's `handle_failure` call site, but other callers (drafter, ship-to-site, librarian) are still vulnerable to any payload containing user-controlled strings with quotes.

**Fix:** Rewrite `log_to_audit.sh` to use sqlite3 parameterized inserts. Cleanest: pipe payload through `python3 -c` which uses `sqlite3.connect(...).execute(sql, params)`. ~20 lines, no API change for callers, every agent benefits.

**Trigger to revisit:** Next infra-touching session. Pairs naturally with any other lib/ work.

**Workaround in the meantime:** Caller-side `sed "s/'/''/g"` or `tr "'" '_'` before passing payload.

**Reference:** `cross_agent_failure_modes_2026-05-16.md` F23 + Top-5 #4.

---

## Infrastructure

### D025 — Orchestrator permission policy for Telegram-spawned sessions

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** Non-interactive `claude -p` sessions (Telegram bot routes) fall back to default permission policy. Default denies `Read` on scripts in `~/agents/lib/` and on `~/store/aiteam.db`, so orchestrator can't answer operational queries ("token spend today?", "any unauthorized access?").

**Two candidate fixes:**
- (a) Allowlist via `~/agents/.claude/settings.json` — fast but fragile.
- (b) Deterministic slash commands (`/spend`, `/audit_recent`, `/tasks_open`, `/kill_switches`) — cleaner, no LLM cost, no permission walls.

**Recommendation:** (b).

**Trigger to revisit:** After Phase 2 closes all 4 sub-phases, OR sooner if the next routed Telegram operational query dead-ends.

### D026 — Phase C launchd auto-restart for Telegram bot

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** Bot runs foreground; dies on Mac sleep/reboot/crash. Operator's stated reason for deferring: "computer hardly ever shuts off." Real risk: silent failure when bot dies and operator doesn't notice for hours.

**Planned scope:** `~/Library/LaunchAgents/com.aiteam.telegram.plist` with `RunAtLoad: true`, `KeepAlive: true`. ~1-1.5 hrs build + reboot verification.

**Trigger to revisit:** (a) first time the bot dies silently, (b) before any externally-publishing agent goes live, (c) Phase 2 close-out.

### D027 — Add `*.bak` to `~/brain/.gitignore`

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** `/ctx` pipeline writes `<save_path>.bak` skeleton backups that get committed by nightly auto-commit, bloating brain repo with stale content.

**Fix:** Single line in `~/brain/.gitignore`. `git rm --cached` existing `.bak` files.

**Trigger to revisit:** Next time `.gitignore` is edited for any reason; fold this in.

### F17/F18 — Daily API spend hard cap

**Status:** Open (cross-agent audit 2026-05-16)

**Problem:** Three caps disagree:
- `orchestrator/failure_modes.md`: `DAILY_API_BUDGET_USD=10` observability-only, no enforcement.
- `assignment-drafter/config/asbestos.yaml`: `daily_pipeline_budget_usd=15.00` enforced for pipeline fires only, fixed $2.50/fire estimate (not real-cost reconciled).
- No system-wide hard stop.

**Fix:** Enforce at `~/agents/lib/log_token_usage.sh` — after every insert, query today's sum, touch `/tmp/aiteam_budget_exceeded` if > cap, which `check_kill_switches.sh` treats as `SYSTEM_PAUSED=true`. Mtime-refresh picks it up within ~2s in every wrapper.

**Trigger to revisit:** After operator answers cross-agent audit §7.2 (what's the right number, and is a hard stop the right shape?).

**Reference:** `cross_agent_failure_modes_2026-05-16.md` F17, F18, Top-5 #2.

### D045 — `ai-do.sh` rewire (operator-named, context-incomplete) — ✅ CLOSED-OBSOLETE (2026-05-18)

**Closure note (2026-05-18):** Premise stale per pre-flight 2026-05-18 — writer/auditor already on Sonnet (`run-batch.sh` lines 107-108 hardcode `--model sonnet` as the default for both `WRITER_MODEL` and `AUDITOR_MODEL`). The "~75% cost reduction via Opus → Sonnet" framing no longer applies. Real remaining value (kill-switch enforcement + per-call token attribution) moved to **D061**. Vestigial `--sonnet` / `--sonnet-audit` CLI flag cleanup moved to **D062**.

**Status:** ~~Open~~ Closed-obsolete

**Problem (historical):** Operator named this as an open decision from this session, but context for the specific rewire isn't captured in any of the build reports or context-saves I read. Likely related to either: (a) the `AGENT_MAX_TURNS_OVERRIDE=1` pattern's edge cases, (b) the model-pinning verification routine (orchestrator/failure_modes.md mentions historical Opus-routing bug that invalidated step 17), or (c) something I haven't surfaced yet.

**Action:** ~~Operator to clarify the specific change envisioned.~~ Superseded by D061 + D062.

**Trigger to revisit:** N/A — closed.

### D046 — `notify.sh` env-override bug

**Status:** Open (Phase 1, 2026-05-16 assignment-drafter patch)

**Problem:** Inline `TELEGRAM_OUTBOUND_ENABLED=false` env override doesn't stub the send, because `~/agents/lib/notify.sh` re-sources `~/agents/config/.env` and overwrites the override. The intended testing pattern (`TELEGRAM_OUTBOUND_ENABLED=false bash notify.sh "..."`) doesn't work.

**Worked-around at the caller:** `~/agents/assignment-drafter/lib/notify_operator.sh` uses `NOTIFY_DEBUG=1` env var to print-instead-of-send, bypassing `notify.sh` entirely.

**Fix:** Make `notify.sh` respect inline env overrides. Either: (a) source `.env` only if `TELEGRAM_OUTBOUND_ENABLED` is unset (`: ${TELEGRAM_OUTBOUND_ENABLED:=...}` pattern), or (b) drop the inline source and require callers to source it themselves.

**Trigger to revisit:** Next time anyone needs to test a new notify-using agent without spamming the operator.

---

## Pipeline hardening

### D040 — Curator idempotency duplicate-row guard

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** Curator writes to `~/brain/expertise/cowen/<group>/<slug>/{synthesis.md, evidence-log.md, predictions-log.md}` without a "already-processed-this-video" guard. A re-run on the same video appends duplicate rows.

**Trigger to revisit:** Next curator touch.

### D041 — Curator topic fan-out for analyst-untagged topics

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** Analyst sometimes produces brief content that touches a topic not in the curator's seed taxonomy. Currently the curator either misses it or files into `_misc/`. Better routing rule would map untagged content to the closest existing seed (similarity > threshold) or auto-create a new seed if the topic recurs.

**Trigger to revisit:** After 30 days of curator runs accumulate data on miss patterns.

### D042 — `seed_taxonomy.sh` Casper preserve guard

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** `~/agents/market/curator/seed_taxonomy.sh` clobbers accreted Casper outlook-history rows on re-run. 5-min fix: skip writes when target row_count > 0.

**Trigger to revisit:** Next curator touch.

### D045 (Cowen, prior session, distinct from operator's D045 above)

**Status:** Tracked in archived context save `MarketAgents_S2_Closeout`, not currently active.

**Problem:** Opus prompt-template constraint to use existing seed slugs (vs producing non-canonical slugs that don't map to curator seeds).

**Trigger to revisit:** When `/important` invocation frequency exceeds ~2-3/week.

*(Operator: the D045 above in §Infrastructure refers to a separate "ai-do.sh rewire" item you named this session. There are now two D045s in flight — recommend renumbering the older one to D050 next time DEFERRED is touched.)*

### F4 / F19 — Per-slug `flock` around long-running operations

**Status:** Open (cross-agent audit 2026-05-16)

**Problem:** Two concurrent `run-batch.sh` invocations for the same slug (drafter cron fires while operator manually retries) race on `drafts/`, `feedback/`, `.cache/`. Same hazard for ship-to-site stage.sh and config-synthesizer overwrites.

**Fix:** `flock -n /tmp/aiteam-pipeline-<slug>.lock` wrapper. macOS flock via homebrew. 3-5 lines per call site.

**Trigger to revisit:** First time a race actually corrupts state, OR cross-agent audit §7.5 operator-policy answer.

**Reference:** `cross_agent_failure_modes_2026-05-16.md` F4, F19, Top-5 #5.

### F14 — Pre-ship operator approval gate

**Status:** Open (cross-agent audit 2026-05-16)

**Problem:** Auditor false positives (per editor-tier-test findings — Haiku is systematically generous; Sonnet less so but similar on subtle semantic checks) reach live site unattended. Recovery (F15) is 9 manual steps per bad slug.

**Fix:** Telegram-driven approval gate for first 5-10 slugs after any config change. After burn-in, auto-ship for unanimous-PASS slugs (no feedback in any of the 3 rounds). Editor agent (idle, scored r=0.933) could slot in as the score-based gate.

**Trigger to revisit:** After operator answers cross-agent audit §7.1 + §7.4. Or first time bad content lands live.

**Reference:** `cross_agent_failure_modes_2026-05-16.md` F14, F15, Top-5 #3.

### D052 — drafter.sh exit-2 vs exit-1 routing — ✅ CLOSED 2026-05-17

**Status:** ✅ CLOSED 2026-05-17. `maybe_fire_pipeline()` now case-routes `fire_pipeline.sh`'s rc: 0=FIRED, 2=SKIPPED_NO_CONFIG (soft skip, no double-audit, no ❌ Telegram), other=FIRE_FAILED. Caller bucket + notify_operator.sh footer added for the new state. Commit `4d45b95`. Canonical audit row id=1153 (`E3A78F6C-CD03-4EA8-99BC-7912902B484B`); supersedes intermediate rows 1151 (placeholder SHA bug) and 1152 (wrong supersedes-target).

**Original status:** Open (surfaced 2026-05-16 during F2 fix sign-off review)

**Problem:** `~/agents/assignment-drafter/drafter.sh` `maybe_fire_pipeline` (lines 184-193) treats any non-zero exit from `fire_pipeline.sh` as a hard fire-failure: captures stderr, logs `pipeline_fire_failed` to audit, buckets into `FIRE_FAILED_SLUGS`, Telegram says "❌ Fire FAILED." With F2 (commit `0df8cf6`) a missing-config slug now exits 2 from `fire_pipeline.sh` — which is a soft skip, not a failure. Result: two audit rows (`pipeline_skip_no_config` from `fire_pipeline.sh` + `pipeline_fire_failed` from `drafter.sh`) and a ❌ Telegram for what's really a benign skip.

**Fix:** `case` block around `fire_rc` in `drafter.sh:184-193`:
- exit 0 → existing fired-OK path (unchanged)
- exit 2 → new `SKIPPED_NO_CONFIG_SLUGS` bucket, ⚠ Telegram footer
- exit 1 / anything else → existing `FIRE_FAILED` path (unchanged)

**Trigger to revisit:** Next drafter.sh-touching session.

**Reference:** F2 fix at commit `0df8cf6`; `cross_agent_failure_modes_2026-05-16.md` F2 + Top-5 #1.

---

## Content / operational

### D028 — Diary's `hive_mind` data source incomplete

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** `write_diary.sh`'s `AGENT_RUNS` query reads from `hive_mind`, but briefer/scribe/analyst/curator don't write there. Diary undercounts daily agent activity.

**Three candidate fixes:** (a) instrument the agents, (b) expand the diary's source to `token_usage` + `audit_log`, (c) scope the diary to "AITEAM-fleet" and accept non-fleet agents are invisible.

**Trigger to revisit:** SSG Agent 1 build. The decision made there carries forward.

### D033 — Cron activation for Market Scribe poll.sh + briefer

**Status:** Day 1 of 1-3 gate began 2026-05-16

**Problem:** Manual-only Market Scribe runs require operator memory. Cron activation is gated on 1-3 days of clean manual runs to validate the full chain on fresh data.

**Trigger to revisit:** End of day 3 if all 3 days clean.

### D039 — outlook-history TL;DR repetition on rows 2+ from same video

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** When multiple per-asset rows derive from the same source video, the TL;DR field repeats verbatim on rows 2+ instead of being abbreviated or distinguished.

**Trigger to revisit:** Re-evaluate after 7+ days of real outlook-history data.

### D043 — `mentioned_assets` multi-asset convergence

**Status:** Built; only 1-asset case exercised

**Problem:** Convergence logic (analyst → briefer) supports multiple assets in a single brief but only the 1-asset code path has been exercised on real data.

**Trigger to revisit:** When a real Cowen/Casper video discusses multiple assets (ETH/SOL/etc.).

### D047 — Pre-patch chrysotile draft rejected by new validator

**Status:** Open (Phase 1, 2026-05-16 assignment-drafter patch)

**Problem:** The `chrysotile-attic-insulation-removal` draft from the pre-patch session contains legacy `<!-- OPERATOR EDIT -->` HTML comments + `TBD` / `Customize` strings. The new autonomous-mode validator (patch 2) correctly rejects it; the draft sits in `state/failed-...md`.

**Operator decision needed:** either (a) re-author the draft cleanly via the new autonomous flow, (b) hand-edit the rejected draft to strip the legacy markers and move it to the pipeline output path, or (c) discard and skip the slug.

**Trigger to revisit:** Next time operator wants the chrysotile-attic-insulation-removal page live.

---

## Hygiene

### D048 — Strip page.tsx template provenance comment before stage

**Status:** ✅ SHIPPED 2026-05-16 (commit `3b43c11`)

**Resolution:** `ship-to-site/lib/stage.sh` now does `tail -n +25` before the awk substitution, stripping the 24-line provenance comment block from `templates/page-tsx-template.tsx` before write. Verified: stripped output is byte-identical to a real shipped wrapper (slug-swapped).

### D051 — `~/agents/` working-tree hygiene

**Status:** Open (Phase 1, 2026-05-16)

**Problem:** Substantial work uncommitted in `~/agents/`:
- **Untracked dirs/files:** `assignment-drafter/` (full agent + patch 2), `editor/failure_modes.md`, `librarian/failure_modes.md`, `lib/{log_to_conversation.sh, log_token_usage.sh}`, `market/analyst/CLAUDE.md`, `market/briefer/`, `market/curator/`, `orchestrator/{commands/,failure_modes.md}`, `scripts/brain-autocommit.sh`, `telegram/`, `tg-monitor/`
- **Modified files:** `dashboard/server.js`, `lib/{ai-cheap.sh, ai-do.sh, ai-think.sh, log_to_audit.sh, notify.sh, run_agent.sh}`, `market/analyst/{analyze.sh, brief_casper.template.md, brief_cowen.template.md, prompt_template.md}`

**Why deferred:** Not a single-feature commit — needs grouping into multiple coherent commits (one per agent or per concern). Last session's commits (`69dd30f`, `3b43c11`) explicitly staged only specific paths to avoid sweeping unrelated work in.

**Fix:** A cross-cutting session focused on committing each agent + each lib modification with descriptive messages. Probably ~5-8 commits depending on grouping.

**Trigger to revisit:** Operator bandwidth for hygiene work, OR before any major refactor that would touch `~/agents/` broadly.

### D053 — assignment-drafter state files not gitignored

**Status:** Open (surfaced 2026-05-16 during F2 fix sign-off)

**Problem:** `assignment-drafter/state/daily_pipeline_spend.txt` got tracked in commit `0df8cf6`. Other state files likely also missing from `.gitignore`: `state/completed.log`, `state/failed.log`, `state/last_run.txt`, `state/pipeline_runs.log`. Will dirty git status on every cron tick and risk sweeping runtime artifacts into future commits.

**Fix:** Add `state/` blanket entry (or per-file entries) to `assignment-drafter/.gitignore`. `git rm --cached` for any already-tracked.

**Trigger to revisit:** Bundle with D051 (`~/agents/` repo hygiene session).

**Reference:** Surfaced during F2 fix sign-off (commit `0df8cf6`).

### D054 — Audit all cron-invoked scripts for PATH dependencies — ✅ CLOSED (2026-05-17)

**Closure note (2026-05-17):** Resolved via Path (c) — crontab-level PATH header prepended: `PATH=/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin`. Hybrid: crontab header covers all 8 direct cron-invoked scripts + all transitive children (`ai-do.sh`, `run-batch.sh`). `build_verify.sh` per-script guard (0be9d1f) preserved as belt-and-suspenders for non-cron callers. Crontab snapshot pre-change at `/tmp/crontab-snapshot-pre-D054-1779000335.txt`. Audit_log closure row: `D79474F2-B88F-4B97-9510-BF93EA37ED65` (positional fields landed malformed — see D064; row content is correct, queryable by substring on target).

**Status:** ~~Open~~ Closed

**Problem (historical):** Ship-to-site's `build_verify.sh` failed silently under cron because `/opt/homebrew/bin` was not in cron's PATH (`npm: command not found`, audit row 536). Same risk existed for any cron-invoked script that calls a Homebrew-installed binary (`npm`, `node`, `python3`, `jq`, `sqlite3`, `gh`, etc.). No project-wide convention for ensuring cron's PATH matches interactive shell.

**Fix options considered:**
- (a) one-time audit of every cron-invoked script for missing PATH preludes
- (b) standardize a "cron PATH harden" snippet in a shared lib that every cron entry point sources
- (c) set PATH explicitly in crontab itself — **selected**

**Reference:** Ship-to-site `build_verify.sh` fix at commit `0be9d1f`; audit_log row 536. Closure commit: no code change required (crontab edit only).

### D057 — Remove deploy throttle when quality is consistent

**Status:** Open (introduced 2026-05-17 alongside the throttle itself)

**Problem:** Ship-to-site was throttled from `*/15` cadence to a single 23:00 PT batch deploy with a 20:00 preview ping (commit `3ed57dc` in `~/agents/`). This is temporary scaffolding — designed to be reversible — but reversibility means nothing if nobody remembers to flip it back. Without removal, time-to-indexed permanently sits at +12h median for every guide.

**Fix:** Reversal procedure documented in `~/brain/projects/aiteam/docs/deploy_throttle_build_2026-05-17.md` §"Reversal procedure". Three-line crontab edit: comment out the new 20:00 + 23:00 lines, uncomment the original `*/15` line. The two new scripts can stay in the repo (idempotent on empty queue) or be deleted. No agent code change needed.

**Trigger to revisit:** After 2-3 weeks of clean preview pings with no manual skips needed (i.e. operator never deletes a slug from `approved/guides/` between 20:00 and 23:00). Signal that audit_guide.py + drafter quality is reliable enough that the human gate is overhead rather than value.

**Reference:** Build report at `~/brain/projects/aiteam/docs/deploy_throttle_build_2026-05-17.md`; crontab snapshot pre-change at `/tmp/crontab-snapshot-pre-throttle-1778996892.txt`.

---

### D058 — Vercel preview URLs in the 20:00 preview ping

**Status:** Open (surfaced 2026-05-17 during throttle build)

**Problem:** The 20:00 preview ping ships without per-slug preview URLs because none exist today — Vercel auto-deploys `main` to production, and nothing in `ship-to-site/` constructs per-commit Vercel preview URLs. Operator currently triages on slug + word count + score + audit-reasoning excerpt only. For tonight's review pattern this is fine; if score-based triage proves insufficient, click-to-preview would be the natural escalation.

**Fix options:**
- (a) **Branch-based preview deploys** — push each ready slug to a `preview` branch first, let Vercel auto-deploy it, capture the per-commit `*.vercel.app` URL, include in the ping. Forwards `main` only at 23:00 after operator approval. Adds branch hygiene + race-with-main concerns.
- (b) **Vercel CLI per-deploy** — install `vercel` CLI, use `vercel deploy --prebuilt` per slug to mint a preview URL without touching git. Captures URL in stdout. Self-contained but requires Vercel auth keys in env + a separate build per preview.
- (c) **Local screenshot to Telegram** — preview_ping.sh stages each slug locally, runs `npm run build`, takes a screenshot (e.g. headless Chrome), attaches to Telegram. Phone-friendly, heavy, fragile, local build may not match prod exactly.

**Trigger to revisit:** After 2-3 weeks of throttle in production, if score-based triage proves insufficient AND operator wants click-to-preview before the 23:00 fire. Otherwise this stays deferred indefinitely (the throttle window itself is the safety mechanism; preview URLs are a UX upgrade, not a safety mechanism).

**Reference:** Build report `~/brain/projects/aiteam/docs/deploy_throttle_build_2026-05-17.md` §"Spec correction — Vercel, not Cloudflare".

---

### D056 — Editor verdict persistence — ✅ CLOSED 2026-05-17

**Status:** ✅ CLOSED 2026-05-17. Agents commits `6895dcc` (score.sh production runner + CLAUDE.md wording), `dc05c51` (threshold tune 3.8 → 3.6 per burn-in distribution), `1323040` (ship.sh editor verdict gate in ship_one_slug step 2.5).

**Closure note:** Shipped the production single-slug editor runner (`~/agents/editor/score.sh`), persisted verdicts to `auditor_verdicts` with `scorer='editor-v1'`, and wired the gate into `ship_one_slug` between validate.sh and the dry-run short-circuit per POLICY Q7 (editor runs AFTER audit_guide.py, both must pass). 4-slug burn-in distribution produced a tight composite range (3.20-3.80) with structural ceilings on 3 of 5 axes driven by audit_guide.py pre-filtering; threshold lowered from 3.8 to 3.6 to bisect the empirical format-quality ordering. Integration test: SQL-trace proof for both PASS and FAIL branches (live dry-run blocked because all 32 approved JSONs are already in the asbestoshq-site whitelist, so validate.sh always short-circuits at slug_already_shipped before reaching the editor gate). Live-gate first execution is deferred to the next genuinely-new pipeline slug — tracked as **D074**. Truncation follow-up tracked as **D072**; rubric-redundancy review as **D073**.

**Commit-message correction:** `dc05c51` body contained a parenthetical asserting "only shingles + dispose-of remain queued" — actually all 3 fail-at-3.8 slugs (shingles, how-to-dispose-of, ceiling-tile) remain queued per the operator's explicit "leave verdicts as-is" instruction. Captured here for the record (per no-amend rule, the commit stands).

**Original problem (preserved for history):** The `auditor_verdicts` table shipped 2026-05-17 (`4e2fe0a` in `~/agents/`) is populated by `audit_guide.py` (commit `0e8089b` in asbestos-contractors) but the editor agent is NOT yet writing to it. Triage-agent coverage is therefore half: only mechanical-check verdicts are queryable, not editor-rubric scorecards. The original build prompt asked for both halves; editor half hit two HALT conditions and was deferred.

**Two blockers:**
1. **No production editor runner exists.** `~/agents/editor/` only has `run_tier_test.sh` (calibration). No script today takes one real draft, scores it, and would be the natural place to hook persistence.
2. **No pass threshold documented.** The 1–5 composite is defined, but no `passed = 1 vs 0` cutoff exists in CLAUDE.md, rubric.md, agent.yaml, or the runner.

**Fix:** Three pre-reqs before persistence can ship:
- A production editor runner (e.g. `~/agents/editor/score_one.sh`) shaped like `audit_guide.py`'s pattern (one invocation = one row).
- An operator-stated pass threshold (constant in the runner or a field in `agent.yaml`).
- A decision about where editor is wired into the pipeline (ship-to-site pre-ship gate? Inside `fire_pipeline.sh`? Standalone cron?). This is the answer to Q7.

Once those exist, the persistence pattern is the same shape shipped today — add ~40 LOC mirroring `audit_guide.py`'s `persist_verdict()` helper.

**Trigger to revisit:** After operator answers Q7 (editor as pre-ship gate) in `cross_agent_failure_modes_2026-05-16.md` §7, AND a production editor runner exists.

**Reference:** Build report `~/brain/projects/aiteam/docs/verdict_persistence_build_2026-05-17.md` §"Editor persistence — deferred"; Q7 in `cross_agent_failure_modes_2026-05-16.md` §7. Note: build prompt asked for this to be tracked as "D053" but that number was already taken; D056 chosen per the file's "next free D-number" rule.

### D072 — Editor `score.sh` chunk-and-aggregate for full-article scoring

**Status:** Open (surfaced 2026-05-17 during D056 burn-in)

**Problem:** Current `score.sh` truncates articles to 8KB (head only) to match the calibration contract from `run_tier_test.sh`. Production asbestos guides run 1800-2800 words (~12-18KB raw), so the tail 30-50% of every article is invisible to the editor. **All 4 burn-in slugs hit the 8KB cap** (`truncated: true` on every scored row). The editor is structurally blind to article endings — exactly where weak guides tend to wander.

**Fix:** Chunk article into 8KB segments, score each segment independently, aggregate. Two aggregation candidates:
- Mean per axis across chunks (smooths noise; favors consistent quality)
- Min per axis across chunks (catches the worst section; supports POLICY Q1 FP-tolerance)

Recommendation lean: min-per-axis matches the "false positives are much worse" policy. Re-calibrate threshold after fix lands — current 3.6 was tuned against head-only scores.

**Trigger:** After 10+ slug burn-in distribution data accumulates AND truncation pattern is confirmed to be hiding real quality signal in the tail. Cheap pre-fix experiment: re-score one slug on its tail 8KB only, compare to head 8KB score. If sub-scores diverge >0.5 on any axis, the head-only score is missing real signal.

**Reference:** D056 closure note + audit_log row `<d056_closed correlation_id>`. Per-row `truncated` flag in `editor_scored` payload lets future re-calibration sessions filter on it.

### D073 — Review editor rubric axes for redundancy vs `audit_guide.py` mechanical gate

**Status:** Open (surfaced 2026-05-17 during D056 burn-in)

**Problem:** 4-slug burn-in showed 3 of 5 editor axes have structural ceilings driven by `audit_guide.py` pre-filtering:
- **Originality** (mean 2.75, range 2-3): SEO conformance (keyword density, required entities) caps creative framing
- **Hook** (mean 3.0, zero variation): S5/S6 audit-guide opener rules (first sentence ≤15 words, first para ≤3 sentences) produce uniformly competent openers
- **Evidence** (mean 3.75, range 3-4): mechanically propped by SPEC1 (≥3 specifics) + L4 (.gov authority link)

Only **clarity** (mean 3.5, range 3-4) showed real discriminating variation. **Evergreen** (4.5, 4-5) floors at "high" for the asbestos topic regardless of article quality. Net: editor is measuring partially redundant signals on a pre-filtered corpus.

**Fix options:**
- (a) Refine editor rubric to test what `audit_guide.py` CAN'T enforce (voice consistency, narrative arc, depth of analysis, internal contradiction detection)
- (b) Reduce editor to a clarity-axis-only discriminator
- (c) Re-position editor as advisory (verdict logged but not gating)
- (d) Some hybrid: keep all 5 axes but weight them so clarity dominates

**Trigger:** After burn-in N>=10 confirms the corpus-distribution pattern, OR after first editor false-positive (a clearly-bad slug passes at composite ≥3.6).

**Reference:** D056 closure note + 4-slug distribution table in `dc05c51` commit body.

### D074 — Verify editor gate executes live on first new pipeline slug

**Status:** Open (surfaced 2026-05-17 at D056 close-out)

**Problem:** D056 closed with SQL-trace proof for both PASS and FAIL branches of the editor gate in `ship.sh:ship_one_slug step 2.5`. Live integration test could not exercise the gate because **every approved JSON at close time was already in the asbestoshq-site whitelist** (validate.sh short-circuits at `slug_already_shipped` before reaching step 2.5). The gate is implemented and statically reachable, but has never executed end-to-end against ship.sh.

**Verification plan (run on next new pipeline slug):**
The gate will first execute live when a genuinely-new approved slug enters via `drafter_queue.txt` → `run-batch.sh` and lands in `content/asbestos/approved/guides/` not-yet-whitelisted. When that happens, verify in order:

1. `ship.sh` log line: `[editor] <slug>: no verdict; running score.sh...`
2. `score.sh` fires inline, writes raw response to `~/agents/editor/workspace/responses/<slug>_<ts>.txt` BEFORE parse
3. `auditor_verdicts` row landed with `scorer='editor-v1'`, `scorer_version='v1'`, `composite_score` populated, `rubric_json` valid 5-axis dict
4. `ship.sh` re-queries verdict and branches correctly: PASS → falls through to dry-run/stage; FAIL → `[skip] ... failed editor gate (routed to needs_review by score.sh)`
5. `audit_log` captures the full sequence: one `editor_scored` row from score.sh + one `ship_to_site_skipped` (with `reason="editor_failed"`) or `ship_to_site` row from ship.sh

**Where to look:** `~/agents/ship-to-site/ship.log`, `~/agents/watchdog/watchdog.log` (cron tick will show it), the next 20:00 preview ping, and `audit_log` queries by date.

**Trigger:** First new approved slug from the asbestos pipeline post-D056 close (or the next manual operator-initiated test).

**Reference:** D056 closure note. SQL-trace proof was the close-out test surrogate; this item upgrades it to a live observation requirement.

**GEO Optimizer scope extension (added 2026-05-17):** D074 now also tracks live-gate verification for all three GEO Optimizer phases:

- **Phase 1** (`~/agents/geo-optimizer/score.sh`, commit `0335838`): production single-slug scorer. Smoke-verified at Phase 1 ship; nothing further to verify here.
- **Phase 2** (ship.sh gate, commit `32b3ea3`): runs `score.sh` inline at deploy time when no fresh verdict exists. Verification: on first new approved slug post-Phase 2, ship.log should show a `geo-v1` score.sh invocation and PASS routes to deploy / FAIL routes to `needs_review_queue.txt`. Same shape as the editor-gate verification above.
- **Phase 3** (run-batch.sh writer-revision loop, commit `624d334`): GEO gate fires inside the `if [ "$audit_passed" = true ]` block after AuditKit passes. PASS → fall through to APPROVED + AUDITED. FAIL → write feedback at `${FEEDBACK}/${page}-r${round}.md`, mv back to drafts/, increment round. MAX_ROUNDS → audit_log `geo_escalated` + `notify.sh` Telegram alert. Hard error (exit 1) → log + fall through (do NOT block APPROVED). Verification: first organic FAIL on a real new pipeline slug should produce one `geo_fail` audit row per round, one feedback file per round, and (if all 3 rounds fail) one `geo_escalated` audit row + one Telegram message in operator's chat.

Phase 3 PASS path smoked at audit_log id=1252, auditor_verdicts id=8. FAIL path code-reviewed, mirrors AuditKit L1099-1140; awaits first organic fail on a real new pipeline slug.

---

### D055 — Site repo node_modules disappeared

**Status:** Open (deps reinstalled this session as part of build_verify fix)

**Problem:** `~/projects/asbestoshq-site/node_modules/` was present when the 31 guides were manually shipped earlier, but absent when ship-to-site tried autonomously (surfaced as `sh: next: command not found` after the PATH fix landed). Unknown what removed it — could have been a fresh clone, `git clean`, manual `rm`, or something else. Risk: if it disappears again, every cron ship fails until rediscovered.

**Fix options:**
- (a) accept and watch (deps reinstalled, move on)
- (b) add a one-time `[ -d node_modules ] || npm ci` self-heal check in `ship.sh` or `stage.sh` (NOT `build_verify.sh` — keep build_verify scoped)
- (c) treat `node_modules` as a known-precondition documented in ship-to-site README + check at agent startup

**Trigger to revisit:** If it disappears again, OR next ship-to-site polish session.

**Reference:** audit_log row 833 (`sh: next: command not found`); paired with D054 (same incident).

### D060 — 3 untracked production scripts (zero git history) — ✅ CLOSED 2026-05-17

**Status:** ✅ CLOSED 2026-05-17. Auto-closed by D066 hygiene audit. brain-autocommit.sh committed in `b76832d`; reader.py + analyzer.py committed in `6a547ce`. Audit row id=1145 (`9A3A7ABD-D427-40C1-9C94-074769567CA2`).

**Original status:** Open (surfaced 2026-05-16 during D054 recon)

**Problem:** Three production scripts run on cron but have never been `git add`-ed in `~/agents/`. All three return zero rows from `git log -- <path>` and `git status` shows `??`:
- `scripts/brain-autocommit.sh` (cron 23:55 daily)
- `tg-monitor/reader.py` (cron */5)
- `tg-monitor/analyzer.py` (cron 07:00 daily)

Pattern caught 4th time (prior: ship-to-site, assignment-drafter, config-synthesizer). Production scripts running silently with no version history is a recurring hygiene gap, making rollback and change-attribution impossible.

**Fix:** Commit each script in `~/agents/` with a descriptive message, verify presence in git log. Standalone hygiene pass, not bundled with feature work. Could fold into D051 if that session happens first.

**Trigger to revisit:** Next agent-hygiene session.

**Reference:** Surfaced during D054 cron-script PATH audit (this session). D054 audit_log closure row: `D79474F2-B88F-4B97-9510-BF93EA37ED65`.

### D063 — Asbestos run-batch.sh SITE_MAP grep latent bug (mirrors D-SSG-05)

**Status:** Open (surfaced 2026-05-17 during D-SSG-05 fix)

**Problem:** `~/projects/asbestos-contractors/content/run-batch.sh` at line ~269 uses the same `grep -E '^\/' "$SITE_MAP" | sed 's/ \[PLANNED\]//' | tr -d ' ' > "$VALID_LINKS"` pattern that was just fixed in SSG. Today `ASBESTOS_SITE_MAP.md` is a 2-line stub, so the grep correctly returns empty — bug doesn't manifest. The instant the asbestos SITE_MAP is filled in beyond stub state (especially if it adopts the same markdown-table format as SSG's), `VALID_LINKS` will silently come out empty and writers lose the valid-internal-links allowlist.

**Fix:** Pattern-copy the D-SSG-05 fix (commit `1096584` in `ssg-content`). Replace the line with the section-bounded, table-row-filtered, backtick-stripped pipeline:
```
sed -n '/^## Slug catalog/,/^## Forbidden link targets/p' "$SITE_MAP" \
  | grep -E '^\|' \
  | grep -oE '`/[a-z0-9/-]+/`' \
  | tr -d '`' > "$VALID_LINKS"
```
Adjust section-header names if asbestos's SITE_MAP picks different ones.

**Trigger to revisit:** When `ASBESTOS_SITE_MAP.md` is filled in beyond stub state.

**Reference:** D-SSG-05 fix at commit `1096584` (ssg-content). D-SSG-05 audit_log closure row: `41E6B47B-3199-433F-8F90-ADBE52826344`.

### D064 — `log_to_audit.sh` invocation hygiene — wrapper or schema check

**Status:** Open (surfaced 2026-05-17 during T2 closure-status audit)

**Problem:** `log_to_audit.sh` accepts six positional args (`ACTOR_TYPE ACTOR_ID ACTION TARGET PAYLOAD CORRELATION_ID`). Easy to malform by passing `key=value` strings positionally — the whole `key=value` literal lands in the wrong column with no error. Happened 3× in the 2026-05-17 session. The affected closure rows exist but are unfindable by standard queries like `WHERE action='deferred_closed'` because the `action` column literally contains `target=D054` etc. Substring search on `target` still finds them, but no caller writes queries that way.

Affected audit_log correlation IDs from 2026-05-17:
- `42098527-9BE9-4CCA-86B7-9001CE656A98` (intended: D045 closure)
- `D79474F2-B88F-4B97-9510-BF93EA37ED65` (intended: D054 closure)
- `41E6B47B-3199-433F-8F90-ADBE52826344` (intended: D-SSG-05 closure)

**Fix options:**
- (a) **Add arg-shape validation inside `log_to_audit.sh`** — reject `ACTOR_TYPE` containing `=`, or any positional arg containing `=` before the PAYLOAD slot. Cheap, prevents silent corruption fleet-wide. **Recommended minimum.**
- (b) **Provide a named-arg wrapper helper** (e.g., `log_to_audit_kv.sh actor=operator action=deferred_closed target=D054 payload='{...}'`) that parses `key=value` and forwards positionally. Most ergonomic but adds a second tool to keep in sync.
- (c) **Accept and rely on operator discipline.** Status quo. Will keep happening.

Do NOT retroactively edit the 3 malformed rows from this session — substring-findable is acceptable; retroactive edits risk worse drift.

**Trigger to revisit:** Next audit-log-touching session, OR when the first `WHERE action='deferred_closed'` query returns wrong results.

**Reference:** Surfaced during 2026-05-17 cleanup session. T2.5 commit. D044-redundancy audit_log row (correctly formed): `DFD1AF8B-7C6F-42CC-8DDD-4EE148C37F0F`.

### D027 — Add `*.bak` to `~/brain/.gitignore`

(Already documented in §Infrastructure — re-listed here as a hygiene item for convenience.)

### Provenance comment block on generated `page.tsx` — operator decision

**Status:** Decided 2026-05-16 — strip via stage.sh (D048).

**Background:** The page.tsx template carries a 24-line documentation comment block. Initially shipped with substitution; this session added the strip step so generated wrappers stay byte-identical to existing shipped pages. No further action needed unless operator wants to revisit.

### D066 — Systemic untracked-production-code pattern (4th instance) — ✅ CLOSED 2026-05-17

**Status:** ✅ CLOSED 2026-05-17. Audit row `F9D65187-374C-430C-95F3-D26886D26E8D` (audit_log id=1143).

**Closure note:** Executed 2026-05-17. 16 commits across 3 repos (~/agents/=12, asbestos=4, ssg-content=2). 50 untracked production files committed in 12 logical groups (A–L). 8 gitignore patterns landed. 6 scratch files deleted. 3 operator UNCLEAR items decided + executed (drafter_queue.txt gitignored, tg-monitor tests deleted, ssg .bak files deleted). One mid-flight HALT handled cleanly via Option 1 (2 follow-up commits for surfaced sub-gitignores + telegram/.env.example template + telegram/.venv ignore). Audit row 1142 malformed (D064 bit on closure call — 4 positional args instead of 5, JSON blob landed in target slot); corrected sibling row inserted at id=1143 with supersedes_malformed_row=1142 in payload. D064 priority bumped — wrapper fix lands in next hygiene cluster.

**Follow-up recon — ~/agents/ half (2026-05-19):** Read-only recon confirmed working tree clean (0 uncommitted files); HEAD at `1323040` (sister's D056 editor verdict gate); 2 unpushed commits both sister-chat editor work, handled by 23:50 `agents-autocommit` cron. 2026-05-17 audit headline ("15+ uncommitted files") swept by intervening commit discipline (D068 watchdog, D056 editor, D070 agents-vault backup). No dedicated audit session needed. Closure audit row: `E5C322E4-2D48-40EB-AC92-5FC6622EEED4`. Prior projects-half audit: `02E772CB-7FE7-4C39-A129-A460A4493D80`.

**Trigger:** Next hygiene-pass session.

**What:** Four distinct sessions this month have encountered production code in `~/agents/` that was running on the host but never `git add`ed. Each instance was surfaced incidentally during unrelated feature work; each required a surgical extraction or "commit pre-existing work first" detour before the actual task could proceed cleanly.

**Instances observed:**
1. Ship-to-site (pre-c1aa9c0 era) — scripts running in cron without git history (resolved organically).
2. **D060** — three untracked production scripts in `~/agents/` (still open, queued).
3. `dashboard/server.js` slash-commands + multi-round discuss + token-spend chart (uncommitted across two sessions; surfaced in c56209f and 60b2486; finally committed in `a7c2c71` 2026-05-17).
4. `lib/log_token_usage.sh` — the actual token_usage writer that every `ai-*.sh` wrapper depends on, untracked. Surfaced during F17/F18 cost-cap pre-flight 2026-05-17; committed as `7446717` in the same session.

**Suggested action:** One-time audit pass over `~/agents/`:
```bash
# Files on disk but not tracked
comm -23 \
  <(cd ~/agents && find . -type f \! -path './.git/*' \! -path '*/node_modules/*' \! -name '*.log' | sort) \
  <(cd ~/agents && git ls-files | sed 's|^|./|' | sort)
```
Triage each match: commit, gitignore, or delete. Then institute a process check (pre-cron-add lint? weekly cron of the above audit posting to Telegram?).

**Process gap hypothesis:** Operator + sister-chat CC sessions write production scripts directly into `~/agents/` (often via Telegram-driven session) without ever returning to `git add`. The next session inherits the file as background state, doesn't notice it's untracked, and ships features on top of it.

**Reference:** D060 (predecessor; consolidate or supersede if D066 audit takes the same scope).

### D067 — Recover SSG data-driven rewrite plan from chat history — ✅ CLOSED 2026-05-17

**Status:** ✅ CLOSED 2026-05-17. Brain commit `b1c4214`. Audit row `0C9C5E2D-E5FD-4EF1-9353-C475C42473C5`.

**Closure note:** Plan was authored in chat `5c03da96-ae1c-4431-851e-41a618741049` on 2026-05-16 (as `AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-16.md`) but never saved to disk — original lost. Reconstructed verbatim via `conversation_search` 2026-05-17 and landed as `~/brain/projects/aiteam/docs/AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-17.md` (375 lines, 16245 bytes). Spec-on-disk before SSG Lane B execution. Brain repo had pre-existing dirty state at close time (modified HANDOFF/LESSONS, 11 untracked context saves, 15 unpushed commits); D067 commit isolated to the recovered plan only by staging the exact path.

**Reference:** Plan file at `~/brain/projects/aiteam/docs/AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-17.md`. Recovery pattern (spec-recovered-from-chat) candidate for `LESSONS.md` next pass.

### D068 — Watchdog hygiene pass (3 fixes) — ✅ CLOSED 2026-05-17

**Status:** ✅ CLOSED 2026-05-17. Agents commits `5c7d85c` (FIX A), `19f44aa` (FIX B), `84808e2` (FIX C). Audit row `591EE19F-E8B2-4421-9660-C5AAFDE039C5`.

**Closure note:** Three hygiene nits surfaced during yesterday's watchdog tuning session (38ed525, 1053913, 26d3311) but were intentionally out of scope at the time. Closed in one session:

- **FIX A — `compute_threshold()` helper**: collapsed 4 inline `NOW - window_hours*3600 - grace_minutes*60` sites (3 probes + main loop) into one helper. Same arithmetic, byte-identical dry-run output. e1/e2 grace flip (grace=0+window=0 → drafter [missing]; grace=35+window=0 → drafter [healthy]) still passes.
- **FIX B — drop `_stale` qualifier from probe returns (option B2)**: `probe_logfile_mtime` and `probe_sqlite_row` previously echoed `mtime`/`mtime_stale` and `sqlite_row`/`sqlite_row_stale`; the main loop discarded the qualifier and re-evaluated freshness via threshold anyway. Probes now always echo `mtime` / `sqlite_row` (qualifier dropped). Per-probe `compute_threshold` call removed as vestigial — main loop owns the freshness decision. Probe signatures simplified (window_hours + grace_minutes args dropped). Output diff under healthy state: byte-identical. Stale-path test confirms action column now shows `mtime`/`sqlite_row` instead of `mtime_stale`/`sqlite_row_stale`; status decision unchanged. `probe_audit_log` untouched (always returned real action name).
- **FIX C — single-load state.json at tick start**: 7 actors × 2 fields × per-call `jq` = 14 jq subprocesses per tick → 1 jq invocation in `load_state_cache()` that populates `STATE_<actor>_<field>` shell vars via `@sh`-quoted eval. `state_get` public interface preserved; body rewritten to read from cache. Bash 3.2-safe (no assoc arrays). 3-run avg timing: pre-C 117ms → post-C 105ms (~10% / ~12ms saved; modest because state.json is only 1KB, but structural 14-forks → 1-fork win is real and will pay off on any future state.json growth). Corrupted-state.json gate test confirms FIX C is downstream of the gate (untouched).

Live tick id=1171 (2026-05-17 15:03:05) clean: healthy=6, missing=1, no transitions. All 3 commits independently revertable.

**Phantom SHA note:** Yesterday's context save named `1e90165` as the LESSONS.md doc commit. That SHA was not in `~/agents` — pre-flight grep was wrong-repo. Confirmed `1e90165 docs(lessons): audit_log column name + heartbeat-before-killswitch validation` exists in `~/brain`. Real SHA, real commit, wrong repo expectation in the prompt.

**Reference:** Build context yesterday's session (commits `88f8d7a`, `1053913`, `38ed525`, `26d3311` in `~/agents`).

### D065 — Add `gsc_submission_queue` INSERT hook to SSG `deploy_batch.sh`

**Status:** Deferred (Phase 2, 2026-05-17)

**Trigger:** When SSG ships its first batch — i.e. `~/agents/ship-to-site/config/ssg.yaml` flips to `enabled: true` AND `content/ssg/approved/guides/` becomes non-empty.

**What to do:** Pattern-copy the asbestos hook (commit `c56209f` introduced it, this commit added the `site` column). After the shipped-success branch in SSG's deploy_batch.sh, insert:

```bash
if [ "${site}" = "ssg" ]; then
  slug_esc="${slug//\'/\'\'}"
  sqlite3 "${STORE_ROOT}/aiteam.db" \
    "INSERT OR IGNORE INTO gsc_submission_queue (slug, url, site, published_at) VALUES ('${slug_esc}', '<ssg-live-base-url>${slug_esc}/', 'ssg', strftime('%Y-%m-%dT%H:%M:%fZ','now'));" \
    >/dev/null 2>&1 || true
fi
```

`site='ssg'` must be passed explicitly — the `DEFAULT 'asbestos'` on the column is a one-time safety net, not a fallback. Verify the SSG live base URL convention against `ssg.yaml` before hardcoding (asbestos got this wrong in spec literal; confirmed from yaml at build time).

**Reference:** This commit (multi-site upgrade) + commit `c56209f` (original asbestos hook + dashboard panel).

### D069 — friend-bot untracked production code (D066 pattern, 5th instance)

**Status:** Open

**Trigger:** Next `~/projects/` hygiene session, OR when friend-bot needs modification.

**Problem:** `~/projects/friend-bot/` is not a git repo. Contains live production code + runtime data with no version control — same pattern D066 closed for asbestos-contractors / asbestoshq-site / ssg-content. Surfaced by 2026-05-17 D066-projects-half recon (audit row `02E772CB-FE71-4C3D-A129-A460A4493D80`).

**Inventory at discovery (2026-05-17):**
- `bot.py` — 15K source
- `bot.db` — 57K runtime (modified 2026-05-17 00:27)
- `bot.log` — 22K runtime (modified 2026-05-17 00:27)
- `.env` — secrets
- `memory_prompt.txt`, `system_prompt.txt` — agent config
- `requirements.txt`

**Fix:** Two-step
- (a) `git init` + commit source/configs (`bot.py`, `*.txt`, `requirements.txt`)
- (b) `.gitignore` for `bot.db`, `bot.log`, `.env` BEFORE first commit

**Risk:** Bot is actively running (db + log modified within hours of discovery). Initialize git carefully — don't commit secrets or runtime data.

**Reference:** D066 (untracked-code hygiene audit, 2026-05-17, brain commit `61647e5`). 5th confirmed instance of the same systemic pattern.

### D070 — `~/agents/` git remote upstream not configured — ✅ CLOSED 2026-05-17

**Status:** ✅ CLOSED 2026-05-17. Audit row `FC8F68E8-7082-4648-8ACF-2501F88611FC`.

**Closure note:** Private GitHub backup landed for `~/agents/`. Mirror of brain-vault pattern.

**What landed:**
- Private repo: `github.com/mdp280028-ui/agents-vault` (created manually via web UI — `gh` CLI not installed)
- SSH auth via existing `id_ed25519` (brain-vault key)
- Full history pushed; HEAD SHA `2ee93487e2ecdaa72b1f10104e7749ef4944a305`
- Upstream: `branch.main.remote=origin`, `branch.main.merge=refs/heads/main`
- Local↔remote SHA match verified post-push
- `.gitignore` hardened pre-push (commit `7b8c0d5`): added `.env`, `.env.*`, `**/.env`, `**/*.lock`, `**/state.json`, `**/state/`, `/tmp/`; exceptions for `**/.env.example` + `**/.gitkeep`
- Daily 23:50 cron installed: `agents-autocommit.sh` — staggered 5 min before `brain-autocommit` at 23:55

**Secrets audit (full history, 64 commits):**
- 205 token/key/password/bearer matches → all false positives (`token_usage` table refs, `BOT_TOKEN` env-var reads, never values)
- 133 high-entropy strings → all false positives (paths, comment dividers)
- 0 cloud-provider key patterns
- Only `telegram/.env.example` committed (intentional template, "NEVER commit" comment)

**Verification gap:** Repo visibility verified by operator browser eyeball, not via API. See D071.

**Sister-chat coexistence:** During D070 execution, sister landed `6895dcc feat(editor): score.sh production runner` at 15:25. Autocommit test fire at 15:26 pushed both separately, no bundling.

**Reference:** D066 `~/agents/` half recon, 2026-05-17 (audit row `58839FBF-6ADB-436B-95B8-B0CEEAB8A4B7`).

### D071 — Install `gh` CLI for GitHub API operations

**Status:** Open

**Trigger:** Next time repo creation or visibility verification is needed, OR opportunistically during a low-priority hygiene session.

**Problem:** `gh` CLI is not installed on the Mac Mini. D070 hit this — repo had to be created via web UI and visibility verified by browser eyeball instead of API call. Workable for one repo, but adds operator-touch and verification gap to any future repo work.

**Concrete consequences seen 2026-05-17 during D070:**
- `gh repo create` not available → operator manually created `agents-vault` via web UI
- `gh repo view --json visibility,isPrivate` not available → no API confirmation that visibility was actually private
- Verification fell back to operator browser eyeball

**Fix:**
```bash
brew install gh
gh auth login
```
(Interactive flow — operator drives.)

**Cost:** ~30 sec install + ~2 min auth flow. One time.

**Use cases unlocked:**
- Programmatic private-status verification (close D070's verification gap retroactively)
- D069 friend-bot fix can create its own private repo via `gh repo create`
- Any future repo (third project, archive snapshots, agent fleet split, etc.) skips web UI

**Risk:** Adds one more tool to the system. Mitigation: `gh` is widely used, well-maintained, and brew-installable. Same risk profile as having git itself.

**Reference:** D070 closure note (verification gap).

---

### D075 — Opportunity Scout Agent (business-idea discovery, separate from Agent A)

**Status:** Deferred. Trigger: Agent A (Research/Opportunity = pain-point miner) ships and burns ~1 week clean.

**Distinct from Agent A:**
- Agent A makes asbestoshq.com better → output is *slugs* fed weekly into drafter_queue.txt.
- D075 finds the *next business* → output is *opportunity briefs* fed monthly into operator portfolio decisions.
- Different reviewers in operator's head, different files, different mission-bar relationship. Cramming both into one agent produces a worse version of each.

**Scope:**
- Discovers business opportunities, low-competition niches, and market gaps the AITEAM fleet could profitably run.
- Sources (v1):
  - Reddit business subs: r/Entrepreneur, r/SideProject, r/smallbusiness, r/SaaS, r/passive_income
  - Hacker News (clean Firebase API)
  - Rotating Google searches against operator-curated seed prompts ("low competition affiliate niches 2026", "boring SaaS ideas making money", "AI agent business opportunities", etc.)
- Sources (v1.1, deferred): IndieHackers, ProductHunt — scrapable but fragile, defer until v1 stabilises.

**Output:**
- Written opportunity briefs to ~/brain/projects/aiteam/opportunities/ (NOT drafter_queue.txt — these are business ideas, not slugs).
- Weekly or bi-weekly Telegram digest with top 3-5 opportunities scored against AITEAM's mission bar: $200/mo passive · agent-runnable · low operator-touch.

**Approval pattern:**
- Different cadence than Agent A's per-slug approval.
- Operator reviews briefs and decides "build a site/agent around this?" Monthly-ish, portfolio-level.
- No drafter_queue.txt integration. Approval = operator manually spawns a new agent-build session.

**Architectural notes for the future build:**
- Google search rotation: seed-prompt YAML the operator edits without touching code (mirror tg-monitor/config.yaml pattern).
- Opportunity scoring rubric:
  ```
  revenue_potential × agent_runnability × low_competition × low_launch_time
  ```
  Sonnet emits the four factors + a weighted total per opportunity. `agent_runnability` is the hard constraint — heavily penalize opportunities requiring human sales, onboarding, or ongoing service delivery. AITEAM is fleet-runnable or it's not on the list.
- Coverage check: integrate with `internal_link_inventory` (and any future site inventories) to avoid proposing niches the fleet already covers.
- Hacker News API: Firebase, no key required, simple — start here.
- IndieHackers + ProductHunt: HTML scraping, fragile selectors — defer.
- `softwareengineering.stackexchange.com` and `workplace.stackexchange.com` have business-pain language — flag as v1.1 source candidates alongside IndieHackers/ProductHunt.
- Random-Google-search piece: fire seed prompts against Google search results (via scrape or paid API), Sonnet-extract opportunity signals from titles + snippets. Cheap and interesting.
- Storage: own SQLite DB (`~/agents/opportunity-scout/opportunities.db`), not `aiteam.db` (matches Agent A's "chatty raw data stays local" pattern).

**Estimated build:** 6-8h CC.

**Dependencies:** Agent A pattern (poll.sh fan-out, polymorphic `source_posts` table, Haiku triage → Sonnet extract → weekly Telegram digest, `/approve` Telegram handler) — D075 reuses that template, just swapping the sources and the output format (briefs vs slugs).

---

### D076 — GEO score variance threshold review

**Status:** Open (surfaced 2026-05-17 at GEO Phase 3 close-out)

**Problem:** GEO Optimizer composite scores show meaningful LLM nondeterminism on the same input. Two PASS smokes on `white-asbestos-vs-blue-asbestos.json` (verdict ids 7 and 8, ~52 hours apart) produced composites 3.88 and 3.62 respectively. Threshold is 3.0, so both passed, but the gap to the threshold (0.62) is thinner than the inter-run variance (0.26). Axis-level variance was also material: tables 1 → 3, capsule 5 → 4, citations 5 → 4. A borderline-quality slug with a true composite near 3.0 could swing from PASS to FAIL across runs purely from LLM noise, with no underlying content change.

**Risk surface:** Phase 3 writer-revision loop (commit `624d334`) bounces drafts back to drafts/ on FAIL. A noise-induced FAIL on a 3.0-borderline slug would burn an extra writer round (~$0.50 of Sonnet) and produce churn-feedback that does not reflect real content gaps. Worst case: 3 noisy rounds escalate the slug to needs-review when content was actually fine.

**Fix options (not yet decided):**
- (a) Lower threshold from 3.0 to a value that puts current burn-in composites comfortably above noise floor (e.g., 2.8). Cheap, blunt.
- (b) Score each slug N=3 times and use the median composite. Costs 3x but eliminates single-call noise. Pairs with prompt caching to keep cost reasonable.
- (c) Raise per-axis floors instead of the composite (e.g., require no axis below 2). Better aligns with the "tables=1 means real structural gap" intuition.
- (d) Calibrate empirically: run all ~30 currently-approved guides through score.sh twice, compute per-slug variance distribution, set threshold at the 5th percentile of composites minus 2-sigma of variance.

**Trigger to revisit:** After 4-slug burn-in distribution data exists (same trigger as the existing editor-gate threshold-tune item bundled in D074). Bundle the burn-in for both gates so we get the variance distributions in one pass.

**Reference:** Phase 3 close-out commit `624d334`. Verdict rows id=7 (composite 3.88) and id=8 (composite 3.62) in `auditor_verdicts`.

---

### D077 — CLAUDE.md em-dash rule vs repo precedent

**Status:** Open (surfaced 2026-05-17 at GEO Phase 3 close-out sign-off)

**Problem:** `CLAUDE.md` rule #5 reads "Never use em dashes (—) anywhere. Not in content, metadata, comments, or alt text. Operator flags em dashes as AI-generated. Use periods, commas, or restructure." Recent commit bodies in the asbestos repo contain em dashes:
- `7166c7b` (chore: D062 vestigial WRITER_MODEL/AUDITOR_MODEL indirection)
- `4ea5242` (fix: D061 ai-do.sh routing)
- `624d334` (feat: GEO Optimizer Phase 3, this session)

CC has been generating commit messages with em dashes despite the rule. The rule reads as absolute ("anywhere") but the framing ("flags em dashes as AI-generated") suggests the intent is content-facing copy where AI-detection matters, not git plumbing the operator authors privately.

**Two-way decision:**
- (a) Tighten the rule to be literal: scrub em dashes from all commit messages going forward, optionally amend recent commits to match. CC's default behavior would change accordingly.
- (b) Soften the rule scope: explicitly exempt commit messages, code comments, and other non-public artifacts. Rewrite rule #5 in CLAUDE.md to scope it to content (guide JSONs, site copy, metadata in shipped pages, alt text).

Operator decision needed; either is defensible. Recommend (b) for ergonomics (em dashes in commit prose are useful and never reach the AI-detection surface), but (a) is the safer interpretation of the current literal text.

**Trigger to revisit:** Next CLAUDE.md-touching hygiene session, or first time an external reader (rather than the operator) sees a repo commit message.

**Reference:** CLAUDE.md `DO NOT (Read This First)` rule #5. Commits `7166c7b`, `4ea5242`, `624d334`.

---

### D078 — Sonnet cost in research-opportunity digest archive header

**Status:** Deferred from research-opportunity v1 markdown-log build (2026-05-17).

**Problem:** The brain-tracked digest archive header (`~/brain/projects/aiteam/research-opportunity/digests/<date>.md`) renders all v1 fields cleanly except `Sonnet cost`, which displays `—` as a placeholder. The token + cost data exists in `~/store/aiteam.db.token_usage` but requires a per-week join filtered by `agent_id LIKE 'research-opportunity%'`. Build deferred to avoid scope creep on v1.

**Fix:** In `digest.sh`, after computing `posts_scanned`, add a parallel query against `aiteam.db.token_usage` to sum `sonnet_tokens` + `estimated_cost_usd` for the last 7 days filtered to the agent. Populate the `digest_log.sonnet_tokens` and `digest_log.estimated_cost_usd` columns (currently NULL). Header reads from `digest_log`, not from re-query at archive-write time.

**Trigger:** Bundle with the next task that touches `digest.sh` or that needs token_usage join logic elsewhere (likely a dashboard week-summary view). Cheap to add — ~30 min of CC work.

**Reference:** Pre-flight §5 Gap-2 in the markdown-log build session.

---

### D079 — Haiku CLI overhead ($0.025/call) propagation to fleet cost projections

**Status:** Deferred from research-opportunity Session 2 build (2026-05-17).

**Problem:** Session 2 cost $3.83 across ~149 Haiku triage calls = $0.026/call. Raw-token arithmetic (~700-token prompt × $0.80/M input + ~50-token output × $4.00/M = ~$0.001/call) predicts 30x lower. The gap is the `claude -p --model haiku` agent-loop wrapper: system prompt, CLAUDE.md context, tools loaded per invocation. Every Haiku-using agent in the fleet (research-opportunity, tg-monitor analyzer, librarian if/when) is mis-budgeted if planning from raw token rates alone.

**Fix:** Either (a) re-budget projections for all Haiku-using agents using $0.025/call as the floor, or (b) bypass `claude -p` for triage workloads and call the anthropic SDK directly (~30x cost reduction at the price of losing CLAUDE.md context + tool access per call — fine for stateless triage). Option (b) is the bigger ROI but is also a fleet-wide refactor.

**Trigger:** Next budget-planning session or first agent that scales triage past ~300 rows/day (where the cost differential becomes the dominant operational expense).

**Reference:** Session 2 ctx save 2026-05-17_1921, LESSONS.md "Haiku CLI overhead" entry.

---

### D080 — YouTube triage threshold tune-up (raise to 0.85 if false-positive volume becomes annoying)

**Status:** Deferred from research-opportunity Session 2 build (2026-05-17).

**Problem:** Haiku-4.5 has a persistent topic-context-leakage bias for YouTube comments. Praise comments ("Great video!", "Brilliant work") consistently score 0.70-0.80 when the parent video happens to be asbestos-related. Tightening the triage prompt (CRITICAL RULE + worked examples + stricter scoring guide) reduced but did NOT eliminate the bias — appears to be a deep-model preference Haiku-4.5 will not override via prompt-only fixes. Mitigation in v1: Sonnet extract acts as a reliable second gate and correctly downscores praise to `asbestos_relevance_score=0.10`. Digest ranking by `score × coverage_weight` buries the false positives at the bottom.

**Fix (one-line):** In `~/agents/research-opportunity/extract.sh`, add a per-source threshold lookup. For `source='youtube'`, use `youtube_relevance_threshold` from `config.yaml` (default 0.85). For other sources, keep the existing 0.5. The 0.85 band cuts virtually all praise hallucinations while preserving the 6-8 genuine pain comments per 50 that scored 0.85+ during smoke. Add the config key with a default in `config.yaml` and have extract.sh fall back to `triage.haiku_relevance_threshold` if the new key is absent (backwards-compatible).

**Trigger:** After 2-3 weekly digests (2026-05-25, 2026-06-01, 2026-06-08), if operator notices >2 obviously-praise items in any single digest. Until then, the system self-corrects via Sonnet extract; no urgent fix needed.

**Reference:** Session 2 ctx save 2026-05-17_1921, LESSONS.md "Haiku-4.5 topic-context-leakage" entry. Smoke-test data showing the 0.70-0.80 false-positive cluster: `sqlite3 ~/agents/research-opportunity/pain_points.db "SELECT printf('%.2f', haiku_relevance_score), substr(body_truncated,1,80) FROM source_posts WHERE source='youtube' AND haiku_relevance_score >= 0.65 ORDER BY haiku_relevance_score DESC LIMIT 15;"`

---

### D081 — `deploy_batch.log` 0-byte canary check after first SSG ship

**Status:** Deferred from SSG Lane C Pass 1 recon (2026-05-23).

**Problem:** `~/agents/ship-to-site/deploy_batch.log` is 0 bytes and last-modified `2026-05-16 23:00` — the moment the cron's `>>` redirect first created it. No `audit_log` rows for `actor_id='deploy_batch'` exist either. Either the `0 23 * * *` cron is not actually invoking the script, or the script exits silently on empty queues without writing any output (stdout, stderr, or audit row). Asbestos hasn't had a non-empty queue since the throttle landed, so the question hasn't surfaced.

**Fix:** After SSG Pass 2 ships its first slug via `deploy_batch.sh`, verify `~/agents/ship-to-site/deploy_batch.log` receives output. If the log is still empty post-deploy, investigate cron invocation:
- `crontab -l | grep deploy_batch` — confirm the line is present and uncommented
- Wrap the cron in `(set -x; bash deploy_batch.sh; echo exit=$?) >> deploy_batch.log 2>&1` temporarily to capture invocation evidence
- Audit `deploy_batch.sh` early-exit paths — should at minimum write an `audit_log` row for "deploy_batch_started" before any conditional exit

**Trigger:** After SSG Pass 2 ships its first slug.

**Reference:** SSG Lane C Pass 1 session 2026-05-23. Hook itself verified structurally correct (deploy_batch.sh:73-97); concern is whether the wrapping cron + log redirect chain works at all.

---

### D082 — `ship.sh` pushes commit to remote before detecting build failure

**Status:** Deferred from SSG Lane C Pass 1 recon (2026-05-23). Out of scope for Lane C.

**Problem:** 2026-05-17 incident: ship of asbestos slug `white-asbestos-vs-blue-asbestos`. Commit `3e608a5` was pushed to `mdp280028-ui/asbestoshq-site` `main`, then the local build failed (`[ship] white-asbestos-vs-blue-asbestos: build failed; rolling back`), then local rollback fired and removed the staged files. The local working tree was clean. **But the remote commit stayed.** The slug landed in `needs_review_queue.txt` and ship.sh kept skipping it on subsequent ticks (10× `[skip] ... in needs_review_queue` in ship.log), but the bad commit on `main` was never reverted.

**Fix:** Either of:
- (a) Reorder `ship.sh` to validate the local build BEFORE pushing — fail fast without touching the remote.
- (b) Extend the rollback path: if the local rollback fires after a push has happened, also push a revert commit to undo the bad commit on the remote.

(a) is simpler and matches what a careful operator would do manually. (b) is more robust against build-passes-then-CI-fails scenarios.

**Trigger:** Future `ship.sh` hardening session. Not blocking Lane C (the SSG gsc-INSERT hook fires only on successful ship; failed/rolled-back ships don't reach the INSERT).

**Reference:** SSG Lane C Pass 1 session 2026-05-23. Incident evidence in `~/agents/ship-to-site/state/shipped.log` line 1 (CI status `timeout`) and `~/agents/ship-to-site/ship.log` (build failure + rollback messages, then 10× needs_review_queue skips).

---

### D083 — SSG-fy `propose_backlinks.sh` + `keyword-registry update_registry.py`

**Status:** Deferred from SSG Lane C Pass 1 wire-up (2026-05-23).

**Problem:** The asbestos-new branch in `~/agents/ship-to-site/deploy_batch.sh` (lines 89–97) fires two follow-ons after the gsc INSERT:
- `~/agents/internal-link/propose_backlinks.sh ${slug}` — Sonnet proposes 3-5 backlinks pointing AT the newly-shipped slug. Asbestos-only; no SSG slug-aware version exists.
- `~/agents/keyword-registry/lib/update_registry.py shipped ${slug}` — hybrid source-of-truth write-through. May not handle SSG's `category/slug` slug shape correctly (was built when only asbestos's flat slugs existed).

The SSG branch added in commit `d3025ae` (Pass 1) deliberately omits both. The gsc INSERT alone is sufficient to make the first SSG slug visible to GSC; the internal-link and keyword-registry write-throughs are nice-to-haves that build coverage over time.

**Fix:** Either of:
- (a) Make `propose_backlinks.sh` and `update_registry.py` site-aware so they accept `--site ssg` (or detect site from the slug shape) and behave correctly. Then add the same two calls to the SSG branch in `deploy_batch.sh`.
- (b) Build SSG-flavored equivalents (`propose_backlinks_ssg.sh`, `update_registry_ssg.py`) if the asbestos versions are too entangled with asbestos-specific data/conventions to retrofit.

(a) is preferable — single source of truth, less drift.

**Trigger:** When SSG has shipped 5+ slugs via `deploy_batch.sh` and the internal-link / keyword-registry coverage debt starts mattering. Symptoms: SSG articles lack inbound internal links (hurts SEO crawl depth), keyword_registry_ssg.md's "live" matches don't reflect new shipped slugs.

**Reference:** SSG Lane C Pass 1 session 2026-05-23, commit `d3025ae`. Until then, SSG ships without internal-link suggestions and without registry write-through. Acceptable for the first batch — they can be backfilled by running the agents manually against shipped slugs once both are SSG-aware.

---

### D084 — `keyword_research.md` top-30-by-SV cutoff misses high-CPC vendor-review opportunities

**Status:** Deferred from SSG Lane C Pass 2 file authoring (2026-05-23).

**Problem:** `~/projects/ssg-content/content/ssg/keyword_research.md` (commit `ef32bc3` in ssg-content) ranks keywords per cluster by search volume and cuts at the top 30 per cluster. The SV-only cutoff systematically excludes high-CPC/low-SV vendor-review keywords like `moneypenny reviews` (SV=150, **CPC=$95** — the highest-CPC SSG-direct keyword surfaced across 6 Ahrefs rounds per `SMART_Master_Research_Report_2026-05-17_consolidated.md`). The drafter for `assignment-batch-001.md` (moneypenny-review page) had to source the primary keyword from the SMART report directly because `keyword_research.md` didn't carry it.

The same gap applies to other promising vendor-review keywords below the SV=200 cutoff: `answerconnect pricing` ($25 CPC, SV=200 — just at the threshold), `voicenation reviews` ($30 CPC, SV=150), `patlive reviews` ($12 CPC, SV=100), `abby connect reviews` ($10 CPC, SV=100), and likely 10–15 more across the 3 SSG clusters.

**Fix:** Add a second per-cluster section to `keyword_research.md`:

```markdown
## Cluster N — [name] sub-threshold high-CPC keywords

| Keyword | SV | KD | CPC | Intent | Status | Live slug |
```

Filter rule: include any keyword where CPC ≥ $30 OR (CPC ≥ $10 AND keyword maps to a known affiliate-enrolled vendor). Cap at 15 per cluster to avoid noise. Pull from `keyword_registry_ssg.md` (full file, not just top-30). Sub-threshold table goes after the main top-30 table per cluster, with a header note explaining the filter rationale.

**Trigger:** After first batch of vendor-review articles ships (assignment-batch-001 + 002) and operator wants visibility into what other high-CPC/low-SV opportunities exist for batch 003+. Or sooner if a research-opportunity scan surfaces a sub-threshold keyword that should have been on the drafter's radar.

**Reference:** SSG Lane C Pass 2 session 2026-05-23. File 4 (`assignment-batch-001.md`) flagged the gap when the moneypenny primary keyword had to be sourced from outside `keyword_research.md`. `SMART_Master_Research_Report` §7 (SSG-Direct Fit table) lists 24 sub-threshold keywords that map to existing or proposed SSG pages — that's the upstream source for the next rebuild.

---

## Authoring notes

- New D-items should pick the next free D-number. D050 is unused; D049 is unused. D045 collides between two items (operator's "ai-do.sh rewire" and the archived Cowen-template item). The older one should be renumbered next time DEFERRED is touched.
- F-items (F1-F23) live in `~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md`. Cross-reference rather than duplicate full context here.
- When an item ships, **mark with ✅ SHIPPED + commit SHA + date** and leave in place for one DEFERRED-touch cycle (operator sees the resolution), then prune on the next.

---

## Late additions (synced 2026-05-18)

Items discussed in sister chats during the SSG pipeline migration and asbestos deploy throttle sessions (2026-05-16 / 2026-05-17) but not written to DEFERRED.md at the time. Synced here for single-source-of-truth.

| ID | Item | Trigger | Notes |
|---|---|---|---|
| D059 | SSG deploy throttle (pattern-copy of asbestos throttle at commit 3ed57dc) | When `ssg.yaml` flips to `enabled: true` AND SSG ships its first batch | Reference: `~/brain/projects/aiteam/docs/deploy_throttle_build_2026-05-17.md`. Est. 30-45 min CC session. Building throttle before SSG ships is solving an imaginary problem. |
| D-SSG-01 | 11 remaining asbestos path references in `run-batch.sh` (all in COMMENT lines) | Bundle with D051 hygiene cleanup | Zero functional impact. Left untouched per "leave comments alone" rule to preserve line-number stability for future sessions. |
| D-SSG-02 | Operator review of `SSG_TEMPLATES.md` Cosmetic Tolerance List + Verdict Rules | Before first SSG audit run | CC authored from first principles (asbestos source files were stubs). Needs operator verification before live audit logic depends on it. |
| D-SSG-03 | Confirm `relatedLinks` slug shape in SSG content | Before first SSG batch run | CC chose category-prefixed format with no slashes: `"it-support/managed-it-services-pricing"`. 1-line edit if convention differs. |
| D-SSG-04 | Reconcile AS1-AS5 anti-sameness checks between `SSG_TEMPLATES.md` and `audit_guide.py` | When `audit_guide.py` is migrated to SSG | Two sources of truth right now; need to consolidate before SSG audit goes live. |
| D-SSG-05 ✅ SHIPPED 2026-05-17 (`1096584`) | `SSG_SITE_MAP.md` grep regex fix in `run-batch.sh` line 269 | ~~Before first SSG batch run~~ Shipped | Markdown table format uses `\|` cells; original `grep -E '^\/'` returned empty. Fixed with `sed -n` range (catalog→forbidden) + `grep '^\|'` row filter + `grep -oE '`/[a-z0-9/-]+/`'` backticked-URL extract + `tr -d '`'`. Empirical test: extracts 14 URLs (matches SITE_MAP's own canonical count). Audit_log closure row: `41E6B47B-3199-433F-8F90-ADBE52826344` (positional fields landed malformed — see D064). Latent same-bug in asbestos tracked as D063. |
| D-SSG-06 | `AUDIT_SPEC.md` SPC-1..14 (~160 lines) still has UST/asbestos vocabulary | When SSG service mode activates | Currently gated as inactive code path. Migrate vocabulary when service mode is needed. |
| D-SSG-07 | Stale "2,000-2,500 words" echo on `run-batch.sh` line ~357 | Cosmetic, no urgency | Console output only; no functional impact. |
| D-SSG-08 | `ctx.sh` has 22 asbestos references | Cosmetic cleanup, no urgency | Session-save tool. Convenience cleanup. |
| D-SSG-09 | `audit_guide.py` has 5 asbestos references in docstring/comments | Cosmetic cleanup, no urgency | No functional impact. |
