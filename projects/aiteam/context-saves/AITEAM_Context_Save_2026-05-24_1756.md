# AITEAM Context Save — 2026-05-24_1756
**Generated:** 2026-05-24T17:56:39-0700
**Since last save:** 2026-05-24 17:56:36
**Session topic:** D066 re-audit close + D072 editor truncation-cap fix + asbestoshq.com internal-link audit/fix saga (99-link false-positive incident → revert → corrected dead-target repoint)

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

- **D066 (~/agents half) closed via re-audit, not new work.** `git ls-files --others --exclude-standard` → 0; working tree clean. The D070 hardening `.gitignore` block (`**/.env`, `**/state/`, `**/*.lock`, `**/state.json`) is catching all runtime/secret artifacts. Yesterday's `3631871` auto-commit absorbed prior drift. ~/projects half left to the other chat. Brain commit `22c1a0c`.
- **D072 fix = raise the cap, not build chunk-and-aggregate.** Editor `score.sh` truncated article bodies at 8000 bytes (`head -c 8000`) before scoring. Original D072 plan was to chunk + aggregate (min/mean per axis). Chose the simpler path: raised cap to 60000 bytes via env-overridable `EDITOR_MAX_BODY_BYTES`. Rationale: Sonnet's 200K context handles full guide bodies in one call, so the chunking premise (working around context limits) no longer applies. `TRUNCATED` audit flag stays as a canary. Threshold (3.6) unchanged. Agents commit `deea54d`; brain close commit on DEFERRED.md.
- **asbestos internal-link form is BARE `/<slug>/`, not `/guides/<slug>/`.** The site component `parseInlineLinks` (GuideArticle.tsx) is *designed* to take bare approved-slug paths in the JSON and rewrite them to `/guides/<slug>/` at render time. The JSON should stay bare. This is the load-bearing fact behind the whole revert.
- **Corrected dead-target repoint uses bare→bare mapping.** `/asbestos-abatement/`→`/asbestos-abatement-near-me/`, `/asbestos-inspection/`→`/asbestos-inspection-cost/`, `/asbestos-removal/`→`/asbestos-remediation-cost/`, `/asbestos-testing/`→`/how-to-test-popcorn-ceiling-for-asbestos/`. All targets are in APPROVED_GUIDE_SLUGS so the renderer links them. One self-link in how-to-test collapsed to plain prose. Shipped `c0b6471`, verified live.
- **Reverts via `git revert` (new commits), not history rewrite.** Two bad commits undone safely (`81ab725`, `711748b`), no force-push.
- **D091 (not D087) for the audit_links.sh follow-up.** D087 was already taken (idea-agent follow-ups, sister chat); D090 was the prior highest. Renumbered to D091. Brain commit `c9a69b0`.

## Lessons learned

- **Verify link health against the rendered HTML, not the raw URL inside the markdown.** On a Next.js/React site the component can rewrite paths at render. asbestoshq.com's `parseInlineLinks` rewrites bare `/<slug>/`→`/guides/<slug>/` when the slug is in `APPROVED_GUIDE_SLUGS`, else strips to plain text. I curled bare URLs, got 404s, and "fixed" 99 links that were never broken — the prefix I added broke the render-time whitelist match and stripped 99 working links to plain text in production. Read the render code before touching link formatting.
- **Don't assume HTML attribute order when grepping rendered output.** Body links render as `<a class="guide-inline-link" href="...">` (class-then-href). My verification greps assumed `href`-then-`class` and returned zero matches three separate times, each looking like a failure. Dump the raw tag (`grep -oE '<a[^>]*>'`) before trusting a count.
- **A clean tree on re-audit is a valid, complete deliverable.** D066's ~/agents half needed no commits — the prior gitignore hardening already did the job. "Nothing to do, here's the proof" closes an item.
- **Confirm the schema before querying.** `audit_log` columns are `action` / `payload_json`, not `event` / `payload`. A prompt-supplied query failed; checked `.schema` and adjusted.
- **Pre-flight D-numbers immediately before writing.** The prompt said "add D087" but D087 was taken. Grepping `D0[0-9]{2}` first caught it; used D091. (This is a recurring multi-chat race — already a documented gotcha.)
- **Vercel CDN edge cache masks deploy verification.** `x-vercel-cache: HIT` with rising `age`, and query-string cache-busting doesn't work on statically-generated pages. Polling for a content change is noisy; the deploy eventually propagates. Don't claim a live render you can't observe — distinguish "pushed + code correct" from "confirmed rendered."

## Operator corrections

- **The operator's process caught my error, not a single verbal correction.** The step-by-step dry-run → review → apply → verify → commit cadence (operator gates every mutation) is what surfaced the 99-link regression: when I reported the repoint, the operator asked for a render-path check, which is when I read `parseInlineLinks` and found the bug. Takeaway: on this operator's lanes, the dry-run-first / verify-on-real-output discipline is non-negotiable and it works — keep proposing dry-runs and reading the actual render path before claiming a fix.
- The operator chose the adjusted testing mapping (`/asbestos-testing/`→`how-to-test-popcorn-ceiling-for-asbestos` instead of doubling up on `asbestos-inspection-cost`) after I flagged the double-link-to-same-target redundancy. Confirmed the flag was worth raising.

## What's next

- **Immediate:** asbestos internal-link lane is DONE and verified live. No open action on it. The follow-up (audit_links.sh body-paragraph + render-path blindness) is captured as **D091**, triggered to be fixed during the D083 SSG-fy pass.
- **Carried from sister/market chats (still live, not mine):** operator-side smoke of the three new agents (idea-agent, notes-agent, /model picker); first market-pipeline cron fire; Sunday 09:00/10:00 first fires. See preserved HANDOFF sections.
- **Editor:** D072 closed, but threshold (3.6) was tuned against head-truncated scores — a fresh burn-in on full-body scores may warrant re-calibration (noted in the D072 closure). D073 (rubric redundancy) and D074 (live-gate verification) remain open.
- **Blocked:** nothing in this lane.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-24_1756.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-24 09:40 (parallel sister-chat session: fleet health-check + OAuth-from-cron fix + market pipeline D033 activation) **Last session summary (sister chat — three new agents):** Built three new agents end-to-end. (1) idea-agent v1 — weekly Sunday self-improvement loop, 7-phase chain, smoke produced 5 Track-B ideas at composite 3.8-4.2. (2) /model slash-command picker for 6 agents (operator pivoted from inline-keyboard after I flagged grammy lacked callback infrastructure). (3) notes-agent v1 — Telegram capture (text + image + forwards) → Haiku categorize → weekly Sonnet sort. Phase 2.6 added forward capture with /yes/no/yes_<cat> confirmation flow. Bot.js gained 14 new handlers/commands. Five new SQLite tables (`idea_proposals`, `idea_calibration`, `zombie_resurface_log`, `model_overrides`, `pending_forwards`). One-line `bot-launch.sh` `cd $(dirname)` fix shipped to recover from a manual-restart CWD crash. Bot restarted 4× in session, now under D026 launchd `KeepAlive` (closed by sister chat at `07925f8`). **This chat (fleet health-check + cron-auth fix + D033):** Diagnosed silent rc=1 failure of `claude -p` from cron — root cause: OAuth credentials not loaded in non-interactive context; symptom: tg-monitor 07:00 digest, write_diary.sh, and assignment-drafter Sonnet calls all silently broken since 2026-05-17. Switched cron auth to long-lived `CLAUDE_CODE_OAUTH_TOKEN` via `claude setup-token` (sourced from `~/.claude_oauth_token`, outside repo). Fixed 2 env-leak paths: `check_kill_switches.sh` was bulk-exporting every .env var via `set -a; source; set +a` (leaked DASHBOARD_TOKEN/SERPER_API_KEY/YOUTUBE_API_KEY to every child); `analyzer.py` had `subprocess.run(env=os.environ.copy())` carrying empty `ANTHROPIC_API_KEY` from dotenv. Restored grammy bot under launchd KeepAlive (sister chat used this for D026 closure). Wrote missing `market/scribe/process_queue.sh` batch wrapper (~120 lines, drains .job queue with 3-attempt retry + workspace/failed/ abandon). Activated **D033** — market pipeline 4-stage cron at 07:00/07:15/08:15/08:45 PT. Extended watchdog with new `launchctl_label` signal type + 4 market entries (12 total now, all healthy after grammy-orphan recovery). Commits: agents `da88d09 / 07925f8 / b11a439 / 23ccd52`, brain `4c5d30c`.  
