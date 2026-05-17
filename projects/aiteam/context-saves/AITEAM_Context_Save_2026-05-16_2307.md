# AITEAM Context Save — 2026-05-16_2307
**Generated:** 2026-05-16T23:07:50-0700
**Since last save:** 2026-05-16 00:17:45
**Session topic:** Verdict persistence layer (foundation for future Escalation Triage agent) + deploy throttle with 20:00 preview / 23:00 batch fire. Two of the three spec'd builds shipped; one (Escalation Triage proper) couldn't proceed because its upstream R3 schema doesn't exist in the codebase yet.

---

## Mechanical record

### Git activity since last save
```
3ed57dc feat(ship-to-site): throttle to 23:00 daily deploy + 20:00 preview ping (temporary review-window scaffolding)
0be9d1f Fix ship-to-site cron npm PATH bug
4e2fe0a feat(lib/migrations): auditor_verdicts table for verdict persistence layer
0df8cf6 Fix F2: pre-flight keyword-config check in fire_pipeline.sh
939361d Fix F4/D044: SQL-escape single quotes in log_to_audit.sh
3b43c11 ship-to-site: initial commit + strip template provenance before stage
69dd30f Add config-synthesizer agent
```

### Files changed
```
A	assignment-drafter/.gitignore
A	assignment-drafter/agent.yaml
A	assignment-drafter/CLAUDE.md
A	assignment-drafter/config/asbestos.yaml
A	assignment-drafter/config/ssg.yaml
A	assignment-drafter/drafter.sh
A	assignment-drafter/failure_modes.md
A	assignment-drafter/lib/budget_check.sh
A	assignment-drafter/lib/fire_pipeline.sh
A	assignment-drafter/lib/generate_draft.sh
A	assignment-drafter/lib/load_context.sh
A	assignment-drafter/lib/mark_complete.sh
A	assignment-drafter/lib/notify_operator.sh
A	assignment-drafter/lib/parse_queue.sh
A	assignment-drafter/lib/validate.sh
A	assignment-drafter/prompts/asbestos_drafter_prompt.md
A	assignment-drafter/README.md
A	assignment-drafter/state/.gitkeep
A	assignment-drafter/state/daily_pipeline_spend.txt
A	assignment-drafter/templates/assignment_batch_template.md
A	config-synthesizer/.gitignore
A	config-synthesizer/agent.yaml
A	config-synthesizer/CLAUDE.md
A	config-synthesizer/config/asbestos.yaml
A	config-synthesizer/config/ssg.yaml
A	config-synthesizer/failure_modes.md
A	config-synthesizer/lib/extract_seeds.sh
A	config-synthesizer/lib/notify.sh
A	config-synthesizer/lib/synthesize.sh
A	config-synthesizer/lib/validate.py
A	config-synthesizer/lib/validate.sh
A	config-synthesizer/README.md
A	config-synthesizer/state/.gitkeep
A	config-synthesizer/state/failed/.gitkeep
A	config-synthesizer/synth.sh
A	config-synthesizer/templates/keyword-config-schema.json
A	lib/migrations/2026-05-17_auditor_verdicts.sql
A	ship-to-site/.gitignore
A	ship-to-site/agent.yaml
A	ship-to-site/CLAUDE.md
A	ship-to-site/config/asbestos.yaml
A	ship-to-site/config/ssg.yaml
A	ship-to-site/deploy_batch.sh
A	ship-to-site/failure_modes.md
A	ship-to-site/lib/build_verify.sh
A	ship-to-site/lib/digest.sh
A	ship-to-site/lib/git_ops.sh
A	ship-to-site/lib/rollback.sh
A	ship-to-site/lib/stage.sh
A	ship-to-site/lib/validate.sh
A	ship-to-site/preview_ping.sh
A	ship-to-site/README.md
A	ship-to-site/ship.sh
A	ship-to-site/state/.gitkeep
A	ship-to-site/templates/page-tsx-template.tsx
M	lib/log_to_audit.sh
M	ship-to-site/lib/build_verify.sh
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
|  id  |         ts          |   actor_id   |        action        |                  target                  |
|------|---------------------|--------------|----------------------|------------------------------------------|
| 1097 | 2026-05-16 23:00:05 | ship-to-site | deploy_batch_empty   | _empty                                   |
| 1096 | 2026-05-16 23:00:05 | ship-to-site | ship_to_site_skipped | white-asbestos-vs-blue-asbestos          |
| 1095 | 2026-05-16 23:00:05 | ship-to-site | ship_to_site_skipped | when-was-asbestos-used-in-homes          |
| 1094 | 2026-05-16 23:00:04 | ship-to-site | ship_to_site_skipped | what-does-asbestos-siding-look-like      |
| 1093 | 2026-05-16 23:00:04 | ship-to-site | ship_to_site_skipped | vermiculite-insulation-guide             |
| 1092 | 2026-05-16 23:00:04 | ship-to-site | ship_to_site_skipped | transite-pipe-guide                      |
| 1091 | 2026-05-16 23:00:04 | ship-to-site | ship_to_site_skipped | is-popcorn-ceiling-asbestos              |
| 1090 | 2026-05-16 23:00:04 | ship-to-site | ship_to_site_skipped | how-to-test-popcorn-ceiling-for-asbestos |
| 1089 | 2026-05-16 23:00:04 | ship-to-site | ship_to_site_skipped | how-to-dispose-of-asbestos               |
| 1088 | 2026-05-16 23:00:04 | ship-to-site | ship_to_site_skipped | house-built-1976-asbestos                |
| 1087 | 2026-05-16 23:00:03 | ship-to-site | ship_to_site_skipped | friable-vs-nonfriable-asbestos           |
| 1086 | 2026-05-16 23:00:03 | ship-to-site | ship_to_site_skipped | does-plaster-have-asbestos               |
| 1085 | 2026-05-16 23:00:03 | ship-to-site | ship_to_site_skipped | chrysotile                               |
| 1084 | 2026-05-16 23:00:03 | ship-to-site | ship_to_site_skipped | black-mastic-guide                       |
| 1083 | 2026-05-16 23:00:03 | ship-to-site | ship_to_site_skipped | asbestos-vs-fiberglass                   |
| 1082 | 2026-05-16 23:00:03 | ship-to-site | ship_to_site_skipped | asbestos-under-carpet                    |
| 1081 | 2026-05-16 23:00:02 | ship-to-site | ship_to_site_skipped | asbestos-tile-guide                      |
| 1080 | 2026-05-16 23:00:02 | ship-to-site | ship_to_site_skipped | asbestos-siding-removal-cost             |
| 1079 | 2026-05-16 23:00:02 | ship-to-site | ship_to_site_skipped | asbestos-siding-guide                    |
| 1078 | 2026-05-16 23:00:02 | ship-to-site | ship_to_site_skipped | asbestos-shingles-guide                  |
| 1077 | 2026-05-16 23:00:02 | ship-to-site | ship_to_site_skipped | asbestos-roof-removal                    |
| 1076 | 2026-05-16 23:00:02 | ship-to-site | ship_to_site_skipped | asbestos-remediation-cost                |
| 1075 | 2026-05-16 23:00:02 | ship-to-site | ship_to_site_skipped | asbestos-popcorn-ceiling-vs-non-asbestos |
| 1074 | 2026-05-16 23:00:01 | ship-to-site | ship_to_site_skipped | asbestos-popcorn-ceiling-removal-cost    |
| 1073 | 2026-05-16 23:00:01 | ship-to-site | ship_to_site_skipped | asbestos-pipe-guide                      |
| 1072 | 2026-05-16 23:00:01 | ship-to-site | ship_to_site_skipped | asbestos-inspection-cost                 |
| 1071 | 2026-05-16 23:00:01 | ship-to-site | ship_to_site_skipped | asbestos-glue                            |
| 1070 | 2026-05-16 23:00:01 | ship-to-site | ship_to_site_skipped | asbestos-floor-tile-removal              |
| 1069 | 2026-05-16 23:00:01 | ship-to-site | ship_to_site_skipped | asbestos-duct-wrap                       |
| 1068 | 2026-05-16 23:00:00 | ship-to-site | ship_to_site_skipped | asbestos-drywall-guide                   |
| 1067 | 2026-05-16 23:00:00 | ship-to-site | ship_to_site_skipped | asbestos-ceiling-tile-guide              |
| 1066 | 2026-05-16 23:00:00 | ship-to-site | ship_to_site_skipped | asbestos-air-quality-test                |
| 1065 | 2026-05-16 23:00:00 | ship-to-site | ship_to_site_skipped | asbestos-abatement-near-me               |
| 1064 | 2026-05-16 22:49:45 | ship-to-site | deploy_batch_empty   | _empty                                   |
| 1063 | 2026-05-16 22:49:45 | ship-to-site | ship_to_site_skipped | white-asbestos-vs-blue-asbestos          |
| 1062 | 2026-05-16 22:49:45 | ship-to-site | ship_to_site_skipped | when-was-asbestos-used-in-homes          |
| 1061 | 2026-05-16 22:49:45 | ship-to-site | ship_to_site_skipped | what-does-asbestos-siding-look-like      |
| 1060 | 2026-05-16 22:49:45 | ship-to-site | ship_to_site_skipped | vermiculite-insulation-guide             |
| 1059 | 2026-05-16 22:49:45 | ship-to-site | ship_to_site_skipped | transite-pipe-guide                      |
| 1058 | 2026-05-16 22:49:45 | ship-to-site | ship_to_site_skipped | is-popcorn-ceiling-asbestos              |
| 1057 | 2026-05-16 22:49:45 | ship-to-site | ship_to_site_skipped | how-to-test-popcorn-ceiling-for-asbestos |
| 1056 | 2026-05-16 22:49:45 | ship-to-site | ship_to_site_skipped | how-to-dispose-of-asbestos               |
| 1055 | 2026-05-16 22:49:45 | ship-to-site | ship_to_site_skipped | house-built-1976-asbestos                |
| 1054 | 2026-05-16 22:49:44 | ship-to-site | ship_to_site_skipped | friable-vs-nonfriable-asbestos           |
| 1053 | 2026-05-16 22:49:44 | ship-to-site | ship_to_site_skipped | does-plaster-have-asbestos               |
| 1052 | 2026-05-16 22:49:44 | ship-to-site | ship_to_site_skipped | chrysotile                               |
| 1051 | 2026-05-16 22:49:44 | ship-to-site | ship_to_site_skipped | black-mastic-guide                       |
| 1050 | 2026-05-16 22:49:44 | ship-to-site | ship_to_site_skipped | asbestos-vs-fiberglass                   |
| 1049 | 2026-05-16 22:49:44 | ship-to-site | ship_to_site_skipped | asbestos-under-carpet                    |
| 1048 | 2026-05-16 22:49:44 | ship-to-site | ship_to_site_skipped | asbestos-tile-guide                      |

### Token usage
|           model           | in_tok | out_tok | cost_usd |
|---------------------------|--------|---------|----------|
| claude-sonnet-4-6         | 25     | 29949   | 0.7461   |
| claude-haiku-4-5-20251001 | 45659  | 161     | 0.0465   |

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

- **Halt the Escalation Triage build entirely.** Build prompt described an `audit_log.tier='R3'` query against a drafter→auditor pipeline with R1/R2/R3 verdicts. Recon proved none of that exists: no `tier` column, no R-tier terminology anywhere, no production scorer that persists verdicts. Operator pivoted to "build the persistence foundation first," which became this session's main work. Rejected: stubbing an `r3_queue` table just to satisfy the spec (would have produced dead schema with no upstream writer).
- **Ship verdict persistence as two halves; only the audit_guide half this session.** Editor agent (`~/agents/editor/`) is calibration-only — no production runner, no documented pass threshold. Inserting persistence into `run_tier_test.sh` would have polluted `auditor_verdicts` with 30+ calibration rows per run mixed with real-draft verdicts. Halted on editor; shipped audit_guide.py persistence; tracked editor-half as D056 with three explicit pre-reqs (production runner, pass threshold, wiring decision via Q7). Operator chose this split.
- **Use `ship.sh --dry-run` as the queue source for both throttle scripts.** `ship.sh:189-194` already prints `[would-ship] <site>: <slug>` for the eligible set via the same `process_site()` → `ship_one_slug()` path the live tick uses. Calling it from both `preview_ping.sh` (20:00) and `deploy_batch.sh` (23:00) means the 20:00 message and the 23:00 fire literally cannot drift — same code path produces both. Rejected: re-implementing the eligibility filter (glob minus whitelist minus needs-review) inline, which would have created a second source of truth.
- **Throttle is cron-only, no agent code changes.** Comment out the old `*/15 ship.sh` line (don't delete), add `0 20` + `0 23` lines. ship.sh itself is unchanged — both new scripts wrap it rather than reimplementing the deploy. Reversal is a 3-line crontab edit. Picked this over a feature-flag inside ship.sh because the throttle is explicitly temporary scaffolding and cron-level toggling needs no code review.
- **Ship preview ping without per-slug preview URLs.** Spec assumed Cloudflare Pages preview URLs; this codebase uses Vercel auto-deploy from `main` with no preview-URL machinery. Operator chose "ship without URLs" over the three alternatives (production URL with 404 caveat, real branch-based Vercel preview deploys, local screenshot). Tracked as D058 for later.
- **Defer the populated deploy_batch.sh test to first real 23:00 fire.** Staging a fake slug into `approved/guides/` would publish to production. Operator chose to verify via the real first fire instead of contaminating data.
- **Use D056/D057/D058 for the three new DEFERRED items** even though the build prompt requested D053. D053-D055 were already taken; followed the DEFERRED.md authoring rule ("New D-items should pick the next free D-number").

## Lessons learned

- **Build specs sometimes describe systems that don't exist yet.** The Escalation Triage spec assumed an `audit_log.tier='R3'` schema and a drafter→auditor pipeline producing R-tier verdicts. None of that existed in the codebase. Pattern: when a spec references specific table columns, agent directories, or audit-log actions, `grep` for them BEFORE accepting the premise. A 10-minute recon prevents hours of building against a phantom upstream.
- **When the upstream provides a `--dry-run`, use it as the queue source for any wrapper.** `ship.sh --dry-run` emits exactly the eligible set the live tick would process. Calling it from both `preview_ping.sh` and `deploy_batch.sh` guarantees they can't drift — same code path produces the list for both. Rejected re-implementing the filter inline; that would have created a second source of truth that future-me would forget to update in lockstep.
- **`ship.sh` always exits 0 even when per-slug deploys fail.** `process_site()` swallows `ship_one_slug` failures with `|| true` (ship.sh:321). Wrappers that loop over slugs and need per-slug success cannot rely on the exit code — must parse stdout for `[ship] <slug>: shipped` vs `[ship] <slug>: stage failed` / etc. Documented this in `deploy_batch.sh`'s header comment.
- **`log_to_audit.sh` takes POSITIONAL args, not `--flag` style.** Signature is `<actor_type> <actor_id> <action> <target> <payload_json> [correlation_id]`. The verdict-persistence build spec showed `--actor-type X --actor-id Y …` (flag style); that's wrong. Convention is positional across `ship.sh`, `run_tier_test.sh`, etc. Specs that show `--flag` calls to this helper are spec drift, not API.
- **`audit_guide.py` writes `composite_score=NULL`** because mechanical checks don't produce a composite — only the editor's 5-axis rubric does. Any UI/message rendering composite for an audit_guide row must handle NULL specifically (em-dash, not "0", since 0 would be misleading). preview_ping.sh renders `—`.
- **DEFERRED.md D-numbers collide silently.** Build prompts that ask for "add D053 …" without checking the file are common; D053-D055 in this case were already taken. Pattern: `grep -nE "^### D[0-9]+" DEFERRED.md | tail` before adding a new entry, and follow the file's own "pick the next free D-number" rule. Also flag the spec drift in the build report so the operator knows the rename happened.
- **Approved-guide JSONs store body content in `.paragraphs[]`, NOT `.allText`.** `.allText` exists in the schema but is empty string (deprecated or unused). Word count via `jq -r '.paragraphs | join(" ")' "$file" | wc -w` works; `jq -r '.allText'` returns 0 words. Confirm structure with `jq 'keys'` + per-field type check before assuming any specific accessor.
- **Vercel, not Cloudflare.** Despite the spec's assumption, `~/projects/asbestoshq-site/` deploys via `git push` to `main` → Vercel auto-deploy. There is no `wrangler.toml`, no `.cloudflare/`, no `vercel.json`, no `.vercel/`. The "deploy step" in this codebase is literally `git_ops.sh` doing a `git push`. Future-me: when a build prompt names a deploy provider, verify with `grep -r '<provider>' ~/projects/<site>/` first.

## Operator corrections

- **"There is no drafter→auditor R3 pipeline. Build the persistence foundation first."** When I halted the Escalation Triage build on missing upstream, operator pivoted to building the `auditor_verdicts` schema + audit_guide.py persistence as the foundation. Explicit: don't build the triage agent against a phantom upstream; build the upstream surface first.
- **"Option 2 — persist only audit_guide.py tonight. Editor stays stdout-only until its production role is decided."** When I surfaced two editor HALT conditions (no production runner, no pass threshold), operator chose to skip the editor half entirely rather than guess. Explicit policy: editor's production role is gated on Q7 (cross_agent_failure_modes_2026-05-16.md §7).
- **"Option 1 — ship without preview URLs."** When I surfaced that the spec's Cloudflare preview URLs don't exist in this Vercel-based stack, operator chose the smallest path: list slug + word count + score + audit reasoning, skip URLs entirely. Track URL upgrade as deferred.
- **"For manual test 2 (deploy batch dry run): … recommend skipping staging to avoid contaminating production data; verify on first real deploy."** Explicit guidance that I should NOT stage test slugs into `approved/guides/` for the populated-path test. First real 23:00 fire is the verification event.

## What's next

**Immediate (next session opening move):**
1. Read this context save + `HANDOFF.md` + `DEFERRED.md`.
2. Check `audit_log` for the first real 23:00 deploy_batch fire (it ran at 23:00:00-23:00:05 tonight with an empty queue — `deploy_batch_empty`, no Telegram, working as designed; first non-empty fire will verify the populated path).
3. Verify preview_ping at 20:00 tomorrow delivers the right message format if anything's in the queue.

**Newly unblocked:**
- **Escalation Triage agent (partial)** — can now query `auditor_verdicts WHERE scorer='audit_guide' AND triaged_at IS NULL` to get real failing-audit candidates. Full coverage waits on D056 (editor half).

**Still blocked:**
- **Editor verdict persistence (D056)** — needs production runner, pass threshold, wiring decision (Q7 from `cross_agent_failure_modes_2026-05-16.md` §7).
- **Throttle removal (D057)** — trigger is "2-3 weeks of clean preview pings with no manual skips needed."
- **Preview URLs (D058)** — only relevant if score-based triage proves insufficient.

**Still-open items from the prior HANDOFF, in priority order:**
- F2: drafter pre-flight check landed (commit `0df8cf6`) — this item is now closed, HANDOFF should reflect that.
- D044: `log_to_audit.sh` SQL escape — landed (`939361d`) — also closed.
- F17/F18 cost-cap reconciliation — open.
- F14 pre-ship operator review gate — partially addressed by the deploy throttle (3h review window), but no formal approval mechanism.
- 7 operator-policy questions in §7 of cross_agent_failure_modes.
- D051 `~/agents/` hygiene — most of it landed via intermediate commits this session.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-16_0015.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-16 19:20 **Last session summary:** Config Synthesizer agent built and committed (`69dd30f`) with `--validate-config-only` flag added to `audit_guide.py` (`e504793` in asbestos-contractors). 4 test configs synthesized + triaged + deleted. page.tsx template extracted from 3 real shipped wrappers; ship-to-site committed for the first time (`3b43c11`) including the provenance-strip tweak to `lib/stage.sh`. Cross-agent failure-modes audit landed at `~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md` documenting 23 failure modes across the now-autonomous asbestos chain.  --- 
