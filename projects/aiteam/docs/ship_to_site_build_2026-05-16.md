# Ship-To-Site — Build Report (2026-05-16)

## TL;DR

The agent is built and the cron is live. Six of ten sign-off items are ✅;
four are ⚠ because they require either a new pipeline-approved slug (none
currently exists — pipeline and site are fully in sync as of commit
`4a0d91c Batch 3: ship 10 guides to site`) or an operator-side action
(bot restart). The architecture, scripts, configs, kill switches, and
audit-log writes are all in place and exercised by the tests that *could*
run today.

Code: `~/agents/ship-to-site/`. Cron is installed; first */15 tick will fire
at the next quarter-hour after this report. Daily digest fires at 07:00.

## Sign-off status

### ✅ 1. Dry-run mode works
`ship.sh --dry-run` produces no writes. With every pipeline-approved slug
already in `APPROVED_GUIDE_SLUGS`, the dry-run is silent on stdout but
audits each as `ship_to_site_skipped` with `reason='slug_already_shipped'`
(audit rows `263..271` from the dry-run pass; see
[Test 1 evidence](#test-evidence)). When eligible slugs do appear,
`ship.sh --dry-run` will print `[would-ship] <site>: <slug>` per slug and
write a `ship_to_site_dry_run` audit row each.

### ⚠ 2. End-to-end test (ship one real asbestos slug to live site)
**Deferred — no eligible slug exists at the time of this build.** The
pipeline has 31 approved guides in
`~/projects/asbestos-contractors/content/asbestos/approved/guides/`. All 31
are in `APPROVED_GUIDE_SLUGS` in
`asbestoshq-site/src/components/GuideArticle.tsx`. Most recent site commit
`4a0d91c Batch 3: ship 10 guides to site` closed the gap.

**To exercise this sign-off item, do ONE of:**

  a) Run the pipeline to produce a new approved guide:
     `cd ~/projects/asbestos-contractors && bash content/run-batch.sh
     <new-slug> --guide`. When `approved/guides/<new-slug>.json` lands, the
     next */15 tick will ship it autonomously, or run
     `~/agents/ship-to-site/ship.sh --site asbestos --slug <new-slug>`
     manually.
  b) Operator confirms a slug to un-ship as a controlled E2E:
     remove a slug from `APPROVED_GUIDE_SLUGS` + remove the corresponding
     `src/data/guides/<slug>.json` + remove `src/app/guides/<slug>/`,
     commit + push that "rollback" first, then let ship.sh re-ship it.

Once that happens, this report should be updated with: the slug shipped,
the commit SHA, the live URL, and the `ship_to_site` audit row ID.

### ⚠ 3. Rollback test
**Partial — passed in isolated fixture; not exercised against the live
build_verify failure path.** I built a temp-directory fixture site repo
(initialized with `git init`, populated with a fake `APPROVED_GUIDE_SLUGS`
file and a minimal pipeline JSON containing a `relatedLinks` entry that
referenced a slug NOT in the fake whitelist), ran `stage.sh` against it,
verified all three file ops landed and the relatedLinks filter dropped the
non-whitelisted entry, then ran `rollback.sh` and verified the working tree
returned to clean (zero `git status` changes; JSON file removed; page.tsx
directory removed; whitelist file restored).

The full sign-off shape "deliberately break the page.tsx template, attempt
a real ship, verify rollback" requires an eligible pipeline-approved slug
(see item 2). The lib script contract is validated; the integration with
build_verify failure was not run end-to-end. Recommend revisiting once
item 2 is unblocked.

### ✅ 4. Halt test
- `touch ~/agents/ship-to-site/halt.flag` then `bash ship.sh` → silent exit
  in <10ms, `state/last_run.txt` did not advance.
- `rm halt.flag` then `bash ship.sh` → tick ran, `state/last_run.txt`
  advanced.

Mid-tick halt is also wired: ship.sh re-checks `halt.flag` between slugs in
`process_site`, exits cleanly with `ship_halted_midtick` audit row. Not
exercised at runtime because no multi-slug iteration was possible today,
but the code path is straight-line and was reviewed.

### ⚠ 5. Slash commands work end-to-end via Telegram
**Code is in place; bot restart required by operator.** `bot.js` was patched
to add `bot.command("halt_shipping", ...)` and `bot.command("resume_shipping", ...)`.
`node --check bot.js` passes. Both `/start` and `/help` were updated to list
the new commands. Path constant `SHIP_HALT_FLAG` added near the existing
`*.SH` constants.

**Spec deviation:** the spec used hyphenated `/halt-shipping` /
`/resume-shipping`. Telegram bot commands cannot contain hyphens (BotFather
rule — `^[a-z][a-z0-9_]{0,31}$`). I used underscores
(`/halt_shipping`, `/resume_shipping`) to match the existing bot's
convention (`/morning_brief`, `/weekly_review`).

**To complete this item:** operator restarts the running bot.js process
(it doesn't appear under launchd by name; check `ps aux | grep bot.js`).
After restart, `/halt_shipping` and `/resume_shipping` will fire and write
`ship_halted` / `ship_resumed` audit rows.

### ✅ 6. Daily digest fires correctly
`lib/digest.sh --print` produces a clean message (verified). Live send
(`bash lib/digest.sh`) wrote an audit row `shipping_digest_sent` (row 272)
and goes through `notify.sh` which is currently in `TELEGRAM_OUTBOUND_ENABLED=true`
mode. Format matches the spec — yesterday-by-site, needs-review,
7-day rollup, random sample, last-run freshness.

Sample output (today, zero shipping activity):
```
📦 SHIPPING DIGEST — 2026-05-16

Shipped yesterday (2026-05-15):
  • asbestos: 0 guides
  • ssg: 0 (config disabled)

Needs review (audit failed, awaiting operator):
  (none)

7-day rollup:
  • Shipped: (none)
  • Build failures (caught + rolled back, no push): 0
  • Push failures: 0

Random sample (verify quality):
  (no shipped slugs yet)

Last run: 2026-05-16T17:49:33-0700 (0 min ago)
```

### ✅ 7. Cron is installed
Two lines added to operator's crontab:
```
*/15 * * * * /Users/mmm2/agents/ship-to-site/ship.sh >> /Users/mmm2/agents/ship-to-site/ship.log 2>&1
0 7 * * *    /Users/mmm2/agents/ship-to-site/lib/digest.sh >> /Users/mmm2/agents/ship-to-site/ship.log 2>&1
```
`state/last_run.txt` updates every */15 tick; first auto-tick will land at
the next quarter-hour after the build wraps. Manual ticks during testing
have already proven `state/last_run.txt` writes correctly.

### ⚠ 8. Audit log has ≥3 distinct action types
Currently 2 distinct action types from this agent's actor_id:
- `ship_to_site_skipped` (62 rows from initial dry-run + halt/resume tests)
- `shipping_digest_sent` (1 row, id 272)

A 3rd action type (`ship_to_site` for a real ship, OR `ship_halted` /
`ship_resumed` from Telegram slash commands, OR `ship_to_site_dry_run`
once eligible slugs exist) will land as soon as item 2 or item 5
unblocks. The audit-write call sites for `ship_to_site`,
`ship_to_site_build_failed`, `ship_to_site_push_failed`,
`ship_to_site_stage_failed`, `ship_to_site_dry_run`, `ship_halted_midtick`
are all in `ship.sh`; reviewed via code-read.

### ✅ 9. failure_modes.md has ≥6 modes
8 modes documented in `~/agents/ship-to-site/failure_modes.md` (cause,
detection, kill switch, recovery, fix-forward):
1. Site repo dirty (uncommitted changes)
2. `npm run build` failure after stage
3. `git push` rejected
4. CI poll timeout
5. APPROVED_GUIDE_SLUGS file moved / refactored
6. Pipeline duplicate approved JSON
7. GitHub API rate limit
8. Pipeline approved/ directory missing

### ✅ 10. README.md has operator usage examples
`~/agents/ship-to-site/README.md` covers: state inspection, manual
invocations (4 forms), pausing (3 paths), audit-row pulls (3 example
sqlite queries), force-running digest, cron schedule, what NOT done, hard
constraints.

## Spec deviations (and reasoning)

1. **`slugs_whitelist_file: src/components/GuideArticle.tsx`**, not
   `src/lib/site-config.ts` as the spec config example suggested. The spec
   itself said "VERIFY before committing this value" — verified, the recon
   doc was correct, the spec config example was speculative. The Set lives
   in `GuideArticle.tsx` along with the renderer's link-parsing logic, with
   31 entries as of this build. Documented inline in `config/asbestos.yaml`.

2. **Slash commands use underscores, not hyphens.** Spec said
   `/halt-shipping` and `/resume-shipping`. Telegram bot commands can't
   contain hyphens. Used `/halt_shipping` and `/resume_shipping` to match
   the bot's existing convention (`/morning_brief`, `/weekly_review`).

3. **Digest sends as its own 07:00 cron line, not bundled with market
   briefer.** Spec mentioned "bundled with market briefer cron tick (already
   existing)." No market briefer cron line currently exists. Rather than
   invent or modify an unrelated cron, ship-to-site got its own 07:00 line.
   If market briefer is added later and the operator prefers bundling, the
   digest can move into a wrapper script trivially.

4. **CI poll uses unauthenticated GitHub API** (60 req/hour limit).
   `gh` CLI is not installed on this box. The spec allowed "GitHub free
   API tier." Rate-limit behavior is documented in failure mode 7.

5. **One audit-log table, no per-site queue file partition.** The
   `state/needs_review_queue.txt` is a flat list across sites, not a
   per-site file. With only asbestos currently enabled and the slug
   namespaces non-overlapping in practice, this is fine. If ssg comes
   online, the digest may need to split-by-site (currently shows a flat
   "N slug(s) awaiting operator" rather than per-site counts).

## Test evidence

Audit row IDs created during the build session (newest at top):

| ID | Action | Target | Reason / Notes |
|----|--------|--------|----------------|
| 272 | `shipping_digest_sent` | `digest` | live `notify.sh` path; payload includes 7-day rollup |
| 263–271 | `ship_to_site_skipped` | `<slug>` (9 of 62 shown) | `reason=slug_already_shipped` from dry-run pass |

Run-tick proof: `state/last_run.txt` advanced from `2026-05-16T17:44:52-0700`
(pre-halt) to `2026-05-16T17:49:33-0700` (post-resume), and the
intermediate halted tick did NOT advance it.

Rollback fixture test: ran `stage.sh` against a fresh `git init` temp
directory with a synthetic 2-slug whitelist + a pipeline JSON whose
`relatedLinks` contained one whitelisted entry + one non-whitelisted entry.
After stage: JSON copied (relatedLinks reduced from 2 entries to 1 —
filter works), page.tsx generated, whitelist appended (slug count goes
from 2 to 3). After rollback: working tree returned to clean (`git status`
empty), JSON gone, page.tsx directory gone, whitelist back to 2 entries.

Slash command code review: `bot.js` lines added at the path-constants
block, after the `/important` handler (before voice handler), and inside
`/start` + `/help` reply text. `node --check ~/agents/telegram/bot.js`
passes. Live behavior depends on bot restart.

## DEFERRED — known work the spec acknowledges or this build leaves open

- **Live E2E ship** — requires a new pipeline-approved slug (or operator
  decision to un-ship + re-ship one for testing). See item 2 above for
  exact prerequisite.
- **Live build_verify failure rollback test** — same prerequisite as E2E.
  Rollback contract is fixture-validated but not exercised under a real
  failed `npm run build`.
- **Live slash command test** — operator restart of the Telegram bot to
  pick up the bot.js patch.
- **SSG enablement** — `config/ssg.yaml` is `enabled: false`. SmartSourceGuide
  has no data-driven renderer and no JSON content layer; each article is
  inlined as JSX. The config has TODOs documented inline; before flipping
  to enabled, the SSG site needs (a) a renderer analogous to
  `GuideArticle.tsx`, (b) a slug whitelist file, (c) the SSG pipeline to
  produce approved JSONs. None of those exist yet.
- **Content-drift detection** — failure mode 6 notes a future
  `ship_to_site_content_drift` action could be added. Out of v1 scope.
- **`/halt_shipping` and `/resume_shipping` rename to hyphenated form** —
  if a future Telegram client allows hyphens in commands, or if a wrapper
  for hyphenated commands is added to bot.js, revert to spec form. Today's
  underscore form is operator-final.
- **Multi-day commit cleanup** — referenced in the spec's DEFERRED list.
  Out of v1 scope; nothing to clean up yet because nothing has shipped
  through this agent.
- **Failure modes 2, 3, 4, 5, 6, 7, 8 are documented but not all
  exercised at runtime.** Only failure mode 1 (site_repo_dirty) is reliably
  reachable today (operator can manually dirty the tree to repro). The
  others require the precondition of an eligible slug.

## Next operator actions

1. Decide on item 2 (E2E test): either run the pipeline to produce a new
   slug, or pick an existing slug to un-ship for controlled re-ship.
2. Restart `bot.js` so `/halt_shipping` and `/resume_shipping` go live.
3. Verify `state/last_run.txt` updates after the next */15 tick (it should
   advance to a time ≤15 min from when cron was installed).
4. Watch `~/agents/ship-to-site/ship.log` after the first cron tick. It
   should be empty unless something printed (all skips are silent on
   stdout; only audit_log records them).
5. After the first real ship, verify the live URL (the agent prints it in
   the `ship_to_site` audit row's `payload.live_url`).

## Files added (in `~/agents/ship-to-site/`)

```
agent.yaml                CLAUDE.md                README.md
failure_modes.md          ship.sh                  .gitignore
config/asbestos.yaml      config/ssg.yaml
lib/validate.sh           lib/stage.sh             lib/build_verify.sh
lib/git_ops.sh            lib/rollback.sh          lib/digest.sh
templates/page-tsx-template.tsx
state/.gitkeep            (state/*.log,txt are gitignored runtime state)
```

## Files modified

```
~/agents/telegram/bot.js  — added SHIP_HALT_FLAG constant, /halt_shipping
                            and /resume_shipping handlers, updated /start
                            and /help. node --check passes. Restart required
                            for changes to apply.
```

## Crontab change

```
+ */15 * * * * /Users/mmm2/agents/ship-to-site/ship.sh >> /Users/mmm2/agents/ship-to-site/ship.log 2>&1
+ 0 7 * * *    /Users/mmm2/agents/ship-to-site/lib/digest.sh >> /Users/mmm2/agents/ship-to-site/ship.log 2>&1
```
