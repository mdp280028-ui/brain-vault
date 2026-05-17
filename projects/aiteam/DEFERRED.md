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

### D045 — `ai-do.sh` rewire (operator-named, context-incomplete)

**Status:** Open

**Problem:** Operator named this as an open decision from this session, but context for the specific rewire isn't captured in any of the build reports or context-saves I read. Likely related to either: (a) the `AGENT_MAX_TURNS_OVERRIDE=1` pattern's edge cases, (b) the model-pinning verification routine (orchestrator/failure_modes.md mentions historical Opus-routing bug that invalidated step 17), or (c) something I haven't surfaced yet.

**Action:** Operator to clarify the specific change envisioned. Marked context-incomplete per the DEFERRED.md authoring rule.

**Trigger to revisit:** Next session with operator clarification.

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

### D052 — drafter.sh exit-2 vs exit-1 routing

**Status:** Open (surfaced 2026-05-16 during F2 fix sign-off review)

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

### D027 — Add `*.bak` to `~/brain/.gitignore`

(Already documented in §Infrastructure — re-listed here as a hygiene item for convenience.)

### Provenance comment block on generated `page.tsx` — operator decision

**Status:** Decided 2026-05-16 — strip via stage.sh (D048).

**Background:** The page.tsx template carries a 24-line documentation comment block. Initially shipped with substitution; this session added the strip step so generated wrappers stay byte-identical to existing shipped pages. No further action needed unless operator wants to revisit.

---

## Authoring notes

- New D-items should pick the next free D-number. D050 is unused; D049 is unused. D045 collides between two items (operator's "ai-do.sh rewire" and the archived Cowen-template item). The older one should be renumbered next time DEFERRED is touched.
- F-items (F1-F23) live in `~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md`. Cross-reference rather than duplicate full context here.
- When an item ships, **mark with ✅ SHIPPED + commit SHA + date** and leave in place for one DEFERRED-touch cycle (operator sees the resolution), then prune on the next.
