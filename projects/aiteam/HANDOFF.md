# AITEAM Project Handoff
**Last updated:** 2026-10-31 ~16:15 PT (FIX-AND-AUDIT-1, Claude Code)
**Last session summary:** The Mac was moved to a new room on 2026-10-31 (clean shutdown 14:41, boot 15:29); WAKE-UP-CHECK-1 passed with two hardware items for Mark (Wi-Fi instead of wired, no UPS seen). FIX-AND-AUDIT-1 then applied fixes F1-F8, each tested and committed, and ran a read-only fleet audit. Report: `docs/AITEAM_Fix_And_Audit_2026-10-31.md` (copy in `~/Downloads/`).

---

## Current phase
**SSG shipping; asbestoshq.com PARKED (site live, pipeline off).** Deploy cap 2/site/night. Revenue: none yet (Scaling Rules: 1+ sites live, 0 revenue = do not scale).

## State (verified 2026-10-31)
- **Host:** booted 15:29 after the move. Default route is **en1 (Wi-Fi)**, not en0 (wired); **no UPS** listed by `pmset -g ps`. `majordomo/check.sh` now WARNs on both (31 checks: 29 OK, 2 WARN). Both are MARK items (cable / UPS USB lead).
- **Drives:** `ActiveSSD`, `Backups` (nightly mirror), `TimeMachine`. `AgentSSD` and `WDArchive` no longer exist. The backup finds `Backups` by volume UUID `E77DF891-2B8F-4649-B034-DB321978C57C`, then by name (no more `disk5s1`).
- **Jobs:** crontab == `lib/cron.txt` (domain-hunter 06:30 line now commented, REPLACED by domain-lab). 8 `com.aiteam.*` launchd jobs loaded. Watchdog watches `domain-lab` (lab.db digests) instead of `domain-hunter`.
- **Models:** aliases only (haiku/sonnet/opus → 5.5). Claude Code 2.1.316.
- **Kill switches (.env):** unchanged from 2026-10-26 (`RESEARCH_OPPORTUNITY_ENABLED=false`, `LINK_MONITOR_ASBESTOS_ENABLED=false`).

## Completed 2026-10-31 (FIX-AND-AUDIT-1)
- [x] Move done; WAKE-UP-CHECK-1 passed (`~/Downloads/AITEAM_WakeUp_2026-10-31.md`).
- [x] F1 `~/agents e0e6e35` backup finds the drive by volume UUID or name; `--resolve-only`; not found → Telegram + exit 2.
- [x] F2(a) `~/agents 27760a9` domain-hunter-lab added to the nightly backup (first night copies ~9.8 GB). F2(b) private GitHub remote: **not done, gh is not installed → MARK-ONLY.**
- [x] F3 `~/agents 051286b` domain-lab cutover (D132 closed): 5 good lab mornings; old cron line off; watchdog swapped.
- [x] F4 `~/agents 90c87d2` check.sh WARNs when the default route is not en0 or no UPS is seen.
- [x] F5 `ssg-content 936f54f` source gate: TIER1 only on a domain that belongs to the quoted vendor (vendor-to-domain map); vendr.com and similar marketplaces only as attributed, dated TIER3. Offline re-gate: coupa-alternatives and vendr-alternatives would now FAIL (not edited; see report).
- [x] F6 `ssg-content c2741a3` adjudicator verdicts stored and reused for identical figure + sentence + passage (cause of the Bill.com $0 flip: sampled model, no temperature control).
- [x] F7 `~/agents a497bca` one Telegram outcome line per drafter-fired run (`🧾 ssg <slug>: <state> — failing: <ids>`).
- [x] F8 (this commit) drive names corrected in current docs; D132 closed; this HANDOFF.
- [x] Independent verifier (fresh sub-agent, SHAs + claimed tests only): **F1–F8 CONFIRMED** (F8 on re-check, after a line in the report was reworded), **0 reverted**. Follow-up it raised: `ssg-content dafaacc` (F5 live test keeps the F6 adjudication store in its temp dir), also CONFIRMED.
- Fix table with verifier verdicts and revert commands: report § Fix table.

## In progress / next session should
1. **Read the report's top-5 findings and the A7 decision list** (`docs/AITEAM_Fix_And_Audit_2026-10-31.md`).
2. **Watch tonight:** 20:00 preview, 23:00 deploy, 23:30 backup is the first with the UUID lookup and the domain-hunter-lab source (expect a longer first run). 09:00 tomorrow: first drafter run with the outcome line.
3. **Guides the new gate would fail** (not edited): vendr-alternatives (Tropic/Vertice prices need a date) and coupa-alternatives (Zip table row unbacked). Fix on their next refresh, or Mark decides (A7).
4. **MARK:** plug in Ethernet (en0); connect or confirm the UPS; install `gh` + `gh auth login`, then create the private `domain-hunter-lab` repo (command in the report).

## Blocked
- F2(b) on `gh` (not installed). D131/D134 on Mark (M6/M3).

## Known bugs (top, open)
| Bug | Severity | Where | Note |
|---|---|---|---|
| Single alert channel (Telegram only) | high | `lib/notify.sh` | D123; needs Mark's channel choice |
| 17 of 23 approved SSG guides fail the current source gate even with every figure assumed present | high | `content/ssg/approved/guides/` | pre-existing (old and new policy alike); see report A3 |
| Pipeline↔site fork / unapproved drafts staged inside `approved/` | high | pipelines vs site repos; `run-batch.sh` | D124, D125 |
| First adjudicator verdict for a new claim is still a model sample | medium | `scripts/verify_sources.py` | F6 makes it repeatable, not right |
| ~190 direct `sqlite3` writes without busy timeout | medium | fleet | D127 (watchdog logged `query_failed_rc5` today) |
| ai-think.sh has no failure audit path | medium | `lib/ai-think.sh` | D130 |

## Gotchas (new)
- Tracked runtime files exist (`watchdog/probe_unknown_logged.date`): they show as modified in `git status` between autocommits. Don't stage them with a fix.
- `pipeline_runs.log` has bare `\r` spinner output: Python's universal newlines and `grep -n` count lines differently.
- `claude -p` loads the cwd's CLAUDE.md: the same prompt is ~3.3k tokens larger from inside `~/projects/ssg-content`.
- A domain's own name in a claim sentence is not evidence it is the vendor's site (F5). Add a vendor's real domain to the slug's keyword config (required_entities / ngram_ignore) or to `scripts/vendor_domains.json`.
- Older gotchas (2026-10-26): `claude -p` drains stdin (`</dev/null`); bash 3.2 empty arrays under `set -u`; zsh doesn't word-split; `.env` vars aren't exported; a `*_sent` row after `|| true` is a claim; never `pgrep -f` your own pattern; `ship.sh --dry-run` ends with `[scan-complete]`.
