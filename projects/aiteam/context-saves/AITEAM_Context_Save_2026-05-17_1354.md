# AITEAM Context Save — 2026-05-17_1354
**Generated:** 2026-05-17T13:54:39-0700
**Since last save:** 2026-05-17 00:34:09
**Session topic:** GSC submission queue → cost-cap enforcement → POLICY Q5 lockfile → D066 hygiene audit (16 commits) → cluster #9 (D064/D060/D052/run-batch cleanup) → D061+D062 (route run-batch.sh through ai-do.sh + drop vestigial flags)

---

## Mechanical record

### Git activity since last save
```
26d3311 tune(watchdog): tighten assignment-drafter check_window to 1h
5d68876 feat(ai-do): add AI_DO_SKIP_PERMISSIONS env hook (opt-in --dangerously-skip-permissions pass-through for non-interactive cron callers)
4d45b95 fix(drafter): D052 — case-route fire_pipeline.sh rc; treat exit 2 as SKIPPED_NO_CONFIG soft skip
38ed525 feat(watchdog): wire grace_minutes into threshold math
ebb0507 fix(lib): D064 — log_to_audit.sh arg-shape validation (reject <4 args, ACTOR_TYPE with =, JSON in TARGET slot)
1053913 fix(watchdog): guard against corrupted state.json — single alert + flag dedup
f062e6a feat(telegram): land .env.example template (Group B follow-up)
de719b6 chore(gitignore): land pre-existing tg-monitor + market/curator sub-gitignores + ignore telegram/.venv
3bc2494 chore(gitignore): exclude tg-monitor runtime state + watchdog/state.json
6da6c59 docs: failure-mode docs for editor, librarian, orchestrator agents
b76832d feat(scripts): brain-autocommit.sh (daily 23:55 brain repo auto-commit+push, cron line 4)
166062f feat(orchestrator): morning_brief + weekly_review slash commands (dashboard /<cmd> dispatcher)
d106ef7 feat(analyst): operator-triggered Opus re-analyze via model_override column + mentioned_assets front-matter
50d8405 feat(market): land curator agent (curate_*.sh + parse_outlook + similarity)
14c0ff6 feat(market): land briefer agent (compose_brief.sh + outlook grid renderer)
6a547ce feat(tg-monitor): land Telegram chat reader + analyzer (cron-driven, */5 reader, 0 7 analyzer)
c5aa271 feat(telegram): land bot.js + transcribe.sh + notify.sh live mode + log_to_conversation helper
127e1b3 feat(lib): retrofit ai-*.sh + run_agent.sh to log token_usage via log_token_usage.sh
54a69bd feat(cost-cap): add $25/$50 enforcement; remove obsolete orchestrator $10 + drafter $15 caps (POLICY Q2)
7446717 feat(lib): add log_token_usage.sh — token_usage writer for ai-*.sh wrappers
a7c2c71 feat(dashboard): warroom slash commands + multi-round discuss + token-spend chart
60b2486 feat(gsc-queue): add site column + dashboard site badge (multi-site prep)
88f8d7a watchdog: add detector agent + expected schedule + state-tracked dedup
fb53d04 drafter: add per-tick heartbeat row for watchdog (D-WATCHDOG-PREP)
c56209f feat(gsc-queue): add gsc_submission_queue table, deploy_batch.sh hook, dashboard panel
```

### Files changed
```
A	editor/failure_modes.md
A	lib/check_cost_caps.sh
A	lib/log_to_conversation.sh
A	lib/log_token_usage.sh
A	lib/migrations/2026-05-17_gsc_queue_add_site_column.sql
A	lib/migrations/2026-05-17_gsc_submission_queue.sql
A	librarian/failure_modes.md
A	market/analyst/CLAUDE.md
A	market/briefer/agent.yaml
A	market/briefer/brief.sh
A	market/briefer/CLAUDE.md
A	market/briefer/compose_brief.sh
A	market/briefer/compose_prompt_template.md
A	market/briefer/failure_modes.md
A	market/briefer/render_outlook_grid.py
A	market/curator/.gitignore
A	market/curator/cowen_topic_prompt_template.md
A	market/curator/curate_casper.sh
A	market/curator/curate_cowen.sh
A	market/curator/curate.sh
A	market/curator/parse_outlook.py
A	market/curator/requirements.txt
A	market/curator/seed_taxonomy.sh
A	market/curator/similarity.py
A	orchestrator/commands/morning_brief.sh
A	orchestrator/commands/weekly_review.sh
A	orchestrator/failure_modes.md
A	scripts/brain-autocommit.sh
A	telegram/.env.example
A	telegram/bot.js
A	telegram/destructive_verbs.txt
A	telegram/package.json
A	telegram/transcribe.sh
A	tg-monitor/.gitignore
A	tg-monitor/analyzer.py
A	tg-monitor/archive_helpers.py
A	tg-monitor/config.yaml
A	tg-monitor/init_archive_db.py
A	tg-monitor/init_db.py
A	tg-monitor/reader.py
A	tg-monitor/README.md
A	tg-monitor/requirements.txt
A	watchdog/digest.sh
A	watchdog/expected_schedule.yaml
A	watchdog/lib/.gitkeep
A	watchdog/watchdog.sh
D	assignment-drafter/lib/budget_check.sh
D	assignment-drafter/state/daily_pipeline_spend.txt
M	.gitignore
M	assignment-drafter/config/asbestos.yaml
M	assignment-drafter/drafter.sh
M	assignment-drafter/lib/notify_operator.sh
M	dashboard/server.js
M	lib/ai-cheap.sh
M	lib/ai-do.sh
M	lib/ai-think.sh
M	lib/log_to_audit.sh
M	lib/log_token_usage.sh
M	lib/notify.sh
M	lib/run_agent.sh
M	market/analyst/analyze.sh
M	market/analyst/brief_casper.template.md
M	market/analyst/brief_cowen.template.md
M	market/analyst/prompt_template.md
M	ship-to-site/deploy_batch.sh
M	watchdog/expected_schedule.yaml
M	watchdog/watchdog.sh
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
|  id  |         ts          |      actor_id      |           action            |                            target                            |
|------|---------------------|--------------------|-----------------------------|--------------------------------------------------------------|
| 1159 | 2026-05-17 13:45:01 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1158 | 2026-05-17 13:30:00 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1157 | 2026-05-17 13:30:00 | assignment-drafter | drafter_tick_started        |                                                              |
| 1156 | 2026-05-17 13:15:01 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1155 | 2026-05-17 13:11:38 | manual             | deferred_closed             | D062                                                         |
| 1154 | 2026-05-17 13:11:38 | manual             | deferred_closed             | D061                                                         |
| 1153 | 2026-05-17 13:02:37 | manual             | deferred_closed             | D052                                                         |
| 1152 | 2026-05-17 13:02:20 | manual             | deferred_closed             | D052                                                         |
| 1151 | 2026-05-17 13:02:10 | manual             | deferred_closed             | D052                                                         |
| 1150 | 2026-05-17 13:02:02 | assignment-drafter | skipped_pause_flag          | drafter                                                      |
| 1149 | 2026-05-17 13:02:02 | assignment-drafter | drafter_tick_started        |                                                              |
| 1148 | 2026-05-17 13:02:02 | assignment-drafter | drafter_tick_notification   | asbestos                                                     |
| 1147 | 2026-05-17 13:00:01 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1146 | 2026-05-17 13:00:01 | assignment-drafter | drafter_tick_started        |                                                              |
| 1145 | 2026-05-17 12:57:04 | manual             | deferred_closed             | D060                                                         |
| 1144 | 2026-05-17 12:56:32 | manual             | d064_validation_test        | D064_TEST                                                    |
| 1143 | 2026-05-17 12:50:59 | manual             | deferred_closed             | D066                                                         |
| 1142 | 2026-05-17 12:48:50 | manual             | deferred_closed             | {"target":"D066","commits":16,"repos":3,"files_committed":50 |
|      |                     |                    |                             | ,"gitignore_patterns_added":8,"scratch_deleted":6,"halt_even |
|      |                     |                    |                             | ts":1,"halt_resolution":"option_1_two_followup_commits"}     |
| 1141 | 2026-05-17 12:46:35 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1140 | 2026-05-17 12:46:18 | notify             | notify_sent                 | operator                                                     |
| 1139 | 2026-05-17 12:46:17 | watchdog           | watchdog_state_corrupt      | /Users/mmm2/agents/watchdog/state.json                       |
| 1138 | 2026-05-17 12:45:55 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1137 | 2026-05-17 12:45:27 | notify             | notify_sent                 | operator                                                     |
| 1136 | 2026-05-17 12:45:27 | watchdog           | watchdog_state_corrupt      | /Users/mmm2/agents/watchdog/state.json                       |
| 1135 | 2026-05-17 12:45:01 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1134 | 2026-05-17 12:30:01 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1133 | 2026-05-17 12:30:00 | assignment-drafter | drafter_tick_started        |                                                              |
| 1132 | 2026-05-17 12:15:01 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1131 | 2026-05-17 12:00:01 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1130 | 2026-05-17 12:00:00 | assignment-drafter | drafter_tick_started        |                                                              |
| 1129 | 2026-05-17 11:54:05 | run-batch          | lock_skip                   | asbestos-contractors                                         |
| 1128 | 2026-05-17 11:54:05 | run-batch          | lock_skip                   | ssg-content                                                  |
| 1127 | 2026-05-17 11:53:52 | run-batch          | lock_skip                   | asbestos-contractors                                         |
| 1126 | 2026-05-17 11:53:16 | run-batch          | lock_stale                  | asbestos-contractors                                         |
| 1125 | 2026-05-17 11:53:16 | run-batch          | lock_skip                   | asbestos-contractors                                         |
| 1124 | 2026-05-17 11:45:01 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1123 | 2026-05-17 11:30:00 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1122 | 2026-05-17 11:30:00 | assignment-drafter | drafter_tick_started        |                                                              |
| 1121 | 2026-05-17 11:20:52 | operator           | feature_added               | cost_cap_enforcement                                         |
| 1120 | 2026-05-17 11:20:42 | assignment-drafter | skipped_pause_flag          | drafter                                                      |
| 1119 | 2026-05-17 11:20:42 | assignment-drafter | drafter_tick_started        |                                                              |
| 1118 | 2026-05-17 11:20:16 | cost-cap           | cost_cap_hard_tripped       | cost_caps                                                    |
| 1117 | 2026-05-17 11:20:16 | notify             | notify_stub                 | operator                                                     |
| 1116 | 2026-05-17 11:19:58 | cost-cap           | cost_cap_soft_tripped       | cost_caps                                                    |
| 1115 | 2026-05-17 11:19:58 | notify             | notify_stub                 | operator                                                     |
| 1114 | 2026-05-17 11:18:26 | cron               | caps_reset_dryrun_smoketest | cost_caps                                                    |
| 1113 | 2026-05-17 11:15:02 | watchdog           | watchdog_check_complete     | _summary                                                     |
| 1112 | 2026-05-17 11:15:02 | watchdog           | watchdog_alert              | orchestrator                                                 |
| 1111 | 2026-05-17 11:15:02 | notify             | notify_sent                 | operator                                                     |
| 1110 | 2026-05-17 11:02:45 | operator           | cron_entry_added            | watchdog                                                     |

### Token usage
(no token usage since last save)

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

- **GSC submission queue: site column added pre-data.** With zero rows in the table, `ALTER TABLE ... NOT NULL DEFAULT 'asbestos'` was safe. DEFAULT acts as a safety net for in-flight inserts during the brief migration window before deploy_batch.sh starts passing `site` explicitly. Rejected alternative: backfill rows later — premature given zero rows. SSG hook deferred to D065 (triggered when `ssg.yaml` flips `enabled: true`).
- **F17/F18 hard-cap pause mechanism: rewrite `.env`, not just a flag file.** Pre-flight uncovered that nothing in the codebase reads `~/store/flags/SYSTEM_PAUSED` — SYSTEM_PAUSED is consumed as an env var only via `check_kill_switches.sh` re-sourcing `.env`. Touching the flag file alone would be phantom enforcement. Chose option 1: hard-cap path `sed -i ''`s `.env` to `SYSTEM_PAUSED=true` (which IS what existing kill-switch consumers honor), AND touches the flag file as an idempotency marker. Daily 00:00 cron flips both back. Drift detection: if operator manually un-pauses .env while still over hard cap, helper re-pauses on next call and logs `cost_cap_drift` without re-firing Telegram. Rejected: (a) extend `check_kill_switches.sh` to read the flag — spec forbade; (b) patch each kill-switch consumer — violates single-source-of-enforcement principle.
- **Surgical-extraction pattern from working tree, formalized.** Hit this 4 times when committing single-feature work that lived in a file containing unrelated uncommitted changes. The dance: cp working file to `/tmp/<name>-backup`, `git checkout HEAD -- <path>`, re-apply only the feature hunks via Edit, commit, then `cp /tmp/<name>-backup <path>` to restore unrelated work. The `git diff HEAD` after restore should show zero feature-related diff lines if the extraction was clean.
- **D052 spec interpretation: chose DEFERRED.md body over prompt text.** The prompt described drafter.sh's own top-level exits as the scope, but the DEFERRED.md D052 entry was about handling `fire_pipeline.sh`'s exit-2 (soft skip, F2 follow-up). Drafter.sh has no `exit 1` calls at top level — premise of prompt-text interpretation didn't match code. Recommended + executed the DEFERRED.md fix: case-route fire_pipeline.sh rc in `maybe_fire_pipeline()`, add `SKIPPED_NO_CONFIG_SLUGS` bucket + `⚠️ KW-config missing` Telegram footer.
- **D061 routing: opt-in env hook in ai-do.sh, not a separate wrapper.** Chose option (a) — added `AI_DO_SKIP_PERMISSIONS` env hook to ai-do.sh (lines 27-31) for the cron-pipeline use case. Consistent with existing `AGENT_ID_OVERRIDE` and `AGENT_MAX_TURNS_OVERRIDE` env-hook pattern. Default 0 keeps safe behavior. Rejected: option (b) duplicate kill-switch + token-log logic in run-batch.sh; option (c) new wrapper file.
- **D066 surfaced files: separate follow-up commits, never bundle.** When Group C/E committed parent directories, 4 previously-hidden files surfaced (sub-gitignores, telegram/.env.example, telegram/.venv). Could have bundled into the gitignore commit but operator approved Option 1 (2 honest follow-up commits) per the "commit-message discipline rule, third repeat of this exact pattern this month" guideline.
- **Audit row malformed-row policy: insert sibling, never retroactively edit.** Hit this 3 times (D066 row 1142, D052 rows 1151+1152). When a caller fumbles positional args, payload lands in wrong column. Per project rule: insert a corrected sibling row with `supersedes:[bad_id]` field in payload; cite canonical row in DEFERRED.md closure; leave the bad rows in place as honest history. After D064 validation (commit `ebb0507`) shipped, no new malformed rows landed.

## Lessons learned

- **Audit row anatomy: 5-arg form is non-negotiable.** `log_to_audit.sh actor_type actor_id action target payload [correlation_id]`. If you pass 4 args with a JSON blob as the 4th, the JSON lands in TARGET, not payload_json. D064 validation (commit `ebb0507`) now rejects `$# < 4`, ACTOR_TYPE containing `=`, and TARGET starting with `{` — exits 2 with a specific stderr error. The verify column is `payload_json` (not `payload` — easy slip).
- **Exit codes through pipelines need pipefail or no pipe.** `cmd | head -5; echo $?` captures `head`'s exit, not `cmd`'s. During the D061 smoke test, my initial run showed `exit=0` when ai-do.sh was actually exiting 1 — the pipe to `head` masked it. Either use `set -o pipefail`, or capture directly: `cmd > /tmp/out 2>&1; rc=$?; cat /tmp/out`. Future smoke tests of exit codes should always isolate the capture.
- **`source .env` clobbers inline env overrides.** Discovered during F17/F18 testing: running `COST_CAP_SOFT_USD=0.5 bash check_cost_caps.sh` had no effect because the script's `source ~/agents/config/.env` reset COST_CAP_SOFT_USD to .env's value (25). Fix pattern: capture overrides into prefixed local vars BEFORE sourcing, then prefer them after: `_OVERRIDE_SOFT="${COST_CAP_SOFT_USD:-}"; source .env; SOFT="${_OVERRIDE_SOFT:-${COST_CAP_SOFT_USD:-25}}"`. Same trap applies to any wrapper that `source`s .env — check before assuming inline overrides "just work".
- **Surfaced-files-during-commit are normal in untracked dirs.** When you `git add directory/` and it commits, files INSIDE the dir that were previously masked under the parent `??` become individually visible on the next `git status`. Build for this: after the commit, re-run `git status --short` and expect new untracked entries to surface. The D066 hygiene audit hit this 4 times (sub-gitignores + template files + a stray venv).
- **Working-tree-recovery via cp backup beats `git stash`.** When extracting a single feature out of a working tree with unrelated uncommitted changes, the simplest reliable pattern is: backup-cp → checkout-HEAD → re-apply-feature → commit → restore-cp. Verify with `git diff HEAD -- <path> | grep -E "^[+-]" | grep <feature-marker>` — should be zero feature lines if the extraction succeeded. `git stash` is more fragile for partial-hunk situations and conflicts when restoring on top of new commits.
- **POLICY Q5 lockfile + pre-existing traps need adaptation.** asbestos and ssg run-batch.sh both had `trap 'stop_spinner; ...' INT` and `trap 'stop_spinner' EXIT` at line 99-100, set BEFORE `stop_spinner` is defined. Naively adding `trap 'rm -f $LOCKFILE' EXIT INT TERM` would clobber those. Adaptation: re-register combined traps after lockfile acquire (`trap 'rm -f $LOCKFILE; stop_spinner; ...' INT` etc.). Also: on lock-skip early exit (before stop_spinner is defined), do `trap - EXIT INT` to clear the pre-existing traps — otherwise you get `stop_spinner: command not found` on stderr.
- **ai-do.sh is NOT a drop-in replacement for `claude -p` in cron use.** Three differences a cron caller has to account for: (a) `--dangerously-skip-permissions` is NOT passed (cron hangs on first tool-use permission prompt); (b) `--output-format json` forces JSON capture which changes stdout shape (text goes through `python3 ... result` extraction); (c) `--max-turns "${AGENT_MAX_TURNS}"` imposes a ceiling (default 30) that may not apply to writer/auditor's read+write+revise loops. After D061 (commit `5d68876`), `AI_DO_SKIP_PERMISSIONS=1` env hook solves (a). The `AGENT_MAX_TURNS_OVERRIDE` env hook (pre-existing) provides the escape hatch for (c) if real-world data hits the ceiling.
- **Pre-flight discipline catches premise mismatches before code is touched.** The D052 spec asked about flipping `exit 1`→`exit 2` in drafter.sh but no `exit 1` calls exist there — the actual D-item body was about `fire_pipeline.sh` rc handling. Reporting "premise doesn't match" before touching code saved a no-op edit. Same shape: F17/F18 hard-cap pause mechanism premise (touch a flag file) didn't match reality (nothing reads that path) — caught and resolved before writing the helper.
- **Untracked-production-code is a process gap.** Four instances in one month (ship-to-site era, D060 trio, dashboard work, log_token_usage.sh) — D066 closed all of them in one audit but the underlying pattern is unresolved. D066 itself stays open as a meta-item even though all its in-scope files are now tracked. Process hypothesis: sister-chat CC sessions write production scripts directly into `~/agents/` without returning to `git add`; the next session inherits the file as background state and ships features on top. Audit pattern documented in D066 body.
- **`ctx.sh` mechanical record only covers `~/agents/`, not other repos.** This session's mechanical record shows ~/agents/ commits only — the asbestos-contractors + ssg-content + brain commits (substantial volume this session) aren't in the file-changed list. Cross-repo sessions need narrative sections to surface those commits explicitly.

## Operator corrections

- **"Surgically extract GSC-only" was the operator's call on commit-scope discipline.** When I surfaced the c56209f dashboard pre-existing-work dilemma, the operator chose Option 1 (surgical extraction with backup + checkout-HEAD + re-apply pattern) over bundling. This pattern then got repeated for the multi-site upgrade (commit `60b2486`), F17/F18 (no extraction needed — dashboard wasn't touched), and finally explicitly named in the F17/F18 spec: "Do not bundle this commit with pre-existing uncommitted dashboard work (commit-message discipline rule, third repeat of this exact pattern this month)".
- **"Run the command exactly as written, then verify, then HALT if malformed."** Operator's D066 closure command had a missing positional arg. Per spec, I ran it as written, caught the malformation in verify (JSON in TARGET slot), and HALTed without retroactively editing. Operator then chose Option 1 (insert corrected sibling row, leave 1142 in place). This pattern became the template for handling D052's similar issues (rows 1151, 1152).
- **"Don't downgrade ai-do.sh's character — opt-in env hook is the right pattern."** During D061 recon I floated three options (env hook in ai-do.sh, bypass entirely with new wrapper, or other). Operator approved option (a) explicitly. The opt-in default-0 pattern keeps ai-do.sh's "default work tier" character intact while providing the cron-specific escape hatch.
- **"PRIME.md needs a Read first section AND the stale Hard rules line fixed in the same commit."** When I surfaced the PRIME.md request didn't match the file's structure, operator explicitly named the bundle: "(a) it's the same file you're already editing, (b) it became stale 90 minutes ago in commit 54a69bd, and (c) leaving it is the exact doc-vs-reality drift pattern the project has been correcting all week." Honest commit message updated to name both changes.
- **"Triple-check log_token_usage.sh is untracked, not just modified."** Before committing the cost-cap feature, operator asked for `git log --follow lib/log_token_usage.sh` to confirm zero history before splitting into a standalone commit. Catches the "git status shows ?? but file was actually deleted and re-added" edge case. Verified empty → committed standalone as `7446717` before the cost-cap feature commit `54a69bd` could reference it cleanly.

## What's next

**Immediate (operator-tag dependent):**
- **D056 — editor verdict persistence.** Q7 answered in POLICY.md (editor runs after audit_guide.py as second gate). Pre-reqs from prior verdict_persistence_build_2026-05-17.md still apply: production runner decision, pass threshold env var, wiring decision now resolved by Q7. Composite_score column already reserved in `auditor_verdicts`.
- **SSG-1 — Decide SSG target domain + persona.** Blocks all 8 `claude -p` prompt rewrites in `~/projects/ssg-content/content/run-batch.sh` (now `AI_DO_SKIP_PERMISSIONS=1 bash ai-do.sh ...` after D061 commit `1864971`). All 8 prompts still reference "SmartSourceGuide" persona which may or may not be the final identity.
- **D058 — Vercel preview URLs in 20:00 preview ping.** Trigger: when the deploy throttle removal (D057) starts feeling real — preview URLs are needed before un-throttling.

**Tracked but blocked:**
- **D057 — Remove deploy throttle.** Awaits a stretch of consistent draft quality. First populated 23:00 fire is the verification event — still pending per HANDOFF.md item 0a.
- **D065 — SSG `gsc_submission_queue` INSERT hook.** Triggers when `ssg.yaml` flips `enabled: true` AND `content/ssg/approved/guides/` becomes non-empty. Pattern-copy of asbestos hook (commits `c56209f` + `60b2486`).
- **D066 — Untracked-code pattern audit.** All in-scope files now tracked, but the pattern remains. Next hygiene-pass session should run the audit script in the D066 body and triage any new instances.

**Open D-items (recently surfaced or unchanged from last save):**
- **D-SSG-01..09** — SSG-track grooming items from sister-chat sessions, awaiting SSG-1 unblock.
- **D059** — SSG deploy throttle pattern-copy from `3ed57dc`. Triggers when SSG ships its first batch.
- **D051** — `~/agents/` hygiene. Largely closed via D066 but the meta-item lingers; D066 itself supersedes it operationally.

**Closed this session (no longer in priority list):**
- ~~F17/F18 — Cost cap enforcement~~ (commit `54a69bd`, audit `7B5C2846-9281-4D6C-86F8-7930B88191C9`).
- ~~Q5 — run-batch.sh lockfile~~ (commits `ac2f721` asbestos, `7dd5c27` ssg).
- ~~D066 — untracked production code audit~~ (16 commits, audit row `F9D65187-374C-430C-95F3-D26886D26E8D`).
- ~~D060 — 3 untracked production scripts~~ (auto-closed by D066, audit row `9A3A7ABD-D427-40C1-9C94-074769567CA2`).
- ~~D064 — log_to_audit.sh validation~~ (commit `ebb0507`).
- ~~D052 — drafter exit-2 routing~~ (commit `4d45b95`, audit `E3A78F6C-CD03-4EA8-99BC-7912902B484B`).
- ~~D061 — route run-batch.sh writer/auditor through ai-do.sh~~ (commits `5d68876` ai-do.sh, `4ea5242` asbestos, `1864971` ssg; audit `0381734F-2D7F-4267-A72F-4ADBD146C4B1`).
- ~~D062 — vestigial WRITER_MODEL/AUDITOR_MODEL removal~~ (commits `7166c7b` asbestos, `c71a54c` ssg; audit `0F97332C-5E93-4821-B61E-006C3160CEF1`).
- ~~PRIME.md missing Read first section~~ (commit `af39f8d`).

**Working-tree state at session end:**
- ~/agents/ — only `M watchdog/expected_schedule.yaml` + `M watchdog/watchdog.sh` (cron-tick state from watchdog's 15-min cycle — NOT my work; these belong to the operator/sister-chat watchdog tuning session that also produced commits `26d3311`, `38ed525`, `1053913`).
- ~/projects/asbestos-contractors/ — clean
- ~/projects/ssg-content/ — clean
- ~/brain/ — clean (other than usual operator working files outside aiteam/)

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-17_0030.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-17 00:30 (POLICY.md lock + brain bookkeeping; SSG-scaffolding + D044 cleanup summaries preserved below) **Last session summary:** Brain-only bookkeeping session. Locked 7 operator-policy answers from cross-agent failure modes audit §7 into new file `~/brain/projects/aiteam/POLICY.md` (commit `6e7fb52`) — Q1 false-positives worse than false-negatives, Q2 $25/$50 daily cap, Q3 manual page-only rollback, Q4 4–20 burn-in ships, Q5 serialize run-batch.sh via lockfile, Q6 watchdog now, Q7 editor after audit_guide.py. POLICY.md unblocks D056, F17/F18, watchdog build. Closed D045 as obsolete after pre-flight showed writer/auditor already on Sonnet (`run-batch.sh:107-108`); opened successors D061 (route through ai-do.sh for kill-switch + token attribution — blocker is missing `--dangerously-skip-permissions` pass-through) and D062 (vestigial flag cleanup) in commit `9818e20`. Synced 10 D-items from sister chats into DEFERRED.md (D059 deploy throttle + D-SSG-01..09) in commit `b2a6fb0`. Added correction box to `cross_agent_failure_modes_2026-05-16.md` noting R1/R2/R3 tiering is planning-doc only with zero presence in code (commits `bcc16d5` + `38f77d0`). Doc-vs-reality audit Gaps 2 + 3 verified as zero-edit (active docs already correct). **Prior session summary (preserved):** Pipeline discovery + SSG content-pipeline scaffolding. Cloned 5 production repos from GitHub into `~/projects/`; built `~/projects/ssg-content/` as local-only fork of asbestos-contractors. Moved 101 asbestos content artifacts to `~/projects/_asbestos-reference/`. **D044 cleanup (preserved):** removed redundant `tr "'" '_'` scrub from `config-synthesizer/synth.sh#handle_failure()` after end-to-end verification confirmed shared-lib SQL-escape fix (commit 939361d) handles apostrophes. Single source of truth restored (commit c1aa9c0). Closed D045, D-SSG-05, D054.  --- 
