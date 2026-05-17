# AITEAM Context Save — 2026-05-16_2311
**Generated:** 2026-05-16T23:11:12-0700
**Since last save:** 2026-05-16 23:10:15
**Session topic:** Pipeline-wrap stack expansion + cross-agent failure-modes audit + closing two of its top-5 priority fixes end-to-end (F2 drafter pre-flight, F4/D044 audit_log SQL-escape) + repairing autonomous shipping. Built config-synthesizer agent from spec, extracted page.tsx template from 3 real wrappers, ran a 23-failure-mode cross-agent audit, refreshed HANDOFF + DEFERRED, then fixed two of the audit's top-5 hits, restored ship-to-site's cron-driven shipping (npm PATH bug + missing node_modules), and shipped `white-asbestos-vs-blue-asbestos` live via the now-working autonomous chain (HTTP 200 confirmed). Commits: `~/agents/` (`69dd30f` config-synth, `3b43c11` ship-to-site first commit, `939361d` F4/D044, `0df8cf6` F2 + assignment-drafter first commit, `0be9d1f` ship-to-site PATH fix), `~/projects/asbestos-contractors/` (`e504793` --validate-config-only), `~/brain/` (`e402c5a` HANDOFF+DEFERRED refresh, `d1eecbd` D052, `30f64b6` D053, `74e9400` D054+D055). Total session API cost ~$0.43 (5 Sonnet calls in sign-off testing).

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

- **Config-synthesizer's Gate 2 interprets the spec inverse of literal reading.** Build spec said "Primary keyword appears in secondary_keywords array (must be present)"; all 10 sampled real configs have the primary keyword NOT in `secondary` (would be tautological with audit_guide.py's SEO1-5d counting). Chose the empirical interpretation (primary must NOT also be a secondary key) and documented the deviation in the schema provenance, failure_modes, and build report. One-line flip in `lib/validate.py` if operator preferred the literal reading.
- **Sonnet tier for config-synthesizer**, not Haiku. The recommendations doc proposed Haiku (cheap, structured data assembly) but the build spec specified Sonnet "because Haiku is too generous per editor test results." Followed the build spec; cost differential at this volume is negligible (~$0.07/synth with warm cache).
- **`SYNTH_SYNTHESIZE_OVERRIDE` per-binary env hook in synth.sh** for failure-injection testing, NOT `SYNTH_LIB_OVERRIDE` whole-lib swap. Initial attempt at whole-lib override broke validate.sh's sibling-file lookup (it computes SYNTH_ROOT as parent dir). Per-binary override is minimal, production callers never set it, and gives clean black-box failure injection.
- **page.tsx template ships with a 22-line provenance comment block at the top.** Decision: strip it before write via `tail -n +25` in `ship-to-site/lib/stage.sh`. Generated wrappers stay byte-identical to the 31 existing shipped pages. (My initial count was off by 2 — actual block is 24 lines; corrected during sign-off.)
- **Cross-agent audit's Top-5 #4 → "F4" in operator's working terminology = my F23 (log_to_audit SQL).** The audit numbers failure modes F1-F23 in order found; the Top-5 priority list reorders them. When the operator says "F4 fix prompt" they mean the 4th priority fix (= F23 in the audit doc). Captured the alias in DEFERRED.md priority view + commit messages.
- **Variable-holding-apostrophe escape pattern for D044/F23 fix.** Build spec used `${VAR//\'/\'\'}` which does NOT work in bash 3.2 — `\'` inside double quotes is literal `\'`, not escaped `'`. First test attempt produced `'actor\'\'s_id'` in the generated SQL. Fix: `APOS="'"` then `${VAR//$APOS/$APOS$APOS}`. Verified on 3.2.57(1)-release. Documented in commit message as a deviation from spec.
- **F2 fix is exit 2, not exit 1.** Drafter's `maybe_fire_pipeline` treats any non-zero exit as a hard fire-failure. Using exit 2 lets a future drafter.sh tweak (tracked as D052) route soft-skips into a separate bucket without breaking the failure path for real errors.
- **assignment-drafter committed as one big first commit** (`0df8cf6`), not as a stage.sh-style modification, because the entire agent directory had never been in git despite running in production via cron. Same recovery pattern used for ship-to-site (`3b43c11`) earlier in the session. Both commit messages flag the project-rule issue: production agents landing without end-of-build git commits.
- **Ship-to-site cron PATH fix is the minimum scope.** Operator rejected baking `npm ci` into `build_verify.sh` (Q4 of the option set). Stayed focused on `/opt/homebrew/bin` PATH prepend; `node_modules` restoration was a one-shot operator action this session, with D055 tracking it as a separate hygiene concern.
- **Brain HANDOFF + DEFERRED updates committed separately from agents commits.** Each ~/brain/ commit is one logical concern (HANDOFF+DEFERRED refresh / D052 / D053 / D054+D055), never bundled with the underlying code commits. Pattern: the brain repo describes state; the code repos change it; they get separate commits so future archeology is clean.

## Lessons learned

- **Bash 3.2 doesn't expand `\'` inside double-quoted strings.** Build specs that show `${VAR//\'/\'\'}` for single-quote escaping silently produce literal `\'\'`. The fix is to hold the quote in a variable: `APOS="'"; "${VAR//$APOS/$APOS$APOS}"`. Always test the specific bash version (3.2.57 on Apple Silicon Mac) before trusting a spec's literal escape syntax.
- **Cron's PATH excludes `/opt/homebrew/bin`** on Apple Silicon Macs. Any cron-invoked script calling Homebrew binaries (npm, node, python3, jq, sqlite3, gh, etc.) needs a PATH prelude or breaks silently. Idempotent prelude: `case ":${PATH}:" in *":/opt/homebrew/bin:"*) ;; *) export PATH="/opt/homebrew/bin:/opt/homebrew/sbin:${PATH}" ;; esac`. Tracked as D054 for a systemic audit.
- **`npm run build` runs `next build` via PATH augmentation with `node_modules/.bin`** — fine when node_modules exists, fails with `sh: next: command not found` when missing. The PATH prepend doesn't help because `next` isn't a Homebrew binary; it lives in the project's local node_modules. If a Next.js site repo's node_modules disappears (fresh clone, git clean, manual rm), every ship breaks until `npm install` runs. Tracked as D055.
- **First-commit-of-a-living-agent is a recurring pattern.** Both `ship-to-site/` and `assignment-drafter/` were running in production via cron without ever being in git. Easy to miss because the working tree is "complete" and the agent works — but a hard drive failure or `git clean -fdx` would erase the agent. Spec rule "agents land with their own commit at end of build" is the right invariant; defer-rule "commit when ready" leads here.
- **The recursive-proof commit pattern works for self-healing infrastructure fixes.** D044/F23 (log_to_audit SQL escape) was verified by logging the fix itself with an apostrophe-containing payload via the fixed script. Pre-fix, the row could not have been written. Audit row id 733 stands as the un-fakeable proof. Use this pattern when fixing observability infrastructure.
- **`git add <path>/` stages everything under that path** — when the path is an untracked directory, that's the entire directory. For the F2 commit this swept in `state/daily_pipeline_spend.txt` (a runtime tally), which got tracked. Tracked as D053. Always check what's actually in a directory before `git add`ing it wholesale.
- **operator-named D-numbers can collide with prior-session D-numbers.** D045 collides between operator's "ai-do.sh rewire" (this session) and an older archived "Opus prompt-template" item from Cowen build. DEFERRED.md authoring rule should require `grep -nE "^### D[0-9]+" DEFERRED.md | tail` BEFORE adding any new entry. Flag any collision in the build report.
- **page.tsx template extraction was a 0-variance discovery, not a careful diff.** All 31 per-slug wrappers are exactly 21 lines; byte differences (555-645B) are pure slug-length artifact. 3-way diff of diverse slug shapes (chrysotile/asbestos-shingles-guide/how-to-test-popcorn-ceiling-for-asbestos) reduces to 3 lines, all the slug substring. Confirms the existing manual copy-paste workflow has been disciplined and the wrapper genuinely has zero per-page variance beyond the slug.
- **The autonomous chain (drafter → run-batch.sh → ship-to-site) wasn't actually working until this session.** Cron-driven shipping had been failing silently with `npm: command not found` since the agent landed; only the 31 manually-shipped guides ever made it live. The 10-tick skip-loop on white-asbestos-vs-blue-asbestos was the smoking gun. Lesson: when a cron-driven pipeline claims "fully autonomous", verify by counting successful runs in audit_log, not by reading the build report.
- **Spec's `tail -n +23` was off by 2** because my own extraction-report count was off by 2. The user's prompt repeated my miscount. Lesson: when a number from a spec is questionable, count it again before relying on it. Especially line-count and length-count specs that depend on a separate authored doc.
- **Verifying with byte-diff against a real artifact** beats unit tests for substitution scripts. The provenance-strip verification (`diff actual_substituted real_shipped_wrapper`) caught the 2-line off-by error AND confirmed byte-identical output in one step. Always have a real-world artifact to diff against when validating codemods.
- **`log_to_audit.sh` callers wrapping with `|| true` makes failures invisible.** Every agent that wrote audit rows containing apostrophes was silently dropping them. The defensive scrub in config-synth's `handle_failure` worked around it; the lib-layer fix (D044/F23) is the correct fix because it benefits every caller. Pattern: when callers reflexively swallow exit codes, the lib is shipping latent bugs.

## Operator corrections

- **"Make sure the expanded message explicitly calls out both fixes — something like a second paragraph 'Also bundles: prior-session trailing-} fix in PAYLOAD default (was sitting uncommitted in working tree)'."** When the D044 commit's diff included an unrelated prior fix (the `${5:-{}}` → `${5-}` payload-default tweak), the operator wanted the commit message to be honest about the bundled scope. Captured as a discipline rule: commit messages must mention any bundled changes, even if the spec's literal message text doesn't.
- **"Commit the entire ship-to-site/ dir"** (in response to "ship-to-site has never been committed, how do I handle?"). Same answer applied later to assignment-drafter. Pattern preference: when an agent has never been in git, prefer one big first-commit over many small surgical ones, even if the spec asks for the surgical version.
- **"Just drop all 6 in downloads then"** (when offered multiple-choice drop sets). Pattern: when offered a curated set, operator prefers the inclusive answer over the minimal one. Save the multi-select UI; just drop everything reasonable.
- **"What else should we drop?"** (asked twice with no specific guidance) signals "use your judgment, include what I'd want." Pattern: operator wants comparison/grading artifacts in `~/Downloads/` for phone review (existing memory). Be inclusive on docs, conservative on raw dumps.

## What's next

**Immediate:**
1. **Editor production runner + threshold + wiring decision (D056, item 0 in HANDOFF).** Verdict persistence is half-live; full coverage needs the editor decision. Operator's Q7 in cross-agent audit §7 is the relevant policy question. Until then, Escalation Triage can be partially built against `audit_guide` verdicts only.
2. **Cost cap reconciliation (F17/F18).** Three caps disagree; needs system-wide hard stop at `log_token_usage.sh`. Operator must answer cross-agent audit §7.2 first (what's the right number?).
3. **F14 pre-ship approval gate.** No human review between approved guide and live site. Editor agent (idle) could slot in cheaply once production runner is decided. Pairs with #1.

**Closed this session:**
- F2 — drafter pre-flight ✅ (commit `0df8cf6`)
- F4/D044 — log_to_audit SQL escape ✅ (commit `939361d`)
- Ship-to-site cron PATH ✅ (commit `0be9d1f`)
- ship-to-site, assignment-drafter — first commits ✅ (3b43c11, 0df8cf6)

**New deferred items (D052-D055), full details in DEFERRED.md:**
- D052: drafter.sh exit-2 vs exit-1 routing (UX cleanup for F2-induced double-row audit logging)
- D053: assignment-drafter state files not gitignored
- D054: audit all cron-invoked scripts for PATH dependencies (systemic version of build_verify fix)
- D055: site repo node_modules disappeared (root cause unknown)

**Blocked:**
- D033 cron activation for Market Scribe still gated on 1-3 days clean manual validation. Tomorrow is day 2 of that window.
- Phase 3 still gated on first AITEAM site live + GA4 data + `mission_tasks` rows.
- 7 operator-policy questions in cross-agent audit §7 — all require operator answers, not code.

**Trigger for SSG Agent 1 build (D028 + others):** the SSG content-spec re-skin work the operator did in parallel (5 new build reports landed in `~/brain/projects/aiteam/docs/`) is the unblock. Once SSG has a populated keyword-configs/ dir, config-synthesizer's `ssg.yaml` can flip to `enabled: true`.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-16_2307.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-16 19:20 **Last session summary:** Config Synthesizer agent built and committed (`69dd30f`) with `--validate-config-only` flag added to `audit_guide.py` (`e504793` in asbestos-contractors). 4 test configs synthesized + triaged + deleted. page.tsx template extracted from 3 real shipped wrappers; ship-to-site committed for the first time (`3b43c11`) including the provenance-strip tweak to `lib/stage.sh`. Cross-agent failure-modes audit landed at `~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md` documenting 23 failure modes across the now-autonomous asbestos chain.  --- 
