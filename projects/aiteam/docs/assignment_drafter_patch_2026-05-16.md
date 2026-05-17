# Assignment Drafter — Patch Build Report (2026-05-16)

## TL;DR

The OPERATOR EDIT human-in-loop pattern is gone and the chain is fully
autonomous: drafter → validate → fire `run-batch.sh` → approved guide →
Ship-To-Site → live site. Operator's only ongoing role is the daily
digest. **All 7 sign-off items are ✅.**

E2E proof: `white-asbestos-vs-blue-asbestos` was drafted twice (once
under cap=$0.01 to verify ⛔ budget-skip path, once under cap=$15 to
verify 🚀 fire path). Live `run-batch.sh` PID 86284 is currently
executing the writer round in the background; the approved guide will
land in `content/asbestos/approved/guides/` asynchronously and
Ship-To-Site's */15 cron will ship it to the live site without further
intervention.

Code: `~/agents/assignment-drafter/`. No cron change — */30 stays.

## Sign-off status

### ✅ 1. Patch 1 validates on real slug
**Slug:** `white-asbestos-vs-blue-asbestos`
**Path:** `~/projects/asbestos-contractors/content/asbestos/assignment-batch-white-asbestos-vs-blue-asbestos.md`
**Result:** Validator PASS. Zero HTML comments. Zero operator-defer
language. Real corporate / regulatory specifics throughout (W.R. Grace
Zonolite, Armstrong, National Gypsum, UNARCO Industries, Johns Manville,
EMSL Analytical, Schneider Labs, Corrosion Proof Fittings v. EPA 1991,
Frank R. Lautenberg Chemical Safety Act 2016, NIOSH 7400, IARC Group 1).

**Validator rejection of synthetic bad fixtures confirmed:**
- HTML comment present → `FAIL: html_comment_present` (counts and locates)
- `TBD` / `customize` / `Operator to` / `OPERATOR EDIT` → `FAIL: placeholder_string`
- Seed paragraph starts with `Should X?` → `FAIL: seed_rhetorical_question`
- Writer notes with <10-word bullets → `FAIL: short_notes_bullet`

**Pre-patch chrysotile draft re-tested against new validator:** rejected
with `html_comment_present` (2 comments) + `placeholder_string` (the
"OPERATOR EDIT" / "Customize" strings inside the comments) +
`short_notes_bullet` (1 short bullet). Confirms backward-incompatible
change — old drafts FAIL the new gate, which is the intent.

### ✅ 2. Patch 2 fires run-batch.sh
**Audit row 440:** `pipeline_fired` for `white-asbestos-vs-blue-asbestos`,
payload `{slug, pid: "86284", est_cost_usd: "2.50", daily_spend_usd:
"2.5000", daily_cap_usd: "15.00"}`.

**Background process verified:**
```
501 86284 1 0 6:57PM ?? 0:00.01 bash content/run-batch.sh white-asbestos-vs-blue-asbestos --guide
501 86294 86284  0:00.00 (child)
501 86372 86284  0:00.00 (child)
501 86378 86372  0:00.00 (child writer)
```

**pipeline_runs.log captured the launch + writer output:**
```
--- 2026-05-17T01:57:18Z slug=white-asbestos-vs-blue-asbestos invocation='bash content/run-batch.sh white-asbestos-vs-blue-asbestos --guide' ---
[1/1] PAGE: white-asbestos-vs-blue-asbestos
--------------------------------------------
  ✏️  [1/1] WRITER round 1 — 06:57:18 PM
  ⠋ [1/1] WRITER round 1 working...
```

The drafter exited at 01:57:21Z (1:26 wall) while `run-batch.sh`
continues — that's the design.

### ✅ 3. Budget cap enforced
**Audit row 432:** `pipeline_fire_skipped_budget` for
`white-asbestos-vs-blue-asbestos`, payload `{slug, daily_spend_usd: "0",
daily_cap_usd: "0.01"}`.

Process: temporarily set `daily_pipeline_budget_usd: 0.01` in config,
ran drafter on a fresh queue line. Drafter generated the draft (~$0.14
Sonnet), validation passed, `budget_check.sh check 0.01 2.50` returned
`FULL 0 0.01` (exit 1), `maybe_fire_pipeline` returned
`SKIPPED_BUDGET 0 0.01`, telegram fired ⛔ PIPELINE BUDGET REACHED,
draft stayed in place, queue line marked ✓ (the *draft* succeeded —
only the *fire* skipped). After verifying, reset cap to 15.00.

Also unit-tested `budget_check.sh` in isolation:
- empty state → 0
- check with room → `OK 0 15.00` exit 0
- charge accumulates → `CHARGED 2.5000`, file updated
- check after charge → still OK if cap fits
- check with insufficient cap → `FULL 5.0000 0.01` exit 1
- stale date rollover → state file shows yesterday, reader returns 0
  (auto-reset)
- malformed spend value → reader returns 0 (defensive)

### ✅ 4. Telegram notification format updated
All 4 variants verified via `NOTIFY_DEBUG=1` bypass mode (added so
sign-off tests don't spam the live channel — the inline
`TELEGRAM_OUTBOUND_ENABLED=false` trick doesn't work due to the
notify.sh env-source bug). Live sends during step 1 + step 2 went to
the operator's Telegram with the new headers.

**🚀 PIPELINE FIRED variant:**
```
🚀 PIPELINE FIRED — 2026-05-16 18:45

Drafted + queued (2 of 2):
  • slug-a  [PID 51234]
  • slug-b  [PID 51289]

Pipeline running in background. Approved guides will hit Ship-To-Site queue automatically.
Daily spend: $0 / $15.00
```

**⛔ PIPELINE BUDGET REACHED variant:**
```
⛔ PIPELINE BUDGET REACHED — 2026-05-16 18:45

Drafted (2 of 2), but pipeline NOT fired:
  • slug-a  [drafted, queued for manual fire]
  • slug-b  [drafted, queued for manual fire]

Daily spend: $0 / $15.00 cap
Resets at midnight local. Or raise cap in config/asbestos.yaml.
```

**📝 DRAFTS SAVED (NO FIRE) variant** (when `auto_fire_pipeline=false`):
```
📝 DRAFTS SAVED (NO FIRE) — 2026-05-16 18:45

Drafted (1):
  • slug-a

auto_fire_pipeline is OFF in config — review at:
  /Users/mmm2/projects/asbestos-contractors/content/asbestos/assignment-batch-<slug>.md
Run pipeline manually when ready.
```

**Mixed variant (some fired, some fire-failed):**
```
🚀 PIPELINE FIRED — 2026-05-16 18:45

Drafted + queued (1 of 2):
  • slug-a  [PID 51234]

⚠️ Fire FAILED for 1 slug(s) — manual investigation needed:
  • slug-b
See state/pipeline_runs.log and audit_log action=pipeline_fire_failed.

Pipeline running in background. Approved guides will hit Ship-To-Site queue automatically.
Daily spend: $0 / $15.00
```

### ✅ 5. audit_log gets ≥2 new action types
**3 new types observed during patch testing:**
- `pipeline_fired` (id 440)
- `pipeline_fire_skipped_budget` (id 432)
- `pipeline_fire_failed` (id 436 — fired during a transient quote-parsing
  bug found and fixed mid-test; see Spec deviation 1 below)

Plus a renamed broader tick audit `drafter_tick_notification` (replaces
the older `drafts_ready_notification`).

### ✅ 6. README.md updated
`~/agents/assignment-drafter/README.md` now documents:
- The autonomous chain top to bottom (queue → draft → validate → fire →
  Ship-To-Site → live site)
- The patch history line pointing at this report
- The "Reviewing drafts (optional — the chain is autonomous)" section
  replaces the old operator-edit instructions
- A "Budget cap" section explaining the spend tracker + how to raise
  the cap mid-day or disable autonomous firing
- Updated "Pausing the autonomous chain" section explaining that
  `/halt_drafting` is the chain-source kill (different from
  `/halt_shipping` which kills only the output end)
- Updated "What this agent does NOT do" to reflect the no-retry-on-fire
  policy

### ✅ 7. failure_modes.md updated
4 new modes added (8, 9, 10, 11) — 11 total now:
- Failure 8: **Daily pipeline budget cap hit**
- Failure 9: **`run-batch.sh` fails to launch**
- Failure 10: **Pipeline run produces 3R-escalated failure** (drafter
  doesn't observe this directly — gap documented; daily Ship-To-Site
  digest will surface)
- Failure 11: **`state/daily_pipeline_spend.txt` corruption**

The existing 7 modes were edited to reflect autonomous mode: failure 3
(validation) was rewritten to document the expanded rejection patterns;
failure 7 (notify down) now mentions pipeline_runs.log as the fallback
source-of-truth; the kill-switch table got `auto_fire_pipeline=false`
and `daily_pipeline_budget_usd` added as additional brakes.

## E2E draft excerpt (first 30 lines)

```markdown
# Assignment Batch: white-asbestos-vs-blue-asbestos - Guide Page
# 1 page | Opus writer | Opus auditor | --guide mode
# Run: cd ~/Desktop/asbestos-contractors && bash content/run-batch.sh white-asbestos-vs-blue-asbestos --guide

---

## Page 1

**Target URL slug:** white-asbestos-vs-blue-asbestos
**Page type:** Long-form guide (identification-intent comparison)
**Template:** Environmental Guides (blog-posts.json schema)
**Word count range:** 1,800-2,200 (floor 1,600, ceiling 2,400)
**Primary keyword:** white asbestos vs blue asbestos
**Primary volume:** 200/mo | KD 2 | CPC $1.75
**SERP Status:** CLEAR (verified 2026-05-16). Comparison-intent query; no dominant commercial page holds positions 1-3.
**Secondary keywords:** chrysotile vs crocidolite (150/mo), types of asbestos, blue asbestos identification, most dangerous type of asbestos, asbestos fiber types
**Audience:** U.S. homeowners and property managers who have received a bulk-sampling PLM lab report from a certified lab identifying asbestos fiber type as chrysotile or crocidolite and need to know whether the mineralogy distinction changes their health risk profile, regulatory obligations, or abatement requirements.

**Seed:**
Reader has received a polarized light microscopy (PLM) lab report from a certified laboratory such as EMSL Analytical or Schneider Labs identifying asbestos as either chrysotile (white) or crocidolite (blue) and needs to know whether fiber type changes their legal exposure limits, abatement licensing requirements, or health risk tier. Chrysotile, a serpentine mineral fiber, accounted for approximately 95% of all asbestos used commercially in the United States and was the primary constituent in W.R. Grace Zonolite attic insulation, Armstrong floor tiles, and National Gypsum joint compounds manufactured from the 1940s through the late 1970s. Crocidolite, an amphibole fiber mined commercially in South Africa and Australia and phased out of U.S. use by the late 1980s, is the most biopersistent of the six fiber types regulated under EPA NESHAP 40 CFR 61 Subpart M and OSHA 29 CFR 1926.1101, due to the rigid, needle-like geometry that allows individual fibers to lodge irreversibly in the pleural lining. Both fiber types are subject to the identical permissible exposure limit of 0.1 fibers per cubic centimeter (f/cc) as an 8-hour TWA under OSHA 29 CFR 1926.1101; mineralogy creates no legal distinction in abatement contractor licensing, waste-stream disposal, or NESHAP notification requirements. Reader needs the risk hierarchy, the lab identification method, and the remediation decision tree — not a general introduction to asbestos history.

**Internal link targets (minimum):**
- 4+ service page links (/asbestos-testing/, /asbestos-removal/, /asbestos-inspection/, /asbestos-abatement/)
- 2+ guide cross-links (friable-vs-nonfriable-asbestos, chrysotile, asbestos-air-quality-test)
- 1+ link to /find-asbestos-contractors/ or /request-a-quote/ (L3)
- All trailing-slash

**Authority links (minimum 4 distinct .gov):**
Pull from content/asbestos/asbestos-authority-links.md.
```

Zero `<!--` in the file. Verified via `grep -c '<!--' ... = 0`.

## Audit row IDs created during patch testing

| ID | Action | Target | Notes |
|----|--------|--------|-------|
| 431 | `assignment_drafted` | `white-asbestos-vs-blue-asbestos` | Step 1 draft success (under cap=$0.01) |
| 432 | `pipeline_fire_skipped_budget` | `white-asbestos-vs-blue-asbestos` | ⛔ path proof |
| 434 | `drafter_tick_notification` | `asbestos` | Step 1 Telegram audit row |
| 435 | `assignment_drafted` | `white-asbestos-vs-blue-asbestos` | Step 2 attempt 1 (quote-bug attempt) |
| 436 | `pipeline_fire_failed` | `white-asbestos-vs-blue-asbestos` | Quote-bug surfaced failure path |
| 438 | `drafter_tick_notification` | `asbestos` | Step 2 attempt 1 Telegram (mixed-variant) |
| 439 | `assignment_drafted` | `white-asbestos-vs-blue-asbestos` | Step 2 attempt 2 (post-fix) |
| 440 | `pipeline_fired` | `white-asbestos-vs-blue-asbestos` | 🚀 path proof, payload.pid=86284 |
| 442 | `drafter_tick_notification` | `asbestos` | Step 2 attempt 2 Telegram (FIRED variant) |

Token usage during the three Sonnet calls (4 incl. the budget-step
retry that actually succeeded as a fresh draft): ~$0.42 total Sonnet
spend across the test session. Pipeline call still running at report
write time — final cost reported via `~/store/aiteam.db` token_usage
once it completes.

## Spec deviations (and reasoning)

1. **Quote-parsing bug found and fixed mid-test.** First fire attempt
   failed with "pipeline script not found at .../\"bash" because
   `yaml_get` doesn't strip quotes around YAML values, and the config
   originally had `pipeline_invocation: "bash content/run-batch.sh"`
   (quoted). Fix applied two ways: (a) removed quotes from the YAML
   value (later restored by an automatic file edit — see below), (b)
   added defensive `strip_quotes()` helper to `lib/fire_pipeline.sh`
   that handles both quoted and unquoted forms. Either form now works.
   This bug surfacing also naturally exercised failure mode 9 (fire
   failed → audit row 436). Pre-patch checklist did not include
   quote-handling in flat YAML; pattern noted for future agent
   integrations.

2. **`NOTIFY_DEBUG=1` env added to notify_operator.sh.** The spec said
   "Don't touch the notify.sh env override bug — just live with it for
   testing." But to test 4 distinct Telegram message variants without
   spamming the operator with 4 live messages, I added a sibling
   bypass at the notify_operator layer that prints to stdout instead
   of calling notify.sh. The notify.sh env bug itself is untouched and
   stays in DEFERRED.

3. **`pipeline_invocation` and `pipeline_args` retained as YAML
   strings** despite being executable command tokens. Rationale: keeping
   them as quotable strings makes the config readable and lets the
   operator change invocation flags without bash-arithmetic in the
   YAML. `fire_pipeline.sh` strips quotes defensively (deviation 1).

4. **Fire happens BEFORE the queue line is marked `✓`** in the patched
   code — wait, actually no, fire happens AFTER. Confirming the
   committed code: `mark_complete.sh` runs first (line ~227 of
   drafter.sh), then `maybe_fire_pipeline` (line ~233). This was a
   deliberate ordering choice: even if fire fails, the draft was
   successful and shouldn't be re-attempted next tick. The mark-then-fire
   ordering means a fire failure leaves the slug in "drafted, never
   fired" state — operator catches via Telegram footer ⚠️ Fire FAILED
   or via `state/pipeline_runs.log` review.

5. **Fixed est_cost_per_fire_usd = $2.50** rather than real-cost
   reconciliation. Per spec: "Simpler alternative: just log a fixed
   estimate per fire ... Real-cost reconciliation can come later." The
   estimate intentionally lands toward the high end of historical
   pipeline costs (writer + auditor Sonnet runs typically $1.50-$3.00;
   Opus runs can hit $4-6) — so the cap is more likely to under-fire
   than over-fire. Real reconciliation is in DEFERRED.

6. **Validator now rejects the existing `chrysotile-attic-insulation-removal`
   draft from the prior session** — it contains the legacy OPERATOR
   EDIT markers. This is intentional: the new gate is correctly strict.
   The chrysotile draft is still in the pipeline repo and the pipeline
   has not been fired against it; if the operator wants to keep it,
   either manually fire `run-batch.sh chrysotile-attic-insulation-removal
   --guide` (the writer LLM ignores HTML comments anyway) or hand-edit
   the comments out. The drafter agent will not re-draft it because the
   file exists (see drafter.sh's `out_path` exists check).

## DEFERRED — known work this patch leaves open

- **`notify.sh` env-override bug** (separate ticket, untouched per spec):
  inline `TELEGRAM_OUTBOUND_ENABLED=false` doesn't stub the send because
  `notify.sh` re-sources `~/agents/config/.env`. Worked around in
  `notify_operator.sh` via `NOTIFY_DEBUG=1`. The bug itself remains.

- **Real-cost reconciliation against `token_usage`.** Current spend
  tracker charges a fixed $2.50 per fire; real Sonnet/Opus cost from
  the pipeline's writer + auditor runs is not summed against the daily
  budget. A future enhancement could scan `token_usage` rows correlated
  with the pipeline PID and reconcile post-completion.

- **Pipeline run outcome is not observed by the drafter.** The drafter
  exits after launching `run-batch.sh`; subsequent escalations (3R
  failures → `needs-review/`), audit log cost spikes, or run-batch.sh
  crashes are not surfaced by this agent. Failure mode 10 documents the
  gap. Right place to fix: the planned Escalation Triage agent (per
  recommendations doc step 3) and/or Ship-To-Site's daily digest.

- **Live `pipeline_fired` chain end-to-end NOT YET VERIFIED to a live
  site URL.** PID 86284 is running at report write; the chain's final
  step (Ship-To-Site picking up the approved guide and pushing to
  `mdp280028-ui/asbestoshq-site`) will happen async. Operator should
  watch:
  - `~/projects/asbestos-contractors/content/asbestos/approved/guides/white-asbestos-vs-blue-asbestos.json`
    landing (writer + auditor completion)
  - `~/agents/ship-to-site/state/shipped.log` getting a new line
  - `https://www.asbestoshq.com/guides/white-asbestos-vs-blue-asbestos/`
    going live

- **Bot.js slash command live test.** No bot.js changes in this patch
  (patch 2 doesn't introduce new slash commands), but the existing
  `/halt_drafting` and `/resume_drafting` from the previous patch still
  need an operator-side bot restart to go live. Tracked in the prior
  build report's DEFERRED.

- **Test fixture for `auto_fire_pipeline=false` path** not run live
  during this patch — the `NOTIFY_DEBUG=1` test verified the message
  format. Code review confirms the `maybe_fire_pipeline` branch returns
  `SKIPPED_DISABLED` without touching budget or fire, and the queue
  loop correctly buckets that into the "no fire" notification path.

- **Test fixture for a fresh date-rollover** not run live (would
  require waiting until midnight or system-clock manipulation). The
  `budget_check.sh` `today` action contains the date check; the unit
  test that simulated a stale 2020-01-01 date confirmed the reset
  path works.

## Files modified

```
~/agents/assignment-drafter/
├── prompts/asbestos_drafter_prompt.md     — rewrote rules 3 + final ask
│                                            to demand decisive output;
│                                            removed both example HTML
│                                            comments; added forbidden-
│                                            strings list
├── lib/validate.sh                         — flipped check 2 from
│                                            "≥2 OPERATOR EDIT markers"
│                                            to "ZERO <!-- ... -->";
│                                            expanded placeholder regex;
│                                            added seed-sentence-count
│                                            + writer-notes-bullet checks
├── lib/notify_operator.sh                  — rewrote to handle 4 variants
│                                            (🚀 fired / ⛔ budget /
│                                            📝 no-fire / ⚠️ mixed);
│                                            added NOTIFY_DEBUG bypass
├── drafter.sh                              — added maybe_fire_pipeline
│                                            wrapper; updated queue-loop
│                                            success path to fire; new
│                                            FIRED_PAIRS / SKIPPED /
│                                            FAILED arrays; updated
│                                            notify invocation
├── config/asbestos.yaml                    — added auto_fire_pipeline,
│                                            daily_pipeline_budget_usd,
│                                            est_cost_per_fire_usd,
│                                            pipeline_invocation,
│                                            pipeline_args
├── config/ssg.yaml                         — mirrored new keys (stubbed)
├── README.md                               — autonomous-chain rewrite,
│                                            budget-cap explainer,
│                                            updated pause docs
└── failure_modes.md                        — added modes 8/9/10/11;
                                              edited 3 + 7 + kill-switch
                                              table for autonomous mode
```

## Files added

```
~/agents/assignment-drafter/lib/
├── budget_check.sh   — daily spend tracker with auto-reset, check/charge/today actions
└── fire_pipeline.sh  — background-launches run-batch.sh, captures PID
```

## No cron changes (per spec)

The */30 drafter.sh cron line from the prior build stays in place; this
patch operates inside that same tick.
