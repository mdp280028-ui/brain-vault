# AITEAM Context Save — 2026-05-17_1424
**Generated:** 2026-05-17T14:24:01-0700
**Since last save:** 2026-05-17 13:57:05
**Session topic:** SSG scoping recon + site-repo locator (read-only follow-on to watchdog deployment in same conversation)

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
|  id  |         ts          |      actor_id      |         action          |  target  |
|------|---------------------|--------------------|-------------------------|----------|
| 1162 | 2026-05-17 14:15:01 | watchdog           | watchdog_check_complete | _summary |
| 1161 | 2026-05-17 14:00:01 | watchdog           | watchdog_check_complete | _summary |
| 1160 | 2026-05-17 14:00:00 | assignment-drafter | drafter_tick_started    |          |

### Token usage
(no token usage since last save)

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

**Reading window note.** This save captures 13:54–14:24 — the read-only window of a conversation whose code-writing phase (watchdog deployment + 4 follow-up fixes) was committed BEFORE the 13:54 sister-chat save. Those commits are in that prior save's reference window, not this one's. Decisions below are from the read-only phase.

- **SSG site repo is `~/projects/smartsourceguide/`, NOT lost.** Operator's recon brief assumed "not in ~/projects/" and asked for GitHub-side enumeration via `gh`. The repo is locally cloned, on `main`, last commit `c05c1f6` 2026-05-16. Every marker from the 2026-05-16 dump matches (Next 14, `/app/` at root not `src/app/`, 14 articles matching SITE_MAP.md slugs, hand-rolled CSS, vercel.json, remote `mdp280028-ui/smartsourceguide`). Decision: report finding and NOT clone (would have been destructive of local state). Alternative considered: clone anyway "to be safe" — rejected because the local copy is the authoritative working tree.
- **gh CLI gap re-encountered, sidestepped via existing git remotes.** Yesterday's LESSONS.md line 149 already documented `gh` missing on this Mac. Today the SSG recon spec started with `gh repo list` — fell at the first hurdle. Decision: don't propose `brew install gh` (operator hasn't asked for it as infra); use `git -C asbestoshq-site remote -v` to infer the org name (`mdp280028-ui`), then inspect `~/projects/` directly. Saved a tool-install detour.
- **SSG content-pipeline state captured but NOT acted on.** Recon found `ssg-content/` machinery is fork-complete but zero content inputs exist (no `keyword_research.md`, no `approved-index.md`, no `authority-links.md`, no `drafter_queue.txt`, no exemplars). `ssg.yaml: enabled: false` with explicit TODO listing those 4 prereq files. Decision: report the gap list and stop; no scaffolding work without operator decisions on D-SSG-02, D-SSG-03, target persona/voice, and the missing `AITEAM_SSG_DataDriven_Rewrite_Plan*.md` doc (which is NOT on disk anywhere readable — `/mnt/project/` referenced in the spec doesn't exist on macOS).
- **Watchdog work in same conversation produced 7 commits across two repos, all before 13:54.** Listed for traceability: `fb53d04` (drafter heartbeat), `88f8d7a` (watchdog detector + schedule + state dedup), `61ea5d1` (brain incidents/ scaffold), `1053913` (state-corrupt guard), `38ed525` (grace_minutes wiring), `26d3311` (drafter window 2h→1h), `1e90165` (brain LESSONS.md entries). All sign-offs ✅. Watchdog has been on cron `*/15` + daily 06:00 digest since 11:02 PT. First production tick fired 11:15:00, alerted exactly once on orchestrator (`diary_written` missing in 26h), no churn since. State at end of session: 6 healthy, 1 missing (orchestrator — real signal, awaiting tonight's 23:45 diary).

## Lessons learned

- **`/mnt/project/` is a Linux/cloud convention, not macOS.** Spec referenced a doc at `/mnt/project/AITEAM_SSG_DataDriven_Rewrite_Plan*.md` — `/mnt/` doesn't exist on macOS. When a spec path starts with `/mnt/`, it's a sign the spec was authored on Linux (likely a sister-CC chat running in a cloud sandbox). Translate to macOS: try `~/Downloads`, `~/Documents`, the brain vault, or ask operator to paste. Don't waste a `find` against root.
- **Repo locator: a missing keyword in the repo name doesn't mean missing repo.** `~/projects/smartsourceguide/` IS the SSG site, but `ls | grep -i ssg` only returned `ssg-content`. Operator's recon brief assumed "not in ~/projects/" based on the same grep. Defense: when looking for a domain's site repo, grep also for the domain's product name (in this case "smartsource"), category names ("fleet", "answering"), or check `git remote -v` of nearby repos to learn the org and infer siblings.
- **`gh` CLI gap is recurring blocker; lesson holds from yesterday.** Second time this week a spec started with `gh repo list` and stopped at the first command. The fix is `brew install gh && gh auth login`, but operator hasn't signaled "install gh as infra" — so the workaround is the right move per session: use existing `git remote -v` outputs to infer org names, then `ls ~/projects/` to enumerate local clones. Track as recurrent: if it happens a third time, raise the install question explicitly.
- **State-file validity gating beats post-hoc detection.** Watchdog fix #5 inserted a `jq -e 'type == "object"'` gate at tick start with a flag-file dedup pattern (`~/store/flags/WATCHDOG_STATE_CORRUPT`). Caught both `"not json"` and zero-byte cases; auto-clears on first clean parse. Pattern generalizes: any agent that depends on a parseable state file should validate-and-flag-or-bail at startup, not let downstream silently swallow parse errors. The 14× `jq` per-tick reparse in watchdog (each silently swallowing) was the actual alert-storm vector — gating prevents it.
- **Probe-vs-decision threshold duplication is a units-divergence trap.** Watchdog fix #3 (grace_minutes wiring) had to land in FOUR places: 3 probe functions + main loop. A "minimal" fix that only patched the main loop would have silently failed because probes filter SQL by their own threshold before signals reach the decision branch. Lesson: when an agent computes the same threshold in multiple places (DRY violation), changes to the formula must reach all sites. Better to refactor to a single helper, but if not, at least audit-grep for every occurrence of the formula before claiming the fix.
- **A/B dry-run is the cleanest "this field is consulted" evidence.** For grace_minutes wiring, the spec's literal step e (single run with grace=1) was provably inconclusive — the result could be "missing" with grace=1 OR with grace=0. Strengthened to two runs with everything-else-identical (`window=0 grace=0 → missing`; `window=0 grace=35 → healthy`). The flip is direct evidence the field is consulted; non-flip is a failed fix. Pattern: when adding a new code path that consumes a previously-ignored input, prove it with A/B holding everything else constant.
- **Heartbeat-before-killswitch design validated in the wild.** At 13:02:02 a manual off-cycle drafter run wrote `drafter_tick_started` (audit id 1149) BEFORE `skipped_pause_flag` (id 1150). Watchdog stayed healthy throughout because heartbeat is "cron fired" semantics, not "work happened" semantics. Already captured in LESSONS.md commit `1e90165` — re-stating here because it's broader than drafter: any agent that needs BOTH a kill switch AND watchdog visibility should fire heartbeat first.
- **Spec premises drift fast; re-verify at pre-flight.** "Commit pre-existing dashboard work" task from earlier in this conversation: spec said `dashboard/server.js` had 384/12 uncommitted lines. Pre-flight `git diff HEAD -- dashboard/server.js | wc -l` returned 0 — work had already landed as `a7c2c71` during the same session (sister-chat). Correct response was STOP, not commit-anyway. Pattern: any spec referencing specific line counts or "uncommitted" state must verify against current `git status` / `git log` before acting. Saved a duplicate-commit / merge-conflict mess.

## Operator corrections

- **"GO. Heading A (new ## 2026-05-17). Entry 1 use γ (cross-reference to line 87, not a duplicate)."** Operator picked between three placement options and three duplicate-handling options I'd surfaced in pre-flight. Confirmed the file-convention reading was correct (existing 2026-05-17 wee-hours entries went under ## 2026-05-16 because they were night-of, not because that's the rule). Validates the pre-flight habit of surfacing convention choices BEFORE editing — operator made the right call, would have been wrong to assume.
- **"GO. Plumb grace_minutes into all four threshold sites (three probes + main loop)... Run step e as the strengthened e1/e2 pair... Non-flip is a failed fix — report ⚠ if it happens."** Operator accepted the strengthened test rather than the literal spec form. Confirms: when a CC pre-flight identifies a probative-evidence gap in the spec's test, the right move is to surface the alternative WITH rationale. Operator's GO included a fallback ("report ⚠ if non-flip") that I would not have stated explicitly without prompting — note to file: state the failure-detection rule explicitly in sign-off plans, not just the success rule.

## What's next

**Immediate (operator-decisions still pending for SSG):**

1. **Paste or repath the `AITEAM_SSG_DataDriven_Rewrite_Plan*.md` doc.** Not on disk; `/mnt/project/` path doesn't exist on macOS. Until the plan is visible, recon cannot map current state against rewrite deltas.
2. **D-SSG-02 sign-off** — operator review of `SSG_TEMPLATES.md` Cosmetic Tolerance List + Verdict Rules. CC authored from first principles (asbestos source files were stubs). Blocks first SSG audit run.
3. **D-SSG-03 sign-off** — confirm `relatedLinks` slug shape (CC chose `"it-support/managed-it-services-pricing"`; 1-line edit if convention differs). Blocks first SSG batch run.
4. **Author content inputs** — `keyword_research.md`, `approved-index.md` (starts empty), `authority-links.md`, 2+ exemplar `assignment-batch-*.md` for SSG voice. Listed in `~/agents/assignment-drafter/config/ssg.yaml` TODO.
5. **Decide whether SSG ships into `smartsourceguide` site directly** or stays approved-JSON-only until phase 2.

**Deferred (no urgency):**

- **D056** — editor verdict persistence (Q7 answered; pre-reqs unblocked; no progress this session).
- **D058** — Vercel preview URLs in 20:00 preview ping.
- **D059** — SSG deploy throttle (correctly gated on `ssg.yaml: enabled: true` AND first batch shipping).
- **Watchdog nits logged this session** (not actionable): duplicate threshold math in 4 sites, unused `_stale` action suffix in probe returns, state.json reparsed 14× per tick. All three are quality-of-life refactors, no functional bugs.

**Blocked:**

- Everything SSG-pipeline-related (items 1–5 above gate the autonomous loop).
- Watchdog production observations: first overnight cycle (23:45 diary, 23:00 deploy, 23:55 brain-autocommit, 07:00 digest) hasn't run yet. Recovery → healthy transition for `orchestrator` will fire tonight when `diary_written` lands; check tomorrow morning that exactly ONE recovery Telegram fired.

**Health-check for next session:**

```bash
# Verify watchdog overnight behavior
sqlite3 ~/store/aiteam.db "SELECT id, datetime(ts,'unixepoch','localtime'), action, target FROM audit_log WHERE actor_id='watchdog' AND ts > strftime('%s','now') - 86400 ORDER BY ts;"
# Verify state.json end-of-night
jq '.' /Users/mmm2/agents/watchdog/state.json
# Verify daily digest fired at 06:00
ls -la /Users/mmm2/agents/watchdog/digest.log
```

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-17_1354.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-17 13:55 (multi-cluster execution session — GSC queue → F17/F18 cost cap → POLICY Q5 lockfile → D066 hygiene audit → cluster #9 → D061+D062) **Last session summary:** Heavy execution session. Shipped GSC submission queue + multi-site upgrade (commits `c56209f` + `60b2486`). Committed substantial pre-existing dashboard work via surgical extraction (`a7c2c71`). Built F17/F18 cost-cap enforcement with `$25/$50` POLICY Q2 caps — `~/agents/lib/check_cost_caps.sh` wired after every `token_usage` INSERT; hard-cap path rewrites `.env` `SYSTEM_PAUSED=true` (the actual enforcement signal) AND touches a flag file as idempotency marker; daily 00:00 cron flips both back; removed obsolete orchestrator $10 + drafter $15 caps; commit `54a69bd`. Updated PRIME.md with Read first section + dropped stale DAILY_API_BUDGET_USD rule (`af39f8d`). Shipped POLICY Q5 per-site lockfile in both run-batch.sh forks (`ac2f721` asbestos, `7dd5c27` ssg). Executed D066 untracked-code hygiene audit — 16 commits across 3 repos committed ~50 untracked production files in 12 logical groups (A–L), 8 gitignore patterns added, 6 scratch files deleted. Closed cluster #9: D064 arg-shape validation in `log_to_audit.sh` (`ebb0507`), D060 auto-closed by D066, D052 fire_pipeline.sh rc case-routing (`4d45b95`), run-batch.sh dead-code removal in both repos (`082e957` + `56a86e0`). Closed D061+D062: routed all 16 writer/auditor callsites through ai-do.sh via new `AI_DO_SKIP_PERMISSIONS` env hook (`5d68876` ai-do.sh + `4ea5242`/`1864971` run-batch.sh); removed vestigial `WRITER_MODEL`/`AUDITOR_MODEL` indirection (`7166c7b`/`c71a54c`). 1 mid-session HALT (D066 group C/E surfaced 4 untracked files post-commit — resolved cleanly via Option 1: 2 follow-up commits). 3 audit-row malformed-row corrections via supersedes-pattern (D066/D052) — D064 validation now blocks recurrence. **Prior session summary (preserved):** Brain-only bookkeeping session. Locked 7 operator-policy answers from cross-agent failure modes audit §7 into POLICY.md (commit `6e7fb52`). Closed D045 as obsolete; opened D061 + D062. Synced 10 D-items from sister chats into DEFERRED.md. Added correction box to `cross_agent_failure_modes_2026-05-16.md`. **Prior-prior session (preserved):** Pipeline discovery + SSG content-pipeline scaffolding. Cloned 5 production repos from GitHub into `~/projects/`; built `~/projects/ssg-content/` as local-only fork.  --- 
