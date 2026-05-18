# AITEAM Project Handoff
**Last updated:** 2026-05-17 19:25 (Research/Opportunity Agent Session 2 live: 3 sources, weekly digest, cron firing hourly)
**Last session summary:** Shipped Session 2 of Research/Opportunity Agent. SE poller (149 questions ingested), YouTube auto-discovery (10 videos pinned, 330k-86k views), YouTube comment poller (529 comments, 1 video commentsDisabled handled), tightened YouTube triage prompt (Haiku praise-hallucination persists but Sonnet extract proven to act as reliable second gate — id 12 scored 0.10 with "No homeowner pain question present"). Weekly digest renders cleanly (10 pains across 3 sources, em-dash dividers, phone-readable). Cron wired: hourly poll + weekly Monday 09:00 digest + monthly auto-discovery. Master kill switch flipped `RESEARCH_OPPORTUNITY_ENABLED=true`. Proxy-cron-fire verified 4 audit_log rows with restricted PATH env. Bug fix: stray SHA on .env line 45 was breaking every `source ~/agents/config/.env` call across the fleet — normalized to `SERPER_API_KEY=<value>`. **Prior session summary (preserved):** Session 1 shipped Reddit-only vertical slice (14 posts, 11 triaged, 7 pain_points, /approve_pain handler) at commit 2413a94. D075 Opportunity Scout deferred at brain commit 83cb3ed. 7 architectural decisions pinned upfront, zero scope drift. 3 bash bugs fixed + documented (apostrophe-in-heredoc, sqlite-newlines, PyYAML-PEP668-venv). HANDOFF + LESSONS + ctx save committed at brain 1bf7c9e. Prior to Session 1: D068 watchdog hygiene + D067 SSG plan recovery + D056 editor production runner + GEO Optimizer Phases 1 & 2 + Internal Link Agent v1.

---

## Current phase
**Research/Opportunity Agent FULLY LIVE.** First natural cron tick at next :00 in production. First Monday 09:00 digest fires 2026-05-25. Awaiting operator approval / rejection on 25 unreviewed pain points.

## Completed (cumulative)
- [x] Phase B Telegram bot
- [x] Cost-cap policy (POLICY Q2)
- [x] Watchdog agent + digest
- [x] Assignment Drafter + autonomous fire-pipeline
- [x] SSG pipeline data-driven rewrite (D067)
- [x] Editor production runner + ship.sh editor gate (D056)
- [x] GEO Optimizer Phases 1+2 (rubric + ship.sh GEO gate at step 2.4)
- [x] Internal Link Agent v1
- [x] Research/Opportunity Agent Session 1 (Reddit poll + triage + extract + approval) — commit 2413a94
- [x] **Research/Opportunity Agent Session 2 (SE + YouTube + digest + cron + master switch ON) — commit bc7504b**
- [x] D075 Opportunity Scout deferred to DEFERRED.md (brain commit 83cb3ed)

## In progress
- [ ] First operator approval cycle on the 25 unreviewed pain_points

## Blocked
- none

## Known bugs
| Bug | Severity | File(s) | Notes |
|---|---|---|---|
| Stray SHA on `.env` line 45 broke fleet-wide source | RESOLVED | `~/agents/config/.env` | Normalized to `SERPER_API_KEY=<value>`. Add `validate_env.sh` per LESSONS |
| YouTube Haiku triage scores praise 0.70-0.80 | low | `research-opportunity/triage.sh` | Sonnet extract proven as second gate. If digest noise excessive, raise youtube-specific threshold to 0.85 in config.yaml |
| Bot not currently running | low | `~/agents/telegram/bot.js` | `/approve_pain` `/reject_pain` handlers active on next bot start; CLI path always works |

## Architecture decisions (recent)
| Decision | Reasoning | Date |
|---|---|---|
| Deterministic digest (Python template, not Sonnet) | Operator wants stable phone formatting > creativity. $0 per digest, no fence-wrapping risk, testable | 2026-05-17 |
| Sonnet extract as YouTube praise-hallucination second gate | Haiku-4.5 over-weights video-topic context; persistent ~12% false positive rate at 0.5+ threshold. Sonnet correctly downscores praise to 0.10 | 2026-05-17 |
| YouTube 2-step quota dance (search→videos.list for viewCount) | 101 units/month for view-quality filter. Skipping videos.list saves 1 unit but loses quality signal | 2026-05-17 |
| `commentsDisabled` 403 → sticky `last_polled_ts=NOW` | Without this, dead videos burn 24 units/day in retry loops | 2026-05-17 |
| Cron flip smoke-tested via proxy-fire with PATH-restricted env | Captures the only real cron-vs-shell risk in 30s vs 40min waiting for :00 | 2026-05-17 |
| Single polymorphic `source_posts` table with discriminator | Matches tg-monitor precedent; one index covers all sources | 2026-05-17 |
| Local `pain_points.db` (not aiteam.db) | Raw posts are chatty; isolation keeps aiteam.db lean | 2026-05-17 |
| `/approve_pain` explicit step (NOT direct drafter_queue write) | Provenance + audit trail + operator control of queue depth | 2026-05-17 |

## Deferred items
- **YouTube-specific triage threshold (0.85)** — wire into extract.sh if first 2 digests show too many praise false positives. One-line change.
- **`validate_env.sh`** — pre-commit / cron-startup .env shape validation. Would have prevented today's source-cascade failure.
- **Haiku CLI batching** — if Session N+ scales triage past 200-300 rows/day, batch multiple rows per Haiku call to cut $0.025/call overhead.
- **D075 Opportunity Scout** (business-idea discovery). Trigger: Agent A burns clean ~1 week.
- All prior `~/brain/projects/aiteam/DEFERRED.md` items (D055, D057, D058, D063, D064, D065, D069, D071, D072, D073, D074, D-SSG-01..09).

## Next session should
1. **Confirm first natural cron tick fired clean** at next :00. Watch `tail -f ~/agents/research-opportunity/logs/poll.log` and audit_log: `sqlite3 ~/store/aiteam.db "SELECT datetime(ts,'unixepoch','localtime'), action FROM audit_log WHERE actor_id='research-opportunity' AND ts > strftime('%s','now','-1 hour');"`
2. **Operator approve / reject the 25 pain_points** via `bash ~/agents/research-opportunity/apply_pain_approval.sh <id> <approve|reject>` CLI, or Telegram `/approve_pain <id>` once bot restarts.
3. **Watch first Monday 09:00 digest** delivery (2026-05-25). Verify Telegram receives + format renders cleanly on phone.
4. **Add `validate_env.sh`** to prevent future fleet-wide source failures.
5. **Consider Haiku CLI batching** if triage backlog grows (current: ~580 untriaged across 3 sources, hourly cron should catch up over 24-48h).

## Files to reference
- `~/agents/research-opportunity/CLAUDE.md`: agent orientation, hard rules, kill switches, source spec.
- `~/agents/research-opportunity/config.yaml`: sub allowlist, SE tags, YouTube auto-discovery params, digest cadence.
- `~/agents/research-opportunity/pain_points.db`: 692 source_posts (14 reddit / 149 SE / 529 youtube), 25 pain_points unreviewed.
- `~/agents/research-opportunity/workspace/digest_outbox/2026-05-17.txt`: latest dry-run digest preview.
- `~/store/aiteam.db` `audit_log` filtered by `actor_id='research-opportunity'`: full event trail.
- `~/brain/projects/aiteam/LESSONS.md` Session 1+2 entries.
- `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-17_1921.md`: Session 2 narrative.

## Gotchas for future me
- **`~/agents/config/.env` line 45**: now `SERPER_API_KEY=55b842d3...47de`. Do NOT remove — it's a real Serper.dev API key. Earlier corruption to bare SHA broke fleet-wide `source` — var-name prefix is required for valid bash.
- **bash 3.2 apostrophe rule**: any new prompt string inside `$(cat <<EOF ... EOF)` must avoid contractions. Documented in research-opportunity/CLAUDE.md.
- **sqlite-to-bash extraction rule**: any text column going to `read -r` must be wrapped in `REPLACE(REPLACE(col, CHAR(10), ' '), CHAR(13), ' ')`.
- **Sonnet JSON parsing**: accept both `{...}` and `[...]`; strip ` ```json ``` ` fences first.
- **YouTube praise-hallucination**: Haiku-4.5 cannot reliably ignore video-title context. Don't waste time tightening the prompt further — rely on Sonnet extract as second gate (proven), or raise YouTube-specific threshold to 0.85.
- **Claude CLI overhead**: ~$0.025 per Haiku call (system prompt, CLAUDE.md, tools loaded per invocation). Plan budgets accordingly.
- **Cron PATH**: crontab already exports `PATH=/opt/homebrew/bin:...` at top; new entries inherit.
- **Sister-chat dirty work in tree**: `backlink-prospector/` and `lib/migrations/2026-05-17_outbound_link_prospects.sql` untracked at session end. Not mine.
