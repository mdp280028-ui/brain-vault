# AITEAM Project Handoff
**Last updated:** 2026-05-24 17:58 (THREE parallel chats: internal-link lane [D066/D072/link-audit] + asbestos-cleanup lane [recon/push/D090/gitignore] + agent-build lane [domain-hunter/idea-agent-synth/Issues-Queue])

**This chat (agent-build lane):** Built **domain-hunter** v0.1 end-to-end (expireddomains/godaddy fetchers → no-LLM hard-filter → brandability + Haiku use-case tags → Wayback classify → Telegram digest; cron live 06:30; godaddy edge-blocked, graceful) then **refit to v0.1.5** — Haiku topical-fit gating against 3 sites (ust/asbestos/ssg), surface only `topical_fit_max ≥ 0.3`, brandability demoted to secondary, Ahrefs deep link in digest. Diagnosed + fixed **idea-agent synthesize** (Sonnet rambled 34K tokens / narrated multi-fence; brittle parser): hardened single-JSON prompt + 3-tier parser (exit 4) + retry-once; **removed the budget cap** after a live run proved healthy synth costs $0.35–0.45 (not ~$0.10); cadence **weekly→daily 07:10**. Built the **Issues Queue**: `agent_issues` table + `*/5` capture daemon (fresh-start anchored at audit_log.id 2991, no backfill, `new_issues.log`) + dashboard `/issues` (filters, status/notes, one-click CC-prompt modal). 3 crons installed live (domain-hunter 06:30, idea-agent 07:10, issues-capture */5). Commits: domain-hunter `ee9e34c…044d619`, idea-agent `ca21007…ce46867`, issues `bdc1909…0d099ab`.

**Parallel chat (asbestos-cleanup):** Read-only recon confirmed 4 planned items (GEO Phase 3, D061, D052, dead-code cleanup) were **all already shipped** — no build; brain `DEFERRED.md` confirmed all CLOSED. Reconciled dirty cc-context tracking in `asbestos-contractors` (was an unfinished 09:40 ctx-save snapshot, already correct — not stale). Found + handled a bug: `approved-index-asbestos.md` falsely listed `asbestos-encapsulation-vs-removal` as shipped while its JSON sat in `needs-review/` (failed `audit_guide.py` rounds 2+3) — reverted the false index hunk, committed the rest. **Pushed 15 local-only commits** (D045 Path B, lockfile, D061, D062, dead-code cleanup, GEO Phase 3, + 2 this chat) to `origin/main` HEAD `021fb93` (drive-failure protection; no autocommit on this repo). Logged **D090** (index-drift investigation). Settled `needs-review/` gitignore policy. Commits: asbestos `b645c71 / 4c62966 / 021fb93`, brain `ce93fd7`. Audit id=2893, 2895. ctx narrative: `context-saves/AITEAM_Context_Save_2026-05-24_1756_asbestos-cleanup.md`.

**This chat (internal-link lane + two D-item closes):** Closed **D066** ~/agents half via re-audit (clean tree, 0 untracked-non-gitignored, D070 gitignore hardening doing its job; brain `22c1a0c`). Closed **D072** — editor `score.sh` truncation cap raised 8KB→60KB via env-overridable `EDITOR_MAX_BODY_BYTES` (simpler than the planned chunk-and-aggregate; Sonnet's 200K context makes chunking moot; agents `deea54d` + brain close). Then the big one: **asbestoshq.com internal-link audit**. Recon found the internal-link v1 agent already exists (asbestos-only) and that all guide-body links use bare `/<slug>/` form. I MISDIAGNOSED bare-URL 404s (from `curl`) as broken links and shipped two bad commits (`36cb356` prefixing `/guides/` into 99 body links; `cd03847` repointing 44 dead-targets with `/guides/` prefix). Reading `parseInlineLinks` in `GuideArticle.tsx` revealed the component rewrites bare approved-slug paths to `/guides/<slug>/` at render — my prefix broke the whitelist match and STRIPPED 99 working links to plain text in production. Reverted both (`81ab725`, `711748b`), then shipped the corrected dead-target repoint in bare→bare form (`c0b6471`), verified live (6/6/7/9 body links render on the four sampled pages; 0 dead-targets; 4 repoint targets resolve 200). Logged the lesson (brain `80469d4`) + **D091** for the audit_links.sh body-paragraph/render-path blindness (brain `c9a69b0`).

**Prior session summary (sister chat — three new agents):** Built three new agents end-to-end. (1) idea-agent v1 — weekly Sunday self-improvement loop, 7-phase chain, smoke produced 5 Track-B ideas at composite 3.8-4.2. (2) /model slash-command picker for 6 agents. (3) notes-agent v1 — Telegram capture (text + image + forwards) → Haiku categorize → weekly Sonnet sort, Phase 2.6 forward capture with /yes/no confirmation. Five new SQLite tables. Bot under D026 launchd `KeepAlive` (`07925f8`).

**Prior session summary (market/cron-auth chat):** Diagnosed silent rc=1 of `claude -p` from cron (OAuth not loaded non-interactively) — switched to long-lived `CLAUDE_CODE_OAUTH_TOKEN`. Fixed 2 env-leak paths (`check_kill_switches.sh` bulk-export, `analyzer.py` empty-key subprocess). Activated **D033** market pipeline cron (07:00/07:15/08:15/08:45 PT). Watchdog extended with `launchctl_label` probe + 4 market entries. Commits: agents `da88d09 / 07925f8 / b11a439 / 23ccd52`, brain `4c5d30c`.

---

## Current phase

**Asbestos internal-link corpus clean + verified; three new agents still awaiting first scheduled fires + operator smoke**

- **asbestoshq.com internal links** — DONE. All 32 guides' body links resolve to live `/guides/<slug>/` pages via the render-time rewrite; 44 former dead-targets now point at live approved guides. Verified live.
- **idea-agent / notes-agent / /model picker** — live, awaiting first Sunday cron fires + operator-side smoke (carried from sister chat, unchanged).
- **market pipeline** — first cron fire was 2026-05-24 07:00 PT; brief expected on phone ~08:46 PT (carried from market chat).

## Completed (cumulative)

- [x] **D066 ~/agents half** — re-audit confirmed clean tree, no untracked code, no modifications. Closed in DEFERRED.md (`22c1a0c`).
- [x] **D072 editor truncation cap** — `score.sh` body cap 8KB→60KB, env-overridable `EDITOR_MAX_BODY_BYTES`, `TRUNCATED` flag retained as canary (`deea54d` + brain close).
- [x] **asbestoshq.com internal-link audit + fix** — reverted the two bad-form commits; corrected 44 dead-target links to bare approved slugs (`c0b6471`). Net: corpus is in a better state than session start (dead-targets now link live), zero regression remaining.
- [x] **asbestos pipeline pushed to origin** (parallel chat) — 15 local-only commits incl. D045 Path B, POLICY-Q5 lockfile, D061, D062, dead-code cleanup, GEO Phase 3 → `origin/main` HEAD `021fb93`. Working tree clean.
- [x] **D090 logged + needs-review/ gitignore policy** (parallel chat) — index-drift investigation deferred (`ce93fd7`); `needs-review/*.json` now ignored as operator-triage state (`021fb93`).
- [x] **domain-hunter v0.1 + v0.1.5** (agent-build lane) — `~/agents/domain-hunter/`. Topical-fit gate against ust/asbestos/ssg; cron 06:30. All 4 v0.1 brandability-junk domains → `best_site_match='none'`. godaddy edge-blocked (graceful).
- [x] **idea-agent synthesize hardening** (agent-build lane) — single-JSON prompt + 3-tier parser + retry-once; no cost cap (prompt+parser+retry is the guard); `AI_DO_MAX_BUDGET_USD` passthrough added to `ai-do.sh`; cadence daily 07:10.
- [x] **Issues Queue** (agent-build lane) — `agent_issues` table; `issues-capture/capture.sh` (`*/5`, fresh-start, `new_issues.log`); dashboard `/issues` view + data feed + status/notes/generate-prompt routes + CC-prompt modal + home counter.
- [x] **Prior:** idea-agent v1, /model picker, notes-agent v1 (+ forward capture), cron OAuth fix, env-leak fixes, market pipeline D033, watchdog launchctl probe, Keyword Registry, Research/Opportunity, Editor + verdict persistence, GEO Optimizer, Internal Link Agent v1, SSG pipeline (Lane B+C), Backlink Prospector, Market scribe/analyst/briefer/curator, Telegram bot, Watchdog, Orchestrator, Ship-to-site.

## In progress

- [ ] **Operator-side live smoke of the three new agents** (carried, sister chat):
  - `/model` grid render + one toggle round-trip (set opus → verify token_usage → set off → verify sonnet)
  - Send text + image notes; forward a message; /yes a forward; /no a forward; forward with photo
  - Spot-check the 5 idea-agent proposals in `~/brain/projects/aiteam/ideas/weekly_2026-05-23.md`, mark `operator_action` in `idea_proposals`
- [ ] **First production fire of market pipeline cron** (carried, market chat) — investigation trail in `23ccd52` body if nothing on phone by 09:00 PT.
- [ ] **Watchdog grammy-bot recovery message** — expected missing→healthy transition tick, not noise.
- [ ] **Watch agent-build first fires** — domain-hunter 06:30 (expect "0 candidates" until a topical domain appears), idea-agent daily 07:10 (confirm synth produces clean JSON, no `weekly_run_failed`), issues-capture `*/5`. Plus operator browser-smoke of `/issues` (Generate CC Prompt → clipboard paste) + tap-test domain-hunter's Ahrefs deep link in a real digest.

## Blocked

- none

## Known bugs

| Bug | Severity | File(s) | Notes |
|---|---|---|---|
| Synthetic test forward entries in `inbox_2026-05-24.md` | minor | `notes/inbox_2026-05-24.md` | Two `[forward]` entries with fake senders from sister-chat smoke. Delete before Sunday sort. |
| D044 `log_to_audit.sh` apostrophe escape | live workaround in place | `~/agents/lib/log_to_audit.sh` | APOS pattern works; parameterized-insert rewrite still deferred. |
| Photo album `media_group_id` not in inbox markdown | minor (v1.1) | `notes-agent/capture.sh`, `bot.js` | Tracked in D089. |
| `pending_forwards` rows never deleted | future risk | schema | Forensic record by design. Hard cap if >1000. D089. |
| internal-link `audit_links.sh` blind to body-paragraph links + render-path | latent | `~/agents/internal-link/audit_links.sh` | Only checks `relatedLinks[]`, not `paragraphs[]`; doesn't mirror render-time path rewrite. Tracked in **D091**. |
| approved-index drift on escalated slugs | medium | `content/run-batch.sh`, `approved-index-asbestos.md` | Index claimed a needs-review/ slug shipped. Bad line reverted; root cause open as **D090**. Check before next batch. |
| `asbestos-encapsulation-vs-removal` stuck in needs-review/ | minor | `content/asbestos/needs-review/` (now gitignored) | Failed audit_guide rounds 2+3 (`-r2.md`/`-r3.md`). Operator: hand-recover, re-run, or drop. |
| `/issues` CC-prompt log-path/`cd` guess assumes top-level `~/agents/<agent>` | minor (v1.1) | `issues-capture/capture.sh`, `dashboard/server.js` | Nested agents (e.g. `market/scribe`) show "no log file found" + wrong `cd`. Operator corrects in the generated prompt. |
| domain-hunter godaddy fetcher non-producing | by design | `domain-hunter/fetch/godaddy.py` | Akamai 403 on all GoDaddy auction URLs; detects edge-block, exits 4, run continues on expireddomains. |
| `/issues` clipboard copy unverified headlessly | unknown | `dashboard/server.js` | navigator.clipboard + execCommand fallback shipped; needs an operator browser tap to confirm. |

## Architecture decisions (recent)

| Decision | Reasoning | Date |
|---|---|---|
| asbestos guide-body links stay BARE `/<slug>/` in JSON | `parseInlineLinks` rewrites bare approved-slug paths → `/guides/<slug>/` at render. Prefixing in JSON breaks the whitelist match and strips the link. | 2026-05-24 |
| Dead-target repoint maps to bare approved slugs | So the existing render-time rewrite links them; targets must be in APPROVED_GUIDE_SLUGS | 2026-05-24 |
| D072 fix = raise `head -c` cap, not chunk-and-aggregate | Sonnet 200K context handles full bodies; chunking premise obsolete. Env-overridable `EDITOR_MAX_BODY_BYTES`, default 60000 | 2026-05-24 |
| Undo bad commits with `git revert` (new commits) | Non-destructive, no force-push; preserves the incident in history | 2026-05-24 |
| domain-hunter shipped as v0.1.5 topical-fit, NOT the v0.2 weak/strong-301 spec | The v0.2 prompt assumed columns/concepts (use_case enum, topical_fit_score, backlink/auction data) that never existed in v0.1; flagged at pre-flight, operator re-specced | 2026-05-24 |
| idea-agent synth has NO cost cap | Healthy synth is $0.35–0.45 (18–24K tokens); any cap tight enough to matter aborts real runs. Hardened prompt + 3-tier parser + retry-once is the guard. `claude` CLI has no `--max-tokens` anyway | 2026-05-24 |
| idea-agent kept `run_weekly.sh` + `weekly_run_*` names when going daily | Renaming ripples into `week_of`/`idea_calibration`/`calibrate.sh`; legacy identifiers, documented | 2026-05-24 |
| Issues `/issues` separate ungated page; capture fresh-start anchored, no backfill | Lower risk than the `/` SPA; operator wants only NEW errors from activation forward. Local `new_issues.log`, Telegram paging deferred to v1.1 | 2026-05-24 |
| /model picker = slash-command grid, NOT inline keyboard | bot.js had zero callback infra (~1.5h net-new) | 2026-05-23 |
| Cron `claude -p` auth via `CLAUDE_CODE_OAUTH_TOKEN` | Stays on Pro/Max billing; token sourced from `~/.claude_oauth_token`, out of git | 2026-05-23 |
| Market cron 07:00/07:15/08:15/08:45 PT | Curator runtime ~20 min per audit_log spread; needed buffer | 2026-05-23 |

## Deferred items

(See `DEFERRED.md` for full context per item.)

**This session:**
- **D091** (NEW) — internal-link agent v1 `audit_links.sh` is blind to body-paragraph links + render-path mismatch. Must parse `paragraphs[]` (asbestos) / `body[].text` (SSG) and mirror per-site render logic. **Trigger:** during the D083 SSG-fy pass.
- **D090** (NEW, parallel chat) — investigate approved-index drift on escalated slugs (manual edit vs `run-batch.sh` bug — index write before MAX_ROUNDS escalation?). **Trigger:** before next batch of slugs ships.
- **D066** — CLOSED (~/agents half re-audit clean; ~/projects half owned by other chat).
- **D072** — CLOSED (cap raised; D073 rubric-redundancy + D074 live-gate verification remain open).

**Carried (sister/market chats):** D087 (idea-agent v1.1), D088 (/model v1.1 — wire remaining 12 ai-do.sh callers), D089 (notes-agent v1.1), D083 (SSG-fy propose_backlinks + keyword-registry).

**Agent-build lane follow-ups (not yet D-numbered):**
- **domain-hunter v0.2** — if topical signal proves real after 2–4 weeks + ≥1 domain acquired, evaluate $10/mo ExpiredDomains Pro for backlink (TF/CF/RD) + auction data. **Trigger:** 2–4 weeks of digests.
- **idea-agent** — revisit `week_of`/`idea_calibration` one-row-per-week semantics now that cadence is daily (calibrate runs 7×/week). **Trigger:** after a week of daily runs.
- **issues-capture v1.1** — tighten capture filter if `new_issues.log` is noisy; wire Telegram paging once noise rate is known; nested-agent log-path resolution. **Trigger:** once noise rate observed.

**Highest-priority open (cumulative):** F2 (drafter pre-flight keyword-config check), D044 (log_to_audit apostrophe rewrite), F17/F18 (daily API spend hard cap), F14 (pre-ship operator approval gate).

## Next session should

1. **Live-smoke the three new agents from the operator's phone** (carried) — token_usage after /model toggles, inbox entries after note triggers, forward confirmation flow.
2. **Spot-check idea-agent's first 5 proposals** (carried) — mark `operator_action` in `idea_proposals`.
3. **If touching internal-link agent or SSG-fy (D083/D091):** extend `audit_links.sh` to parse body-paragraph markdown AND mirror each site's render-time path logic. Do NOT audit by curling raw slug URLs.
4. **If re-touching editor:** consider a fresh full-body burn-in to re-calibrate the 3.6 threshold (it was tuned on head-truncated scores pre-D072).

## Files to reference

**Internal-link lane (this session):**
- `~/projects/asbestoshq-site/src/components/GuideArticle.tsx` — `parseInlineLinks` (line ~71) is the render-time bare→`/guides/` rewriter + whitelist gate; `APPROVED_GUIDE_SLUGS` set (line 8). READ THIS before touching any guide link.
- `~/projects/asbestoshq-site/src/data/guides/*.json` — 32 guides; body links live in `paragraphs[]` as bare `/<slug>/` markdown.
- `~/agents/internal-link/audit_links.sh` — v1 audit (relatedLinks-only; D091 to extend).
- `~/agents/editor/score.sh` — `EDITOR_MAX_BODY_BYTES` cap at line ~31.

**Agent-build lane (this session):**
- `~/agents/domain-hunter/` — `scan.sh`, `sites.yaml`, `fetch/`, `filter/{hard_filter,wayback_check}.py`, `score/{value_score,topical_fit,backfill_topical}.py`, `deliver/telegram_digest.sh`.
- `~/agents/issues-capture/` — `capture.sh`, `state/start_anchor_id` (=2991), `log/{capture,new_issues}.log`.
- `~/agents/idea-agent/synthesize.sh` + `prompts/synthesize.md` (hardened) + `run_weekly.sh` (daily 07:10).
- `~/agents/dashboard/server.js` — `/issues` page + `/issues/data` + POST routes + `/` counter.
- `~/agents/lib/ai-do.sh` — `AI_DO_MAX_BUDGET_USD` passthrough + hardened `.get('result')`.

**Prior (new agents):**
- `~/agents/idea-agent/`, `~/agents/notes-agent/`, `~/agents/lib/resolve_model.sh`, `~/agents/lib/ai-do.sh` (`AI_DO_MODEL_OVERRIDE` hook), `~/agents/telegram/bot.js`.

## Gotchas for future me

- **Verify link health against RENDERED HTML, not raw URLs.** asbestoshq.com's `parseInlineLinks` rewrites bare `/<slug>/`→`/guides/<slug>/` at render IF the slug is in `APPROVED_GUIDE_SLUGS` (else strips to plain text). Curling the bare URL returns 404 for every *working* link. This caused the 99-link false-positive incident this session (commits `36cb356`/`cd03847` reverted by `711748b`/`81ab725`). See LESSONS.md "Verify render path, not raw URLs."
- **Don't assume HTML attribute order in greps.** Body links render `<a class="guide-inline-link" href="...">` (class-then-href). An href-then-class grep returns 0 and looks like failure — tripped me 3× this session. Dump the raw tag first.
- **Vercel CDN masks deploy verification.** `x-vercel-cache: HIT` + rising `age`; `?cb=` query-busting doesn't work on static pages. The deploy propagates eventually. Don't claim a live render you can't observe.
- **Confirm table schema before querying.** `audit_log` is `action`/`payload_json`, not `event`/`payload`.
- **D-number race:** RE-CHECK `grep -oE 'D0[0-9]{2}'` immediately before writing to DEFERRED.md. D087 was already taken this session; used D091.
- **Bot under launchd KeepAlive** — `launchctl kickstart -k gui/$UID/com.aiteam.grammy-bot` is the operator-clean restart (plist at `07925f8`).
- **dotenv reads `.env` from CWD, not script dir** — dotenv-using scripts need `cd "$(dirname "$0")"`.
- **macOS cron does NOT catch up on missed firings** after sleep/reboot. Silence-during-downtime is permanent.
- **`KEY=          # required` in .env parses as empty value** — strictly worse than unset (`#` starts a shell comment). Empty `ANTHROPIC_API_KEY` makes `claude` exit rc=1.
- **Auto-commit at 23:50 PT sweeps untracked files** under `agents: auto-commit YYYY-MM-DD`. Commit before then or reference the auto-commit SHA in your message.
- **Each chat keeps its OWN ctx-save narrative file.** The internal-link chat owns `AITEAM_Context_Save_2026-05-24_1756.md`; the parallel asbestos-cleanup chat owns `_1756_asbestos-cleanup.md` (same-minute filename collision — see next). Earlier sister chats own `_0937.md`/`_0940.md`. Don't trample earlier chats' slots.
- **ctx.sh's mechanical git record only scans `~/agents`.** Work in `asbestos-contractors` / `brain` is invisible to it. On a multi-repo session the auto-record can be entirely a *sister chat's* `~/agents` work while your own commits are absent — cross-check `git log` in all three repos and hand-assemble the mechanical record if your work landed elsewhere.
- **Same-minute ctx.sh filename collision is real.** Snapshot names are minute-granular (`_HHMM.md`). Two chats running `ctx.sh` in the same minute clobber each other's skeleton (the later run re-fills the topic). If yours got overwritten with a sister's filled narrative, write yours to a suffixed name (`_HHMM_<topic>.md`). Happened today at 17:56.
- **Index/manifest files can lie about ship status.** Trust file location (`approved/` vs `needs-review/`) + `auditor_verdicts` rows over a derived index. And `.gitignore` comments can contradict tracked reality — verify against `git ls-files`.
- **(agent-build lane) The `claude` CLI (v2.1.150) has NO `--max-tokens`** — output bound is `--max-budget-usd` only (soft, can overshoot). A budget abort returns no `result` field → wrappers must `.get('result') or ''` or KeyError under `set -e`.
- **(agent-build lane) `claude --json-schema` doesn't populate `.result`** in the `ai-do.sh` pattern (tool-use turns, empty result, needs >1 turn). High blast radius to adopt; prefer hardened-prompt + tolerant-parser.
- **(agent-build lane) idea-agent synth legitimately costs $0.35–0.45/run** (18–24K out tokens). Don't cap it low — that aborts healthy runs (empty result → exit 2, no retry). Prompt+parser+retry is the guard.
- **(agent-build lane) Wayback fetch:** `id_` after the timestamp for raw page; detect gzip magic `1f 8b` + decompress (Accept-Encoding ignored, original bytes replayed); strip control bytes before subprocess.
- **(agent-build lane) bash `IFS=$'\t' read` collapses consecutive tabs** → empty fields shift columns. Use `\x1f` for DB-row parsing with possibly-empty fields.
- **(agent-build lane) GoDaddy/Ahrefs/ExpiredDomains public pages edge-blocked 403** to unauthenticated GETs — fetchers detect + degrade, don't fail.
- **(agent-build lane) `INSERT OR IGNORE` on UNIQUE keeps first-seen metadata** (correlation_id) → correlation-keyed funnel counts zero out on dedup; count against the input set.
- **(agent-build lane) Task spec naming nonexistent columns/concepts → STOP at pre-flight, don't fabricate the schema** (domain-hunter v0.2→v0.1.5). The agent-build lane's ctx narrative is `AITEAM_Context_Save_2026-05-24_1758.md`.
