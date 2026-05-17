# AITEAM Context Save — 2026-05-17_0030
**Generated:** 2026-05-17T00:30:42-0700
**Since last save:** 2026-05-17 00:28:00
**Session topic:** Policy lock + brain bookkeeping — POLICY.md created (7 operator answers from cross-agent audit §7), DEFERRED.md synced with sister-chat items, D045 closed obsolete + D061/D062 opened, doc-vs-reality correction box on cross_agent_failure_modes_2026-05-16.md.

**Note on mechanical record:** Empty because `ctx.sh` checks `~/agents/` git activity, not `~/brain/`. This session's commits all landed in `~/brain/` (b2a6fb0, bcc16d5, 38f77d0, 9818e20, 6e7fb52). No `~/agents/` work this window.

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

- **POLICY.md is now the authoritative source for Q1–Q7 operator decisions** (commit `6e7fb52`). Future sessions read POLICY.md instead of re-deriving in conversation. Locked: Q1 false-positives are worse than false-negatives → gates lean strict; Q2 $25 soft / $50 hard daily cap (Max-sub-equivalent); Q3 manual page-only rollback (automate after 3 incidents); Q4 4–20 clean ships burn-in before review-window reduction; Q5 serialize run-batch.sh via lockfile; Q6 watchdog agent built now (not deferred); Q7 editor runs after audit_guide.py as second gate (mechanical-cheap → editor-judgment, both must pass).
- **D045 closed obsolete, replaced by D061 + D062** (commit `9818e20`). Pre-flight on `run-batch.sh` showed writer/auditor already on Sonnet by default at lines 107-108 — D045's "Opus → Sonnet ~75% cost reduction" premise was completely stale. Real remaining value (kill-switch enforcement + per-call token attribution via `ai-do.sh`) carved out as D061; vestigial `--sonnet`/`--sonnet-audit` flag cleanup as D062. Alternative considered: do the naive `sed` swap of `claude -p` → `ai-do.sh`. Rejected because `ai-do.sh` would brick the cron-fired pipeline — it doesn't pass `--dangerously-skip-permissions`, hardcodes `--max-turns ${AGENT_MAX_TURNS}`, and forces `--output-format json`. First Read/Write tool call would hang on a permission prompt with no operator.
- **D-number collision resolved D060 → D061+D062.** Operator requested D060+D061; pre-grep found `D060` already in use at line 380 ("3 untracked production scripts"). Asked operator; chose next-free pair. Renumber-the-existing-D060 path rejected as more disruption than benefit.
- **cross_agent_failure_modes_2026-05-16.md correction strategy: prepend a correction box, don't edit the body.** R1/R2/R3 tiering is referenced throughout the doc but exists nowhere in code (grep `R1`/`R2`/`R3` returns zero in `~/agents/` or `~/projects/`). Editing the body would have rewritten ~865 lines of audit reasoning. Correction box (7-line blockquote at top) preserves the doc as historical record of planning intent while flagging the gap.
- **Untracked-file commit shape: split into two commits.** The failure-modes doc was untracked; committing under the "doc-vs-reality correction" message would have actually committed 872 lines (whole file + 7-line correction) — misleading. Two commits: `bcc16d5` (add the untracked doc as-is) + `38f77d0` (the 7-line correction). Alternative: single combined commit. Rejected because the commit message would lie about the change.
- **Doc-vs-reality Gaps 2 and 3 left untouched after verification — not reflexively edited.** Gap 2 ("production judge" misnomer): all 5 matches in historical context-save / session-handoff files; zero matches in active docs. Gap 3 ("Cloudflare Pages" references): every active-doc reference already states "Vercel, not Cloudflare". Both reported as zero-edit gaps rather than fabricating changes to satisfy task framing.

## Lessons learned

- **Stale-deferred-item premise check belongs in pre-flight, not in execution.** D045 was scoped as a model swap that had already happened. Two minutes of `grep "claude -p"` + reading lines 107-108 surfaced the staleness. If pre-flight had been skipped, the session would have produced a no-op edit and a false "~75% cost reduction shipped" report. Rule: for any deferred item older than a couple of weeks, before executing, grep the current state to confirm the stated premise still holds.
- **`ai-do.sh` is NOT a drop-in `claude -p` replacement for non-interactive callers.** Missing/different flags: no `--dangerously-skip-permissions` (cron-fired pipeline would hang on first tool-use permission prompt), hardcoded `--max-turns ${AGENT_MAX_TURNS}` (currently 30; was unlimited), forced `--output-format json` + Python extraction of `.result` (changes stdout shape — fine for run-batch.sh which doesn't consume stdout, breaks for callers that do). Any "swap `claude -p` for `ai-do.sh`" task needs a per-callsite flag audit. Carried into D061's notes for the eventual build.
- **DEFERRED.md D-number collisions are silent — pre-grep is non-negotiable.** Operator requested D060; D060 was already in use. The file's own authoring notes flag this risk; rule should be `grep -nE "^### D[0-9]+|^\| \*\*D[0-9]+" DEFERRED.md` (covers both detail sections AND the Priority view table) before assigning any number. Two D045s and two D051s and two D027s already exist in the file as evidence of past collisions.
- **Doc-vs-reality audits often find the gap is already mitigated — verify before editing.** 2 of 3 gaps in this session's audit were non-issues. Reflexively editing to satisfy task framing would have introduced churn or, worse, *un*corrected an already-correct doc. Rule: read the surrounding context of every grep hit before editing. "Cloudflare" near "Vercel, not Cloudflare" is a *correction*, not a stale assertion.
- **Untracked files in `~/brain` trap commit messages.** `git ls-files --error-unmatch <path>` reveals whether a "small fix" diff is actually `git add` of a multi-hundred-line file. If untracked: split into add-file + apply-fix commits, or update the commit message to honestly describe the addition. Never let the commit message lie about diff size.
- **POLICY.md pattern: when operator decides ≥3 cross-cutting questions in one session, durable file beats context-save.** Context-saves are time-sliced narrative — they decay as the relevant decisions get buried under newer sessions. POLICY.md is a single grep target: `grep "## Q2" POLICY.md` instantly answers "what's the daily cost cap?" without scrolling through ten context-saves. Future-CC should treat POLICY.md as authoritative and not re-derive Q1–Q7 in conversation.
- **`ctx.sh` mechanical record only covers `~/agents/`, not `~/brain/`.** Brain-only sessions produce blank mechanical records ("no commits since last save"). The narrative section becomes the entire signal. Future-CC reading a brain-only context save: don't conclude "nothing happened" from the empty mechanical block — read the narrative.

## Operator corrections

- **"Use D061 + D062" instead of operator's originally-specified D060 + D061.** I surfaced the D060 collision and offered three numbering paths; operator picked the next-free pair. (Confirmation captured via `AskUserQuestion`.) Not a correction of approach — a correction of input parameters under new information.
- **"Close D045 as obsolete and open successor"** (the actual D045 task framing). Operator's response to my pre-flight HALT report. Initial task framed as "rewire D045"; after pre-flight findings showed the premise was stale, operator pivoted scope rather than push through. Pattern: HALT-with-findings is preferred over execute-anyway, even when execution looks like a one-line sed job.
- **"Two commits"** for the cross_agent_failure_modes correction. Operator chose the cleaner-git-history path over the simpler one-commit path when I surfaced the untracked-file mismatch. (Confirmation captured via `AskUserQuestion`.)
- **No correction on POLICY.md task.** Task was specified verbatim with exact content; my execution was pre-flight → write → verify → commit, no judgment calls escalated. Operator's task description was the policy itself, not a request for me to derive policy.

## What's next

**Immediate (operator can pick up any of these next session):**

1. **Build watchdog agent (Q6).** Policy is "build now". Spec exists in `cross_agent_failure_modes_2026-05-16.md` §6. Monitors alive/dead agents, cron-fired-but-no-completion-row, audit-log anomalies. No upstream blockers.
2. **F17/F18 cost cap build at `log_token_usage.sh` (Q2 answered).** $25 soft warning → Telegram + drafter cron pause. $50 hard stop → `SYSTEM_PAUSED=true` sentinel for `check_kill_switches.sh`. Now unblocked.
3. **D056 — editor verdict persistence (Q1 + Q4 + Q7 answered).** Editor runs AFTER `audit_guide.py` as second gate; FP-leaning default per Q1; `pass_threshold` env var. Persists to `auditor_verdicts` (composite_score column already reserved).
4. **`run-batch.sh` lockfile (Q5).** ~10 lines bash. Check `/tmp/run-batch-asbestos.lock` at start; trap-cleanup on EXIT; log skip to audit if locked.

**Deferred (waiting on triggers):**

- **D061** — `ai-do.sh` rewire of `run-batch.sh`. Trigger: after F17/F18 ships (so the kill-switch hooks have something to enforce) OR sooner if per-call token attribution becomes urgent.
- **D062** — vestigial flag cleanup in `run-batch.sh`. Bundle with D061.
- **D057** — remove deploy throttle. Trigger: 4-20 clean burn-in ships per Q4.
- **D058** — Vercel preview URLs in 20:00 preview ping. Trigger: operator wants them.
- **D-SSG-01..09** — all the SSG-fork items synced from sister chats this session. Triggers vary by item; mostly "before first SSG batch run".

**Blocked:** None. All 7 operator-policy questions are now answered.

**Stale state to watch:**

- The "Last context save" link in the prior HANDOFF.md points to `2026-05-17_0023.md`; this save (`_0030.md`) is now the latest. HANDOFF.md update below refreshes the pointer.
- The Claude.ai project file (separate from on-disk repo) reportedly contains the "editor as production judge" misnomer. Operator said they handle that manually — flagged here so future-CC doesn't try to chase it in `~/brain/`.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-17_0025.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-17 00:25 **Last session summary:** D044 caller-layer cleanup — removed the redundant `tr "'" '_'` scrub from `config-synthesizer/synth.sh#handle_failure()` after end-to-end verification confirmed the shared-lib SQL-escape fix (`log_to_audit.sh`, commit 939361d) handles apostrophes correctly on all six positional fields. Net −7 lines, single source of truth restored (commit c1aa9c0). Closed D045, D-SSG-05, D054 — trigger conditions met by prior commits.  --- 
