# AITEAM open items

**Created:** 2026-11-04 by FLEET-FIX-2 (fix 14). **State as of:** 2026-11-05 09:45 PT.

**Sources, as named in the table:**
- **FA1:** `docs/AITEAM_Fix_And_Audit_2026-10-31.md` (FIX-AND-AUDIT-1).
- **FF2:** `docs/AITEAM_Fleet_Fix_2_2026-11-04.md` (this loop).
- **Resume:** `~/Downloads/AITEAM_Resume_<window>_2026-10-31.md`.

Keep this list current: remove a row when the item closes, and record the closing commit in its source doc.

## Mark only

Recommended answers are condensed from FA1 section A7. "No A7 answer" means A7 did not cover the item, so the recommendation is FLEET-FIX-2's.

| Item | Recommended answer | Owner | Status | Date | Source |
|---|---|---|---|---|---|
| Pick a monetization path for SSG. No page can earn today; revenue is $0 against $200/mo break-even | No A7 answer. FA1 A8 #1: list vendors on live pages with partner programs (Ramp, Brex, Tidio, Gorgias…), then choose affiliate links with a disclosure change, or a "get quotes" form | MARK | open | 2026-10-31 | FA1 A8 #1 |
| Power and network: the Mac is on **Wi-Fi (en1), no UPS** | No A7 answer. Plug in Ethernet and connect a UPS. **The power cut on 2026-11-04 makes the UPS the priority** | MARK | open, still true 2026-11-04 | 2026-10-31 | FA1 top findings; FF2 fix 12 |
| **Auto-login is off.** After the 11-04 power cut the Mac restarted by itself but sat at the login window for 7 h 47 min, with the bot, dashboard, hourly check and other launchd agents down until someone logged in | No A7 answer. Turn on auto-login (FileVault is off, so it is allowed), accepting that anyone with physical access gets a logged-in Mac. Otherwise the agents stay down after every outage until someone logs in | MARK | open (D139) | 2026-11-04 | FF2 fix 12 |
| Submit sitemaps to Search Console (GSC credentials) | **Skip API credentials. Submit both sitemaps once in the Search Console UI** (5 min). 38 SSG + 38 asbestos URLs have never been submitted | MARK | open | 2026-10-31 | FA1 A7 #7, A8 #2 |
| Vercel apex and vercel.app redirects | **308 for both apex → www** (both measured 307 on 10-31; not rechecked) and a 308 from the production *.vercel.app host to www | MARK | open | 2026-10-25 | FA1 A7 #6; Resume SSG-SITE-AUDIT-1, FLEET-AUDIT-55 |
| Backup encryption + retention (D129) | **Encrypt the Backups volume** (APFS), or at least exclude `.env` / `*.session`. **Keep all DB generations; review yearly** | MARK | open | 2026-10-26 | FA1 A7 #4 |
| Second alert channel (D123) | **Email** (free SMTP, e.g. a Gmail app password) for 🔴 lines only, sent when Telegram fails | MARK | open | 2026-10-26 | FA1 A7 #5 |
| F2(b): private GitHub repo for domain-hunter-lab (no remote; `gh` is not installed) | **Install `gh` once and create it:** `brew install gh && gh auth login`, then `gh repo create mdp280028-ui/domain-hunter-lab --private --source ~/projects/domain-hunter-lab --remote origin --push`. Note: "install gh" was auto-killed as D071 by resurfacing, not by a ruling | MARK | open | 2026-10-31 | FA1 A7 #16, F2 |
| M3: repoint keyword-registry to SSG? (D134) | **No, keep it parked.** It is not on the ship path, and the Scaling Rules say no new machinery before revenue | MARK | open | 2026-10-26 | FA1 A7 #1; Resume FLEET-AUDIT-55 |
| M6 remainder: backlink-prospector | **Stay parked.** A7 also recommended freezing the market chain + tg-monitor; Mark's later ruling in the FLEET-FIX-2 prompt overrides that (keep running, Mark reads them) | MARK | open for backlink-prospector only | 2026-10-26 | FA1 A7 #3 |
| coupa-alternatives Vendr wording | **Keep the Vendr figures; attribute and date every instance** ("Vendr buyer data, Oct 2026"), including the 2 table rows; apply on the next source-fix | MARK | open | 2026-10-26 | FA1 A7 #8; Resume FLEET-AUDIT-55 |
| voicenation-vs-patlive: VoiceNation now in Moneypenny's group | **Put it on the rewrite list:** confirm the ownership change on a primary source, add it, re-gate | MARK | open | 2026-10-31 | FA1 A7 #9 |
| SSG "Needs rewrite" list | **Work it closest-first with `--source-fix-only`, one a day** after the drafter: helpscout (24 failing), ramp-vs-expensify (25), front (29), fsm-quickbooks (29) | MARK | open; CC lane runs it once approved | 2026-10-25 | FA1 A7 #10, A8 #8; Resume SSG-SITE-AUDIT-1 |
| Do SAT1 and G1 earn their place on SSG? | **Keep both.** SAT1: stop the writer repeating citation phrases. G1: pass the config's `secondary_nouns` to the writer. The writer half landed tonight (ssg-content `4dbc3dd`); no check or threshold changed | MARK | open | 2026-10-31 | FA1 A7 #11 |
| `DEPLOY_CAP_PER_SITE` | **Keep 2.** About 0.7 approvals a day; the cap bound once in 6 nights | MARK | open | 2026-10-31 | FA1 A7 #12 |
| domain-lab strictness (H1 does not pass; report §9.0 options a/b/c) | **Stay strict; read the daily WATCH list.** Look at the `expireddomains` source (0 rows, `no_table`, every day) before touching any gate | MARK | open | 2026-10-26 | FA1 A7 #13; Resume DOMAIN-LAB-2 |
| D-F5: are tropicapp.io / vertice.one marketplaces? | **Yes, keep them in MARKETPLACES and add a date** to vendr-alternatives' Tropic/Vertice sentences on the next refresh. Alternative if you prefer: a marketplace's own `/pricing` page is TIER1 when the sentence names no other vendor. The verifier calls today's result a possible false positive, so it is your call | MARK | open | 2026-10-31 | FA1 A7 #14 |
| Park scope confirmation: research-opportunity, backlink-prospector and keyword-registry are 100% asbestos, so parking asbestos stopped all three | No A7 answer. Confirm; nothing is blocked on it | MARK | open | 2026-10-25 | Resume RECOVERY-RECON-2 §8 |
| USPTO register not checked for domain dossiers | No A7 answer. Check it by hand before buying any domain | MARK | open | 2026-10-26 | Resume DOMAIN-LAB-2 |
| omnitracs-reviews: quote-only vendor cannot meet CAP1 (finding D-1); needs a redraft decision | No A7 answer. Redraft only with a new sourced figure; do not loosen CAP1 (R1) | MARK | open | 2026-10-26 | Resume SSG-FINISH-1 |
| Site decisions: meta-refresh stubs, byline, policy pages | No A7 answer. Stubs: switch to 308 redirects (the audit found noindex plus refresh sends conflicting signals). Byline: keep "SmartSourceGuide Editorial" and show it on the page; naming a person is your call. Policy pages: decide alongside monetization, since a disclosure change touches them | MARK | open | 2026-10-25 | Resume SSG-SITE-AUDIT-1 |
| Watch Search Console for the 4 title and meta-description changes | No A7 answer. Check after the sitemaps go in | MARK | open | 2026-10-25 | Resume SSG-SITE-AUDIT-1 |
| D142: the 3-failure guard is rarely reachable. `fire-pipeline` skips any slug whose brief exists, and real retries use `run-batch.sh` directly, which writes no outcome row and never meets the guard | No A7 answer. If you want the limit to bite, put it in `run-batch.sh` and have direct runs write an outcome row. That is an SSG-lane change to a live pipeline, so do it between runs. Otherwise leave it as is | MARK | open | 2026-11-05 | FF2 amendment |
| Doc drift in PRIME: it says no publishing without operator approval, but SSG guides auto-approve and deploy unless Mark halts. It says all retrieval goes through `retrieve_context()`, which has no callers (also D128) | No A7 answer. Change PRIME to match the machine, or tell CC to change the machine; the publishing line is the one that matters | MARK | open | 2026-10-31 | FA1 A6 |
| The separate audit of FLEET-AUDIT-55's 30 commits: that window saw no sign it ran. FIX-AND-AUDIT-1 audited the fleet, not those commits one by one | No A7 answer. Low priority: FA1 audited the fleet they changed, though not commit by commit | MARK | open | 2026-10-26 | Resume FLEET-AUDIT-55 |
| Orchestrator diary: no reader found, $0.32/wk | No A7 answer. FA1 A1 calls it an optional freeze. Freeze it unless you read it | MARK | open | 2026-10-31 | FA1 A1 |

## CC lane: fleet (`~/agents`)

| Item | Owner | Status | Date | Source |
|---|---|---|---|---|
| D135: YouTube answers **HTTP 429** on subtitle downloads. First real run 11-05 07:15: 6 videos, all `transcript_fetch_failed` 429, 0 fetched. Upgrading yt-dlp did not help in a scratch test on 1 video. See FF2 fix 7 | CC lane | open | 2026-11-04 | FF2 fix 7 |
| D136: `scripts/tests/drafter_outcome_test.sh` reads old runs from the real `pipeline_runs.log`. That log will rotate at 20 MB (12 MB now), so the test needs a frozen fixture | CC lane | open | 2026-11-04 | FF2 fix 9 |
| D138: `needs_review_queue.txt` still holds 10 parked asbestos slugs and still blocks `ship.sh`. Not displayed, except 7 in pipeline_status's labelled "STALE REVIEW ENTRIES" line. Approved SSG slugs in it are now shown as `ship_block[ssg]`. Clear the parked ones if asbestos unparks | CC lane | open, parked | 2026-11-04 | FF2 fix 2 |
| Open soft issues, 204 at 09:43 on 11-05: idea-agent 114, unknown actor 60, `transcript_fetch_failed` 6, others. Cause not proven gone, so left open | CC lane | open | 2026-11-04 | FF2 fix 11 |
| 57 hard issues: assignment-drafter 41, `timeout_kill` 15, diary 1. Left open on instruction | CC lane | open | 2026-05 to 2026-08 | FF2 fix 11; FA1 A5 |
| The 20:00 preview cap line on 10-26 was never seen directly: the sent text is not logged. Indirect only: 3 slugs against a cap of 2, `preview_ping_sent` notify_rc 0, code at `ship-to-site/preview_ping.sh:147-151` | CC lane | not checked directly | 2026-10-26 | Resume FLEET-AUDIT-55 |
| Night line covers batch runs only: a slug shipped directly by `ship.sh` outside `deploy_batch.sh` (vanta-vs-drata, 23:22 tonight) has no `deployed` row or night line | CC lane | open, minor | 2026-11-04 | FF2 verifier |
| D141: small hardening leftovers, none reachable in normal use today: a failing `awk` inside `--blocked`; a bare `[scan-skipped]` with no reason; a rotation skip has no alert; `research-opportunity/pain_points.db` sidecars are not excluded from the backup (agent frozen, no writer) | CC lane | open, minor | 2026-11-05 | FF2 verifier |
| D142 rest: `fire-pipeline` prints OK when nothing ran; a real exit-2 `srcver_fail` row is not filed (predates D137); `is_gate_verdict` ignores a singular `error` key; the capture.log counter recounts | CC lane | open, minor | 2026-11-05 | FF2 amendment |
| First real runs still to watch: fix 4 night line 11-05 23:00; rotation with `bfb7ee3` 11-05 23:50 | CC lane | to watch | 2026-11-04 | FF2 |
| D127 rest: about 190 direct `sqlite3` writes elsewhere have no busy timeout (watchdog part done in fix 6) | CC lane | open | 2026-10-26 | FA1 A8 #10 |
| D130: `ai-think.sh` has no failure audit path. **Do not edit during overnight runs** | CC lane | open | 2026-10-26 | FA1 A6 |
| D131: market chain truthfulness. Mark ruled the chain stays, so the trigger is met. Briefer briefs on zero input; curator writes `curator_run` when the LLM failed. The `poll.sh` part is stale (fixed 08-09) | CC lane | open | 2026-10-26 | FA1 A6; FF2 fix 8 |
| D124/D125: pipeline↔site fork; unapproved drafts staged inside `approved/` | CC lane | open | 2026-10-26 | HANDOFF known bugs |
| Dead code with no callers or schedule: auto-refresh, fix-runner, skills, _template, exports/bob-kit, config-synthesizer, content-loop | CC lane | open, no action needed yet | 2026-10-31 | FA1 A1 |
| editor and geo-optimizer are idle for SSG (`*_gate_skipped` on every ship); rubrics are asbestos-tuned | CC lane | open | 2026-10-31 | FA1 A1 |
| OL1–OL14 from the 10-25 audit: other-lane findings, listed there. Not rechecked here | CC lane | not checked | 2026-10-26 | Resume FLEET-AUDIT-55 |
| F4 follow-up: only en0 counts as wired, so a USB/Thunderbolt Ethernet adapter would WARN. Left conservative | CC lane | open, minor | 2026-10-31 | FA1 verifier #5 |

## CC lane: SSG content (`~/projects/ssg-content`, read-only for this loop)

| Item | Owner | Status | Date | Source |
|---|---|---|---|---|
| 17 of 23 live pipeline guides fail the current source gate (442 uncited, 130 wrong-tier, 219 unbacked figures, as of 10-31). coupa- and vendr-alternatives also fail after F5 | CC lane (SSG) | open | 2026-10-31 | FA1 top findings, A3 |
| Harden the adjudicator: tools off, fixed cwd, ask again before storing a short ABSENT, expire stored fails. F6 makes the first verdict repeatable, not right | CC lane (SSG) | open | 2026-10-31 | FA1 A8 #6, verifier #2 |
| Unstick fleetio-pricing (pricing guard). syncro-vs-atera went live tonight | CC lane (SSG) | open | 2026-10-31 | FA1 A8 #9 |
| SSG drafts in `needs-review/`: count at close is in FF2, and it moves while SSG-OVERNIGHT-2 runs. Now shown daily | CC lane (SSG) | open | 2026-11-04 | FF2 fix 2 |
| C48-2: LLM auditor S1 range (1,600–2,000 words) conflicts with SSG_TEMPLATES (1,400–1,800) | CC lane (SSG) | open | 2026-10-26 | Resume SSG-CONTENT-1 |
| C48-3: getjobber.com returns 403 to the gate's fetcher | CC lane (SSG) | open | 2026-10-26 | Resume SSG-CONTENT-1 |
| G-1: item-10 nearest-vendor attribution credits the takeaway $59 to Service Fusion | CC lane (SSG) | open | 2026-10-26 | Resume SSG-CONTENT-1 |
| "Housecall" is not mapped to housecallpro.com in the source gate | CC lane (SSG) | open | 2026-10-26 | Resume SSG-CONTENT-1 |
| brex: source-fix writer claimed it removed Ramp's 1.5% from a table cell but did not | CC lane (SSG) | open | 2026-10-26 | Resume SSG-CONTENT-1 |
| nextiva-pricing, zendesk-pricing need a redraft (SAT1); both are pending in the drafter queue | CC lane (SSG) | open | 2026-10-26 | Resume SSG-FINISH-1 |
| omnitracs repair was never run, so the repair-vs-rewrite test has REPAIR n=1 | CC lane (SSG) | open | 2026-10-26 | Resume SSG-REPAIR-TEST-1 |
| SSG-QUEUE-1 queue reorder and Dialpad re-queue never ran | CC lane (SSG) | open | 2026-10-26 | Resume SSG-REPAIR-TEST-1 |
| Re-check the two `ssg-source-verifier` exclusions (report "Attribution correction" section) | CC lane (SSG) | open | 2026-10-26 | Resume SSG-REPAIR-TEST-1 |
| atera-reviews still names the defunct Growth and Power tiers in 3 sentences | CC lane (SSG) | open | 2026-10-25 | Resume SSG-SITE-AUDIT-1 |

## CC lane: sites and domain lab

| Item | Owner | Status | Date | Source |
|---|---|---|---|---|
| Self-host fonts with next/font/google; launch-guide titles over 60 characters; E1/E2 cannibalization edits (recommended, not started) | CC lane (site) | open | 2026-10-25 | Resume SSG-SITE-AUDIT-1 |
| domain-lab `expireddomains` source returns 0 rows (`no_table`); 0 domains sent on the 5 days checked | CC lane (lab) | open | 2026-10-31 | FA1 verifier #4 |
| Stale copy `~/Downloads/AITEAM_SSG_Finish_2026-10-26.md`; the correct one is `-final.md` | CC lane | open, minor | 2026-10-26 | Resume SSG-FINISH-1 |
| The asbestos park plan exists only in `~/Downloads` | CC lane | open, minor | 2026-10-25 | Resume RECOVERY-RECON-2 |

## Closed: appears in the sources, do not reopen

| Item | Closed by |
|---|---|
| Backup rc=23 (FA1 A8 #3) | FF2 fix 1 |
| Needs-review visibility, M4(b) (FA1 A7 #2, A8 #4) | FF2 fix 2 |
| A2 silent failures (FA1 A8 #7) | FF2 fixes 3–7 |
| Freeze idea-agent, research-opportunity (FA1 A8 #5). Market chain + tg-monitor stay, ruled | FF2 fix 8 |
| D133 log rotation | FF2 fix 9 |
| Tracked runtime date marker (FA1 A5) | FF2 fix 10 |
| C7 asbestoshq-site progress notes | FF2 fix 13 |
| D132 domain-lab cutover; `LOAD_PROOF_PLACEHOLDER` no longer in the migration note | FA1 F3; checked 2026-11-04 |
| F7 audit path not exercised live | 4 real `pipeline_outcome` rows, latest 11-04 09:10 |
| A-1, V-1 (SSG-FINISH-1 findings) | FA1 F6, F5 |
| 2-per-night deploy cap, 17 guide backport | FLEET-AUDIT-55; SSG-FINISH-1 `cb37e87` |
| A7 #15 `syncrosecure.com` mapping (Mark's OK) and "never re-point to a redirecting domain" | SSG-OVERNIGHT-2, ssg-content `c8e6220`, `7b8d901` |
| syncro-vs-atera stuck in needs-review | SSG-OVERNIGHT-2, live 2026-11-04 (site `3fa32bb`) |
| C47-1, C47-2 writer brief changes | SSG-OVERNIGHT-2, ssg-content `4dbc3dd` |
| D137 source-gate verdicts filed as issues; 260 existing rows resolved | Mark's ruling; agents `711024c`, `1b0b4b8` |
| D140 3-failure guard read a dead table | Mark's ruling; agents `6d55d44`, `b87839d` |
| FLEET-AUDIT-55 night checks: the 23:00 `deploy_capped` row, `live_check` on each ship row, the first DB generation | Seen in the data: audit rows 40436, 40439, 40443; `aiteam-2026-10-26.db` on the backup drive |
| The 5 uncommitted lab dossiers (Resume DOMAIN-LAB-2) | No longer uncommitted, checked 2026-11-05 |
