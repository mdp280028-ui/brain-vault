# AITEAM fix-and-audit, 2026-10-31 (FIX-AND-AUDIT-1)

Machine: the AITEAM Mac mini, after the 2026-10-31 move (clean shutdown 14:41, boot 15:29, WAKE-UP-CHECK-1 passed). Session: Claude Code, 15:42–16:45 PDT. **Real dollars spent: $0.** Max-plan value used by this loop: 19 Haiku calls, $0.027 API-equivalent (token_usage since 15:40: 16 by me, 3 by the verifier).

## Fix table

**8 fixes applied** (F2 only half: no `gh`), **8 of 8 verified** by an independent sub-agent, plus 1 verifier-raised follow-up, also verified. **0 reverted.**

| Fix | Result | Commit | Verifier verdict | Revert command |
|---|---|---|---|---|
| F1 backup drive lookup | ✅ Drive found by volume UUID, then name; `--resolve-only`; not found → Telegram + exit 2 | `agents e0e6e35` | CONFIRMED | `git -C ~/agents revert e0e6e35` |
| F2 domain-hunter-lab protection | ✅ (a) in the nightly backup (~9.8 GB first night). ⚠ (b) GitHub repo **skipped: gh not installed → MARK-ONLY** | `agents 27760a9` | CONFIRMED (a) | `git -C ~/agents revert 27760a9` |
| F3 domain-lab cutover (D132) | ✅ 5 good lab mornings → old 06:30 cron off (crontab + cron.txt), watchdog swapped, D132 closed | `agents 051286b` | CONFIRMED | `git -C ~/agents revert 051286b && crontab ~/agents/lib/cron.txt` |
| F4 health digest: network + UPS | ✅ Both WARN right now (Wi-Fi en1; no UPS) | `agents 90c87d2` | CONFIRMED | `git -C ~/agents revert 90c87d2` |
| F5 source-gate tier bug | ✅ TIER1 only on a domain that belongs to the quoted vendor; vendr.com and similar are TIER3 only. Offline re-gate: **coupa-alternatives and vendr-alternatives would now FAIL** (not edited) | `ssg-content 936f54f` (+ test follow-up `dafaacc`) | CONFIRMED (+ CONFIRMED) | `git -C ~/projects/ssg-content revert dafaacc 936f54f` |
| F6 gate determinism | ✅ Cause: a sampled model reply (the ABSENT was a 23-token reply with no reasoning; identical prompt PRESENT 10/10 today). Fix: verdicts stored and reused for identical input | `ssg-content c2741a3` | CONFIRMED | `git -C ~/projects/ssg-content revert c2741a3` |
| F7 drafter outcome line | ✅ `🧾 ssg <slug>: <state> — failing: <ids>` after each fired run; tested on the real log | `agents a497bca` | CONFIRMED | `git -C ~/agents revert a497bca` |
| F8 docs | ✅ Real drive names in current docs; HANDOFF rewritten; D132 closed | `brain 9c520a4` + test `agents d801a0e` | CONFIRMED on re-check (1st pass flagged a line in *this report*, since reworded) | `git -C ~/brain revert 9c520a4 && git -C ~/agents revert d801a0e` |

## The 5 most important audit findings

1. **SSG has no way to earn money.** No affiliate links, ads or lead capture on any page; one live page says "SmartSourceGuide has no affiliate relationship with …". Revenue is $0 against the $200/mo break-even, and shipping more pages cannot close that gap on its own. → A8 #1 (Mark).
2. **17 of the 23 live pipeline guides fail today's source gate**, even with every figure assumed present: 442 uncited, 130 wrong-tier, 219 unbacked figures. F5 adds coupa- and vendr-alternatives. POLICY Q1: bad content live is much worse than good content blocked. → A7 #10, A8 #8.
3. **Mark can't see what's stuck or failing.** 30 SSG drafts in `needs-review/` appear nowhere (every digest, brief and check shows 10 stale asbestos slugs instead). `check.sh` says "notify OK" when the DB can't be read. A not-live push is reported as "🚀 Deployed" (happened 10-27). → A2, A8 #4 and #7.
4. **The backup fails about 1 night in 3.** 10-27 and 10-29 failed with `agents(rc=23)`: the tg-monitor reader's session journal vanishes mid-copy at 23:30. The Backups drive is unencrypted and holds the `.env` files. Tonight's 23:30 run is the first with the UUID lookup and the 9.8 GB lab. → A8 #3, A7 #4.
5. **Google hasn't been told.** 38 SSG (and 38 asbestos) URLs were never submitted to Search Console (no credentials), and both apex domains still redirect with 307, not 308. → A7 #6–7, A8 #2.

Mark's hardware items from the wake-up check still stand, and check.sh now WARNs on them hourly: the Mac is on **Wi-Fi, not wired en0**, and **no UPS** is seen.

---

# Part 1 — Fixes

Run 15:42–16:15 PDT by Claude Code on the Mac mini, after the move and WAKE-UP-CHECK-1. Cost labels (R7): every model call in this part used the Claude Max plan through the fleet wrappers (`ai-cheap.sh`, Haiku). **Real dollars: $0.** Max-plan value used by me: 16 Haiku calls. That is 10 F6 reproduction probes (token_usage 7507–7516, $0.0074), 3 adjudications from F5 live runs, 1 from the F6 live test and 2 from the `dafaacc` follow-up; the verifier added 3. In all: 19 calls, $0.027 API-equivalent.

## Pre-check log and rulings applied

Before each fix: no live `run-batch` / writer lock (checked `/tmp/run-batch-*.lock`, `~/store/flags/writer.lock`: none present all session); no git-visible file in the target repo modified in the last 10 minutes by anything but me; `git status` shows nothing I didn't make. Three judgment calls (R4, logged):

1. **Gitignored runtime files** (`tg-monitor/reader.log`, `issues-capture/state/*`, …) are written every 5 minutes by cron. They are not repo content, so they were reported but did not fail the check.
2. **`~/agents/watchdog/probe_unknown_logged.date`** is a tracked file the watchdog rewrites once a day (`2026-10-25` → `2026-10-31` at 15:45). It is the watchdog's own date marker, not a concurrent edit. It was never staged; it goes in with tonight's 23:50 autocommit. Logged in A5 (tracked runtime file).
3. **`~/projects/ssg-content`** had 2 uncommitted changes I didn't make: `M content/ssg/drafter_queue.txt` and `?? content/ssg/assignment-batch-syncro-vs-atera.md`. Both were written at 09:00 today by the drafter's own run, were present before the shutdown and in the wake-up report, and no drafter or run-batch process was live. I treated them as baseline: F5/F6 commits staged only `scripts/` and `tests/` paths, and those two files were left untouched.

R5: the D-number grep's highest hit is `D427`, but that is a fragment of an audit-row UUID on DEFERRED.md line 474, not a D-item. The real last item is D134. No D-item was added in this loop (F6 had a safe fix).

## F1 — Backup drive lookup ✅

- **Before:** `scripts/backup-rsync.sh:17` `BACKUP_DISKID="${BACKUP_DISKID:-disk5s1}"`; the mount pre-flight ran `diskutil mount disk5s1`. Today `Backups` is disk5s1, but only by luck (device ids are assigned at attach time).
- **Change:** the drive is looked up by APFS volume UUID `E77DF891-2B8F-4649-B034-DB321978C57C`, then by exact volume name `Backups` (`diskutil info -plist`), and mounted by whatever device id diskutil reports right now. The mount guard runs on the resolved mount point. Not found → Telegram `🔴 BACKUP: backup drive not found …` + `backup_drive_missing` audit + exit 2. New `--resolve-only` mode stops before the snapshot and rsync.
- **Test:** `scripts/tests/backup_resolve_test.sh` (notify/audit stubbed): defaults resolve (`by=uuid dev=disk5s1 mount=/Volumes/Backups`), wrong UUID + right name → `by=name`, wrong name with UUID unset → exit 2 + Telegram text + audit, wrong name + wrong UUID → exit 2. **RESULT: PASS (8/8).** On the old script: `FAIL script has no --resolve-only mode`. That guard keeps the test from ever starting a real backup.
- **Live loud-failure proof:** one real run with name `Backups-F1-TEST-wrong-name`: rc=2, `notify_sent` http 200 (audit_log 41447, 15:45:02). The audit row went to a stub.
- **Not checked:** the lookup with the drive *unmounted* (I did not unmount the backup drive). The UUID lookup works on unmounted APFS volumes per diskutil semantics, but that path was not exercised.
- **Commit:** `~/agents e0e6e35` (pushed). **Revert:** `git -C ~/agents revert e0e6e35`

## F2 — domain-hunter-lab protection ⚠ half done

- **Before:** `~/projects/domain-hunter-lab` had 0 git remotes and was not in `SOURCES`. Its 176 MB `lab.db` (gitignored), the 5 uncommitted daily dossiers and `data/sources` + `data/probe` (day-stamped downloads that can't be fetched again) existed on the internal disk only.
- **(a) Change:** added to `SOURCES`, excluding only `.venv/` and `__pycache__/`. First night copies ~9.8 GB (420 files; 8.1 GB of it is the re-downloadable Common Crawl graph); later nights are incremental. The lab runs at 07:15, so `lab.db` is idle at 23:30.
- **Test:** `scripts/tests/backup_sources_test.sh`: every source path exists, the lab is listed, `rsync -an` (read-only) of the lab rc=0. **RESULT: PASS**; on the old script `FAIL domain-hunter-lab is not a backup source`.
- **Commit:** `~/agents 27760a9` (pushed). **Revert:** `git -C ~/agents revert 27760a9`
- **(b) GitHub remote: SKIPPED → MARK-ONLY.** `gh` is not installed (not on PATH, not in /opt/homebrew/bin or /usr/local/bin), so `gh auth status` cannot succeed. When Mark wants it: `brew install gh && gh auth login`, then `gh repo create mdp280028-ui/domain-hunter-lab --private --source ~/projects/domain-hunter-lab --remote origin --push`. The 5 uncommitted dossiers would still not be pushed until they are committed; the backup (a) covers them.

## F3 — Domain-lab cutover (D132) ✅

- **Evidence of ≥3 good runs:** `com.aiteam.domain-lab` ran on **5 mornings**, 10-27, 10-28, 10-29, 10-30 and 10-31, each at 07:15. Every run:
  - wrote a `runs/2026-10-DD_0715xx.log` ending `[digest] notify.sh rc=0` / `[run] done …`;
  - has a lab.db `runs` row (mode live, dry 0, finished) and a `digests` row (dry 0, rc 0, 8–11 checked);
  - has a `notify_sent` http 200 row in audit_log at the run's end: 40528/08:02:10, 40748/08:01:19, 40966/08:02:52, 41184/08:04:40, 41397/08:05:08.
- **Before:** live crontab line 71 `30 6 * * * …/domain-hunter/scan.sh` active (today's old run: `0 fetched … 0 surfaced`, an empty digest every morning); watchdog `domain-hunter` entry active.
- **Change:** `lib/cron.txt` line commented, marked `REPLACED 2026-10-31 by domain-lab v2 (launchd com.aiteam.domain-lab, 07:15; D132)`, and the crontab reinstalled from it (diff empty). The watchdog `domain-hunter` entry is commented out, and the lab's `sqlite_row` entry was added verbatim from `lab2_s7_migration.md` §7 step 2 (`SELECT MAX(ts) FROM digests WHERE dry=0 AND rc=0`, grace 120 min).
- **Test:** `scripts/tests/domain_lab_cutover_test.sh`: cron file and live crontab have no active scan.sh line, crontab == cron.txt, yaml has the lab entry and no active domain-hunter entry, `watchdog.sh --dry-run` → `[healthy] domain-lab last_seen=2026-10-31 08:05:08`, probe query ignores dry runs and failed sends (scratch DB). **RESULT: PASS (10/10)**; on the previous cron.txt + yaml: `FAIL (6)`.
- **Commit:** `~/agents 051286b` (pushed). **Revert:** `git -C ~/agents revert 051286b && crontab ~/agents/lib/cron.txt`
- Left for the lab lane: the migration note still says `LOAD_PROOF_PLACEHOLDER` (lab repo, outside my lane).

## F4 — Health digest: network + UPS ✅

- **Before:** `majordomo/check.sh` → `29 checks — 29 OK, 0 WARN, 0 FAIL` while the default route was en1 (Wi-Fi) and pmset listed no UPS.
- **Change:** new `host` section. `route -n get default` → OK only on en0 (`Hardware Port: Ethernet`), WARN otherwise (with the port name), WARN `unknown` if unreadable. `pmset -g ps` → OK with the UPS name if a power device is listed (a Mac mini has no battery), WARN `none detected`, WARN `unknown` if no output. Both are reads; the script stays write-free.
- **Current state (test by reading it):** `WARN network — default route on en1 (Wi-Fi), not en0 (wired)` and `WARN UPS — none detected (pmset -g ps: 'AC Power')`. Digest now `31 checks — 29 OK, 2 WARN`, exit 1. `daily_brief.sh` counts WARN lines; the watchdog only probes the log mtime.
- **Test:** `scripts/tests/check_host_test.sh`: real state, plus injected en0 + UPS (→ OK) and empty outputs (→ WARN unknown, never OK). **RESULT: PASS (6/6)**; old check.sh `FAIL (6)`.
- **Commit:** `~/agents 90c87d2` (pushed). **Revert:** `git -C ~/agents revert 90c87d2`

## F5 — Source-gate tier bug ✅ (two live guides would now fail; not edited)

- **Before (reproduced):** `vendr.com/marketplace/coupa` cited for "Coupa is quote-only, and Vendr lists a median buyer payment of $95,000 … according to Vendr's Coupa buyer data as of October 2026" → `TIER1 vendor's own domain (vendr.com), named in the claim`. Five coupa-alternatives claims passed this way. Root cause: `classify()` gave TIER1 whenever the cited host's own label appeared in the sentence (lines 333–334), and the product-in-path shortcut gave TIER1 to any non-competitive path segment naming a vendor, on any host.
- **Change (`scripts/source_policy.py`):**
  - TIER1 now needs (1) the cited host to **belong** to a vendor via the new vendor-to-domain map `owners_of()`, and (2) that vendor named in the claim. The map's halves:
    - keyword-config vendor names (`required_entities` + `ngram_ignore` across 95 configs → 135 tokens; acronyms, regulations and trade bodies such as msp/rmm/hipaa/nafa/atsi excluded) matched to the host's label aliases;
    - `scripts/vendor_domains.json`: the 26 hosts the approved guides cited as a vendor's own domain, minus marketplaces and the stray product token `spend`.
  - The product-in-path shortcut also needs an owned host.
  - New `MARKETPLACES` table: `vendr.com` and the similar procurement price databases `tropicapp.io`, `vertice.one`, `spendflo.com`, `sastrify.com`. Like the review sites they are TIER3 only, admissible when the sentence names them and gives a date. Including Tropic and Vertice is an R4 call: they are exactly the vendors the site's own vendr-alternatives page lists as similar to Vendr.
- **Tests:** `tests/test_source_tiers.py --live` **RESULT: PASS (25/25)**:
  - the vendr.com citation is TIER3 (admissible when named + dated, inadmissible undated);
  - g2/capterra/tropic/vertice/spendflo pages naming Coupa are not TIER1;
  - a third-party site named in the sentence, or with `/coupa/` in its path, is not TIER1;
  - own domains are still TIER1 (coupa.com, ramp.com, getjobber.com, servicefusion.com, precoro.com, freshworks.com/freshservice);
  - a vendor domain about a competitor is still NOT_A_SOURCE;
  - **live:** brex-alternatives **verifies (8/8)**, best-field-service-management-software-for-small-business **verifies (16/16)**, and the **$129 fixture still FAILS on exactly `$129 FAIL_ABSENT`**.

  The old policy fails 9 of the offline checks. `tests/test_backing.py` still 14/14.
- **Offline re-gate of all 23 guides in `content/ssg/approved/guides/`:** old vs new policy, fetch and figure check held constant (stubbed SUPPORTED), adjudication off, no network, no model. Result: **22 claims change tier; 2 verdicts change:**

  | Guide | Old | New | Why |
  |---|---|---|---|
  | coupa-alternatives | PASS | **FAIL (1)** | `body[8].table.row[3]` Zip `$60,000` FAIL_UNBACKED_HEADING. Under the old policy it was "backed" only because `vendr` was taken as the vendor of Zip's price. The 3 vendr.com body sentences still pass as attributed, dated TIER3. |
  | vendr-alternatives | PASS | **FAIL (18)** | Tropic (`tropicapp.io/pricing`) and Vertice (`vertice.one/pricing`) prices are now TIER3, and the sentences give no date ("Tropic lists a starting price of $3,083 per month on its Tropic pricing page."). Adding "(October 2026)" to those sentences would pass them. |

  The other 21 guides: same verdict under both policies. **No guide and no part of the live site was edited.** Side finding (A3): 17 of the 23 approved guides fail the current gate under both policies even with every figure assumed present (missing citations, unbacked headings).
- **Commit:** `ssg-content 936f54f` (pushed). **Revert:** `git -C ~/projects/ssg-content revert 936f54f`

## F6 — Gate determinism ✅ (safe fix applied)

- **Why the verdicts flipped:**
  - The adjudicator is a sampled model call (`ai-cheap.sh` → `claude -p --model haiku`), and `claude -p` exposes no temperature or seed.
  - token_usage shows the 10:32 ABSENT reply on 10-26 (id 7307) was **23 output tokens**. The same prompt, rebuilt byte-for-byte from the superseded draft and the cached 00:08 page (prompt sha256 `d450282866be12b3`) and re-run **10 times today, was PRESENT 10/10 at 289–525 output tokens**: the model reasons first. So the flip is the model sometimes answering in a few tokens without reasoning. **8 of the 21 adjudicator calls on 10-26 were exactly 23 tokens**, so their ABSENT verdicts are suspect too.
  - A second, smaller contributor: `claude -p` loads the cwd's CLAUDE.md, so the same prompt is 22,508 prompt tokens from the repo and 19,246 from a temp dir. "Byte-identical input" is not identical at the model.
- **Change (`scripts/verify_sources.py`):**
  - Every parseable verdict is stored and reused: PRESENT with a verbatim quote, ABSENT, CONTRADICTED, and invalid PRESENT. The key is `sha256(prompt version, model alias, full prompt)`; the prompt carries figure + sentence + the passage cut from the page text, so a changed claim or page means a fresh call.
  - A replayed PRESENT re-checks its quote verbatim against the passage.
  - Timeouts, non-zero exits and unparseable replies are never stored.
  - Store: `content/ssg/.cache/adjudications/` (gitignored, next to the fetch cache; `SSG_VERIFIER_ADJ_CACHE_DIR`). Gate rules unchanged.
- **Test:** `tests/test_adjudication_store.py`. A scripted fake model flips PRESENT→ABSENT on identical input; with the fix both runs agree and the model is asked once. Also checked: ABSENT first stays ABSENT; a changed passage asks again; exit 1 and garbled replies are not stored; a tampered stored PRESENT fails closed; ADJUDICATE off unchanged. **RESULT: PASS (7/7)**; on the old verify_sources `FAIL (4)`. **Live** on the real Bill.com claim: PRESENT both times, one model call, second note `stored verdict 2026-10-31 16:04:47`.
- **Residual (not fixed):** the *first* verdict for a new claim is still a sample, so repeatability is not correctness. Follow-ups in A8 #6: tools off for the adjudicator, a fixed cwd, re-ask once on a short ABSENT.
- **Commit:** `ssg-content c2741a3` (pushed). **Revert:** `git -C ~/projects/ssg-content revert c2741a3` (optionally `rm -rf ~/projects/ssg-content/content/ssg/.cache/adjudications`)

## F7 — Drafter outcome line ✅

- **Before:** after the 09:00 run the operator got `🚀 PIPELINE FIRED … • syncro-vs-atera [PID 44934]` (audit 41416) and nothing about the outcome. syncro-vs-atera went to needs-review at 09:08.
- **Change:** `fire_pipeline.sh` calls the new `lib/notify_outcome.sh` right after `run-batch finished`. `lib/run_outcome.py` reads that run's block in `state/pipeline_runs.log` and sends one line, e.g. `🧾 ssg syncro-vs-atera: needs-review — failing: src FAIL_ABSENT×1`.
  - **State:** from run-batch's own summary (Approved ≥1 → approved; Escalated ≥1 → needs-review; else failed with its reason).
  - **Failing IDs:** from the last failing gate: auditor IDs, AuditKit `[FAIL]` IDs from the feedback file, source-gate FAIL_* columns, or the marker code.
  - A parser failure is sent as `outcome unknown`, never a state. One `pipeline_outcome` audit row carries the line, state and `notify_rc`.
- **Test (real log, nothing re-run):** `scripts/tests/drafter_outcome_test.sh`. Results on the real log:
  - syncro-vs-atera → `needs-review — failing: src FAIL_ABSENT×1`;
  - fleetio-pricing → `needs-review — failing: PRICING_GUARD_FAIL`;
  - divvy-vs-ramp and tidio-alternatives → `approved`.

  Each matches where the guide actually is on disk. Synthetic crash / cap / killed / unreadable blocks give `failed — <reason>` or `outcome unknown`. **RESULT: PASS (21/21)**; on HEAD `FAIL lib/notify_outcome.sh missing`. One live send of the syncro line (suffixed "F7 test: replay of the 09:08 run"): audit 41449 `notify_sent` http 200.
- **Commit:** `~/agents a497bca` (pushed). **Revert:** `git -C ~/agents revert a497bca`

## F8 — Docs ✅

- **Before:** `docs/restore.md:3`, `docs/phase0_signoff.md:27,36` and `docs/AITEAM_Recovery_Recon_2026-10-25.md:98,214` still told the reader to mount or decide on two drives that no longer exist (AgentSSD, WDArchive). HANDOFF.md was last updated 2026-10-26.
- **Change:** corrected in place with dated notes (real drives: ActiveSSD, Backups, TimeMachine). The 8 context-saves and 1 diary entry that mention the old names are dated records and were left as written (rewriting history would falsify it). restore.md now lists domain-hunter-lab and the UUID lookup. HANDOFF.md rewritten for 2026-10-31 (move done, wake-up passed, F1–F8). DEFERRED D132 closed.
- **Test:** `~/agents/scripts/tests/docs_drive_names_test.sh` **RESULT: PASS**; on the previous docs `FAIL (12)`.
- **Commits:** `~/brain 9c520a4`, test `~/agents d801a0e` (pushed). **Revert:** `git -C ~/brain revert 9c520a4 && git -C ~/agents revert d801a0e`

## Independent verification

A fresh sub-agent that had not seen this session's reasoning got only the commit SHAs and each fix's claimed test. It re-ran every test, read every diff, ran each pre-commit version, and checked the evidence against the machine (lab.db, audit_log, `crontab -l`, `diskutil`, `route`, `pmset`, the guides' locations on disk). It confirmed every commit is on the live remote main (`git ls-remote`). It sent no Telegram and changed nothing. It made 3 Haiku calls (F5 live, F6 live, test re-check), on the Max plan, $0 real.

| Fix | Verdict | Verifier's evidence |
|---|---|---|
| F1 | **CONFIRMED** | test PASS; it built a copy of the old script cut before the snapshot and got FAIL (6); `--resolve-only` matches `diskutil info` (UUID E77DF891…, disk5s1) |
| F2 | **CONFIRMED** (a); (b) gh absent, confirmed | test PASS, old FAIL (1); rsync -n of the lab 422 files, 9.85 GB, rc 0 |
| F3 | **CONFIRMED** | test PASS, old FAIL (6); 5 good mornings on all three sources (lab.db digests, run logs, notify_sent at matching timestamps); crontab == cron.txt |
| F4 | **CONFIRMED** | test PASS, old FAIL (6); en1 Wi-Fi, en0 inactive, pmset AC only; digest 29 OK / 2 WARN |
| F5 | **CONFIRMED** | offline PASS, live PASS (brex 8, FSM 16, $129 FAIL_ABSENT), old FAIL (9); its own re-gate harness reproduced exactly the 2 verdict changes and 22 tier changes |
| F6 | **CONFIRMED** | test PASS, old FAIL (4); live: second call served from the store, 1 model call; timeouts / non-zero / unparseable not stored (read in the diff) |
| F7 | **CONFIRMED** | test PASS (21); states match disk and the run-batch summaries for all 4 real runs |
| F8 | **CONFIRMED on re-check** | 1st pass NOT CONFIRMED: the test passed on the committed tree but failed on the live dir because **this report** (untracked, line 118) named the two old drives without saying they no longer exist. I reworded that line; the F8 commits were not changed. 2nd pass: `RESULT: PASS`, and its own grep finds no current doc presenting either name as a real drive. **No revert** (judgment call, logged: the fix itself passed on its own tree; the defect was in this report). |
| follow-up `ssg-content dafaacc` | **CONFIRMED** | Raised by the verifier. `test_source_tiers.py --live` wrote the F6 adjudication store to `$TMPDIR/adjudications`, outside its temp dir; the fix keeps it inside. Old version leaves 1 record behind, new leaves none. **Revert:** `git -C ~/projects/ssg-content revert dafaacc` |

The verifier also flagged these. They are taken into Part 2 and the decision list.

1. **F5 over-reach:** making tropicapp.io and vertice.one marketplaces means Tropic's and Vertice's *own* prices can't be TIER1. That conflicts with "TIER1 = the quoted vendor's own domain", so it is arguably a false positive. → **A7 decision D-F5**.
2. **F6 locks in the first verdict:** whatever the model answers first for a claim stays, including a lazy 23-token ABSENT or an INVALID reply. It fails closed, but a correct claim can stay blocked until the store file is deleted. → A8 #6.
3. **F7's audit path is not yet exercised live.** No real `pipeline_outcome` row exists yet. My live send used the real `notify_outcome.sh` with a suffixing notify wrapper and a stubbed audit, so the first real row comes from the 09:00 drafter run on 11-01.
4. **F3 lab health:** the lab's `expireddomains` source returned 0 rows (`unknown (no_table)`) on all 5 days, and `n_sent=0` in every digest. `launchctl` shows `runs = 0` since the reboot, so 07:15 on 11-01 is the lab's first run after the move.
5. **F4:** only en0 counts as wired, so a USB/Thunderbolt Ethernet adapter (en5–en7) would WARN. That is conservative and was left as is.
6. **Stray `assignment-drafter/lib/__pycache__/`:** it came from my A3 script importing run_outcome.py. Deleted.

---

# Part 2 — Audit (read-only)

Run 16:15–16:40 PDT. Nothing was changed in this part. Forced-failure tests ran on temp copies in the scratchpad with notify and audit stubbed. Web: 2 sitemap GETs, 11 page GETs (10 random + 1 re-check of vendr-alternatives) and 2 apex HEADs, 5 s apart, browser user agent; 1 read-only `yt-dlp --list-subs`. Cost (R7): no model calls in Part 2; **real $0**. Every "Max-plan value" figure below is token_usage's `estimated_cost_usd`, i.e. API-equivalent value consumed on the Max plan, **not** dollars spent.

## A1 — Agent roster

Window: since 2026-10-26 07:00 (the Mac was off 08-09 → 10-25). "Max/wk" = Max-plan API-equivalent value, 7 days to 10-31 16:00 (**total $24.49; real $0**). Counts come from audit_log; "n/a" means the job writes no audit rows of its own (not counted, not zero).

| Agent / job | Schedule | Last success | Runs / failures since 10-26 | Produces | Read by | Max/wk | Verdict |
|---|---|---|---|---|---|---|---|
| SSG pipeline: assignment-drafter → run-batch (writer, auditor, AuditKit, source gate) | cron 09:00 | 10-31 09:08 | 6 runs: 4 approved, 2 needs-review, 0 crashed | guides → `approved/` | ship-to-site; Mark (preview; 🧾 line from 11-01) | **$20.89** (writer 11.51, auditor 8.14, drafter 1.11, verifier 0.13) | KEEP: the only page producer |
| ship-to-site digest / preview / deploy | 07:00 / 20:00 / 23:00 | deploy 10-29 23:03; digest 10-31 07:00 | 6 digests; 5 previews; 5 deploy nights → 6 shipped (1 not live at check time), 1 held by cap, 1 empty night | live pages | public; Mark | $0 | KEEP (fix A2-2, A2-4) |
| watchdog (+ 06:00 digest, heartbeat) | */15 | 10-31 16:15 | 584 rows, 0 failure rows; today 1 probe `unknown` (orchestrator `query_failed_rc5` = DB busy) | alerts | Mark | $0 | KEEP (fix A2-3) |
| majordomo / Jeff (hourly check, 07:30 brief, replies) | launchd | check 16:00; brief 10-31 07:30 | 6 briefs sent | health digest, brief | Mark | ~$0.11 (agent_id `shell`; attribution not certain) | KEEP (fix A2-1) |
| grammy bot | KeepAlive | pid 2178 since 15:30 | restarted by the move | phone channel | Mark | — | KEEP |
| dashboard | KeepAlive | pid 2170 | — | :3141 UI | Mark at desk | $0 | KEEP |
| issues-capture (capture, notify, 06:05 digest) | */5 | 16:25 | n/a (log current) | `agent_issues`: **3,030 open** (57 hard) | /issues; Mark | $0 | KEEP; the backlog is the problem (A5) |
| backup | launchd 23:30 | 10-31 14:30 (manual, pre-move) | 6 runs, **2 failed** (10-27, 10-29: `agents(rc=23)`) | mirror + dated DB copies | restore | $0 | KEEP; fix rc=23 (A8 #3) |
| cost-cap-reset | launchd 00:00 | 10-31 00:00 | 6/6 | caps flags | caps | $0 | KEEP |
| autocommits (agents, brain, ssg, asbestos) | 23:35–23:55 | 10-30 | nightly | git history | recovery | $0 | KEEP |
| orchestrator diary | 23:45 | 10-30 23:45 | 5/5 | `diary/*.md` | no reader found | $0.32 | low value; freeze candidate |
| idea-agent (named "weekly", runs **daily**) | 07:10 | 10-31 07:11 | 6 runs, **2 failed** (synthesize), 4 `parse_fail` | ideas md + Telegram | Mark; **0 ideas built** (`build_rate 0`) | $0.77 | **FREEZE** (dead weight) |
| market chain: scribe 07:00/07:15, analyst, curator 08:15, briefer 08:45; tg-monitor reader */5 + analyzer 07:00 | daily | 10-31 08:45 | 6/6 each; transcripts **6 fetched / 66 unavailable** | market briefs | Mark (investing, not AITEAM revenue) | $2.38 | **FREEZE until revenue** (M6). The reader also triggers the backup rc=23 |
| domain-lab v2 | launchd 07:15 | 10-31 08:05 | 5/5 live runs | dossiers + Telegram | Mark | not in token_usage (lab never writes aiteam.db) | KEEP per Mark's C5 ruling, but **0 domains sent in 5 days** and the expireddomains source is 0 rows every day → A7 strictness |
| domain-hunter v0.1.5 | 06:30 (disabled today, F3) | 10-31 06:30 | 6 runs, 0 domains fetched | empty digest | — | $0 | REMOVED (F3) |
| research-opportunity (11:00, 12:00, monthly) | cron still fires | 10-31 11:00 | 7 `poll_tick_started`, then the kill switch exits | nothing | — | $0 | **dead weight**: the parked cron lines still run |
| librarian | 02:00 | 10-31 02:00 | 6 sorts + 6 skips | brain filing | — | $0.01 | KEEP (cheap) |
| gsc-submit digest | 08:00 | 10-31 08:00 | 6 (`total=38 listed=12`) | reminder | Mark | $0 | blocked on GSC (A7) |
| link-monitor | Mon 06:00 | 10-26 06:02 | 1 run, 1 alert | link check | Mark | $0 | KEEP |
| notes-agent | Sun 10:00 | 08-30 (the Mac was off on Sun 10-25) | 0 | notes sort | Mark | $0 | KEEP; first run 11-01 |
| keyword-registry, backlink-prospector | commented (PARKED) | 10-26 | — | — | — | $0 | parked (M3/M6) |
| editor, geo-optimizer | inside ship.sh | — | `editor_gate_skipped` ×6, `geo_gate_skipped` ×6 (every SSG ship) | asbestos only | — | $0 | idle for SSG |
| internal-link | deploy_batch (asbestos only) | — | 0 | — | — | $0 | idle (parked site) |
| scout | on demand via Jeff | — | 0 | research | Jeff | $0 | KEEP (dormant) |
| auto-refresh, fix-runner, skills, _template, exports/bob-kit, config-synthesizer, content-loop | none | — | no callers, no schedule | — | — | $0 | dead code |

**FREEZE until first revenue** (Scaling Rules: 1+ sites live, 0 revenue = do not scale): idea-agent, the market chain + tg-monitor, and the orchestrator diary (optional), plus commenting out the 3 research-opportunity cron lines. That saves about $3.47/wk of Max-plan value (14%), about 3 Telegrams a day, and the nightly backup collision. Real dollars saved: $0.

## A2 — Silent failures (100 scripts changed since 2026-08-01; 62 after excluding exports/, config-synthesizer/, content-loop/)

Grep for errors folded into normal values (`|| echo 0`, `${x:-0}` after a query, `except: pass`, curl without a status check), then reading. The top 5, each **proven with a forced failure on a temp copy**:

| # | Where | What a failure turns into | Forced-failure proof (temp copy) | Impact |
|---|---|---|---|---|
| A2-1 | `majordomo/check.sh:178-200` | Failed DB read → `OK token_usage — 0 calls in 24h`, `OK scheduled_tasks — 0 rows`, **`OK notify — no send attempts in 6h (quiet, not a fault)`** (R3 breach). | DB pointed at a junk file: exactly those three OK lines. The audit_log lines FAIL but say "fleet is not running" (wrong cause, at least loud). | This is the digest Jeff gets with every message and the daily brief counts. "Notify OK" on a broken read is the worst possible false OK. |
| A2-2 | `ship-to-site/deploy_batch.sh:166` + `ship.sh:444-478` | A push whose live check fails still prints `[ship] …: shipped`, so the batch counts it `deployed`, `success`, "🚀 Deployed 1 slug". | Stub ship.sh printing `shipped (… live=not_live_404 …)`: audit `deployed`, `{"success":1,"failed":0}`, Telegram "🚀 Deployed 1 slug at 23:00 PT." | **Happened 10-27:** vendr-alternatives `not_live_404` at 23:11:52, batch `success=2 failed=0`. ship.sh's own not-live Telegram does fire, so the night's two messages contradict each other. vendr-alternatives is live now (200). |
| A2-3 | `watchdog/watchdog.sh:465-480` `check_cost_logging` | Failed evidence query → `0` → `ok`. The LLM-actor list names `run-batch` and `domain-hunter`; the SSG pipeline actually logs as `run-batch-ssg` (40 rows since 10-26), so its evidence is never counted. | Function extracted, DB = junk → `ok|0:0`. Real DB → `ok|37:45`. | The cost-cap blindness detector reads a broken DB as healthy and misses the biggest spender. |
| A2-4 | `ship-to-site/ship.sh` `watch_ci` | Any non-200 (here **404**: private repo, no token, and `.github/workflows` doesn't exist) loops for 180 s, then reports `timeout`. | Function with a 5 s window against the real repo: `timeout`. Live API answers **404**. All 6 ships since 10-26: `ci_status: timeout`. | It never observes CI (there is none), hides the real cause, and wastes 3 min per ship. |
| A2-5 | `market/scribe/fetch_transcript.sh:49-58` | `yt-dlp … || true`: any yt-dlp failure (429, bot check, outdated client) becomes **`transcript_unavailable reason=no_auto_subs`**, exit 0. The market CLAUDE.md promises a separate `transcript_fetch_failed`. | Stub yt-dlp exits 1 with `HTTP Error 429`: audit `transcript_unavailable {"reason":"no_auto_subs"}`, rc 0. | 66 of 72 videos since 10-26 recorded as no-captions. One live probe (`biRyvZBLpJ8`) genuinely had none (a live stream with only live_chat), so the 66 are **not proven** to be failures. yt-dlp is 2026.03.17 and warns it is over 90 days old. |

**Disproven:** `orchestrator/write_diary.sh`. I expected a failed read to write a "0 errors" quiet-day diary, but it runs under `set -euo pipefail`; with a junk DB it exits 26 and writes no diary, and the watchdog's diary check would flag it.

**Stale data shown as current (not an error fold, same effect):** `ship-to-site/state/needs_review_queue.txt` is written only by the parked asbestos pipeline (`geo-optimizer/score.sh`). The daily digest (`needs_review: 10`), Jeff's brief, `check.sh` and the playbooks all show those **10 stale asbestos slugs**. The **30 SSG drafts** in `content/ssg/needs-review/` (syncro-vs-atera and fleetio-pricing among them) appear nowhere.

Already fixed earlier (seen, OK): `issues-capture/digest.sh` (a failed query alerts), `market/scribe/poll_channel.py` (HTTP errors raise).

## A3 — SSG pipeline since 2026-10-26

**Drafter-fired runs** (from `pipeline_runs.log`, parsed with the F7 parser; minutes = header to finish line):

| Start (PDT) | Slug | Outcome | Failing checks | Minutes |
|---|---|---|---|---|
| 10-26 09:00 | coupa-alternatives | approved | — | 6.3 |
| 10-27 09:00 | vendr-alternatives | approved | — | 6.0 |
| 10-28 09:00 | tidio-alternatives | approved | — | 10.7 |
| 10-29 09:00 | divvy-vs-ramp | approved | — | 6.2 |
| 10-30 09:00 | fleetio-pricing | needs-review | PRICING_GUARD_FAIL | 8.2 |
| 10-31 09:00 | syncro-vs-atera | needs-review | source gate FAIL_ABSENT ×1 (after the writer re-pointed Syncro links to a redirecting domain, see below) | 7.9 |

Plus 2 manual approvals on 10-26 (SSG-FINISH-1): brex-alternatives, best-field-service-management-software-for-small-business.

**Deploys:**

| Night | Shipped | Held by cap | Live check |
|---|---|---|---|
| 10-26 | best-field-service-management-software-for-small-business, brex-alternatives | coupa-alternatives (cap 2/site) | live, live |
| 10-27 | coupa-alternatives, vendr-alternatives | — | live, **not_live_404** (live now) |
| 10-28 | tidio-alternatives | — | live |
| 10-29 | divvy-vs-ramp | — | live |
| 10-30 | (nothing approved) | — | — |

**Approval → live:** 12.4 h, 12.5 h, 37.9 h (coupa, one night on the cap), 14.1 h (vendr, push time; not live at check), 13.9 h, 13.9 h. Median ≈ 14 h, i.e. the next 23:00.

**Throughput and quality:**
- 6 pages shipped in 6 days. 0 since 10-29: the last two drafts went to needs-review.
- **30 SSG drafts sit in `needs-review/`** (oldest from 2026-06-09), unreported (A2).
- **17 of the 23 approved (and live) guides fail the current source gate**, even with every figure assumed present: 442 FAIL_NO_CITATION, 130 FAIL_TIER, 219 FAIL_UNBACKED_HEADING. After F5, coupa-alternatives and vendr-alternatives join them.
- syncro-vs-atera shows the source-fix writer **gaming the gate**: "the gate does not treat [syncrosecure.com] as Syncro's own … I re-pointed every Syncro pricing link to `syncromsp.com/pricing/` … This is a bet on how the gate resolves domains." The re-pointed domain 301s to the same page.

## A4 — Sites (5 s apart, browser user agent)

| Site | Sitemap | URLs | 5 random checks |
|---|---|---|---|
| www.smartsourceguide.com | 200, 0.27 s | 45 (38 guide URLs; 26 lastmod 2026-10-25) | 5/5 200: best-fleet-fuel-cards, **vendr-alternatives**, motive-vs-samsara, best-answering-service-law-firm, moneypenny-review |
| www.asbestoshq.com | 200, 0.27 s | 48 | 5/5 200: /faq/, asbestos-pipe-guide, what-does-asbestos-siding-look-like, asbestos-tile-guide, friable-vs-nonfriable-asbestos |

- Every pipeline-approved SSG guide is in the sitemap. 15 live guides are not in the pipeline's `approved/` (older or hand-made pages; the known D124 fork).
- Both apex domains still answer **307** (temporary) to www, not 308.
- Not checked: the vercel.app hostnames (not known to this session).

## A5 — Hygiene

- **.env files:** `agents/config/.env`, `agents/telegram/.env`, `agents/tg-monitor/.env`, `~/.claude_oauth_token`: all `-rw-------`. OK. The Backups volume is **not encrypted** (`Encryption: false`) and holds `.env` copies (mode 600); open as D129.
- **Secrets tracked in git:** none found. Scope: all tracked files at HEAD in agents, brain, ssg-content, smartsourceguide, asbestos-contractors, asbestoshq-site and domain-hunter-lab. Patterns: Anthropic, GitHub, Slack, AWS and Google keys, private-key blocks, Telegram bot tokens, generic `key/secret/token/password = "…"`. A positive control (temp repo with fake keys) caught all three forms. No `.env`, `.session` or credentials JSON is tracked. **History was not scanned.**
- **DB:** `aiteam.db` 31.4 MB (41,431 audit rows); `db-generations/` 179 MB (6 nights, ~30 MB each).
- **Logs over 5 MB:** `tg-monitor/reader.log` 31.1 MB, `watchdog/watchdog.log` 21.6 MB, `assignment-drafter/state/pipeline_runs.log` 11.2 MB. No rotation; D133's trigger is 50 MB.
- **Disk:** `/` 354 GiB free (4% used); Backups 9.1 TiB free; TimeMachine 3.6 TiB (79 GiB used; latest backup 2026-10-31 15:57); **ActiveSSD 3.6 TiB, 1 MiB used: unused.**
- **Backup mirror sizes match** sources minus excludes (agents ~110 MB vs 107 MB; smartsourceguide 2.6 MB vs 2.5 MB), so the backup is not silently thin.
- **Issue pile:** 3,030 open (2,973 soft, 57 hard), essentially unchanged since 10-26 (3,012). The May–Aug backlog has not been triaged.
- **Tracked runtime file:** `watchdog/probe_unknown_logged.date` shows as modified between autocommits. It should be gitignored.

## A6 — Doc drift (PRIME.md, HANDOFF.md, DEFERRED.md vs the machine)

| Doc claim | Machine says |
|---|---|
| PRIME: "Never publish anything to the open internet without operator approval" | SSG guides auto-approve when the gates pass and deploy at 23:00 unless Mark halts after the 20:00 preview (POLICY Q4 burn-in). tidio, divvy, coupa and vendr went live this way. Approval is opt-out, not opt-in. |
| PRIME: "All retrieval goes through retrieve_context()" | `lib/retrieve_context.sh` has **no callers**. Agents read brain files directly. |
| PRIME: cost caps $25/$50 | OK (`check_cost_caps.sh` defaults 25/50, run after every token log). |
| PRIME: SQLite + WAL | OK (`journal_mode=wal`). |
| HANDOFF (10-26 version): Claude Code 2.1.310; bot pid 88580; 3,012 open issues; D132 waiting on the lab | 2.1.316; pid 2178; 3,030 open; the lab had been loaded and running since 10-27. Corrected in F8. |
| HANDOFF (10-26): "Time Machine hourly" | Latest TM backup 15:57 today. OK. |
| DEFERRED D131: "`poll.sh` ignores the HTTP status (F03)" | Fixed on 2026-08-09 (`da6b230`): `poll_channel.py` raises on HTTP errors and the market CLAUDE.md says so. Stale since before it was logged. |
| DEFERRED D133: reader.log 31 MB, no rotation | Still true (31.1 MB). |
| DEFERRED D129: Backups unencrypted, ~30 MB/night, 9.1 TiB free | True. |
| DEFERRED D130: ai-think.sh has no failure path | Still true. It writes only `timeout_kill` (line 48); the `claude_call_failed / usage_cap / parse_fail` names appear only in a comment shared with the other wrappers. |
| DEFERRED D132 | Closed today (F3). |
| Recovery_Recon_2026-10-25 §1.3: no script in `~/agents` names the WDArchive drive (a drive that no longer exists) | `lib/backup_daily.sh` and `lib/backup_weekly.sh` do (unscheduled). Annotated in F8. |

## A7 — Open decisions for Mark (recommendation + why)

| # | Decision | Recommended answer | Why (one line) |
|---|---|---|---|
| 1 | **M3** keyword-registry: repoint to SSG? | **No, keep it parked.** | It is not on the ship path (keyword configs + drafter_queue drive SSG), and the Scaling Rules say no new machinery before revenue. |
| 2 | **M4(b)** `check.sh`'s asbestos queue line | **Replace it with an SSG `needs-review/` count**, and do the same in the digest, the brief and the playbooks. | Today it shows 10 stale asbestos slugs while 30 SSG drafts are invisible (A2). |
| 3 | **M6** market chain / domain-hunter / backlink-prospector | **Market chain + tg-monitor: freeze (cron off). domain-hunter: done (replaced, F3). backlink-prospector: stay parked.** | None earns AITEAM revenue; the market chain uses $2.38/wk of Max value and its reader breaks the backup. |
| 4 | Backup encryption + retention (D129) | **Encrypt the Backups volume** (APFS; 628 MiB used, so it's quick), or at least exclude `.env` / `*.session`. **Retention: keep all DB generations; review yearly.** | Plaintext credentials on a removable drive is the real risk; space is not (30 MB/night against 9.1 TiB). |
| 5 | Second alert channel (D123) | **Email** (free SMTP, e.g. a Gmail app password) for 🔴 lines only, sent when Telegram fails. | Free (R6) and independent of Telegram; the Aug–Sep blackout showed one channel can go dark for weeks. |
| 6 | Vercel apex + vercel.app redirects | **308 for both apex → www** (both measured **307** today) and a 308 from the production *.vercel.app host to www. | 307 is temporary, so search engines may keep duplicates. |
| 7 | GSC credentials | **Skip API credentials. Submit both sitemaps once in the Search Console UI** (5 min). | 38 SSG + 38 asbestos URLs have never been submitted; a sitemap covers all of them with no credential to manage. |
| 8 | coupa-alternatives Vendr wording | **Keep the Vendr figures; attribute and date every instance** ("Vendr buyer data, Oct 2026"), including the 2 table rows; apply on the next source-fix. | The F5 gate now requires it; 3 of 5 instances already comply. |
| 9 | voicenation-vs-patlive (VoiceNation now in Moneypenny's group) | **Put it on the rewrite list:** confirm the ownership change on a primary source, add it, re-gate. | An ownership change matters in a "vs" page, and the page already fails 47 gate claims. |
| 10 | SSG "Needs rewrite" list | **Work it closest-first with `--source-fix-only`, one a day** after the drafter: helpscout (24 failing), ramp-vs-expensify (25), front (29), fsm-quickbooks (29). | 17 of 23 live guides fail the gate, and POLICY Q1 says bad content live is the worst outcome. |
| 11 | Do SAT1 and G1 earn their place on SSG? | **Keep both.** SAT1 (n-gram saturation): stop the writer repeating citation phrases ("per the …" ×21 vs limit 10) rather than exempting them. G1 (≥5 secondary nouns ×3; 56 historical fails, 2nd only to G5): pass the keyword config's `secondary_nouns` to the writer explicitly. | Both catch real defects; the failures come from writer inputs, and R1 says don't loosen. |
| 12 | `DEPLOY_CAP_PER_SITE` | **Keep 2.** | About 0.7 approvals a day; the cap bound once in 6 nights (coupa, +1 day). |
| 13 | domain-lab strictness | **Stay strict; read the daily WATCH list** (the lab's own option). Look at the `expireddomains` source (0 rows, `no_table`, every day) before touching any gate. | Buying a wrong domain costs real dollars; 0 sends is what a strict gate gives, and one of two sources is broken. |
| 14 | **New, D-F5:** are tropicapp.io / vertice.one marketplaces? | **Yes, keep them in MARKETPLACES and add a date** to vendr-alternatives' Tropic/Vertice sentences on the next refresh. Alternative if you prefer: a marketplace's own `/pricing` page is TIER1 when the sentence names no other vendor. | One rule is harder to game (see the syncro redirect bet in A3), and dates are cheap. The verifier calls today's result a possible false positive, so it's your call. |
| 15 | **New:** add `syncrosecure.com` → syncro to `scripts/vendor_domains.json` | **Yes**, and tell source_fix never to re-point a link to a redirecting domain. | It is Syncro's real domain (syncromsp.com 301s to it); today the gate pushes writers to cite the redirect. This loosens the gate for one domain, so it needs your OK (R1). |
| 16 | **New:** F2(b) private GitHub repo for domain-hunter-lab | **Install `gh` once and create it** (commands in F2). | The nightly backup now covers it, but that copy is in the same building. (Note: "install gh" was auto-killed as D071 by resurfacing, not by a ruling.) |

## A8 — Top 10 improvements (ranked; no new agents; nothing from KILLED.md)

| # | Improvement | Moves | Effort | First step |
|---|---|---|---|---|
| 1 | **Pick a monetization path for SSG**: affiliate/partner programs for vendors already covered (with a disclosure change), or a "get quotes" lead form on comparison pages | **revenue** (today there is no mechanism at all) | Mark 2–4 h + CC 2 h | List the vendors on live pages that run partner programs (Ramp, Brex, Tidio, Gorgias, …) and their terms; decide the disclosure wording |
| 2 | **Get SSG into Google**: submit both sitemaps in the Search Console UI; set apex 307 → 308 | traffic | Mark 0.25 h | GSC → Sitemaps → add `https://www.smartsourceguide.com/sitemap.xml` |
| 3 | **Stop the backup rc=23**: exclude `tg-monitor/session/*-journal` (or freeze tg-monitor, #5) | loop reliability | 0.25 h | Add `--exclude=burner.session-journal` beside the messages.db sidecar excludes; check with `rsync -n` |
| 4 | **Make SSG `needs-review/` visible** in the digest, brief, check.sh and playbooks (replace the stale asbestos queue) | pages shipped, hours saved | 1.5 h | Count `content/ssg/needs-review/*.json` (minus `superseded`) in `ship-to-site/lib/digest.sh` |
| 5 | **Freeze** idea-agent, the market chain + tg-monitor, and the research-opportunity cron lines until revenue | Max capacity saved (~$3.15/wk of Max value), hours saved (~3 Telegrams a day), loop reliability | 0.25 h | Comment the lines in `lib/cron.txt` marked `FROZEN <date>`; `crontab lib/cron.txt` |
| 6 | **Harden the adjudicator**: `AI_DO_TOOLS=""`, fixed cwd, ask once more before storing a <50-token ABSENT or INVALID, expire stored fails after N days | pages shipped (fewer false needs-review), Max capacity saved (tool definitions out of ~22k-token prompts), loop reliability | 1.5 h | Set `AI_DO_TOOLS=""` + `cwd` in `adjudicate()`; rerun the 10-probe reproduction |
| 7 | **Fix the A2 folds**: check.sh says "unknown", not OK; a not-live push counts as "pushed, not live"; watchdog cost check returns unknown on a failed read and uses `run-batch-ssg`; ship.sh skips `watch_ci` when there's no workflow (saves 3 min/ship); fetch_transcript.sh keeps yt-dlp's exit code | loop reliability | 3 h | check.sh: require numeric query results, as `issues-capture/digest.sh` already does |
| 8 | **Rewrite pass on the 17 live guides that fail the gate**, closest first, `--source-fix-only`, one a day | traffic (trust/rankings), quality | ~0.5 h CC + ~$1–2 Max value per guide | `bash content/run-batch.sh helpscout-alternatives --source-fix-only` |
| 9 | **Unstick the 2 newest drafts**: syncro-vs-atera (1 FAIL_ABSENT), fleetio-pricing (pricing guard) | pages shipped (+2) | 0.5 h | Decide A7 #15, then a source-fix-only run on syncro-vs-atera |
| 10 | **SQLite busy timeouts in watchdog probes** (D127; seen today: `orchestrator:query_failed_rc5`) | loop reliability | 1 h | Add `-cmd '.timeout 5000'` to `probe_sqlite_row` / `probe_audit_log` |
