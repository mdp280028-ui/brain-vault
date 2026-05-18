# AITEAM Project Handoff
**Last updated:** 2026-05-17 17:36 (D068 watchdog hygiene + D067 SSG plan recovery + D056 editor production runner + GEO Optimizer Phases 1 & 2 + Internal Link Agent v1)
**Last session summary:** Closed three deferred items and shipped a new verdict-gate agent. D068 watchdog hygiene — 3 refactor commits (compute_threshold helper, drop `_stale` qualifier, single-load state.json cache) all byte-identical except FIX B's narrow intentional stale-path diff; live tick id=1171 clean. D067 SSG data-driven rewrite plan recovered from chat history to `~/brain/projects/aiteam/docs/` (brain commit `b1c4214`). D056 editor production runner shipped end-to-end: `score.sh` + threshold tune 3.8→3.6 per 4-slug burn-in + ship.sh editor gate at step 2.5 (commits `6895dcc`, `dc05c51`, `1323040`); SQL-trace integration test (live blocked by whitelist coverage of all 32 approved slugs); 3 new D-items opened (D072 truncation, D073 rubric redundancy, D074 live-gate verification). GEO Optimizer Phases 1 + 2 shipped: 8-axis rubric + `geo-optimizer/score.sh` (Phase 1, commit `0335838`) + ship.sh GEO gate at step 2.4 BEFORE editor gate at 2.5 (Phase 2, commit `32b3ea3`); smoke test on white-asbestos-vs-blue scored composite 3.88 PASS with predicted floors confirmed (tables=1, quotations=1, other 6 axes 4-5). Stack now: `validate → GEO (2.4) → editor (2.5) → dry-run/stage/build/push`. Sister chat shipped internal-link agent v1 mid-session (`928671a` + `ec3f087`) — not yet inspected. **Prior session summary (preserved):** Watchdog agent built and activated. Drafter heartbeat row (`fb53d04`); watchdog scaffold (`88f8d7a`); cron activated 11:02 PT; first production tick fired 11:15:00 alerting once on orchestrator (real signal — `diary_written` missing in 26h). Four follow-on fixes landed: state-corrupt guard (`1053913`); grace_minutes wired (`38ed525`); drafter check_window 2h→1h (`26d3311`); LESSONS.md docs (`1e90165` brain). SSG site repo located at `~/projects/smartsourceguide/`.

---

## Current phase

**Phase 1, late-mid-phase. Verdict-gate stacking now live.**

The asbestos pipeline remains fully autonomous (23:00 PT batch deploy, 20:00 PT preview ping). Watchdog runs `*/15` + 06:00 digest. ship.sh now stacks three pre-flight gates per slug:

1. **validate.sh** — pipeline JSON exists, site repo clean, slug not in whitelist
2. **GEO gate (step 2.4, GEO Phase 2)** — queries `auditor_verdicts` for `scorer='geo-v1'`; fires `geo-optimizer/score.sh` inline if no row; skip on `passed=0` with `reason='geo_failed'`
3. **editor gate (step 2.5, D056)** — same shape with `scorer='editor-v1'` and `editor/score.sh`
4. **dry-run short-circuit / stage / build / push** — unchanged

All three gates must pass per POLICY Q7 (gates stack). audit_guide.py runs upstream inside asbestos `run-batch.sh` and produces approved JSONs that ship.sh then sees.

Verdict persistence is now full-coverage: `audit_guide` (mechanical, asbestos repo), `editor-v1` (Sonnet, ~/agents/editor/score.sh), `geo-v1` (Sonnet, ~/agents/geo-optimizer/score.sh). All three write to the shared `auditor_verdicts` table.

## Agents live

Unchanged from prior handoff except:
- **New agent: `editor-v1`** — production scorer `~/agents/editor/score.sh`, Sonnet, 5-axis rubric, threshold 3.6 (env-overridable `EDITOR_PASS_THRESHOLD`)
- **New agent: `geo-v1`** — production scorer `~/agents/geo-optimizer/score.sh`, Sonnet, 8-axis rubric, threshold 3.0 (env-overridable `GEO_PASS_THRESHOLD`)
- **`internal-link` v1** — Rank #2 agent. 3-table SQLite schema (`internal_link_inventory`, `backlink_proposals`, `audit_link_failures`); 4 scripts at `~/agents/internal-link/` (build_inventory, audit_links, propose_backlinks, apply_approval). `propose_backlinks.sh <new_slug>` calls Sonnet at ~$0.07/call (cap $0.50) to pick 3-5 existing slugs that should backlink TO the new one. Operator approves per-proposal via Telegram `/approve <id>` or `/reject <id>` (new bot.js handlers). Approval appends source_slug to `~/agents/ship-to-site/state/updated_slugs_queue.txt`. `deploy_batch.sh` consumes the queue at 23:00 as `kind=refresh` (skips gsc INSERT + propose_backlinks side effects). `audit_links.sh` wired into asbestos `run-batch.sh` as advisory (logs to `audit_link_failures`, never blocks). Live Telegram + first re-ship deferred (bot is off; chrysotile→white-asbestos-vs-blue queued).

## Completed (this session window: ~14:30-17:35)

- [x] **D068 watchdog hygiene closed** — 3 refactor commits (FIX A `5c7d85c` compute_threshold helper / FIX B `19f44aa` drop `_stale` qualifier / FIX C `84808e2` single-load state.json cache). Live tick id=1171 clean. Closure audit row `591EE19F-E8B2-4421-9660-C5AAFDE039C5`. Brain DEFERRED commit `2241611`.
- [x] **D067 SSG data-driven rewrite plan recovered to disk** — `~/brain/projects/aiteam/docs/AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-17.md` (375 lines reconstructed from chat history). Brain commits `b1c4214` + `936038f`. Closure audit row `0C9C5E2D-E5FD-4EF1-9353-C475C42473C5`.
- [x] **D056 editor production runner** — `~/agents/editor/score.sh` (commit `6895dcc`), threshold tune 3.8→3.6 per 4-slug burn-in (commit `dc05c51`), ship.sh editor gate at step 2.5 (commit `1323040`). Closure audit row `ACBEB80B-5A18-4B35-A3BA-51D0EA55E7E6`. Brain DEFERRED commit `2bc5518`. **5 editor verdicts in DB** (ids 3-7 from burn-in + the white-vs-blue re-score during D056 testing).
- [x] **D072 + D073 + D074 opened** as D056 follow-ups (truncation fix / rubric redundancy review / live-gate first-execution verification).
- [x] **LESSONS.md updates** — sister-chat 24-line addition committed separately (`c57201d`), then 2 new lessons from D068/D056 (`f9f5cbf`), then 7 more from D056 + GEO integration in this ctx save's LESSONS.md append.
- [x] **GEO Optimizer Phase 1** — agent.yaml + CLAUDE.md + rubric.md (8 axes, integer 1-5, mean composite) + score.sh + (slug,scorer) composite index migration (commit `0335838`). Smoke test on white-vs-blue: composite 3.88 PASS at threshold 3.0.
- [x] **GEO Optimizer Phase 2** — ship.sh GEO gate at step 2.4 BEFORE editor gate (commit `32b3ea3`). SQL-trace both branches verified. Spec-deviation flagged: `local src_json` dedup vs strict "byte-for-byte mirror."
- [x] **Internal-link agent v1** — commits `928671a` (agents: migration + folder), `ec3f087` (agents: deploy_batch.sh new/refresh column + bot.js /approve+/reject), `370f62c` (asbestos-contractors: run-batch.sh advisory audit_links hook + first approved backlink in chrysotile.json), `41c5429` (asbestoshq-site: GuideArticle.tsx "Related Reading" render section). Build-verified visual render via `npm run build`. 1 real propose_backlinks call: $0.0746 (cap $0.50). 5 proposals generated, 1 approved (chrysotile→white-asbestos-vs-blue), 1 rejected, 3 still proposed. No ship.sh stacked gate — internal-link runs post-deploy as a hook in deploy_batch.sh (not in ship.sh's pre-flight chain).

## In progress

- (none — D068/D067/D056 all closed; GEO Optimizer Phase 3 not started)

## Blocked

- **GEO Optimizer Phase 3** (Slot A in asbestos `run-batch.sh` writer-revision loop) — operator-gated; lands when operator is ready. Requires cross-repo commit + new GEO feedback file format (mirror of AuditKit feedback).
- **D074** (live editor + GEO gate verification) — waiting on the next genuinely-new pipeline slug from asbestos drafter to enter `approved/guides/` not-yet-whitelisted.
- **SSG content pipeline first batch** — still blocked on 5 items: (1) operator decisions on D-SSG-02/03; (2-4) authoring keyword_research.md, approved-index.md, authority-links.md, 2+ exemplar assignment-batch-*.md; (5) ship destination decision (smartsourceguide direct vs approved-JSON-only). With D067 closed, the spec doc is now on disk — first blocker resolved.
- **Internal-link live `/approve` Telegram round-trip + first real re-ship** — bot.js handlers committed but bot has been off since 2026-05-15. Once operator starts bot: `/approve 1` for chrysotile→white-asbestos-vs-blue would no-op (already approved during smoke); approve/reject siblings #2/#3/#4 to exercise live path. Then 23:00 deploy_batch.sh consumes `updated_slugs_queue.txt` (currently contains `chrysotile`) and re-ships via the new `kind=refresh` branch — first end-to-end production cycle. Internal-link is NOT a ship.sh stacked gate; it's a post-deploy hook in deploy_batch.sh, so no GEO/editor-style stacking concerns.

## Known bugs

| Bug | Severity | File(s) | Notes |
|---|---|---|---|
| (none new this session) | — | — | — |

## Architecture decisions (recent)

| Decision | Reasoning | Date |
|---|---|---|
| Editor gate slots into ship.sh `ship_one_slug`, not stage.sh | stage.sh is pure file-staging (copy JSON, generate page.tsx, append whitelist); no DB access; placing the gate there would mix concerns | 2026-05-17 |
| Editor fail-routing reuses ship-to-site `needs_review_queue.txt` | Single source of truth; ship.sh step 1 already consults it; new file in asbestos repo would create dual source | 2026-05-17 |
| Editor threshold default 3.6 (was 3.8) | 4-slug burn-in showed structural ceilings on originality/hook/evidence from audit_guide.py pre-filtering; 1/4 pass at 3.8 vs 2/4 at 3.6 | 2026-05-17 |
| GEO Optimizer threshold default 3.0 | 8 axes vs 5 changes distribution math; permissive starting bar pending GEO-specific burn-in | 2026-05-17 |
| GEO + editor as separate gate blocks (not merged into one) | Independent revertability; different scorers, different prompts, different versioning; SQL queries cleanly distinguished by scorer column | 2026-05-17 |
| Stacked-gate `src_json` declared once in GEO (upstream); editor reuses | Avoids redundant `local` re-declaration + duplicate identical computation in same function scope. Spec-deviation from strict "byte-for-byte mirror," flagged in commit body. | 2026-05-17 |
| D056 integration test = SQL-trace both branches | Live dry-run blocked by whitelist coverage of all 32 approved slugs; SQL-trace is honest verification of the wire | 2026-05-17 |
| GEO Optimizer hybrid integration (Slot A + Slot B) | Slot A (run-batch.sh writer-revision) is the authoritative path per project instructions; Slot B (ship.sh defensive) handles cold-start for slugs predating GEO deploy | 2026-05-17 |

## Deferred items

(Full list in `DEFERRED.md`. Newest live items in priority order:)

- **D074** — verify editor + GEO gates execute live on first new pipeline slug post-close. **NEW THIS SESSION.**
- **D072** — editor/GEO score.sh chunk-and-aggregate for full-article scoring (every burn-in slug truncated at 8KB). **NEW THIS SESSION.**
- **D073** — review editor rubric axes for redundancy vs audit_guide.py mechanical gate. **NEW THIS SESSION.**
- **D057** — remove deploy throttle once draft quality consistent.
- **D058** — Vercel preview URLs in 20:00 preview ping.
- **D065** — SSG `gsc_submission_queue` INSERT hook (gated on SSG enabled).
- **D066** — untracked-code pattern meta-item (closed for current files; pattern unresolved).
- **D051** — `~/agents/` hygiene meta-item.
- **D-SSG-01..09** — SSG-track grooming items (D067 closed reduces this set by one).
- **D059** — SSG deploy-throttle pattern-copy from `3ed57dc`.

## Next session should

1. **Read POLICY.md** (authoritative for Q1–Q7). Do not re-derive.
2. **Check overnight watchdog behavior:** `sqlite3 ~/store/aiteam.db "SELECT id, datetime(ts,'unixepoch','localtime'), action, target FROM audit_log WHERE actor_id='watchdog' AND ts > strftime('%s','now') - 86400 ORDER BY ts;"` — expect `orchestrator:missing→healthy` recovery row tonight after `diary_written` lands at 23:45; expect `watchdog_digest_sent` at 06:00.
3. **Check D074 trigger condition:** `ls -la ~/projects/asbestos-contractors/content/asbestos/approved/guides/ | wc -l` — if count > 32, a new slug landed; tail `~/agents/ship-to-site/ship.log` for the `[geo]` / `[editor]` `no verdict; running score.sh...` lines on the next tick after the new slug arrives.
4. **Check overnight 23:00 fire result:** `sqlite3 ~/store/aiteam.db "SELECT * FROM audit_log WHERE actor_id='ship-to-site' AND action IN ('deployed','deploy_failed','deploy_batch_complete') ORDER BY ts DESC LIMIT 10;"`
5. **Check overnight cost-cap state:** `ls ~/store/flags/ 2>/dev/null; grep '^SYSTEM_PAUSED=' ~/agents/config/.env; sqlite3 ~/store/aiteam.db "SELECT COUNT(*), SUM(estimated_cost_usd) FROM token_usage WHERE date(ts,'unixepoch','localtime')=date('now','localtime');"`
6. **Pick a build:**
   - **GEO Optimizer Phase 3** (Slot A run-batch.sh writer-revision) — operator-gated, cross-repo commit.
   - **Internal-link v1 live cycle close** — start bot.js, send `/help` to verify new commands, `/approve` siblings #2/#3/#4, watch 23:00 deploy_batch.sh fire the `kind=refresh` branch + re-ship chrysotile, eyeball the "Related Reading" section on live https://www.asbestoshq.com/guides/chrysotile/.
   - **SSG content pipeline activation** — D067 resolved the spec-doc blocker; 4 authoring items + 1 ship-destination decision remain.
   - **D058 Vercel preview URLs** — preps for D057 throttle removal.

## Files to reference

- **POLICY.md** — authoritative Q1–Q7 (commit `6e7fb52`).
- **PRIME.md** — north-star hard rules + Read first section.
- **DEFERRED.md** — D-numbered deferred work (D072/D073/D074 added this session).
- **LESSONS.md** — 130+ gotchas; 7 added this session under `### D056 + GEO Optimizer integration lessons (2026-05-17 16:00-17:35 window)`.
- **Build reports (this session window):**
  - `~/brain/projects/aiteam/docs/AITEAM_SSG_DataDriven_Rewrite_Plan_2026-05-17.md` (recovered via D067)
- **New scripts this session:**
  - `~/agents/editor/score.sh` (5-axis production scorer)
  - `~/agents/geo-optimizer/score.sh` (8-axis production scorer)
  - `~/agents/geo-optimizer/rubric.md` (canonical GEO rubric)
  - `~/agents/lib/migrations/2026-05-17_geo_verdicts_index.sql` (composite slug+scorer index)
- **ship.sh stacked gates:** `~/agents/ship-to-site/ship.sh:188-261` (GEO gate 2.4 + editor gate 2.5)

## Recent commits (this session)

| repo | sha | summary |
|---|---|---|
| ~/agents/ | `5c7d85c` | refactor(watchdog): collapse 4 duplicate threshold computations into single helper |
| ~/agents/ | `19f44aa` | refactor(watchdog): drop _stale qualifier from probe returns + remove dead per-probe threshold |
| ~/agents/ | `84808e2` | perf(watchdog): single-load state.json at tick start, eliminate 14× jq reparse |
| ~/agents/ | `6895dcc` | feat(editor): score.sh production runner + CLAUDE.md wording |
| ~/agents/ | `dc05c51` | tune(editor): lower default threshold 3.8 → 3.6 per 4-slug burn-in distribution |
| ~/agents/ | `1323040` | feat(ship-to-site): editor verdict gate in ship_one_slug (D056) |
| ~/agents/ | `0335838` | feat(geo-optimizer): Phase 1 — rubric + score.sh + verdicts index migration |
| ~/agents/ | `32b3ea3` | feat(ship-to-site): GEO verdict gate at step 2.4, before editor gate (GEO Phase 2) |
| ~/agents/ | `928671a` | feat(internal-link): v1 agent — inventory + audit-links + backlink-proposals |
| ~/agents/ | `ec3f087` | feat(deploy-batch+telegram): wire internal-link agent into pipeline |
| ~/projects/asbestos-contractors/ | `370f62c` | feat(run-batch+chrysotile): internal-link audit gate (advisory) + first approved backlink |
| ~/projects/asbestoshq-site/ | `41c5429` | feat(GuideArticle): render suggestedLinks as 'Related Reading' section |
| ~/brain/ | `b1c4214` | docs(ssg): recover SSG data-driven rewrite plan from 2026-05-16 chat history |
| ~/brain/ | `936038f` | docs(deferred): close D067 — SSG data-driven rewrite plan recovered to disk |
| ~/brain/ | `2241611` | docs(deferred): close D068 — watchdog hygiene pass (FIX A/B/C) |
| ~/brain/ | `2bc5518` | DEFERRED: close D056 (editor production runner + gate live); open D072, D073, D074 |
| ~/brain/ | `c57201d` | LESSONS: sister-chat 2026-05-17 entries (multi-cluster execution + SSG recon) |
| ~/brain/ | `f9f5cbf` | LESSONS: D064 validator catches Claude-authored prompts; cross-repo SHA qualification (2026-05-17 D068/D056) |

## Sign-off discipline

A step is ✅ only when the build plan's pass condition has been executed and produced the expected result. Smoke tests run, audit row landed correctly, working tree clean. Honest ⚠ beats premature ✅.

This session window: 0 HALTs. 0 audit-row malformed corrections (D064 validator caught my own malformed `key=value` invocation early in D068 close-out — corrected via clean positional re-run). 0 retroactive commit edits (1 commit-message inaccuracy in `dc05c51` flagged + captured in D056 close-out audit `commit_msg_corrections` field; no amend). 1 spec deviation flagged in `32b3ea3` body (editor gate `local src_json` dedup).

## Gotchas for future me

(Unchanged gotchas from prior handoff still apply. New this session:)

- **Whitelist coverage of asbestoshq-site is total: 32/32.** Every approved JSON is in `src/components/GuideArticle.tsx`'s `APPROVED_GUIDE_SLUGS`. validate.sh skips every one as `slug_already_shipped` BEFORE editor/GEO gates run. Live gate testing requires a NEW slug (D074) or queue-state mutation (which doesn't bypass validate). Plan SQL-trace for any gate verification while the corpus is whitelist-saturated.
- **`auditor_verdicts.scorer` column distinguishes three sources:** `audit_guide` (mechanical, asbestos repo's audit_guide.py), `editor-v1` (Sonnet, ~/agents/editor/score.sh), `geo-v1` (Sonnet, ~/agents/geo-optimizer/score.sh). Composite index `idx_verdicts_slug_scorer` makes per-slug per-scorer lookups fast (added 2026-05-17 via `2026-05-17_geo_verdicts_index.sql`).
- **Editor `EDITOR_PASS_THRESHOLD` env-overridable; default 3.6.** GEO `GEO_PASS_THRESHOLD` env-overridable; default 3.0. Inclusive `>=` means a composite of exactly the threshold value PASSES.
- **score.sh routes its own fail to `needs_review_queue` BEFORE returning exit 2.** ship.sh gate doesn't need to re-append; it just logs the skip + returns. Slug appearance in queue correlates with the `*_scored` audit row's `passed:0` field, not the `ship_to_site_skipped` row.
- **8KB truncation universal in editor + GEO burn-in.** Every asbestos approved guide >8KB raw; current scoring sees only the head. `truncated: true` flag in audit payload for filtering. D072 tracks the chunk-and-aggregate fix.
- **ship.sh stacked-gate `src_json` declared in GEO (step 2.4) only.** Editor gate (step 2.5) reuses without re-declaration. If you remove the GEO gate, you must re-add `local src_json` to the editor gate's `local` line.
- **GEO Optimizer rubric criterion 6 is RELAXED** ("structured presentations" includes comparison-shaped paragraphs + ranked lists + structured timelines, not just HTML tables). Criterion 8 (expert quotations) is STRICT (named experts with credentials). Both can floor at 1 across asbestos corpus per smoke test — corpus signal, not bug.
- **Sister-chat work surfaces mid-session.** Stage commits by exact path (`git add ship-to-site/ship.sh`) to keep your commit isolated. Combined commits dishonestly attribute sister-chat diffs to your commit message. See LESSONS.md "Untracked files in `~/brain` trap commit messages" entry (line 87 area).
