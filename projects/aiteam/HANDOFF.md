# AITEAM Project Handoff
**Last updated:** 2026-05-17 22:00 (Keyword Registry live + brain-tracked pain log + digest archive · 4 production hooks · nightly rebuild cron firing)
**Last session summary:** Shipped Keyword Registry agent (`~/agents/keyword-registry/` + `~/brain/projects/aiteam/keyword_registry/`). 162 canonical keywords tracked across 32 live asbestoshq.com slugs, alias-collapsed (chrysotile/white-asbestos/amosite/crocidolite etc — 14 seed rules). Three lib scripts: rebuild_registry.py (full regen, 0.5s), update_registry.py (write-through helper, verb dispatch), backfill_registry.sh (cron wrapper). Four production hooks wired best-effort: deploy_batch.sh new+refresh branches, internal-link/apply_approval.sh, research-opportunity/extract.sh pre-Sonnet shortcut (multi-word title match, 31 of 162 keywords qualify). Nightly cron at 0 3 * * * installed. Brain commit `6c15f3f`, agents commit `6047fe0`. Earlier this wall-clock window: brain-tracked pain_points_log + digest archive (commit 3f7145e) + Gap-1 fix for digest_log.posts_scanned + D078/D079/D080 deferred. Sister chat shipped Backlink Prospector v0.1 (b873996) in parallel — not inspected here. **Prior session summary (preserved):** Session 2 of Research/Opportunity Agent live (SE + YouTube + weekly digest + cron + master switch ON, commit bc7504b). Session 1 (Reddit vertical slice) at commit 2413a94. D075 Opportunity Scout deferred at brain 83cb3ed. Prior: D068 watchdog hygiene + D067 SSG plan recovery + D056 editor production runner + GEO Optimizer Phases 1 & 2 + Internal Link Agent v1.

---

## Current phase
**Keyword Registry FULLY LIVE.** Three-agent observability stack now operational: research-opportunity (pain miner) ↔ keyword-registry (coverage truth) ↔ internal-link (backlink agent). First natural 03:00 cron tick fires tonight. Awaiting operator validation on `white-asbestos-vs-blue-asbestos` h1-derived primary_kw notes flag.

## Completed (cumulative)
- [x] Phase B Telegram bot
- [x] Cost-cap policy (POLICY Q2)
- [x] Watchdog agent + digest
- [x] Assignment Drafter + autonomous fire-pipeline
- [x] SSG pipeline data-driven rewrite (D067)
- [x] Editor production runner + ship.sh editor gate (D056)
- [x] GEO Optimizer Phases 1+2
- [x] Internal Link Agent v1
- [x] Research/Opportunity Agent Session 1 (Reddit vertical slice) — commit 2413a94
- [x] Research/Opportunity Agent Session 2 (SE + YouTube + digest + cron, master switch ON) — commit bc7504b
- [x] Brain-tracked pain_points log + digest archive + Gap-1 posts_scanned fix — commit 3f7145e
- [x] D075 Opportunity Scout deferred to DEFERRED.md — brain 83cb3ed
- [x] D078/D079/D080 deferred (Sonnet cost in digest header / Haiku CLI overhead / YouTube triage threshold) — brain 2e58c23
- [x] **Keyword Registry agent + brain seed (162 keywords / 14 aliases / 4 hooks / nightly cron) — agents 6047fe0, brain 6c15f3f**
- [~] Sister-chat: Backlink Prospector v0.1 (b873996) — shipped but not inspected by this chat

## In progress
- [ ] First operator review cycle on the 25 unreviewed pain_points (CLI: `bash ~/agents/research-opportunity/apply_pain_approval.sh <id> approve`; Telegram: `/approve_pain <id>` once bot restarts)

## Blocked
- none

## Known bugs
| Bug | Severity | File(s) | Notes |
|---|---|---|---|
| `white-asbestos-vs-blue-asbestos` primary_kw derived from h1 | low | `~/brain/.../keyword_registry/keyword_registry.md` notes column | Either create `keyword-configs/white-asbestos-vs-blue-asbestos.json` OR accept h1-derived "white asbestos vs blue asbestos" as canonical. Notes flag visible to operator |
| Bot not currently running | low | `~/agents/telegram/bot.js` | Approval handlers active on bot start; CLI path works regardless |
| Stray SHA on `.env` line 45 → bare line | RESOLVED | `~/agents/config/.env` | Now `SERPER_API_KEY=...` (valid bash). Add `validate_env.sh` per LESSONS (D-number not yet allocated) |
| YouTube Haiku triage scores praise 0.70-0.80 | low (D080) | `research-opportunity/triage.sh` | Sonnet extract is reliable second gate. Raise threshold to 0.85 if digest noise becomes annoying |

## Architecture decisions (recent)
| Decision | Reasoning | Date |
|---|---|---|
| Keyword registry storage = markdown table in `~/brain/` | Phone-readable, auto-committed, no schema migration | 2026-05-17 |
| Hybrid source-of-truth (write-through + nightly rebuild) | Write-through alone risks silent drift; nightly-only adds read latency | 2026-05-17 |
| v1 alias seed (14 rules) NOT deferred | 26/32 slugs touch asbestos-type aliases. Defer = 80% under-counting | 2026-05-17 |
| Pre-Sonnet shortcut: multi-word primary keywords only | Single-word "asbestos" would over-match; 2+ word phrases keep FP risk low | 2026-05-17 |
| update_registry.py single helper + verb dispatch | Stable surface, callers don't pick scripts, swap implementation v2-friendly | 2026-05-17 |
| Brain-tracked agent outputs use "Auto-maintained — do not edit" header | Prevents operator hand-edits being silently overwritten on next regen | 2026-05-17 |
| Deterministic digest (Python template, not Sonnet) | Stable phone formatting > creativity. $0/digest, no JSON-fence risk | 2026-05-17 |

## Deferred items
- **`white-asbestos-vs-blue-asbestos` keyword-config** — operator choose: create config file, or accept h1-derived primary_kw and edit notes flag.
- **`validate_env.sh`** — pre-commit/cron-startup .env shape validation. Would have prevented the SHA cascade.
- **Per-row registry UPDATE** — current update_registry.py delegates to full regen. Fast enough at 162 keywords. Bundle into "D081 — per-row registry UPDATE" if hook latency becomes a problem.
- **D078** Sonnet cost in digest archive header (token_usage join).
- **D079** Haiku CLI $0.025/call overhead propagation to fleet cost projections.
- **D080** YouTube triage threshold tune-up (raise to 0.85 if digest false-positives become annoying).
- **D075** Opportunity Scout. Trigger: Agent A burns clean ~1 week.
- All prior items in `~/brain/projects/aiteam/DEFERRED.md` (D055, D057, D058, D063-D074, D076-D080, D-SSG-01..09).

## Next session should
1. **Verify first natural cron tick at 03:00 PT** — `tail -f ~/agents/keyword-registry/logs/rebuild.log` + audit_log query for `actor_id='keyword-registry'`.
2. **Resolve white-vs-blue notes flag** (one-line operator decision).
3. **Watch for `registry_shortcut_hit`** audit rows on next extract.sh tick that processes fresh Reddit/SE comments. Spot-check the matches.
4. **Recon `backlink-prospector/`** (sister chat). Understand overlap with internal-link + keyword-registry.
5. **Watch first Monday 09:00 digest delivery (2026-05-25)** to confirm Telegram path (requires bot running by then).
6. **Add `validate_env.sh`** to prevent future fleet-wide source failures.

## Files to reference
- `~/agents/keyword-registry/CLAUDE.md`: agent orientation, hard rules, hybrid source-of-truth model.
- `~/brain/projects/aiteam/keyword_registry/keyword_registry.md`: 162-keyword auto-maintained table.
- `~/brain/projects/aiteam/keyword_registry/keyword_aliases.md`: 14 operator-editable alias rules.
- `~/agents/keyword-registry/lib/rebuild_registry.py`: full ground-truth regen (0.5s for 32 slugs).
- `~/agents/keyword-registry/lib/update_registry.py`: write-through helper. `shipped`/`refreshed`/`backlinked`/`lookup` verbs.
- `~/brain/projects/aiteam/research-opportunity/pain_points_log.md`: 25-entry append-only pain log.
- `~/brain/projects/aiteam/research-opportunity/digests/2026-05-17.md`: first digest archive (outbox_fallback delivery).
- `~/store/aiteam.db` `audit_log` filtered by `actor_id IN ('keyword-registry','research-opportunity')`: full event trail.

## Gotchas for future me
- **bash 3.2 `case` inside `while` inside `$()` with quoted patterns** breaks the parser. Drop to Python heredoc — see LESSONS 2026-05-17 entry.
- **Pre-Sonnet shortcut signal is the post TITLE, not Sonnet output.** YouTube comments (no title) always fall through. Documented in `keyword-registry/CLAUDE.md`.
- **`update_registry.py` v1 does full regen per call** (~0.5s). At 500-slug scale this hits ~3-5 sec per call. Watch for hook latency complaints; promote to per-row UPDATE if needed.
- **Brain-tracked agent outputs use "Auto-maintained — do not edit" header.** Apply to any new file written by agents into `~/brain/projects/aiteam/`.
- **`~/agents/config/.env` line 45** is `SERPER_API_KEY=55b842d3...47de`. Real key. DO NOT remove. Earlier corruption to bare SHA broke fleet-wide `source`.
- **Sister-chat dirty work today:** `backlink-prospector/` (commit b873996) + `lib/migrations/2026-05-17_outbound_link_prospects.sql`. Not built by this chat; recon when context budget allows.
