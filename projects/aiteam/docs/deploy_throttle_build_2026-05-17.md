# Deploy throttle + preview ping — build report (2026-05-17)

## Scope shipped

Temporary review-window scaffolding. The asbestos pipeline previously
deployed every 15 minutes via `*/15 ship.sh`; now it batches all
ready-to-ship slugs to a single 23:00 PT fire, with a 20:00 PT Telegram
preview ping listing what's about to deploy. Designed to be reversed
when draft quality is consistent (see "Reversal procedure" below).

| Artifact | Path | Commit |
|---|---|---|
| Preview ping | `~/agents/ship-to-site/preview_ping.sh` | `3ed57dc` |
| Batch deploy | `~/agents/ship-to-site/deploy_batch.sh` | `3ed57dc` |
| Crontab | (system crontab) | not git-tracked; snapshot at `/tmp/crontab-snapshot-pre-throttle-1778996892.txt` |

Drafter (`*/30`) and `audit_guide.py` are unchanged — only the deploy step
is throttled.

## Spec correction — Vercel, not Cloudflare

The build prompt assumed Cloudflare Pages and asked for a per-commit
Cloudflare preview URL pattern. **There is no Cloudflare in this
codebase.** Confirmed during pre-flight:

- No `wrangler.toml`, no `.cloudflare/` dir.
- `~/agents/ship-to-site/lib/git_ops.sh` does `git push` to
  `mdp280028-ui/asbestoshq-site` `main`.
- Vercel auto-deploys from the `main` branch; live URL pattern is
  `https://www.asbestoshq.com/guides/<slug>/` (from `asbestos.yaml:
  live_base_url`).
- No `.vercel/` dir in the site repo, no `vercel.json`. Nothing in
  `ship.sh` captures or constructs Vercel's per-commit preview URLs.

Because no preview URL scheme exists today, the preview ping ships
without URLs (operator decision: triage on slug + word count + score +
audit reasoning instead). Standing up real Vercel preview deploys is
tracked as D058.

## Ready-to-ship definition (resolved)

**Source of truth: `~/agents/ship-to-site/ship.sh --dry-run` output.**

Per `ship.sh:189-194`, dry-run emits one line per eligible slug:
`[would-ship] <site_name>: <slug>`. It runs the exact same
`process_site()` → `ship_one_slug()` code path the live tick uses,
including:

- File present at `${pipeline_repo}/${pipeline_approved_glob}`
  (asbestos: `~/projects/asbestos-contractors/content/asbestos/approved/guides/*.json`)
- Slug NOT in `APPROVED_GUIDE_SLUGS` whitelist
  (`~/projects/asbestoshq-site/src/components/GuideArticle.tsx`)
- Slug NOT in `~/agents/ship-to-site/state/needs_review_queue.txt`
- Site repo clean
- Pipeline JSON parses

Both `preview_ping.sh` (20:00) and `deploy_batch.sh` (23:00) call
`ship.sh --dry-run` to get the list. **They cannot drift from each
other** — the 20:00 preview describes exactly what the 23:00 deploy
will see, modulo files added/removed between 20:00 and 23:00 (which is
the operator's intended skip channel).

## Crontab diff

Snapshot of the pre-change crontab at:
`/tmp/crontab-snapshot-pre-throttle-1778996892.txt`

### Before
```
55 23 * * * /Users/mmm2/agents/scripts/brain-autocommit.sh
45 23 * * * /Users/mmm2/agents/orchestrator/write_diary.sh
*/5 * * * * cd /Users/mmm2/agents/tg-monitor && ... reader.py ...
0 7 * * *  cd /Users/mmm2/agents/tg-monitor && ... analyzer.py ...
*/15 * * * * /Users/mmm2/agents/ship-to-site/ship.sh >> ... ship.log 2>&1
0 7 * * * /Users/mmm2/agents/ship-to-site/lib/digest.sh >> ... ship.log 2>&1
*/30 * * * * /Users/mmm2/agents/assignment-drafter/drafter.sh >> ... drafter.log 2>&1
```

### After
```
55 23 * * * /Users/mmm2/agents/scripts/brain-autocommit.sh
45 23 * * * /Users/mmm2/agents/orchestrator/write_diary.sh
*/5 * * * * cd /Users/mmm2/agents/tg-monitor && ... reader.py ...
0 7 * * *  cd /Users/mmm2/agents/tg-monitor && ... analyzer.py ...
# DISABLED 2026-05-17 — throttled to 23:00 daily for review window. Restore by uncommenting.
# */15 * * * * /Users/mmm2/agents/ship-to-site/ship.sh >> ... ship.log 2>&1
0 7 * * * /Users/mmm2/agents/ship-to-site/lib/digest.sh >> ... ship.log 2>&1
*/30 * * * * /Users/mmm2/agents/assignment-drafter/drafter.sh >> ... drafter.log 2>&1
0 20 * * * /Users/mmm2/agents/ship-to-site/preview_ping.sh >> ... preview_ping.log 2>&1
0 23 * * * /Users/mmm2/agents/ship-to-site/deploy_batch.sh >> ... deploy_batch.log 2>&1
```

The 07:00 digest line and all non-ship lines are untouched. No collision
risk at 23:00: `write_diary` runs at 23:45, `brain-autocommit` at 23:55.
deploy_batch at 23:00 finishes well before either.

## Reversal procedure

To restore the previous 15-minute deploy cadence:

```
crontab -e
```

Then, inside the editor:
1. Comment out the two new lines:
   ```
   # 0 20 * * * /Users/mmm2/agents/ship-to-site/preview_ping.sh ...
   # 0 23 * * * /Users/mmm2/agents/ship-to-site/deploy_batch.sh ...
   ```
2. Uncomment the original 15-min line:
   ```
   */15 * * * * /Users/mmm2/agents/ship-to-site/ship.sh >> /Users/mmm2/agents/ship-to-site/ship.log 2>&1
   ```
3. Remove the `# DISABLED 2026-05-17 …` comment line above it (optional
   cosmetic).

No agent code change required — the throttle is purely cron-level. The
two new scripts can stay in the repo (idempotent on empty queue) or be
deleted; they don't affect anything that isn't scheduled.

## Cost / time-to-indexed delta

| Metric | Before (15-min) | After (23:00 daily) | Delta |
|---|---|---|---|
| Worst-case time from drafter-output to live | ~15-30 min | up to 23h45m (drafted at 23:01 waits ~23h59m) | up to +24h |
| Best-case (drafted at 22:50) | ~15-30 min | ~10 min | comparable |
| Median (uniform drafter cadence) | ~7-15 min | ~12h | ~+12h |
| Operator review window | 0 min | 3h (20:00 ping → 23:00 fire) | +3h of human gate |
| Time-to-indexed (Google) | dominated by Vercel deploy + Googlebot crawl, ~hours-to-days regardless | same | unchanged |

The Google-indexing layer dominates time-to-traffic, so the +12h median
deploy delay is small in relative terms but provides a meaningful human
gate. Trade is favorable while draft quality is being established.

## Sign-off

| Test | Result |
|---|---|
| Pre-flight checks | ✅ (with HALT-1 surfaced — Cloudflare vs Vercel; resolved by operator decision) |
| "Ready to ship" definition resolved | ✅ — `ship.sh --dry-run` |
| Existing cron line commented out | ✅ — `# DISABLED 2026-05-17 …` comment preserved above it |
| New cron lines added (20:00, 23:00) | ✅ |
| preview_ping.sh executable + syntax-clean | ✅ |
| deploy_batch.sh executable + syntax-clean | ✅ |
| Test 1 — preview_ping format | ✅ (empty path verified via Telegram delivery, audit_log row id=1031, http 200 38 bytes; populated path sanity-checked by synthesizing `[would-ship]` line through formatter against `asbestos-shingles-guide` + `asbestos-don't-demolish-test` — word count 2006, score em-dash for NULL composite, audit-line truncation at 60 chars + `...` confirmed, apostrophes preserved) |
| Test 2 — deploy batch populated path | ⚠ deferred to first real 23:00 fire (operator direction: avoid staging test data into production). Empty path verified: exits 0, writes `deploy_batch_empty` audit row id=1064, no Telegram noise. |
| Test 3 — empty queue terse message | ✅ (collapsed into test 1; queue was empty at run time) |
| Reversibility check | ✅ — `crontab -l \| grep DISABLED` shows the preserved comment block with explicit restore instruction |
| `~/agents/` commit | ✅ — `3ed57dc` |

## What CC could not verify and why

The populated-queue path of `deploy_batch.sh` (test 2) requires either
(a) staging a test slug into `~/projects/asbestos-contractors/content/asbestos/approved/guides/`,
which would publish to production at 23:00 tonight, or (b) waiting for
real drafter output to land. Per operator direction, deferred to the
first real 23:00 fire — verification will come from the Telegram summary
message + the `deployed` rows in `audit_log` after tomorrow's run.

If the first real fire fails, the failure path will produce:
`❌ Deploy: X ok, Y failed (slug1 slug2 …). See deploy_batch.log.`
plus per-slug `deploy_failed` audit_log rows with the same correlation_id
as the `deploy_batch_complete` summary row.

## Open spec drift (operator-facing)

1. **Cloudflare → Vercel** — corrected throughout, see §"Spec correction"
   above.
2. **Preview URL** — spec assumed Cloudflare Pages preview URLs. None
   exist today. Shipped without URLs per operator direction (option 1);
   real preview deploys tracked as D058.
3. **Manual test 2** — populated deploy_batch deferred per operator;
   first real fire is the verification event.
