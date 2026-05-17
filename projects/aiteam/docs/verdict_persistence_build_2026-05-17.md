# Verdict persistence layer — build report (2026-05-17)

## Scope shipped

Foundation work for the future Escalation Triage agent. **Half-coverage by
design** — audit_guide.py persistence ships; editor persistence is deferred
pending operator policy decisions (see §"Editor persistence — deferred").

| Artifact | Path | Commit |
|---|---|---|
| Schema migration | `~/agents/lib/migrations/2026-05-17_auditor_verdicts.sql` | `4e2fe0a` (`~/agents/`) |
| audit_guide.py persistence | `~/projects/asbestos-contractors/scripts/audit_guide.py` | `0e8089b` (`~/projects/asbestos-contractors/`) |

## What persists now

Every invocation of `audit_guide.py` writes one row to `auditor_verdicts`
and one matching row to `audit_log` (action `verdict_recorded`), tied
together by `correlation_id = audit_guide-<ts>-<pid>`.

`auditor_verdicts` columns populated by audit_guide:

| Column | Source |
|---|---|
| `ts` | `int(time.time())` at persist call |
| `slug` | `result['slug']` (added to `audit()` return dict; falls back to `--slug` arg) |
| `source_path` | `os.path.abspath(args.guide)` |
| `scorer` | literal `'audit_guide'` |
| `scorer_version` | `git rev-parse --short HEAD` from `~/projects/asbestos-contractors/`; `'unknown'` on failure |
| `passed` | `1` if `result['fail_count'] == 0` else `0` (matches existing `sys.exit` semantic) |
| `composite_score` | `NULL` (mechanical checks don't produce a composite) |
| `rubric_json` | `{<code>: <pass-bool>}` for all 39 mechanical checks (S1, S2, …, G1, …, L5) |
| `reasoning` | `"; ".join("<code>: <msg>")` over failing checks; empty string when all pass |
| `tier` | `NULL` (reserved for future tier-rubric work — see §"Open spec drift" below) |
| `triaged_at` | `NULL` (escalation-triage agent will populate) |
| `correlation_id` | `audit_guide-<ts>-<pid>` |

Stdout output of `audit_guide.py` is **unchanged**. Persistence is purely
additive — verified by running against a real guide and observing the
identical report.

Persistence is **best-effort**: SQLite or `log_to_audit.sh` failures log to
stderr but never break the audit run itself.

## Sign-off — what was tested

| Test | Bar | Result |
|---|---|---|
| **Test 2** (audit_guide row persists) | Run audit_guide.py against a real guide; row lands in `auditor_verdicts` and matching `audit_log` row emitted with same correlation_id | ✅ Ran against `_asbestos-reference/approved-guides/asbestos-shingles-guide.json` with its keyword-config; row id=1, correlation_id `audit_guide-1778994650-14958` matches audit_log row id=867 |
| **Test 3** (apostrophe / F4 regression) | Force apostrophes through slug, source_path, rubric_json, reasoning; row inserts cleanly, no SQL syntax errors | ✅ Direct `persist_verdict()` call with `slug="asbestos-don't-demolish-test"`, source_path with apostrophe, rubric containing `"can't pass"`, reasoning with multiple apostrophes + embedded `'quotes'`. Row id=2, all apostrophes preserved verbatim. Validates both the Python `sqlite3` parameterized path AND the F4-fixed `log_to_audit.sh` escape path (bash `${VAR//$APOS/$APOS$APOS}`). |
| **Test 4** (idempotent migration) | Migration script runs twice without error | ✅ Re-applied twice after the initial apply; both subsequent runs returned cleanly, row count unchanged (`CREATE TABLE IF NOT EXISTS` + `CREATE INDEX IF NOT EXISTS`). |

Test 1 (editor manual test) was **not run** — editor persistence is deferred.

## Editor persistence — deferred

The build spec's editor half (modify `~/agents/editor/<runner>` to persist
verdicts) was **not shipped**. Two explicit HALT conditions surfaced:

### HALT-1 — no production editor runner exists

`~/agents/editor/` contains:
- `agent.yaml`, `CLAUDE.md`, `rubric.md` — prompt / config
- `failure_modes.md` — docs
- `run_tier_test.sh` — **calibration** script (its line-3 comment: "score every article in `editor_test_corpus/` … emit `tier_results.csv` + `tier_report.md` with Pearson correlations")

The editor's CLAUDE.md explicitly states "Your only job (for the tier test)".
The agent is currently a model invocation pattern (system prompt + rubric +
`ai-do.sh` wrapper) without any production caller that scores **one real
draft** at a time and persists the verdict.

The closest emit point is `score_one()` at line 90 of `run_tier_test.sh`,
but that loops over a fixed test corpus and runs each article **twice**
(Haiku + Sonnet). Wiring persistence there would dump 30+ rows per
calibration run into `auditor_verdicts`, mixing test-corpus calibration
scores with real-draft verdicts. Wrong table semantics.

### HALT-2 — no pass threshold defined

`composite_score` is defined (arithmetic mean of 5 axes), but no
`passed = 1 vs 0` threshold is documented in `CLAUDE.md`, `rubric.md`,
`agent.yaml`, `run_tier_test.sh`, or `failure_modes.md`. The build spec
suggested `composite >= 3.5` as a default but explicitly forbade guessing.

### Gating question

Editor's production role is gated on **Q7 from
`~/brain/projects/aiteam/docs/cross_agent_failure_modes_2026-05-16.md` §7**:

> 7. **Editor agent's role in the production pipeline.** `editor/` exists
>    and is tuned (rubric, tier-test) but currently isn't called by drafter,
>    ship-to-site, or run-batch.sh. Was the intent for editor to be a
>    pre-ship gate (slotting in similar to Top-5 #3)? If so, that's a much
>    smaller build than building approval-gate from scratch — editor's `do`
>    tier already produces a 5-axis scorecard. Operator could set a score
>    threshold for auto-ship vs require-review.

### Files that need to exist before editor persistence can ship

1. **A production editor runner** — a script (e.g. `~/agents/editor/score_one.sh`)
   that takes one draft path + slug, calls `~/agents/lib/ai-do.sh` once,
   parses the TSV output, and is the place persistence can hook into. Shape
   should mirror `audit_guide.py`'s pattern: one invocation = one row.

2. **A documented pass threshold** — either as a constant in the new
   runner, or as a config field somewhere (e.g. `agent.yaml`). The
   threshold should be operator-stated, not invented, since it directly
   determines the publish gate.

3. **A decision about where editor is called from** — the answer to Q7.
   Is editor a pre-ship gate inside `ship-to-site/run.sh`? Inside
   `assignment-drafter/lib/fire_pipeline.sh`? A standalone cron? The
   wiring location dictates what `slug` and `source_path` the persist
   call has access to.

Once those three exist, editor persistence is a small additive change
following the same shape as the audit_guide work shipped today.

## Schema reference

```sql
CREATE TABLE auditor_verdicts (
  id              INTEGER PRIMARY KEY AUTOINCREMENT,
  ts              INTEGER NOT NULL,
  slug            TEXT    NOT NULL,
  source_path     TEXT,
  scorer          TEXT    NOT NULL,    -- 'editor' | 'audit_guide'
  scorer_version  TEXT,                -- git short SHA at run time
  passed          INTEGER NOT NULL,    -- 0 or 1
  composite_score REAL,                -- editor only; NULL for audit_guide
  rubric_json     TEXT,                -- structured per-check / per-axis detail
  reasoning       TEXT,
  tier            TEXT,                -- reserved for future tier-rubric work
  triaged_at      INTEGER,             -- reserved for escalation-triage agent
  correlation_id  TEXT
);

CREATE INDEX idx_verdicts_slug      ON auditor_verdicts(slug);
CREATE INDEX idx_verdicts_untriaged ON auditor_verdicts(triaged_at) WHERE triaged_at IS NULL;
CREATE INDEX idx_verdicts_failed    ON auditor_verdicts(passed, ts) WHERE passed = 0;
```

## Open spec drift (operator-facing)

These items in the build prompt didn't match repo reality. Building straight
through would have produced incorrect work or collided with existing state.
Flagging here for next-session resolution:

1. **`log_to_audit.sh` signature.** Build prompt showed `--actor-type`,
   `--actor-id`, etc. (flag-style). The actual helper at
   `~/agents/lib/log_to_audit.sh` takes positional args (`<actor_type>
   <actor_id> <action> <target> <payload_json> [correlation_id]`),
   matching the convention used by `run_tier_test.sh` and elsewhere. The
   shipped `persist_verdict()` uses the positional form.

2. **"D052 (tier rubric)" reference is wrong.** The spec referred to a
   D052 tracking tier rubric work, used to justify leaving `tier` NULL
   for now. Actual `DEFERRED.md` D052 is "drafter.sh exit-2 vs exit-1
   routing" — unrelated. No DEFERRED item currently exists for a tier
   rubric specifically. `tier` is still left NULL per the build prompt's
   instruction, but the cross-reference needs a new D-item if the
   operator wants it tracked formally.

3. **"D053" requested for the new deferred item is taken.** The build
   prompt said "add D053 'Editor verdict persistence'". DEFERRED.md
   already uses D053 (assignment-drafter state files not gitignored),
   plus D054 and D055. **Used D056 instead** for the new editor-
   persistence deferred item, following the file's own authoring rule
   ("New D-items should pick the next free D-number"). Operator can
   rename if a different number is preferred.

## Next-session pointers

- D056 (new this session) tracks editor verdict persistence — see
  `DEFERRED.md`.
- Triage-agent build can now partially proceed: any agent reading
  `auditor_verdicts WHERE scorer='audit_guide' AND triaged_at IS NULL`
  has real data. Full coverage waits on editor.
- Q7 is the policy blocker for both editor persistence AND wider
  pre-ship gating; resolving it unlocks ≥2 dependent builds.
