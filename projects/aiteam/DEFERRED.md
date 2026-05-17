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

### D056 — Editor verdict persistence

**Status:** Open (surfaced 2026-05-17 during verdict-persistence build)

**Problem:** The `auditor_verdicts` table shipped 2026-05-17 (`4e2fe0a` in `~/agents/`) is populated by `audit_guide.py` (commit `0e8089b` in asbestos-contractors) but the editor agent is NOT yet writing to it. Triage-agent coverage is therefore half: only mechanical-check verdicts are queryable, not editor-rubric scorecards. The original build prompt asked for both halves; editor half hit two HALT conditions and was deferred.

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
