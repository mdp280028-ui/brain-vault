# AITEAM Project Handoff
**Last updated:** 2026-10-26 ~11:15 PT (FLEET-AUDIT-55, carried out by the OVERNIGHT-AUDIT-1 session on Mark's amendment (2))
**Last session summary:** Recovery after the Mac was off from about 2026-08-09 to 2026-10-25, preceded by a 21-day CLI + network blackout (2026-08-13 → 09-03). An overnight read-only audit (`~/aiteam-overnight-2026-10-26/AITEAM_Overnight_Audit_2026-10-26.md`) produced 39 fleet findings. This session then applied the time-critical park step (C2), the publish- and data-safety fixes, F-02, park commits C1/C3/C4/C6, the rulings, and most of the reliability queue: 30 single-purpose commits in `~/agents` and 3 in `~/brain`, each with its verify output in `docs/AITEAM_Opus55_Fleet_Audit_2026-10-25.md`. **A separate session will audit these fixes.**

---

## Current phase
**asbestoshq.com PARKED** (site stays live; pipeline, research-opportunity, keyword-registry, backlink-prospector and link-monitor's asbestos half are off). **SSG is shipping again:** 3 guides approved 10-26 (coupa, brex, best-field-service-management-software-for-small-business). Deploy cap 2/site/night.

## State (verified 2026-10-26)
- **Models:** wrappers use aliases only. `haiku` → claude-haiku-5-5, `sonnet` → claude-sonnet-5-5, `opus` → claude-opus-5-5 (live probe 10:4x, token_usage id 7317). No pinned model id anywhere in `~/agents`. Claude Code 2.1.310 (auto-updated 10:39 today; `majordomo/check.sh` now checks `claude --version` hourly).
- **Wrapper timeouts:** ai-cheap 1200 s, ai-do 3600 s, ai-think 1200 s. The old HANDOFF's "now 1800" and recon R#22 were wrong.
- **Kill switches (.env):** `RESEARCH_OPPORTUNITY_ENABLED=false`, `LINK_MONITOR_ASBESTOS_ENABLED=false` (park). `TELEGRAM_DEFAULT_AGENT=bob` (Mark's choice; unknown values now fall back to majordomo).
- **Crontab == lib/cron.txt.** keyword-registry (03:00) and backlink-prospector (Sun 08:00) lines are commented `PARKED 2026-10-25`.
- **Issues:** 3,012 open (the May–Aug 12 backlog), 57 hard. The blackout storm (2,196) and 23 mis-captured successes were resolved after `.backup` (`~/store/aiteam.db.bak-d6-*`, `.bak-d7-*`).
- **Backups:** nightly mirror + snapshot at 23:30, plus (from tonight) dated `db-generations/aiteam-YYYY-MM-DD.db`. Restore tested 2026-10-26 (`docs/restore.md`). Time Machine hourly.
- **Telegram:** delivering (rc 0, message ids returned). The bot was restarted 10:47 (pid 88580).

## Completed 2026-10-26 (agents repo unless noted)
- [x] Park: C2 `0dc87ac` (research-opportunity off + 3 detectors; verified at the real 11:00 run), C1 `c0946a6`, C3 `9022eed` (+ crontab), C4 `6a32677`, C6 `9850573`. C5 withdrawn (Mark: keep hunting asbestos domains).
- [x] Publish safety: `99e6528` ship.sh holds a site while run-batch holds its lock (F-05); `463c5cc` 2-per-site deploy cap; `dd53056` /approve refuses forked guides (F-03); `80b3a2c` post-push live-URL check (canonical url; the ship row's old SSG url was a 404); `e142f11` a crashed dry-run is a failure, not an empty queue (F-01).
- [x] Data safety: `579e105` log_to_audit waits 5 s on a locked DB; `ad816a1` dated DB generations outside the --delete mirrors.
- [x] F-02 `032189d`: `</dev/null` on `claude -p` in all three wrappers. Before, the first model call ate the drafter's queue (proven at $0: loop saw "a" vs "a b c").
- [x] Rulings: `2cda014` notify_operator logs the real site; `40701ac` opus-alias comments (model check: nothing to repoint).
- [x] Reliability: `a56a276` bot default fails safe; `ca19b80` + `4e1c3a4` notify_rc in sent rows + preview footer/cap line; `cc91ce1` issues-capture value check; `cfb2de8` AI_CALLER from $2; `fcb727a` librarian sort_failed; `20f6bae` bash-3.2 empty arrays; `b42bf75` CLI health check; `2384b36` kill-switch lib exports every call / no pipefail abort; `4f4102a` digest unknown ≠ missing, failed query ≠ quiet; `c29d92b` deploy log line; `037846e` diary window/attribution; `9af354f` link-monitor checks body links; `e88aee2` digest lists live detectors only; `25aa5a4` netdiag on transport failure.
- [x] Brain: `c0b9ef3` DEFERRED D123–D134 (closed D098, D115) + rewritten `docs/restore.md`; this HANDOFF.
- [x] Orphan `pgrep` loops from the domain-hunter-lab session (pids 50839/51502) killed after matching their command lines.

## In progress / next session should
1. **Audit today's fixes** (separate session, per Mark): each fix's verify command and output is in `docs/AITEAM_Opus55_Fleet_Audit_2026-10-25.md` § Fix log.
2. **Watch tonight:** 20:00 preview should show `⏳ Cap 2/site: tonight ships ssg: first 2 of 3`. 23:00 deploy ships best-field-service-management-software-for-small-business + brex-alternatives and holds coupa-alternatives (`deploy_capped`); each ship row gets `live_check`. 23:30 writes the first `db-generations/aiteam-2026-10-26.db`.
3. **D132 domain-lab cutover:** once DOMAIN-LAB-2 loads `com.aiteam.domain-lab` (`launchctl list`) and fills the migration note's load proof, comment out crontab line 71 (domain-hunter) + cron.txt, and swap the watchdog entry (note §7).
4. **Mark decisions:** M3 (repoint keyword-registry to SSG?), M4(b) (`majordomo/check.sh:240` stale asbestos queue line), M6 (market chain / domain-hunter / backlink-prospector still wanted?), backup volume encryption + generation pruning (D129), second alert channel (D123), apex 307 → 308 in Vercel, GSC credentials (32 SSG URLs never submitted).
5. **Other lanes** (not this lane's to do): SSG content defects OL1–OL8 (SSG-SITE-AUDIT-1), the pipeline back-port OL9 and draft staging OL11 (SSG-CONTENT-1, root fix for F-05), and the coupa Vendr-cited figures flagged in `ssg-content/docs/AITEAM_SSG_Finish_2026-10-26.md:39`.

## Blocked
- D132 on the lab loading its plist. D131/D134 on Mark (M6/M3).

## Known bugs (top, open)
| Bug | Severity | Where | Note |
|---|---|---|---|
| Single alert channel (Telegram only) | high | `lib/notify.sh` | D123; needs Mark's channel choice |
| Pipeline↔site fork (SSG 17/17, asbestos 24/41) | high | pipelines vs site repos | guard in place (`dd53056`); back-port = other lanes; D124 |
| Unapproved drafts staged inside `approved/` during gating | high | `ssg-content/content/run-batch.sh` | ship.sh holds on the lock (`99e6528`); root fix D125/OL11 |
| ~190 direct `sqlite3` writes without busy timeout | medium | fleet | D127 |
| Most `*_sent` rows still unverified delivery | medium | ship-to-site digest, briefs, research digest | D126 |
| ai-think.sh has no failure audit path | medium | `lib/ai-think.sh` | D130 |
| 41 doc-vs-code drifts | low | CLAUDE.md files | D128 |

## Gotchas (new)
- `claude -p` drains a piped stdin. Any model call inside `while read … done < <(…)` needs `</dev/null` (the wrappers have it now).
- macOS cron runs `/bin/bash` 3.2: `"${arr[@]}"` on an empty array under `set -u` is fatal. Use `${arr[@]+"${arr[@]}"}`.
- zsh (the interactive shell) doesn't word-split `$var`. Run sweeps under `/bin/bash`.
- `.env` assignments are not exported; pass flags to children explicitly.
- A `*_sent` row written after `notify … || true` is a claim, not a delivery.
- Never `pgrep -f` a pattern that appears in your own command line (it matches itself forever).
- `ship.sh --dry-run` now ends with `[scan-complete]` (or `[scan-skipped] <reason>`); callers treat its absence as a crash.
