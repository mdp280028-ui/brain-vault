# AITEAM Context Save — 2026-05-17_1736
**Generated:** 2026-05-17T17:36:40-0700
**Since last save:** 2026-05-17 17:35:36
**Session topic:** Internal Link Agent v1 — inventory + audit-links + backlink-proposals (this chat)

**Note on mechanical record:** The 17:35 save (`AITEAM_Context_Save_2026-05-17_1735.md`, generated 60s before this one) captured the actual commits + files from this session — see that save's mechanical-record section. This 1736 save's mechanical record is empty because nothing happened in the 60-second window between the two ctx fires. The 1735 save's narrative slots are empty (sister-chat will fill them); this save covers the internal-link-agent slice of the day's work.

---

## Mechanical record

### Git activity since last save
```
(no commits since last save)
```

### Files changed
```
(no file changes)
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
(no audit rows since last save)

### Token usage
(no token usage since last save)

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

**HALT 1 — apply_approval re-ship trigger.** Three options for "who triggers ship-to-site after a /approve writes new suggestedLinks into the source JSON". Picked **(b) automatic via deploy_batch.sh queue**: `apply_approval.sh` appends source_slug to `~/agents/ship-to-site/state/updated_slugs_queue.txt`; next 23:00 `deploy_batch.sh` consumes the queue and re-ships. Rejected (a) manual (too slow) and (c) immediate ship.sh call (commit spam if many approvals fire in quick succession). The 23:00 throttle is already the right cadence; no new cron added.

**HALT 2 — bot.js extension.** Added `/approve <id>` and `/reject <id>` slash command handlers to `~/agents/telegram/bot.js` (operator's prompt referenced `~/agents/grammy-bot/bot.js` but that path doesn't exist; canonical is `~/agents/telegram/bot.js`). Mirrors the `/halt_shipping` + `/morning_brief` patterns exactly: allowlist middleware, `cancelPendingDestructiveIfAny`, `logConversation`, `logAudit`, `spawn` of `apply_approval.sh`. /help updated. Live Telegram test deferred (bot was off) — Option 1 for the 10.5 verification gap.

**HALT 3 — per-proposal Telegram approval intentionally diverges from ship-to-site's "auditor is the gate" convention.** Documented in `~/agents/internal-link/CLAUDE.md`. Rationale: backlink proposals are NEW unreviewed agent output proposing edits to already-approved guides. Different stage, different gate. Ship-to-site's no-approval rule applies to content that already passed audit_guide.py + editor.

**HALT 4 — cost budget.** `propose_backlinks.sh` empirical cost in smoke: **$0.0746/call** at 31 candidate slugs, 5 proposals, ~3-5K input tokens. Cap stayed at $0.50/call (override via `PROPOSE_BACKLINKS_COST_CAP`). Build cap $5 — used 1.5%. Operator flagged for monitoring after 5 real deploy cycles.

**Step 13 audit-gate strictness — picked Advisory.** `audit_links.sh` runs after audit_guide.py passes in run-batch.sh. Failures land in `audit_link_failures` + audit_log; approval/ship proceed. Rejected hard-gate (false-positives expected on cross-batch relatedLinks references where target slug is still in drafts/). Rejected defer-to-v1.1 (the table already exists, the script already works, advisory adds zero risk to ship flow).

**Step 14 close — picked visual-render check via local `npm run build`.** Copied chrysotile.json (with first approved suggestedLinks) into src/data/guides/, ran `next build`, grepped the prerendered HTML, confirmed the "Related Reading" section renders with the italic anchor "white asbestos vs blue asbestos" → /guides/white-asbestos-vs-blue-asbestos/. Reverted the smoke-test stage copy before committing; the real pipeline (ship.sh + stage.sh) will re-stage at next 23:00 deploy. Telegram round-trip + production deploy deferred.

**Smoke-test data retained as real first approval.** Proposal #1 (chrysotile → white-asbestos-vs-blue-asbestos) was approved via apply_approval.sh during Step 10 smoke. Sonnet's top pick was the chrysotile-EPA-ban guide pairing — high-relevance, sensible. Operator chose option (a) keep + commit it. Proposals #2-4 remain in `proposed` status for later operator decision.

## Lessons learned

- **`ctx.sh` last-save check window can be sub-minute.** If two ctx fires happen within 60s, the second save's mechanical record is empty even though the prior save just rolled up the whole multi-hour session. When this happens, fill the LATER save's narrative + cross-reference the earlier save for the mechanical record, rather than trying to retroactively trample the earlier save. (Encountered during this very save.)

- **bash 3.2 is the Mac Mini default — no `${var@Q}`, no associative arrays.** Initial draft of `build_inventory.sh` used `${slug@Q}` quoting and would have silently dropped quoting on apostrophes. Rewrote using env-var-pass to `python3` heredoc (matches existing `log_token_usage.sh` convention). All new agent scripts should follow this pattern.

- **Operator's spec paths can be aspirational.** Build prompt referenced `~/agents/grammy-bot/bot.js` (doesn't exist) and `~/projects/asbestos-contractors/run-batch.sh` (actually at `content/run-batch.sh`). Always verify path existence with `ls` BEFORE editing — don't trust the spec verbatim.

- **`updated_slugs_queue.txt` consumption needs a 'kind' marker.** `deploy_batch.sh` originally fired `gsc_submission_queue` INSERT + (now) `propose_backlinks.sh` per success — both designed for first-time deploys. A refresh ship (suggestedLinks edit) must skip both: gsc INSERT is no-op (UNIQUE url), but `propose_backlinks.sh` would re-Sonnet against an existing slug, wasting $0.07. Added 3rd column to PAIRS_FILE: 'new' vs 'refresh' to gate side effects.

- **ship.sh refuses to ship if site_repo is dirty.** Anything that runs after touching `~/projects/asbestoshq-site/` (e.g., the `GuideArticle.tsx` edit in this session) must be committed before deploy_batch.sh can fire. Order of operations matters: commit-then-ship, not ship-then-commit.

- **Sister-chat continued shipping commits to `~/agents/` during this session.** New commits seen: `6895dcc` (editor score.sh + CLAUDE.md), `dc05c51` (editor threshold tune), `1323040` (editor verdict gate in ship_one_slug), `0335838` + `32b3ea3` (GEO Optimizer Phase 1 + Phase 2). My commits stacked cleanly on top because the lanes don't overlap. The D066-style "watch for sister-lane collisions" caution remained necessary — both `bot.js` and `deploy_batch.sh` are shared infrastructure that sister could plausibly have edited.

- **`npm run build` is fast enough (~30s) to be a reliable type-check + render-check before committing UI edits.** Prerendered HTML lives in `.next/server/app/guides/<slug>.html`; grepping it confirms both the markup is generated AND the props serialize correctly. Better than `tsc --noEmit` alone (catches React rendering bugs in addition to type errors).

## Operator corrections

- **Operator interrupted the AskUserQuestion at HALT 3/4 to give direct GO with explicit decisions inline.** No re-derivation requested. Pattern: operator prefers single-message GO with all branch decisions over multi-question prompts when the path is well-understood.

- **"Defer to Step 14" was the right call for the bot.js handler verification gap.** Operator preferred shipping with documented partial-pass over the operational risk of starting the bot mid-session.

- **Step 14 close: operator picked the visual-render variant over the "skip + trust type-check" variant.** Explicit choice = visual verification matters more than build speed.

## What's next

**Immediate (next session, when bot is started):**
1. Start `~/agents/telegram/bot.js` (`node bot.js` or however the launchd plist will be set up — D026 is still open).
2. Send `/help` — verify `/approve <id>` and `/reject <id>` appear in the command list.
3. Approve or reject proposals #2, #3, #4 via Telegram. Each `/approve` should: append source_slug to `updated_slugs_queue.txt`, edit the source JSON, audit-log row.
4. Wait for (or manually fire) `~/agents/ship-to-site/deploy_batch.sh` — verify the 'refresh' code path consumes the queue, re-ships affected slugs via ship.sh, commits + pushes to asbestoshq-site. **No propose_backlinks fires for refresh; no gsc_submission_queue INSERT.**
5. Eyeball https://www.asbestoshq.com/guides/chrysotile/ — confirm "Related Reading" section renders below "Related Guides" with the italic styling.

**Deferred (v1.1 / v2):**
- Batch approval flow (3-5 proposals/day in one Telegram message) — v1.1.
- `suggest_for_draft.sh` (writer-phase suggestion path) — v2.
- Internal-link cost-monitoring rollup after first 5 real deploy cycles. If avg propose cost > $0.50/cycle, halve candidate set or move screening pass to Haiku.

**Blocked:**
- Live `/approve` + `/reject` Telegram round-trip — blocked on bot-startup state.
- Production deploy of chrysotile's first suggestedLinks — blocked on next 23:00 cron (or manual deploy_batch.sh fire).

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-17_1735.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-17 14:24 (watchdog deployment + 4 hardening/tuning fixes + SSG scoping recon + SSG site-repo locator) **Last session summary:** Watchdog agent built and activated. Drafter heartbeat row (`fb53d04`); watchdog scaffold + expected_schedule.yaml + state-tracked dedup (`88f8d7a`); brain incidents/ scaffold (`61ea5d1`); cron activated at 11:02 PT (audit id 1110); first production tick fired 11:15:00, alerted exactly once on orchestrator (real signal — `diary_written` missing in 26h). Four follow-on fixes landed: state-corrupt guard with `jq -e 'type == "object"'` + flag-file dedup (`1053913`); grace_minutes wired into all 4 threshold sites with A/B-proven flip evidence (`38ed525`); drafter check_window tightened 2h→1h, now catches the second missed `*/30` tick (`26d3311`); LESSONS.md cross-reference + heartbeat-before-killswitch validation (`1e90165` brain). Read-only follow-on: SSG content-pipeline scoping recon — `ssg-content/` machinery is fork-complete but zero content inputs exist; `ssg.yaml: enabled: false` with explicit TODO listing 4 prereq files; D-SSG-02/03 sign-offs still pending; the `AITEAM_SSG_DataDriven_Rewrite_Plan*.md` doc referenced in operator's spec is not on disk (path `/mnt/project/` is Linux-only, doesn't exist on macOS). SSG site repo locator: NOT lost — at `~/projects/smartsourceguide/` (remote `mdp280028-ui/smartsourceguide`, last commit `c05c1f6` 2026-05-16, all 14 canonical slugs from SITE_MAP.md present). Watchdog now running 4 ticks deep on the new code; production behavior matches dry-run exactly. **Prior session summary (preserved):** Heavy execution session. Shipped GSC submission queue + multi-site upgrade (commits `c56209f` + `60b2486`). Committed substantial pre-existing dashboard work via surgical extraction (`a7c2c71`). Built F17/F18 cost-cap enforcement with `$25/$50` POLICY Q2 caps — `~/agents/lib/check_cost_caps.sh` wired after every `token_usage` INSERT; hard-cap path rewrites `.env` `SYSTEM_PAUSED=true` (the actual enforcement signal) AND touches a flag file as idempotency marker; daily 00:00 cron flips both back; removed obsolete orchestrator $10 + drafter $15 caps; commit `54a69bd`. Updated PRIME.md with Read first section + dropped stale DAILY_API_BUDGET_USD rule (`af39f8d`). Shipped POLICY Q5 per-site lockfile in both run-batch.sh forks (`ac2f721` asbestos, `7dd5c27` ssg). Executed D066 untracked-code hygiene audit — 16 commits across 3 repos committed ~50 untracked production files in 12 logical groups (A–L), 8 gitignore patterns added, 6 scratch files deleted. Closed cluster #9: D064 arg-shape validation in `log_to_audit.sh` (`ebb0507`), D060 auto-closed by D066, D052 fire_pipeline.sh rc case-routing (`4d45b95`), run-batch.sh dead-code removal in both repos (`082e957` + `56a86e0`). Closed D061+D062: routed all 16 writer/auditor callsites through ai-do.sh via new `AI_DO_SKIP_PERMISSIONS` env hook (`5d68876` ai-do.sh + `4ea5242`/`1864971` run-batch.sh); removed vestigial `WRITER_MODEL`/`AUDITOR_MODEL` indirection (`7166c7b`/`c71a54c`). 1 mid-session HALT (D066 group C/E surfaced 4 untracked files post-commit — resolved cleanly via Option 1: 2 follow-up commits). 3 audit-row malformed-row corrections via supersedes-pattern (D066/D052) — D064 validation now blocks recurrence. **Prior session summary (preserved):** Brain-only bookkeeping session. Locked 7 operator-policy answers from cross-agent failure modes audit §7 into POLICY.md (commit `6e7fb52`). Closed D045 as obsolete; opened D061 + D062. Synced 10 D-items from sister chats into DEFERRED.md. Added correction box to `cross_agent_failure_modes_2026-05-16.md`. **Prior-prior session (preserved):** Pipeline discovery + SSG content-pipeline scaffolding. Cloned 5 production repos from GitHub into `~/projects/`; built `~/projects/ssg-content/` as local-only fork.  --- 
