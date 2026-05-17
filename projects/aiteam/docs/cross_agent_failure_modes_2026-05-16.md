# Cross-Agent Failure Modes — 2026-05-16

System-level audit covering what happens when AITEAM agents interact, fail in
combination, or pass bad outputs to each other. Companion to the per-agent
`failure_modes.md` files (which document each agent in isolation). Read-only
investigation — no code changes here.

Sources consulted: every agent's `agent.yaml`, `CLAUDE.md`, `failure_modes.md`;
shared lib (`ai-cheap`, `ai-do`, `ai-think`, `run_agent`, `check_kill_switches`,
`log_to_audit`, `notify`, `retrieve_context`); `~/store/aiteam.db` schema;
`crontab -l`; pipeline state under `~/projects/asbestos-contractors/`; brain
docs (`ssg_pipeline_recon`, `ssg_pipeline_automation_recommendations`,
`assignment_drafter_build`, `assignment_drafter_patch`, `ship_to_site_build`,
`config_synthesizer_build`).

---

## 1. Agent fleet map

### Production pipeline agents (asbestos / SSG content fork)

```
                    ┌────────────────────────────────────────────┐
                    │ OPERATOR                                   │
                    │  - edits drafter_queue.txt                 │
                    │  - runs config-synthesizer/synth.sh        │
                    │  - flips kill switches in config/.env      │
                    │  - reviews ship-to-site/lib/digest.sh out  │
                    └─────┬────────────────────────────┬─────────┘
                          │                            │
                          ▼                            ▼
  ┌──────────────────────────────────────┐   ┌──────────────────────────────┐
  │ config-synthesizer                   │   │ assignment-drafter           │
  │   operator-invoked, no cron          │   │   cron */30                  │
  │   tier: sonnet                       │   │   tier: sonnet               │
  │   reads: 3 example configs           │   │   reads: queue + 4 context   │
  │   writes: keyword-configs/<slug>.json│   │           files in repo      │
  │   gate: audit_guide.py               │   │   writes: assignment-batch-  │
  │         --validate-config-only       │   │           <slug>.md          │
  └─────────────────────┬────────────────┘   │   then: nohup run-batch.sh   │
                        │                    │         <slug> --guide &     │
                        │                    │   budget: $15/day (estimate) │
                        │                    └──────────────┬───────────────┘
                        │                                   │
                        │                                   ▼
                        │   ┌─────────────────────────────────────────────────┐
                        │   │ run-batch.sh (NOT an AITEAM agent — pipeline)   │
                        │   │   reads: BOTH assignment-batch AND keyword-     │
                        │   │          config for the slug                    │
                        │   │   3-round writer ↔ auditor ↔ AuditKit Pass 1+2  │
                        └──>│   writes: approved/guides/<slug>.json on PASS   │
                            │           needs-review/<slug>.json on 3R fail   │
                            │   logs: content/asbestos/logs/batch-*.log       │
                            └────────────────────────┬────────────────────────┘
                                                     │
                                                     ▼
                                ┌────────────────────────────────────────────┐
                                │ ship-to-site                               │
                                │   cron */15                                │
                                │   tier: script (no LLM)                    │
                                │   reads: approved/guides/*.json            │
                                │   writes: SITE_REPO/src/data/guides/...    │
                                │           SITE_REPO/src/app/guides/.../    │
                                │           SITE_REPO/src/components/        │
                                │             GuideArticle.tsx (whitelist)   │
                                │   then: npm build, git commit + push       │
                                │   gate: GitHub schema-check.yml polls      │
                                └────────────────────────┬───────────────────┘
                                                         │
                                                         ▼
                                          Vercel auto-deploy → live site
```

### Shared infrastructure

- **`~/store/aiteam.db`** (SQLite, WAL). Tables touched cross-agent:
  - `audit_log` — every agent writes via `~/agents/lib/log_to_audit.sh`
  - `token_usage` — every LLM call (Haiku/Sonnet/Opus) via `log_token_usage.sh`
  - `conversation_log`, `warroom_transcript`, `hive_mind`, `memories` —
    orchestrator + warroom only
  - `scheduled_tasks`, `mission_tasks` — schema present, **empty** as of audit
- **`~/agents/config/.env`** — global env. `SYSTEM_PAUSED`,
  `LLM_SPAWN_ENABLED`, `<AGENT>_ENABLED`, `DAILY_API_BUDGET_USD=10`,
  `AGENT_MAX_TURNS=30`. Mtime-refreshed (~2s liveness) via
  `check_kill_switches.sh`.
- **`~/agents/lib/notify.sh`** — Telegram outbound. Stubbed when
  `TELEGRAM_OUTBOUND_ENABLED=false`. Every agent uses it for operator-facing
  notifications.
- **`~/projects/asbestos-contractors/`** — pipeline repo. Read by config-synth,
  read+written by drafter, read by ship-to-site, written by pipeline.
- **`~/projects/asbestoshq-site/`** — site repo. Read+written by ship-to-site
  ONLY. Operator may also pull/edit (race surface).

### Non-pipeline agents (out of scope for cross-agent analysis but listed for completeness)

- `orchestrator/` — operator-facing war room + nightly diary cron (23:45)
- `librarian/` — nightly brain inbox sort (Haiku); not currently in cron
- `editor/` — content quality scorer (Sonnet); not currently in production
  pipeline; called by tier-test only
- `market/{analyst,briefer,curator,scribe}` — equity-research stack, distinct
  domain
- `telegram/bot.js` — slash-command dispatcher (`/halt_shipping`, etc.)
- `tg-monitor/` — analyzes Telegram archives (own SQLite at `tg-monitor/archive.db`)
- `dashboard/server.js` — observability + war-room API
- `_template/` — scaffolding only, not an active agent

### Cron timing (active schedules)

| When | What | Notes |
|---|---|---|
| `*/5 * * * *` | tg-monitor `reader.py` | local SQLite write |
| `*/15 * * * *` | `ship-to-site/ship.sh` | scans approved/, may push |
| `*/30 * * * *` | `assignment-drafter/drafter.sh` | Sonnet + bg run-batch.sh |
| `0 4 * * *` | `backup_daily.sh` | rsync to external SSD |
| `0 5 * * 0` | `backup_weekly.sh` | rsync to WD HDD |
| `0 */6 * * *` | `backup_db.sh` | SQLite snapshot |
| `0 7 * * *` | `tg-monitor/analyzer.py` | daily Telegram analysis |
| `0 7 * * *` | `ship-to-site/lib/digest.sh` | daily ship rollup |
| `23:45 * * *` | `orchestrator/write_diary.sh` | nightly diary |
| `23:55 * * *` | `scripts/brain-autocommit.sh` | brain git push |
| `*/15` + `*/5` | `convert_dropzone.sh`, `rebuild_command_center.sh` | local only |

---

## 2. Cross-agent failure modes

### Data-handoff failures

#### F1. Config-synth writes a config for a slug with no assignment-batch

- **Trigger.** Operator runs `synth.sh --slug X --primary-keyword "..."` for a
  slug that hasn't been enqueued in `drafter_queue.txt` and has no
  hand-authored `assignment-batch-X.md`. Config lands at
  `<pipeline_repo>/content/asbestos/keyword-configs/X.json`.
- **Detection.** None automatic. The config sits idle. There's no agent that
  scans for "orphan configs without companions." Operator notices only via
  `git status` or by `ls keyword-configs/` and remembering.
- **Damage radius.** ~$0.07 Sonnet spend on the synth itself (cache-warm).
  Configs file is git-untracked; doesn't pollute production until committed.
  If never paired with an assignment-batch, no downstream effect.
- **Recovery.** Delete the orphan file (`rm` + verified by `git status`), or
  pair it with an assignment-batch and let the autonomous chain pick up.
  Triage workflow used in `synth_configs_triage_2026-05-16.md` is the
  template.
- **Prevention.** A pre-flight check in `synth.sh` that warns
  ("no assignment-batch found for this slug — config will sit idle unless you
  enqueue") would surface the orphan at synth time. Small change.

#### F2. Drafter writes assignment-batch for a slug with no keyword-config (CRITICAL)

- **Trigger.** Operator enqueues `slug-X|primary kw` in `drafter_queue.txt`
  without first running config-synthesizer for `slug-X`. Drafter (cron */30)
  generates `assignment-batch-X.md`, fires `run-batch.sh X --guide` in
  background. Pipeline's pre-flight (`run-batch.sh:1119-1126`) hard-fails:
  `MISSING_CONFIG` status, no writer call. **OR** worse: if a stale or
  hand-rolled keyword-config exists with wrong fields, pipeline runs the
  writer (Opus), then AuditKit Pass 2 fails, drafts get fed to round 2 and 3,
  burning more Opus tokens, ending in `needs-review/`.
- **Detection.** Audit log row `pipeline_fire_failed` IF run-batch.sh's
  startup error is caught by `fire_pipeline.sh`'s rc check. If pipeline gets
  past pre-flight and fails later, the drafter has already exited — failure
  surfaces only in `content/asbestos/logs/batch-*.log` or the absence of an
  approved guide hours later. Operator sees no shipped guides in the next
  ship-to-site digest.
- **Damage radius.** **Up to ~$5-10 in wasted writer/auditor calls per slug**
  (3 rounds × writer Opus + auditor Opus). Drafter's $15/day fixed-estimate
  cap still ticks ($2.50 per fire), so 6 missing-config fires = budget
  exhausted with **zero approved guides**. Trust erosion: operator sees
  pipeline activity but no output.
- **Recovery.** Run config-synthesizer for the slug, then either re-run
  `bash content/run-batch.sh <slug> --guide --force` manually, or wait for
  next drafter tick to re-fire (drafter won't re-fire automatically because
  the assignment-batch already exists).
- **Prevention.** Drafter's `maybe_fire_pipeline` should refuse to fire if
  `<pipeline_repo>/content/asbestos/keyword-configs/<slug>.json` doesn't
  exist. One-line shell test before launching `nohup run-batch.sh`. Costs
  nothing, catches the most expensive cross-agent failure.

#### F3. Ship-to-site ships a guide whose keyword-config was deleted/edited post-approval

- **Trigger.** Approved guide lands in `approved/guides/<slug>.json`. Operator
  (or a future config-cleanup) deletes or rewrites `keyword-configs/<slug>.json`.
  Next ship-to-site tick copies the approved JSON to the site — but the audit
  that approved it is no longer reproducible.
- **Detection.** None. ship-to-site doesn't read keyword-configs at all.
  Forensic: `git log content/asbestos/keyword-configs/<slug>.json` shows
  when/why the config changed.
- **Damage radius.** Low in immediate terms (the audit already passed, the
  content is what it is). Real damage is the *next* re-audit of the same slug
  fails or produces a different verdict — operator can't reproduce the
  history. Audit reproducibility for SEO calibration / future tuning
  decisions weakens.
- **Recovery.** `git checkout` the prior config from git history. Pipeline
  repo is tracked.
- **Prevention.** Don't allow keyword-config deletion for any slug present
  in `approved/guides/`. A pre-commit hook in the pipeline repo, or a
  drafter-fired sanity check before fire. Medium effort.

#### F4. Two agents write to the same file at the same time

- **Trigger.** Three realistic variants:
  - (a) Two CC sessions both invoke `synth.sh --overwrite` for the same slug.
  - (b) `drafter.sh` (cron */30) launches `run-batch.sh` for slug X; operator
    simultaneously runs `bash content/run-batch.sh X --guide` manually.
  - (c) `ship-to-site` (cron */15) stages slug X while operator manually edits
    `src/components/GuideArticle.tsx` to add slug Y.
- **Detection.** None automatic. Symptoms: garbled JSON (last-writer wins),
  truncated files, git merge conflicts on next operation, build errors on
  next `npm run build`.
- **Damage radius.** Case (a) — last writer wins, no real harm (operator
  notices via `git diff`). Case (b) — two parallel run-batch.sh processes
  for same slug compete over `drafts/<slug>.json` and `.cache/`; either
  succeed and the second overwrites the first, or one crashes mid-write.
  Could leave drafts/ in a corrupt state. Case (c) — git index may stage a
  hybrid; ship-to-site's `git_ops.sh` may commit operator's WIP unintentionally.
- **Recovery.** Variant-specific. (a) re-run synth, accept new output.
  (b) `rm` drafts/<slug>.json + drafts/.cache/* for the slug; re-run
  pipeline. (c) `git restore --staged .` + redo carefully.
- **Prevention.** File-system advisory locks (`flock`) around the
  long-running operations: `run-batch.sh` invocations per slug, ship-to-site
  per slug stage. Small additions; flock is in macOS via `/usr/bin/flock`
  through homebrew or coreutils. Medium effort.

### Pipeline / orchestrator failures

#### F5. Pipeline approves a guide with broken internal links → ships → 404 on live

- **Trigger.** Writer generates a body paragraph with an inline link like
  `[asbestos abatement](/asbestos-abatement-cost/)`. The auditor checks
  `L1-L7` (internal-link counts + .gov authority + duplicates) but does NOT
  verify the link target exists. Approved JSON lands. ship-to-site's
  `stage.sh` filters `relatedLinks` array against `APPROVED_GUIDE_SLUGS` —
  but does NOT touch inline body links in `paragraphs[]`. Site builds (TS
  strings, no validation), Vercel deploys, page is live with a dead link
  to `/asbestos-abatement-cost/` (a slug that doesn't exist).
- **Detection.** Site renderer (`GuideArticle.tsx`) detects when parsing
  inline markdown: if the slug is not in `APPROVED_GUIDE_SLUGS`, the link
  is silently rewritten to plain text (no `<a>`). So most "stale" links
  degrade gracefully to text — no 404. But for slugs that ARE published
  yet linked with the wrong path (typo in writer's output), the rendered
  link will be clickable but 404. Operator notices via manual click-through,
  GSC crawl errors, or weekly `weekly-link-audit.yml` GitHub Action (runs
  Mondays 9am PT — opens PR with fixes).
- **Damage radius.** SEO penalty (404s on internal links hurt crawl
  budget), reader frustration. Single page, fixable. Auto-detected within
  one week by `weekly-link-audit.yml`.
- **Recovery.** Edit the JSON manually (in site repo or pipeline repo +
  re-ship), commit, push.
- **Prevention.** Add a link-target check to `audit_guide.py` L-series:
  for every inline `]/(/...)` and every `relatedLinks.slug`, verify the
  target exists in either `APPROVED_GUIDE_SLUGS` (for guides) or the site
  static-pages set. Medium effort. Alternative: add to ship-to-site's
  `validate.sh` so the check runs at ship time on the JSON-to-be-shipped.

#### F6. ai-cheap/do/think returns JSON-valid but semantically wrong content

- **Trigger.** Sonnet/Opus/Haiku produces output that parses (valid JSON,
  valid TSV, valid markdown) but is content gibberish: e.g., config-synth
  returns `intent_concepts: ["foo", "bar"]`; editor returns
  `5\t5\t5\t5\t5` for a junk article; drafter writes a brief with
  hallucinated regulatory citations.
- **Detection.** None at the LLM-wrapper level. Per-agent gates catch
  *some* — config-synth's `validate.py` Gate 3 catches "primary not in
  body" via the auditor's own checks, but only because the *real* audit
  later catches gibberish. drafter's `validate.sh` catches placeholder
  strings (TODO, OPERATOR EDIT, TBD) but not "hallucinated company
  name." Editor has no semantic gate. Operator spot-check is the only
  safety net.
- **Damage radius.** Wide. A hallucinated regulatory cite (e.g.
  "40 CFR 999.999") that survives the pipeline → live page making a
  false legal claim. Reader trust + potential liability exposure.
- **Recovery.** Manual edit + re-ship. If detected after publication,
  same as F15 (live-content rollback path).
- **Prevention.** Hard problem. Layered defenses:
  (a) tighten prompts with explicit "do not invent" instructions;
  (b) add entity-validity checks against an authoritative source list
  (e.g. `asbestos-authority-links.md` for citations);
  (c) sample a fraction of LLM outputs through a second LLM as a
  cross-check (expensive — would need cost analysis first).
  Large effort, high value. Not in v1.

#### F7. Orchestrator delegates to non-existent agent

- **Trigger.** Operator's `@agentname` typo or operator references an
  agent that was renamed/deleted.
- **Detection.** **LOUD.** `run_agent.sh:16` does `[ -d "$AGENT_DIR" ] ||
  { echo "ERROR: agent dir not found: $AGENT_DIR" >&2; exit 1; }`.
  Warroom dispatcher returns 422 with `error: 'no matching registered
  agents'` (per orchestrator/failure_modes.md). No silent path —
  dispatcher rejects pre-fan-out, no LLM spend.
- **Damage radius.** None. Loud rejection, $0 spend.
- **Recovery.** Operator re-types the agent name.
- **Prevention.** Not needed. Already handled.

### Shared state failures

#### F8. audit_log writes from two agents share a correlation_id

- **Trigger.** correlation_id is NOT a unique key. Each agent
  generates one with `uuidgen` per invocation; collisions are
  astronomically unlikely (~10^-37). But agents intentionally pass a
  shared correlation_id when grouping related rows (e.g., ship-to-site's
  whole tick shares one UUID).
- **Detection.** N/A — this is by-design behavior, not a fault.
- **Damage radius.** None. correlation_id is an index for grouping
  related rows; the `id` PK is the unique key.
- **Recovery.** None needed.
- **Prevention.** None needed. Design is correct.

#### F9. token_usage rows reference orphan agent_id (deleted/renamed agent)

- **Trigger.** `token_usage.agent_id` is free-form text (no FK to a
  registry). If `config-synthesizer` is renamed to `cs` next month,
  historical rows still say `config-synthesizer`. If a typo agent_id is
  passed (e.g., `AGENT_ID_OVERRIDE=conf-syn`), that string lands in the
  table forever.
- **Detection.** `SELECT DISTINCT agent_id FROM token_usage` vs
  `ls ~/agents/`. Operator-driven, no automation.
- **Damage radius.** Data hygiene only. Cost-attribution queries by
  agent return slightly fragmented results.
- **Recovery.** None required. If material, an `UPDATE token_usage SET
  agent_id = 'X' WHERE agent_id = 'X-typo'` cleans up.
- **Prevention.** Validate `agent_id` against `ls ~/agents/` inside
  `log_token_usage.sh` and reject unknowns with a warning. Small change.
  Not high-priority.

#### F10. SQLite WAL checkpoint fails

- **Trigger.** Default auto-checkpoint at 1000 pages. If the
  `aiteam.db-wal` file can't be merged into the main db (disk full,
  permission issue, FS corruption), WAL grows indefinitely. Eventually
  reads slow + disk fills.
- **Detection.** None automatic. `ls -la ~/store/aiteam.db-wal` would
  show growth. `backup_db.sh` runs every 6 hours and snapshots — if the
  snapshot is large or fails, that's the signal. **No alarm wired today.**
- **Damage radius.** Disk fill, then total system stop. Recoverable but
  the failure happens silently until a hard symptom.
- **Recovery.** `sqlite3 ~/store/aiteam.db "PRAGMA wal_checkpoint(TRUNCATE);"`
  to force-merge. Or move/delete WAL file (data loss for unmerged
  transactions — rare with current write volume).
- **Prevention.** Add `PRAGMA wal_checkpoint;` to the end of
  `backup_db.sh` (every 6 hours). Or add a disk-space check to the
  dashboard. Small change, eliminates whole class of silent failure.

### Operator-in-the-loop failures

#### F11. Operator deletes a file the agent fleet expects

Per-agent behavior is already documented; aggregating cross-agent:

| File deleted | Affected agents | Failure mode |
|---|---|---|
| `~/projects/asbestos-contractors/` (the whole repo) | drafter, ship-to-site, config-synth | drafter exits 2 (FATAL load_context); ship-to-site logs `_site_scan` missing_dir; config-synth pre-flight fails. **All loud.** |
| `keyword-configs/<slug>.json` after approval | (none directly) | ship-to-site doesn't check. Audit reproducibility lost (see F3). |
| `assignment-batch-<slug>.md` after pipeline run | (none directly) | run-batch.sh already finished. Re-running pipeline for that slug becomes impossible until reauthored. |
| `approved/guides/<slug>.json` | ship-to-site | next tick: nothing to ship for that slug. No alarm — silent absence. If the slug was ALSO removed from `APPROVED_GUIDE_SLUGS`, eventually re-shippable via re-run of pipeline. |
| `drafter_queue.txt` | drafter | logs `drafter_queue_missing`, exits 0. Next tick same. Operator restores. |
| `~/agents/config/.env` | every agent that sources it | LOUD — `source` fails, scripts exit. |

- **Detection.** Most are loud (audit row + script exit). The silent ones
  are F3-class (audit reproducibility) and ship-to-site's "nothing to ship"
  (visible only via missing-row in `state/shipped.log`).
- **Damage radius.** Low if loud; medium if silent (operator may not
  notice missing output for a day).
- **Recovery.** `git checkout` whatever was tracked; reauthor whatever
  wasn't.
- **Prevention.** Move `state/` and per-agent ephemeral files into git
  (currently `.gitignore`d). Drift detection becomes git diff.

#### F12. Operator manually edits a file an agent is currently writing

- **Trigger.** Operator opens `keyword-configs/<slug>.json` in their
  editor right as config-synthesizer is writing the same file (or any
  similar race).
- **Detection.** None. POSIX `rename(2)` is atomic on macOS APFS for the
  file-replacement step (config-synth does `mv tmp dst`), so the file
  itself never appears half-written. But the operator's editor may still
  hold a stale buffer and re-save it later, overwriting the agent's write.
- **Damage radius.** Depends on what operator saves. Typically reversible
  via `git checkout`.
- **Recovery.** `git diff` + `git checkout` if tracked, else manual
  re-authoring.
- **Prevention.** Hard problem (file locks don't help against an editor
  holding a buffer). Mitigation: agents should `cp -p` to a `.bak` before
  overwriting, so manual recovery is fast. Small change.

#### F13. Operator `git pull` on site repo while ship-to-site is mid-stage

- **Trigger.** ship-to-site has run stage.sh (3 files written) and is
  mid-`npm run build`. Operator does `git pull --rebase origin main` in
  another terminal.
- **Detection.** **Possible silent failure.** Two outcomes:
  - (a) Pull succeeds with no conflicts: rebase rewrites local commits
    (ship-to-site hasn't committed yet — files are working-tree only).
    Operator's pull pulls in remote changes; ship-to-site's staged
    changes survive in working tree. Next git_ops.sh commits them. Fine.
  - (b) Pull has conflicts: working tree becomes hybrid; ship-to-site's
    `npm run build` may pass or fail depending on what conflicts. If
    build passes, git_ops.sh commits the hybrid (operator-merged +
    ship-to-site-staged). If build fails, rollback.sh runs and may
    `git restore .` operator's pending merge state (DATA LOSS RISK).
- **Damage radius.** (b) can lose operator's in-flight pull. Rare but
  possible.
- **Recovery.** Operator inspects `git reflog`, recovers.
- **Prevention.** Ship-to-site should take an exclusive lock on the site
  repo for the duration of stage+build+push. `flock` on a sentinel file
  in `.git/`. Small change.

### Quality-gate failures

#### F14. Pipeline auditor passes a guide that shouldn't have passed

- **Trigger.** `run-batch.sh` auditor (Opus by default; Sonnet with
  `--sonnet-audit`) is generous on the LLM checks (AUDIT_SPEC V-series,
  C-series, GSPC-series). AuditKit mechanical gates catch the binary
  thresholds (word count, etc.) but can't catch "the prose says
  asbestos is safe to inhale." Editor-tier test findings show Haiku is
  systematically generous; the pipeline doesn't use Haiku for auditor,
  but Sonnet is no worse than Haiku and arguably similar on subtle
  semantic checks.
- **Detection.** None at the gate. Operator review on approved/ before
  shipping (NOT currently done — ship-to-site auto-ships).
- **Damage radius.** Bad content live (see F15 for downstream).
- **Recovery.** F15.
- **Prevention.** Add an explicit final-review gate between
  `approved/guides/` and ship-to-site. Could be: a Telegram-driven
  "approve this slug for live publication?" prompt before ship; or a
  second auditor pass with Opus on a different prompt. Both block the
  autonomous chain. Operator-policy decision required (see §7).

#### F15. Bad guide reaches live site — rollback path

- **Trigger.** Composite of F5, F6, F14. Bad guide is now at
  `https://www.asbestoshq.com/guides/<slug>/`, deployed by Vercel,
  potentially indexed by Google.
- **Detection.** Operator manual review, daily digest, reader
  complaint, GSC manual action, Google ranking drop.
- **Damage radius.** Variable — from "one wrong fact, fix and re-ship"
  to "manual action from Google for misinformation, site-wide penalty."
- **Recovery.** Multi-step, no automation:
  1. `cd ~/projects/asbestoshq-site`
  2. Remove from `APPROVED_GUIDE_SLUGS` in `src/components/GuideArticle.tsx`
  3. `rm src/data/guides/<slug>.json`
  4. `rm -r src/app/guides/<slug>/`
  5. `git commit -m "remove <slug>"`
  6. `git push`
  7. Wait for Vercel redeploy
  8. GSC: submit URL for removal + sitemap update
  9. Decide whether to leave a 410 Gone or a 301 redirect (no
     redirect infrastructure exists today)
  Total operator time: 15-30 minutes per slug, all manual.
- **Prevention.** Anything in F14 reduces likelihood; if it still
  happens, an explicit `ship-to-site --unship <slug>` mode would
  shortcut the recovery from 9 steps to 1. Medium effort.

#### F16. Anthropic API rate-limit or outage

- **Trigger.** `ai-cheap.sh` / `ai-do.sh` / `ai-think.sh` call to claude
  CLI fails (API 429, 500, network).
- **Detection.** Per-wrapper: each script `set -euo pipefail` and exits
  1 on failure. Caller behavior varies:
  - **drafter** — retries once after 30s; writes to `state/failed.log`;
    queue line NOT marked, next tick retries
  - **config-synth** — no retry, saves raw to `state/failed/`, returns 1
  - **ship-to-site** — no LLM dependency, immune
  - **librarian** — would fail the nightly run, no retry
  - **editor** — would fail the tier-test call
- **Damage radius.** No cascading retry storm because failures are
  isolated. Worst case: drafter's */30 tick fires, fails, next */30 tick
  fires the same slug again, fails again. 48 retries/day = $0 spend on
  outage (failed calls aren't billed) but wasted cron ticks.
- **Recovery.** Wait for API to recover. Operator can flip
  `LLM_SPAWN_ENABLED=false` to stop the retry pattern.
- **Prevention.** Each retrying agent (only drafter today) should
  exponential-backoff after N failures to avoid hammering a recovering
  API. Small change.

### Cost-runaway failures

#### F17. Bug causes ai-think.sh to be called in a tight loop

- **Trigger.** Hypothetical: a new agent has a bug where it calls
  `ai-think.sh` in a loop without bounds. Real example: a daemon
  agent's "wait for X" loop that doesn't sleep, or a recursive
  invocation through `run_agent.sh`.
- **Detection.** **None automatic.** `token_usage` rows pile up but
  no agent watches rate-of-change. Dashboard cost chart shows the
  growth but operator must look. `DAILY_API_BUDGET_USD=10` is
  observability-only per orchestrator/failure_modes.md.
- **Damage radius.** Unlimited in principle. At Opus rates (~$15-75
  per ~10K-call burst), real money. If the operator is asleep or
  doesn't see the dashboard for hours, this is the worst-case
  exposure.
- **Recovery.** Flip `SYSTEM_PAUSED=true` — all wrappers exit 1
  within ~2s (mtime-refresh). Then debug.
- **Prevention.** Hard enforcement of `DAILY_API_BUDGET_USD` inside
  `log_token_usage.sh`: after writing the row, query today's sum; if
  > cap, write a sentinel file (e.g. `/tmp/aiteam_budget_exceeded`)
  that `check_kill_switches.sh` reads and treats as
  `SYSTEM_PAUSED=true`. Small change, eliminates worst-case
  cost-runaway. Trade-off: a legitimate large day's work hits the cap.

#### F18. Daily API spend crosses unstated threshold

- **Trigger.** Combined drafter + config-synth + orchestrator + editor
  + tg-monitor spend exceeds `DAILY_API_BUDGET_USD=10` (or any other
  threshold operator has in mind).
- **Detection.** Dashboard cost chart. **Manual eyeball only.**
  Drafter has its OWN $15/day cap that's enforced for pipeline fires
  — but this cap and the global $10 cap are **not reconciled**.
  Drafter could hit $15 in pipeline fires alone while the system cap
  is supposedly $10.
- **Damage radius.** Operator's monthly bill. Currently no cap
  enforcement so no hard ceiling.
- **Recovery.** `SYSTEM_PAUSED=true`.
- **Prevention.** Same as F17 (hard enforcement at the
  `log_token_usage.sh` layer). Also: reconcile the two caps — either
  drafter's pipeline cap is a *subset* of the system cap, or the
  system cap is the only one and drafter inherits it.

### Additional cross-agent failure modes found in audit

#### F19. Concurrent run-batch.sh invocations for the same slug

- **Trigger.** drafter (cron */30) launches `nohup run-batch.sh X
  --guide`. Operator simultaneously runs `bash content/run-batch.sh X
  --guide` manually (e.g., to test or to retry after a budget cap).
  Both processes try to write to `drafts/X.json`, `feedback/X-r*.md`,
  `.cache/assembled-X.md` simultaneously.
- **Detection.** None. Pipeline doesn't take per-slug locks.
- **Damage radius.** Corrupted draft state (one process's write
  overwrites the other's mid-flight); possibly two competing approved
  JSONs ending up where only one should land; double Opus spend.
- **Recovery.** `rm` per-slug files under `drafts/`, `feedback/`,
  `.cache/`, `approved/guides/` if hybridized. Re-run cleanly.
- **Prevention.** Per-slug `flock` in run-batch.sh (or in drafter's
  `fire_pipeline.sh` wrapper). Small change.

#### F20. Approved guide sits unshipped because ship-to-site is halted

- **Trigger.** Operator touched `~/agents/ship-to-site/halt.flag` (or
  `/halt_shipping` Telegram). Meanwhile drafter+pipeline produces an
  approved guide. The guide sits in `approved/guides/` indefinitely.
- **Detection.** ship-to-site's daily digest shows "Shipped: 0".
  Operator must remember the halt is on.
- **Damage radius.** Reversible. When operator removes halt.flag,
  next */15 tick ships the backlog (one slug per tick).
- **Recovery.** `rm ~/agents/ship-to-site/halt.flag` or
  `/resume_shipping`.
- **Prevention.** Notify-on-backlog: if halt.flag is present AND
  `approved/guides/` has un-shipped slugs, daily digest sends a
  reminder. Small change.

#### F21. Mid-tick SYSTEM_PAUSED set is not honored

- **Trigger.** Operator sets `SYSTEM_PAUSED=true` in
  `~/agents/config/.env` while drafter is mid-Sonnet-call (already
  past `drafter.sh:57`'s pre-flight check).
- **Detection.** No mid-tick re-check after the entry guard. The
  Sonnet call completes; the draft is written; the pipeline fires.
  The PAUSE only takes effect on the NEXT */30 tick.
- **Damage radius.** One slug's worth of Sonnet + one pipeline fire
  ($2.50 est + real Opus cost). Recoverable.
- **Recovery.** Wait for in-flight tick to finish; subsequent ticks
  honor the pause.
- **Prevention.** `check_kill_switches.sh` exists for mtime-refresh
  during long-running scripts. drafter could call it between slugs
  in the queue loop, similar to ship-to-site's mid-tick halt.flag
  re-check. Small change.

#### F22. Operator-system $10 vs drafter-fire $15 caps disagree

(Already covered in F18 prevention — listed separately because it's a
config-drift bug, not a runtime failure.)

#### F23. notify.sh single-quote injection in SQL payloads

- **Trigger.** Any agent passes a payload to `log_to_audit.sh` that
  contains a single quote (e.g. an error message saying
  `"primary 'foo'"`). Discovered during config-synth build —
  `log_to_audit.sh` builds SQL by shell-expanding values, so a
  single quote terminates the SQL string and the INSERT fails
  silently (caller wraps with `|| true`).
- **Detection.** **Silent.** Caller's `|| true` swallows the
  error. Audit_log row missing. No row to query.
- **Damage radius.** Audit-log integrity. Any failure with a
  quote-containing reason fails to be logged. Cross-agent
  observability degraded.
- **Recovery.** Defensive scrub in caller (config-synth's
  `handle_failure` now does `tr "'" '_'`). Pre-existing bug in
  the shared lib.
- **Prevention.** Fix `log_to_audit.sh` itself. Use `sqlite3`
  parameterized inserts via `printf '.parameter set ...'` or
  pipe payload through `python3 -c`. Small change, eliminates a
  whole class of silent failures across every agent.

---

## 3. Severity ranking — 2×2

Damage axis: **HIGH** = irreversible or expensive (bad live content, large
$ wasted, audit reproducibility lost). **LOW** = reversible by one operator
action in <5 min.

Likelihood axis: **HIGH** = happens in normal operation weekly+. **LOW** =
requires multiple coincident faults.

```
                       LOW likelihood              HIGH likelihood
                   ┌──────────────────────────┬──────────────────────────┐
HIGH damage        │  F5  broken inline links │  F2  drafter fires w/o   │
                   │  F6  semantic gibberish  │      keyword-config      │
                   │  F13 git-pull race       │  F14 auditor pass-but-   │
                   │  F15 rollback path       │      shouldnt            │
                   │  F19 concurrent run-batch│  F17 cost runaway loop   │
                   │      for same slug       │  F18 unstated budget cap │
                   │                          │                          │
                   ├──────────────────────────┼──────────────────────────┤
LOW damage         │  F10 SQLite WAL fail     │  F1  orphan config       │
                   │  F12 operator mid-write  │  F3  config edited       │
                   │  F16 API outage          │      post-approval       │
                   │  F22 cap config drift    │  F4  same-file races     │
                   │                          │  F11 operator deletes    │
                   │                          │      expected file       │
                   │                          │  F20 shipped during halt │
                   │                          │  F21 mid-tick pause skip │
                   │                          │  F23 single-quote SQL    │
                   │                          │                          │
                   │                          │  (F7, F8, F9 — already   │
                   │                          │   handled or non-issues) │
                   └──────────────────────────┴──────────────────────────┘
```

**High/High quadrant (priority fixes):** F2, F14, F17, F18.

---

## 4. Top 5 priority fixes

Picked from §3's high/high quadrant + one straggler (F23) because it
silently degrades the observability that everything else relies on.

### 1. Drafter pre-flight check for keyword-config existence (fix F2)

- **Change.** In `~/agents/assignment-drafter/lib/fire_pipeline.sh`,
  before launching `nohup run-batch.sh`, test:
  `[ -f "${pipeline_repo}/content/asbestos/keyword-configs/${slug}.json" ]
  || { echo "FATAL: no keyword-config for ${slug}" >&2; exit 1; }`.
  The drafter already buckets fire failures into `FIRE_FAILED_SLUGS` and
  surfaces them in Telegram; this just teaches the gate to fire earlier.
- **Lives in.** `~/agents/assignment-drafter/lib/fire_pipeline.sh`.
- **Complexity.** **Small** — single bash test.
- **Retrofit?** Yes. Pure addition to existing agent. No coordination
  with other agents needed.
- **Why first.** Highest expected dollar-waste reduction. Currently any
  enqueued slug without a config burns $1-5 in Opus + propagates to
  needs-review/ with no actionable diagnosis. Fix is a one-line test.

### 2. Hard budget cap enforced at log_token_usage.sh (fix F17 + F18)

- **Change.** In `~/agents/lib/log_token_usage.sh`, after inserting the
  row, run `SELECT SUM(estimated_cost_usd) FROM token_usage WHERE ts >
  strftime('%s','now','start of day');`. If sum > `DAILY_API_BUDGET_USD`,
  `touch /tmp/aiteam_budget_exceeded` AND set
  `SYSTEM_PAUSED=true` in `~/agents/config/.env` (so the mtime-refresh
  picks it up everywhere). Print a clear error to stderr so the offending
  agent's stdout shows the cause.
- **Lives in.** `~/agents/lib/log_token_usage.sh`.
- **Complexity.** **Small-medium** — 15-20 lines including the
  edge-case where two agents racing past the cap each trip it.
- **Retrofit?** Yes. Every agent already calls log_token_usage via
  the ai-* wrappers. No agent code changes.
- **Why second.** Eliminates the worst-case exposure (cost runaway).
  Trade-off: a legitimate big day hits the cap and operator must raise
  it manually. Operator should set the cap with that in mind.

### 3. Pre-ship operator approval for first N publications (fix F14 + F15)

- **Change.** Add a Telegram-driven approval gate between `approved/guides/`
  and ship-to-site for the **first 5-10 slugs after enabling** any new
  pipeline configuration. After the burn-in period, fall back to
  auto-ship for slugs whose audit verdict was unanimous-PASS (no
  feedback in any of the 3 rounds).
- **Lives in.** `~/agents/ship-to-site/ship.sh` + new
  `~/agents/ship-to-site/lib/approval_gate.sh` + telegram bot handler.
- **Complexity.** **Medium** — gate logic + Telegram conversation state.
- **Retrofit?** Yes. Ship-to-site already has `halt.flag`; the approval
  gate is a per-slug version of the same idea.
- **Why third.** Fully autonomous chain means bad content can land live
  with zero human review. Operator-policy decision required (see §7) —
  this fix sets a defensive default that can be tightened or loosened.

### 4. Fix log_to_audit.sh SQL-injection hazard (fix F23)

- **Change.** Rewrite `~/agents/lib/log_to_audit.sh` to use sqlite3's
  parameterized inserts. Cleanest path: pipe payload through
  `python3 -c` which uses `sqlite3.connect(...).execute(sql, params)`.
- **Lives in.** `~/agents/lib/log_to_audit.sh`.
- **Complexity.** **Small** — 20 lines, no API change for callers.
- **Retrofit?** Yes. Drop-in replacement for the existing shell
  function. Every agent benefits automatically.
- **Why fourth.** Eliminates a silent failure class that affects every
  agent's audit visibility. Discovered during config-synth build —
  failure reasons with apostrophes never reach the audit log.

### 5. Per-slug flock around run-batch.sh (fix F4 + F19)

- **Change.** Wrap run-batch.sh invocations in `flock -n
  /tmp/aiteam-pipeline-<slug>.lock` (from coreutils flock). If lock is
  held, log a `pipeline_fire_locked` audit row and skip. Same for
  ship-to-site's stage.sh per-slug.
- **Lives in.** `~/agents/assignment-drafter/lib/fire_pipeline.sh` and
  `~/agents/ship-to-site/ship.sh`.
- **Complexity.** **Small** — 3-5 lines per call site; macOS flock via
  homebrew if not present.
- **Retrofit?** Yes. Pure addition; no behavior change in non-race cases.
- **Why fifth.** Race conditions are rare but ugly when they hit
  (corrupt drafts, doubled spend). Cheap insurance.

---

## 5. Detection gaps (silent failures)

Failure modes that produce NO operator-facing signal until secondary
symptoms appear. These are the most dangerous because operator trust
erodes without anyone flagging it.

1. **F1 — orphan keyword-configs.** Sits in `keyword-configs/` until
   operator notices via `git status`. No alarm.
2. **F2 — drafter fires without config.** `pipeline_fire_failed` audit row
   IF run-batch.sh fails at startup; otherwise hidden in
   `content/asbestos/logs/batch-*.log` until next ship-to-site digest
   shows "Shipped: 0".
3. **F3 — config edited/deleted post-approval.** No agent checks.
4. **F5 — broken inline links shipped.** Detected only by weekly
   `weekly-link-audit.yml` (Mondays 9am PT) or reader complaint.
5. **F6 — semantically gibberish output.** No gate catches it. Only
   operator spot-read.
6. **F10 — SQLite WAL bloat.** No size monitoring.
7. **F11 (silent subset) — approved guide deleted from pipeline.**
   ship-to-site's "nothing to ship" is indistinguishable from "nothing
   approved this tick."
8. **F14 — auditor false positive.** Auditor signs off; no second
   opinion before ship.
9. **F17 — cost runaway.** Dashboard cost chart shows the growth but no
   alarm.
10. **F18 — unstated budget cap exceeded.** Same as F17.
11. **F20 — backlog while halted.** Digest shows zero shipped; operator
    must remember the halt is on.
12. **F23 — single-quote audit_log failures.** Caller's `|| true`
    swallows. No row to query later.

The common thread: there's no agent in the fleet whose job is to **watch
the rest of the fleet's outputs and flag patterns**. Each agent reports
its own actions to audit_log; no one looks at audit_log holistically.

---

## 6. Recommended monitoring/alerting layer

A 7th agent (`watchdog/` or `fleet-monitor/`) — read-only over audit_log,
token_usage, the pipeline repo, and the site repo. Cron */15 or hourly.
**Not building it; spec only.**

### What it would watch for

| Watch | Query / check | Alert (Telegram) when |
|---|---|---|
| F2 detector | For each drafter `assignment_drafted` row in last 24h, check `keyword-configs/<slug>.json` exists at that moment in time (via git log if needed) | Mismatch within ±30 min of drafter run |
| F10 monitor | `ls -la ~/store/aiteam.db-wal` | WAL > 50 MB |
| F11 detector | Diff `approved/guides/` vs `audit_log` `ship_to_site` rows by slug | Approved-but-unshipped for > 6 hours AND no halt.flag |
| F17/F18 monitor | `SELECT SUM(estimated_cost_usd) FROM token_usage WHERE ts > <today>;` | > 80% of `DAILY_API_BUDGET_USD` |
| F20 monitor | `[ -f ~/agents/ship-to-site/halt.flag ]` AND backlog > 0 | Once per day when both true |
| F23 detector | `SELECT * FROM audit_log WHERE action='X' AND ts BETWEEN ...` for known agent activity windows where rows should exist | Expected-row missing |
| Orphan agent_id | `SELECT DISTINCT agent_id FROM token_usage` vs `ls ~/agents/` | New unknown agent_id appears |
| Cron staleness | mtime of `state/last_run.txt` per agent | > 2× expected interval (e.g., > 60 min for */30 cron) |
| Pipeline 3R escalation | Count files in `needs-review/` | New files since last check |

### Architecture sketch

- Tier: `cheap` (Haiku) — only for summarizing the alert payload. Most
  work is shell/SQL.
- Inputs: SQLite reads only, plus a few filesystem `stat` checks. Zero
  API spend in normal operation.
- Outputs: Telegram messages via `~/agents/lib/notify.sh`. No writes to
  any pipeline or site repo.
- Failure mode of the watchdog itself: silent. Would need an external
  health check (e.g., orchestrator's nightly diary mentions the
  watchdog's last_run).

### Sequencing

This is post-Top-5 work. Several Top-5 fixes (F17 budget enforcement,
F2 pre-flight) eliminate the worst silent failures at their source —
the watchdog is a backstop, not the primary defense.

---

## 7. Open questions for operator

These can't be decided unilaterally — they're policy questions:

1. **False-positive vs false-negative tolerance for the auditor.** If
   the auditor flags 1 good guide as needing review (false positive)
   per 50 guides vs lets 1 bad guide through (false negative) per 100
   — which is worse? The pre-ship approval gate (Top-5 #3) trades
   false-positive cost (operator time) for false-negative cost (live
   bad content). What's the right balance?

2. **Daily API spend hard cap.** Currently `DAILY_API_BUDGET_USD=10`
   in `~/agents/config/.env`, observability-only. Drafter has its own
   `daily_pipeline_budget_usd: 15.00` enforced on pipeline fires.
   Three policy questions:
   - What's the actual daily ceiling operator is willing to spend?
   - Should the system-wide cap and drafter's pipeline cap be
     reconciled (system >= drafter)?
   - Hard stop (Top-5 #2) — accept that legitimate big days hit the
     cap and require manual override?

3. **Rollback policy for bad live content.** When F15 fires (bad
   content on live site), is the policy:
   - (a) Immediately unship + GSC removal (operator does it within
     an hour of detection)?
   - (b) Mark wrong and edit in place (less SEO disruption, slower)?
   - (c) Take down the entire site temporarily?
   Each implies different tooling. Right now no tooling exists for any
   of them — all manual.

4. **Burn-in publication count for approval gate.** Top-5 #3 suggests
   5-10 slugs require pre-ship approval after any config change. Is
   that the right number? Lower = more risk to live content. Higher =
   more operator burden.

5. **Concurrent run-batch.sh policy.** Currently no lock. If operator
   wants to manually run for a slug while drafter cron might fire the
   same slug, options:
   - Lock (Top-5 #5) — operator must wait or use a different slug
   - Don't lock — accept the race and clean up after
   - Lock with override — `--force-no-lock` for operator manual runs
   Which UX is right?

6. **Watchdog priority.** §6 specs a 7th agent. Is this:
   - Next-build priority? (recommended after Top-5)
   - Wait until a real silent-failure incident motivates it?
   - Not needed if Top-5 closes enough gaps?

7. **Editor agent's role in the production pipeline.** `editor/` exists
   and is tuned (rubric, tier-test) but currently isn't called by
   drafter, ship-to-site, or run-batch.sh. Was the intent for editor
   to be a pre-ship gate (slotting in similar to Top-5 #3)? If so,
   that's a much smaller build than building approval-gate from scratch
   — editor's `do` tier already produces a 5-axis scorecard. Operator
   could set a score threshold for auto-ship vs require-review.

---

## Sign-off

- ✅ All 7 sections present
- ✅ All 18 spec'd failure modes documented; 5 additional (F19-F23) found
- ✅ 2×2 severity matrix populated — every mode placed
- ✅ Top 5 fixes are concrete (file, change, complexity, retrofit)
- ✅ Detection gaps is a real list (12 items)
- ✅ Agent fleet map references actual agents (10 active in `~/agents/`)
- ✅ Read-only — no agent edits, no git operations
