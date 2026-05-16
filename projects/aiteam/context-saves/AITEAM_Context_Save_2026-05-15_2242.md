# AITEAM Context Save — 2026-05-15_2242
**Generated:** 2026-05-15T22:42:31-0700
**Since last save:** 2026-05-15 20:11:39
**Session topic:** Market Scribe session 2 — built and signed off the Curator (Cowen knowledge accretion + Casper outlook-history) and the Briefer (morning Telegram roll-up), wired notify.sh through to live Telegram Bot API, added `/important` slash command + `model_override` schema + D032 Sonnet-vs-Opus blind comparison (Sonnet stays default), and shipped D043 mentioned_assets-driven ETH/SOL convergence path with 1-asset case exercised on today's real briefs. 31 audit rows (#141–#172), total session cost ~$1.20.

---

## Mechanical record

### Git activity since last save
```
8560861 diary: step 19 closure + cron schedule fix
```

### Files changed
```
M	orchestrator/write_diary.sh
```

### Diary entries since last save
- 2026-05-15.md: 2026-05-15


### Agent activity (audit_log)
| id  |         ts          |   actor_id   |          action          |                        target                         |
|-----|---------------------|--------------|--------------------------|-------------------------------------------------------|
| 172 | 2026-05-15 22:42:22 | claude       | regression_fixed         | briefer_canonical_brief_discovery                     |
| 171 | 2026-05-15 22:37:08 | briefer      | briefer_run              | aiteam_market_briefer                                 |
| 170 | 2026-05-15 22:37:08 | notify       | notify_sent              | operator                                              |
| 169 | 2026-05-15 22:36:17 | briefer      | briefer_run              | aiteam_market_briefer                                 |
| 168 | 2026-05-15 22:36:17 | notify       | notify_stub              | operator                                              |
| 167 | 2026-05-15 22:35:11 | briefer      | briefer_run              | aiteam_market_briefer                                 |
| 166 | 2026-05-15 22:35:11 | notify       | notify_stub              | operator                                              |
| 165 | 2026-05-15 22:30:40 | stonecreed   | config_change            | crontab                                               |
| 164 | 2026-05-15 22:10:36 | claude       | tuning_decision          | market_analyst_default_model                          |
| 163 | 2026-05-15 21:58:51 | bot          | slash_important_complete | cowen:FgxAe_NAh5c                                     |
| 162 | 2026-05-15 21:58:51 | analyst      | brief_generated          | cowen:FgxAe_NAh5c                                     |
| 161 | 2026-05-15 21:58:18 | bot          | slash_important_invoked  | cowen:FgxAe_NAh5c                                     |
| 160 | 2026-05-15 21:56:44 | orchestrator | diary_written            | /Users/mmm2/brain/projects/aiteam/diary/2026-05-15.md |
| 159 | 2026-05-15 21:53:26 | bot          | startup                  | bot                                                   |
| 158 | 2026-05-15 21:53:25 | bot          | shutdown                 | bot                                                   |
| 157 | 2026-05-15 21:40:59 | briefer      | briefer_run              | aiteam_market_briefer                                 |
| 156 | 2026-05-15 21:40:59 | notify       | notify_sent              | operator                                              |
| 155 | 2026-05-15 21:27:20 | briefer      | briefer_run              | aiteam_market_briefer                                 |
| 154 | 2026-05-15 21:27:20 | notify       | notify_sent              | operator                                              |
| 153 | 2026-05-15 21:25:11 | briefer      | briefer_run              | aiteam_market_briefer                                 |
| 152 | 2026-05-15 21:25:11 | notify       | notify_stub              | operator                                              |
| 151 | 2026-05-15 21:24:24 | briefer      | briefer_run              | aiteam_market_briefer                                 |
| 150 | 2026-05-15 21:24:24 | notify       | notify_stub              | operator                                              |
| 149 | 2026-05-15 21:21:18 | notify       | notify_stub              | operator                                              |
| 148 | 2026-05-15 21:11:33 | curator      | curator_run              | aiteam_market_curator                                 |
| 147 | 2026-05-15 21:08:11 | claude       | deferred_items_logged    | aiteam_market_curator                                 |
| 146 | 2026-05-15 21:08:00 | claude       | tuning_decision          | curator_group_threshold                               |
| 145 | 2026-05-15 21:08:00 | claude       | tuning_decision          | curator_similarity_threshold                          |
| 144 | 2026-05-15 21:04:07 | curator      | curator_run              | aiteam_market_curator                                 |
| 143 | 2026-05-15 20:59:11 | curator      | curator_run              | aiteam_market_curator                                 |
| 142 | 2026-05-15 20:50:30 | curator      | curator_casper_run       | aiteam_market_curator                                 |
| 141 | 2026-05-15 20:49:07 | claude       | tuning_decision          | curator_similarity_threshold                          |

### Token usage
|           model           | in_tok | out_tok | cost_usd |
|---------------------------|--------|---------|----------|
| claude-sonnet-4-6         | 54     | 23890   | 0.9195   |
| claude-opus-4-7           | 6      | 1777    | 0.1852   |
| claude-haiku-4-5-20251001 | 82861  | 448     | 0.0851   |

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

- **Curator built end-to-end across two paths.** Cowen path = LLM-heavy synthesis (per-topic Sonnet call with max-turns=1, three delimited output blocks for synthesis/evidence/predictions). Casper path = pure mechanical (parse outlook table → append rows to per-asset outlook-history). Both share a single audit row at curate.sh top level. Designed as an orchestrator + two sub-scripts to keep failure surfaces small and re-runnable.
- **Curator similarity threshold tuned twice based on calibration data.** Started at 0.90 (spec default), dropped to 0.50 after checkpoint 2 rank test showed scores cluster in 0.3-0.6 for short-query-vs-long-anchor, then dropped to 0.35 / group 0.15 after checkpoint 4 showed top-1 correct in 5/5 cases but two correct mappings (spx-secondary-correction → equities-spx-correlation at 0.38, inflation-reacceleration → inflation-outlook at 0.47) were below the 0.50 floor. Logged as audit rows #141, #145, #146.
- **Seed-stub-anchored embedding on cold start.** Each of the 28 Cowen topics gets a 1-2 sentence seed description in synthesis.md at folder creation time so similarity can discriminate before any briefs accrete. Anchor text per topic = `slug-as-words + seed_description + current_thesis`. After briefs accrete, current_thesis dominates.
- **Operator-supplied seed-description pattern note:** descriptive ("Cowen covers X using Y framework") embeds better than positional ("Cowen believes X means Y") for similarity-index purposes — captured for future seed work.
- **Briefer = deterministic stitching + small LLM narrative call.** The header, per-video sections, and outlook grids are bash-built. The LLM (one Sonnet call, max-turns=1) composes only THESIS_SHIFTS (per-topic 1-liner from updated Current Thesis) and CONVERGENCE (per-asset 4-line block). Total cost shape: $0.04-0.08 per daily brief.
- **notify.sh extended inline with the briefer build.** Direct Telegram Bot API curl, reading `BOT_TOKEN` + `ALLOWLIST_USER_IDS` from `~/agents/telegram/.env`. Stub-first → live-flip discipline; `TELEGRAM_OUTBOUND_ENABLED` kill switch retained. Every delivery attempt audit-logged as `notify_sent` / `notify_failed` / `notify_stub`. Operator confirmed phone receipt with monospace grid rendering correctly in `<pre>...</pre>`.
- **D032 Sonnet-vs-Opus verdict: Sonnet stays default.** Reasoning logged as audit row #164 with operator verdict text: Sonnet's deeper key-point reasoning chains preserve the why behind calls, and Opus tends to generate non-canonical topic slugs (`fed-chair-transition` vs `fed-policy-transition`) that don't map cleanly to curator seeds. Cost difference is real (Opus ~5× per brief steady-state, though tonight's Opus run came in CHEAPER at $0.20 than Sonnet at $0.26 due to model behavior differences), but the quality delta doesn't justify the downstream slug-compat burden.
- **Opus override is `/important <video_id>` only.** model_override column on seen_videos, consumed-on-success (cleared to NULL after successful run), produces parallel `<date>_<vid>_opus.md` file that does not displace the canonical Sonnet brief. The /important Telegram handler validates the video_id, writes the column, spawns analyze.sh in the chat queue, and sends a follow-up reply with both file paths + cost.
- **AGENT_MAX_TURNS_OVERRIDE pattern added to ai-do.sh** (mirrors AGENT_ID_OVERRIDE). Lets non-agentic text-in/text-out callers (curator topic updates, briefer compose) pin max-turns=1 without editing the wrapper or the .env. Set inline before invocation; wrapper checks after `source .env` overrides.
- **mentioned_assets frontmatter contract added to analyst (D043).** Frontmatter field `mentioned_assets: [<list>]` is a strict subset of the Casper outlook-history asset list (btc/eth/sol/xrp/total/total2/total3). Cowen rule = substantive discussion (1+ minute or directional call); Casper rule = mechanical from outlook arrows. Briefer parses this from each new brief, computes intersection across channels, and loops the convergence section per asset in fixed order. 1-asset case exercised tonight ([btc]); multi-asset waits on real-world data.
- **Briefer canonical-brief discovery filter** added mid-validation when stub run double-counted the Cowen video after the Opus brief landed earlier in the session. `grep -Ev '_(opus|haiku)\.md$'` on both Cowen and Casper discovery globs. Logged as `regression_fixed` audit row #172.

## Lessons learned

- **all-MiniLM-L6-v2 sentence-transformers scores cluster low (0.3-0.6 range) for short-query-vs-long-anchor matches.** Don't set a flat 0.50+ threshold without calibrating against real query/anchor pairs. Tonight's working values: 0.35 for slug→thesis matching, 0.15 for group-mean placement. Top-1 was correct in 5/5 of the off-threshold cases — ranking discriminates well even when absolute scores look "weak."
- **`close` is a reserved word in awk** (builtin function). Using it as a `-v` variable name (`-v close="<<<END>>>"`) produces a misleading "syntax error" at the WRONG line. Cost: the curator burned $0.28 on LLM responses we couldn't parse. Use `startmark`/`endmark` or any non-builtin name.
- **`paste -sd ", " -` on BSD/macOS uses each char as a cycling delimiter**, NOT a single multi-char delimiter. To join with ", " use `tr '\n' ',' | sed 's/,/, /g'`. Hit during briefer's first stub run on Cowen "Topics touched" output.
- **Persist LLM responses to disk BEFORE parsing** for any agent with non-trivial per-call cost. The curator's awk-bug failure burned cost on outputs that were `rm`'d on parse failure. Now every Cowen topic call writes `workspace/responses/<correlation_id>.md` before any block extraction. A parse-only bug becomes free to retry.
- **`log_to_audit.sh` embeds the payload in a single-quoted SQL string** with no escaping. Any `'` in the payload (apostrophes, embedded bash like `'_(opus)\.md$'`) breaks the INSERT. Workaround: pipe payload through `sed "s/'/''/g"` before passing. Real fix tracked as D044.
- **Opus on `claude -p` does NOT internal-multi-turn like Sonnet at max-turns=30.** Opus on the analyst prompt wrote 1.7K output tokens and stopped; Sonnet writes 12-13K of internal reasoning before producing the final brief. Result: a single Opus brief run cost less than the equivalent Sonnet run ($0.20 vs $0.26) on this workload. Cost decisions about Sonnet-vs-Opus must benchmark actual model behavior on the specific prompt, not assume "Opus is always more expensive."
- **Idempotent setup scripts must distinguish "seed" from "reset."** `seed_taxonomy.sh` unconditionally overwrites all template files — fine for the Cowen synthesis stubs but it wiped the Casper btc.md rows that the curator had already accreted. The fix is to write Casper outlook-history files only when missing OR `row_count: 0`. Tracked as D042.
- **Spec text can drift from spec body.** The curator seed-taxonomy spec said "24 topics" in the header but listed 27 in the body (then 28 after the oil-price-outlook addition). When this happens, the body is authoritative — flag and reconcile, don't average.
- **Model-override briefs are operator spot-check artifacts, not pipeline inputs.** `_opus.md` files made the briefer double-count the Cowen video the first time stub-mode ran with both Sonnet and Opus briefs in the same date. Discovery globs across the pipeline must exclude model-suffix files explicitly. Tonight's filter pattern `_(opus|haiku)\.md$` is a start; if more tiers get added, extend the pattern.
- **Telegram outbound via parse_mode=HTML is the lightest path** for our use case. `<pre>...</pre>` for monospace grids, escape only `<`, `>`, `&` in regular text. Avoid MarkdownV2 unless we want to escape every `_*[]()~>#+-=|{}.!`.
- **Cron jobs need their own outbound delivery mechanism.** grammy's `ctx.reply` only works inside inbound handlers — for cron-spawned outbound (briefer, future scheduled agents), call the Bot API directly via curl. Don't try to bridge through the long-polling bot process.
- **`AGENT_MAX_TURNS_OVERRIDE` pattern (like `AGENT_ID_OVERRIDE`)** is the right way to pin per-call max-turns for non-agentic uses. Set inline before invocation; the wrapper applies it after `source .env`. Don't edit .env globally — that affects every agent.
- **MEMORY-stored operator preferences need active recall.** Operator's MEMORY note "put output files in ~/Downloads/" was relevant to D032's blind-comparison artifacts but I didn't apply it until operator asked. Future similar phone-readable artifacts: copy to `~/Downloads/` proactively without being asked.

## Operator corrections

- **D043/D044 ID collision.** Verbatim from operator: *"Before anything else: verify whether D043 was overwritten when the D032 verdict step accidentally collided IDs."* I had used "D043" for the log_to_audit SQL-escape item in conversational prose, but operator had intended D043 for ETH/SOL convergence. Investigation showed nothing was actually written to canonical docs (no real collision); resolution was to renumber my SQL-escape claim from D043 → D044. Lesson: when claiming a D-number, check whether the operator may have intended it elsewhere first, especially when D-numbers haven't been ratified in HANDOFF/DEFERRED yet.
- **Threshold-tune authoritative direction.** Verbatim: *"Tune similarity threshold to 0.50 for checkpoint 3 onward... 0.90 was anchored to a different reference workload."* And then at checkpoint 4: *"Apply Option 1 (your recommendation): lower flat threshold from 0.50 → 0.35, group threshold from 0.50 → 0.15."* The operator explicitly named the group threshold (which I'd left ambiguous in my Option 1 wording). Pattern: when I propose options with a recommendation, the operator's acceptance often clarifies sub-parameters I conflated.
- **Seed-description embedding style coaching** (verbatim): *"Pattern note for any future seed work: descriptive ('Cowen covers X using Y framework') embeds better than positional ('Cowen believes X means Y') for similarity-index purposes."* Captured for future seed/taxonomy work.
- **notify.sh extension scope** (verbatim): *"yes, inline. Add the audit-log + HTTP error handling. Keep the stub-first → live-flip test discipline."* This locked in the stub-first → live-flip pattern as the standard for any future outbound-side infrastructure work.
- **D032 verdict reasoning** (operator's basis text, captured in audit row #164): *"Opus distills harder but generates non-canonical topic slugs that don't map cleanly to curator seeds, which would force threshold re-tune or slug normalization. Sonnet's deeper key-point reasoning chains preserve the why behind calls. Quality delta does not justify 5x cost or downstream slug-compat burden."* The decision was operator's, made by reading both briefs blind without my summary — exactly the discipline that was specified in the build spec ("DO NOT summarize or compare the briefs — operator will read both blind and decide").
- **"drop copies of these files in my downloads"** — operator asked for D032 comparison files in `~/Downloads/` rather than the brain-buried path. Reinforces the MEMORY note about phone/desk-side grading material. Going forward: proactively place comparison artifacts in `~/Downloads/` when they're for operator reading.

## What's next

**Immediate (tomorrow morning, 2026-05-16):**
- Operator runs `~/agents/market/briefer/brief.sh` manually after morning videos drop. Validates the full pipeline (scribe poll → analyst → curator → briefer) end-to-end with fresh data.
- If 1-3 days of manual runs are clean, activate D033 (cron for `*/15 * * * * poll.sh` + a `30 8 * * *` line for briefer).

**Carried-forward deferred items (with trigger conditions):**
- **D033 cron activation** — gated on 1-3 days of manual validation; tomorrow is day 1.
- **D039 outlook-history TL;DR repetition** — rows 2+ from the same Casper video repeat the TL;DR. Re-evaluate after 7+ days of real data.
- **D040 curator idempotency duplicate-row guard** — re-running `curate_casper.sh` on already-processed videos would append duplicates. Add `(date, video_id, horizon)` existence check before append. Trigger: next curator touch or first wild observation.
- **D041 curator topic fan-out** — should the curator add a Sonnet pass to propose touched topics beyond the analyst's tag list? Decide after 30 days of runs show under-tag frequency.
- **D042 seed_taxonomy.sh Casper preserve guard** — script should not clobber `~/brain/expertise/casper/outlook-history/*.md` files that have `row_count > 0`. 5-min fix, do at next curator touch.
- **D044 log_to_audit.sh SQL-escape** — add `sed "s/'/''/g"` in the helper before SQL embed. 3-line fix. Do at next chassis touch.

**Carried-forward from prior sessions:**
- **D025** orchestrator permission policy for Telegram-spawned `claude -p` sessions
- **D026** Phase C launchd auto-restart for the Telegram bot (bot is currently `nohup`'d, dies if Mac reboots)
- **D027** Add `*.bak` to `~/brain/.gitignore`
- The 6 known bugs from session 1's HANDOFF (per-agent kill-switch defense-in-depth gap, PUBLISHER_DEPLOY_ENABLED has no enforcement, DAILY_API_BUDGET_USD observability-only, historical audit_log trailing-`}` rows, `.bak` files in brain repo, uncommitted Phase 2 telegram-stack work) — all carried.

**Blocked:**
- **D043 ETH/SOL multi-asset convergence code path** is built but only exercises the 1-asset branch tonight. Will get exercised organically when Cowen next mentions ETH or SOL (or when Casper expands its outlook template beyond BTC/ETH). No action needed.
- Phase 3 (silver platters, kanban) gated on: first AITEAM site live + GA4 data, `mission_tasks` having rows. No internal blockers.

**Architectural questions parked for future operator-led design:**
- Should the analyst constrain its slug vocabulary to the curator's seed taxonomy explicitly (vs the soft "use existing seeds when applicable" rule in `analyst/CLAUDE.md`)? Decide if /important Opus usage rises.
- Multi-asset convergence section will get more interesting when intersections grow (e.g. Cowen + Casper both touching BTC+ETH+SOL). The compose prompt's per-asset block format works structurally but the synthesis quality at 3+ assets is unproven.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_2010.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-15 19:56 **Last session summary:** Market Scribe + Analyst session 1 — built and signed off the YouTube polling + transcript-fetching + brief-generating pipeline for two channels (Cowen rich, Casper lean). 10 steps shipped in one session with live smoke test producing one brief per channel. `session_1_complete` audit row #140 at 19:55:56. Total session cost $0.33. Curator + Briefer + cron activation deferred to session 2.  --- 
