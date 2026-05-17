# AITEAM Context Save — Autonomous Loop Live End-to-End (Asbestos)

**Date:** 2026-05-16 (evening session, ~7 PM – ~11 PM Pacific)
**Session character:** Cross-agent failure-mode fixes. Three real bugs closed, four deferred items logged, autonomous loop proven functional with first organic slug.

---

## TL;DR

**Milestone:** First organic slug shipped end-to-end with zero human intervention. `white-asbestos-vs-blue-asbestos` traveled from `drafter_queue.txt` → assignment-drafter → run-batch → auditor → `approved/guides/` → ship-to-site → live on `https://www.asbestoshq.com/guides/white-asbestos-vs-blue-asbestos/` (HTTP 200). Asbestos site only — SSG still needs data-driven rewrite + content fork re-skin before its loop can close.

**Bugs closed:** F4/D044 (SQL injection in shared lib), F2 (no pre-flight keyword-config check), ship-to-site cron npm PATH bug.

**Deferred items logged:** D052, D053, D054, D055.

**Mission bar status:** Still $0 revenue toward $200/mo break-even. Loop is now in place — revenue depends on operator running keyword research and enqueuing slugs.

---

## WHAT GOT DONE THIS SESSION

### 1. F4 / D044 — SQL escape in log_to_audit.sh (commit 939361d)

Heredoc with shell-expanded values was vulnerable to apostrophes in any field. Confirmed bug: config-synthesizer/synth.sh line 252 documented a defensive caller-layer workaround. ~20 callers fleet-wide silently broken.

Fix: SQL-escape `'` → `''` via bash 3.2 parameter expansion before heredoc interpolation. Also bundled a prior-session trailing-`}` fix in PAYLOAD default that had been sitting uncommitted.

**Sign-off:** Clean call ✅, apostrophes-everywhere call ✅, config-synth-style payload ✅, recursive proof (script logging its own fix via the fix) ✅.

**Bash 3.2 gotcha worth remembering:** `"${VAR//\'/\'\'}"` does NOT work on macOS bash 3.2 — backslashes leak literal. Pattern that works:
```bash
APOS="'"
VAR_ESC="${VAR//$APOS/$APOS$APOS}"
```

### 2. F2 — pre-flight keyword-config check in fire_pipeline.sh (commit 0df8cf6)

Drafter previously fired run-batch.sh for any slug without verifying a keyword-config existed, burning $1-5 of Opus tokens before failing.

Fix: pre-flight check reads `site_name` from yaml, verifies `${PIPELINE_REPO}/content/${site_name}/keyword-configs/${slug}.json` exists. If missing: log skip to audit + runs log, exit 2. Site-agnostic via existing yaml keys.

Also: first-ever commit of the full `assignment-drafter/` agent. Same pattern as ship-to-site (commit 3b43c11) — agent had been running in production via cron */30 without ever being staged in git. Flagging for project-rule attention.

**Sign-off:** bash -n ✅, missing-config exit-2 + audit row ✅, config-present pre-flight passes ✅, fix logged ✅.

### 3. Ship-to-site cron npm PATH bug (commit 0be9d1f)

**Diagnosis chain (worth preserving as a how-we-debug example):**

Started with: `curl -I https://www.asbestoshq.com/guides/white-asbestos-vs-blue-asbestos/` → 404.

Then: 31 manually-shipped guides live, autonomous slug not. Approved JSON existed in pipeline repo. Audit trail showed 10 silent `ship_to_site_skipped` rows for the slug.

Drilled into payload: `reason=in_needs_review_queue`. The slug was parked.

Walked back further: the first non-skip row (audit row 536) revealed the original failure — `npm: command not found`. Cron's stripped PATH excludes `/opt/homebrew/bin`. Every cron-driven ship had been failing identically; only the 31 manual ships made it live.

Fix: prepend `/opt/homebrew/bin:/opt/homebrew/sbin` to PATH at top of build_verify.sh. Idempotent (case-check before adding).

**Surprise during retry:** PATH fix landed (npm now found), but next failure surfaced — `sh: next: command not found`. Root cause: `~/projects/asbestoshq-site/node_modules/` had vanished. Something wiped it between manual shipping and autonomous retry (cause unknown).

Resolved by `npm install` in site repo (365 packages). Retry then succeeded end-to-end.

**Sign-off:** End-to-end ship of white-asbestos-vs-blue-asbestos ✅. Live HTTP 200 ✅. Audit row 868 captures success.

---

## DEFERRED ITEMS LOGGED THIS SESSION

| ID | Item | Trigger |
|---|---|---|
| D052 | drafter.sh exit-2 vs exit-1 routing — currently both treated as hard fail; needs case block to route soft-skip into `SKIPPED_NO_CONFIG_SLUGS` bucket with ⚠ instead of ❌ Telegram | Next drafter.sh-touching session |
| D053 | assignment-drafter state files not gitignored — `state/daily_pipeline_spend.txt` etc. dirtying git status on every cron tick | Bundle with D051 hygiene session |
| D054 | Audit ALL cron-invoked scripts for PATH dependencies — same npm-not-found risk applies to any script calling Homebrew binaries (npm, node, python3, jq, sqlite3, gh) | Next cron-job-touching session, or when another cron script fails for the same reason |
| D055 | Site repo node_modules vanishing mystery — was present at manual ship time, absent at autonomous retry. Cause unknown. Risk of recurrence. | If it disappears again, or next ship-to-site polish session |

Brain commits: d1eecbd (D052), 30f64b6 (D053), 74e9400 (D054 + D055).

Also surfaced but not logged formally: **config-synthesizer caller-layer workaround at synth.sh:252 is now redundant** (F4 fixed at shared lib level). Remove in next config-synthesizer-touching session.

---

## KEY LEARNINGS

### Bash 3.2 escape syntax (add to project lessons)
`"${VAR//\'/\'\'}"` does NOT work on macOS bash 3.2 — backslashes leak literal into the SQL. Pattern that works:
```bash
APOS="'"
VAR_ESC="${VAR//$APOS/$APOS$APOS}"
```

### Cron PATH is stripped — always
This is a known Unix gotcha, but easy to forget when developing in a rich shell. Anything cron invokes needs explicit PATH hardening, or absolute paths, or an explicit `PATH=...` in crontab. D054 captures the systemic version.

### Audit-trail design pays off
The audit log was good enough to walk from "ship failed silently" to root cause in ~10 minutes of diagnostic SQL. Every silent skip was traceable. Every parked slug had its first-failure row preserved. This is the design working as intended.

### Production agents landing without git commits (project rule pattern)
Three agents now caught running in production with no git history at end-of-build:
- ship-to-site (commit 3b43c11, prior session)
- assignment-drafter (commit 0df8cf6, this session)
- config-synthesizer (handled in parallel chat earlier today via commit 69dd30f)

Pattern worth surfacing: builds end with "✅ complete" without verifying commit landed. Consider adding a final "verify in git" step to build-prompt templates.

### Silent skip reasons need surfacing
ship-to-site logs `slug_already_shipped` and `in_needs_review_queue` reasons silently to keep cron output quiet. Good UX choice for normal operation, but during this session it took an extra step to query the JSON payload to find the real reason. Worth considering: a `--diagnose` flag on ship.sh that surfaces last-N skip reasons for a slug. Not formal deferred — flag if it bites again.

### Operator instinct: don't bundle commits with misleading messages
Caught twice this session — option 1 (expand commit message to reflect bundled fix) was the right call both times. Option 3 (just commit per literal spec, message understates) is the bad option the project rule explicitly forbids.

---

## FINAL STATE (END OF SESSION)

### Live agents (no changes this session)
Orchestrator, Librarian, Editor, Diary Writer, TG Monitor (reader + analyzer), Market pipeline (scribe → analyst → curator → briefer), Ship-To-Site, Assignment Drafter, Config Synthesizer.

### Asbestos site
- 32 guides live (31 manual + 1 autonomous from this session)
- Auto-ship via cron */15 verified functional end-to-end
- Daily 07:00 digest scheduled (next morning: check it fires correctly)

### SSG site
- Still needs data-driven rewrite (Chat B/C planning this in parallel today)
- Content fork still has 36 asbestos path references in run-batch.sh (D-SSG-01 from parallel chat)
- SSG flags remain `enabled: false` in Ship-To-Site + Assignment Drafter

### Mission bar
- $0 revenue → $200/mo break-even target unchanged
- Loop functional; revenue depends on operator running keyword research

### Recent commits

`~/agents/`:
- `0be9d1f` Fix ship-to-site cron npm PATH bug
- `0df8cf6` Fix F2: pre-flight keyword-config check in fire_pipeline.sh
- `939361d` Fix F4/D044: SQL-escape single quotes in log_to_audit.sh
- `3b43c11` ship-to-site: initial commit + strip template provenance before stage

`~/brain/`:
- `74e9400` DEFERRED: add D054 + D055 (cron PATH audit + node_modules vanishing)
- `30f64b6` DEFERRED: add D053 (assignment-drafter state files not gitignored)
- `d1eecbd` DEFERRED: add D052 (drafter.sh exit-2 vs exit-1 routing)
- `e402c5a` brain: update HANDOFF.md, refresh DEFERRED.md

---

## RECOMMENDED NEXT SESSION OPENING MOVE

1. **Verify the 07:00 digest fired correctly** the morning after this session — first organic test of ship-to-site's daily reporting under real conditions.
2. **D045 — rewire run-batch.sh from `claude -p` → `ai-do.sh`** (Opus → Sonnet). Single biggest cost-reduction lever in the project right now. ~75% off pipeline runs. Test on next organic slug after digest verification.
3. **7 operator-policy questions** from cross-agent audit §7 — needs focused operator attention, not rushed:
   - Auditor false-positive vs false-negative tolerance
   - Daily API cap — number + hard or soft stop
   - Bad-content rollback policy
   - Burn-in publication count before re-introducing approval gate
   - Concurrent run-batch.sh — allow or serialize
   - Watchdog agent — build now or after F2/F4 (F2/F4 now done)
   - Editor agent — slot in as pre-ship gate?
4. **Then F14/F17/F18** if not already absorbed by the policy answers.

After that, the remaining roadmap is largely Chat B/C's domain (SSG rewrite + content fork re-skin + 15 JSX → JSON migration) plus operator-only work (keyword research, affiliate enrollment, GSC submission).

---

## OPEN E2E — RESOLVED

White-asbestos-vs-blue-asbestos: ✅ shipped, ✅ live (HTTP 200), ✅ audit trail captured.

No outstanding in-flight pipeline runs at session close.

---

## CRITICAL NOTES FOR NEXT CLAUDE

- Asbestos autonomous loop works. SSG does not yet (data-driven rewrite still pending).
- F4 + F2 closed; the shared lib is safe for new agents. config-synthesizer:252 workaround is now redundant.
- Cron PATH gotcha is a systemic concern (D054). Any new cron-invoked agent script needs explicit PATH hardening.
- node_modules vanishing (D055) is unresolved — root cause unknown. Watch for recurrence.
- Three production agents (ship-to-site, assignment-drafter, config-synthesizer) lacked git history before this session. Pattern of "build complete ≠ committed" — flag if it happens again.
- Don't push operator into content production. Stated preference: build systems first, content second. Mission bar still $0, but the system is the priority.

---

*End of context save. Loop is live; rest is polish, SSG, and operator inputs.*
