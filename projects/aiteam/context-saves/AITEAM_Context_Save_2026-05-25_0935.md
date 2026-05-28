# AITEAM Context Save — 2026-05-25_0935
**Generated:** 2026-05-25T09:35:09-0700
**Since last save:** 2026-05-25 09:35:00
**Session topic:** Verification + multi-build chat — outstanding-work audit → DEFERRED bookkeeping → internal-link SSG-fy + D091 → orchestrator conversation memory → Telegram typing indicator → orchestrator dropzone (text writes + image forwarding). Spanned 2026-05-24 evening into 2026-05-25 morning.

**Note on the empty mechanical record:** all this session's commits landed before the ctx anchor (the dropzone work was swept by the 23:50 auto-commit `74c1cde`), so the auto-generated git/file/audit sections above are blank. Narrative below is the real record. Commits this session: brain `1c35674` + `e4d132e`; agents `934dc76`, `ff63975`, `9f7c5c4`, `dc1dfc4`, `1627643`, and dropzone work inside auto-commit `74c1cde`.

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

**1. Outstanding-work audit (read-only, no changes).** Verified 15 items against disk. Found F17/F18 cost-cap shipped (`54a69bd`) but still Open in DEFERRED; D045 number collision; D087 stale (weekly text vs daily cron); domain-hunter "v0.2" items already shipped in v0.1.5; Issues Queue v1.1 untracked. These fed Phase A.

**2. Phase A DEFERRED bookkeeping (brain `1c35674`).** Closed F17/F18 inline (cost-cap enforcement shipped). Renamed the archived Cowen prompt-template D045 → **D050** to resolve the collision. Reframed D087 weekly→daily (commit `6192499`). **Mid-task collision:** a sister chat shipped + closed Issues Queue v1.1 as **D092** (`1d89771`) while I was editing — dropped my duplicate D092(OPEN), kept only **D093** (domain-hunter v0.2 = ExpiredDomains Pro). My other edits (F17/F18, D050, D087) were swept into the sister's commit; only D093 was mine to push.

**3. Internal-link SSG-fy v2 + D091 (agents `934dc76` + `ff63975`, brain `e4d132e`).** Operator-approved decisions: (a) SSG source = the live SITE repo `~/projects/smartsourceguide/data/guides/<cat>/<slug>.json` because the ssg-content pipeline dir is empty; (b) audit against `APPROVED_GUIDE_SLUGS` parsed from each site's render component, NOT file existence (this is the D091 fix — prevents the curl-404 false-positive). Per-site verdict asymmetry: asbestos non-approved link = **stripped** (advisory), SSG = **broken** (404), because SSG's `inline-md.tsx` emits hrefs verbatim while asbestos `parseInlineLinks` rewrites/strips. Schema migration tested on a `.backup` round-trip before live apply (32/0/5 preserved). **D091 fully closed; D083 left PARTIAL** (propose_backlinks SSG-fied; keyword-registry `update_registry.py` + deploy_batch SSG-branch wiring still open = deferred item 7).

**4. Orchestrator conversation memory (agents `9f7c5c4` + `dc1dfc4`).** Diagnosed as **Mode 2** — `conversation_log` was write-only (105 rows, zero SELECT anywhere). Wired `build_conversation_context.sh` (gated by `agents_with_memory.txt`, currently just `orchestrator`) into `run_agent.sh`, prepending the last 30 msgs / 72h. Excludes the current turn via `source_turn_id`. Text path only — vision left unchanged to avoid breaking the `@path` image attachment. Verified live: input 242→3765 tokens, cache_read 12963 on 2nd call, orchestrator recalled the GTA-6 screenshot it had previously "forgotten."

**5. Typing indicator (agents `1627643`).** Pulse `ctx.replyWithChatAction("typing")` every 4s inside `routeToOrchestrator`, `stopTyping()` in `finally`. Threaded a `ctx` param into the function. Replaces the existing single-shot pings that expired after ~5s.

**6. Orchestrator dropzone (agents, swept into `74c1cde`).** `~/orchestrator_inbox/` + `write_to_dropzone.sh` (md/txt/json/csv, no overwrites, positional `log_to_audit.sh`, path-only stdout). Image archive integrated into BOTH photo paths (direct `message:photo` handler + `handleForwardedMessage`) — NOT a second `bot.on` handler (one already existed for notes/vision). Permission to run the helper required a **cwd-local** `orchestrator/.claude/settings.json` allow-rule (git-root was silently ignored — D025).

## Lessons learned

- **Agent `claude -p` permission allow-rules must live in the cwd-local `~/agents/<agent>/.claude/settings.json`, not the git-root `~/agents/.claude/settings.json` (which is silently ignored).** run_agent.sh `cd`s into the agent dir; that's where Claude Code resolves project settings. Verified empirically wiring the orchestrator dropzone helper (git-root did nothing; cwd-local worked first try). [D025]
- **`log_to_audit.sh` is positional** (`actor_type actor_id action target payload_json [corr]`), not `--flag` style. Flag-style invocation writes a malformed row *silently* (D064 validation only catches `=` in slot 1 / JSON in slot 4). It also echoes the correlation_id on stdout — redirect BOTH streams (`>/dev/null 2>&1`) in any helper whose stdout is a contract (e.g. returns a path). And `audit_log` is `action`/`payload_json`, never `event_type`.
- **Verify features against hard evidence (audit rows + files on disk), never the operator's "it worked" or the bot's normal reply.** The dropzone image archive looked successful (bot replied normally) but had *never executed* — the test photo was a forward, which took an un-wired code path. Only the absence of any `image_received` row exposed it.
- **The 23:50 PT auto-commit sweeps uncommitted work into a generic `agents: auto-commit <date>` commit.** If you want descriptive commit messages, commit before 23:50 — once it's swept and pushed you can't re-message without a force-push (banned). Six descriptive commits this session; the dropzone feature's two intended commits were lost to the sweep (`74c1cde`).
- **Concurrent sister chats share the working tree and push to the same repo.** A sister `git add <file>` can sweep your uncommitted edits into their commit, and D-numbers can be claimed mid-session — re-grep DEFERRED for the next free D-number immediately before writing, and stage exact paths (never `git add -A`).
- **`bot.js` is ESM (`import`), not CommonJS.** Reuse existing helpers (`logAudit()`, `file.download()` from `@grammyjs/files`, `copyFile`/`stat` from `node:fs/promises`) — `require()`/global-`fetch`/`execSync` snippets will crash it.
- **SSG and asbestos render internal links differently.** asbestos `GuideArticle.tsx` rewrites bare `/<slug>/`→`/guides/<slug>/` if approved, else strips to plain text; SSG `lib/inline-md.tsx` emits the href verbatim (no whitelist, no rewrite). So a non-approved internal link is genuinely broken (404) on SSG but merely stripped (advisory) on asbestos — audits must mirror per-site render logic, not curl raw URLs.

## Operator corrections

No hard "you got it wrong" corrections this session; the operator's steers were directional decisions at gates:
- On the D092 collision: approved dropping my duplicate and keeping D093.
- On D083: approved **partial-close** (not full close) with the exact 3-line ✅/🟡 breakdown — "propose_backlinks done; keyword-registry + deploy_batch wiring open."
- On the orchestrator permission wall: chose the **scoped allowlist** over skip-permissions (security) or deferring.
- On the image-archive discrepancy I flagged: chose to **also wire forwarded photos**, then accepted "call it done" with the forward path committed-but-not-live-verified.
- Confirmed the memory phone-test passed before I committed the memory fix.

## What's next

**Immediate / verify-on-next-use:**
1. **Forwarded-photo dropzone archive** is committed + syntax-checked but never run live — confirm an `image_received` row with `forwarded:true` + a `telegram-photo-*.jpg` lands on the next real forward.
2. **Memory v1.1:** vision/photo turns don't receive conversation context (text path only) — wire if continuity gaps show up around image turns.

**Deferred (tracked in DEFERRED.md):**
- **D083 remainder:** `keyword-registry/lib/update_registry.py` SSG `category/slug` handling + `deploy_batch.sh` SSG-branch wiring (item 7) — gated on SSG shipping its first batch.
- **D093:** domain-hunter v0.2 (ExpiredDomains Pro) — gated on 2-4 weeks of v0.1.5 signal + ≥1 domain acquired. Also: domain-hunter CLAUDE.md still mislabels v0.1.5 items as "v0.2" — cleanup at next domain-hunter touch.
- **D087:** idea-agent daily-cadence calibration — `idea_calibration` now gets duplicate per-week rows under daily cadence; revisit row-uniqueness. Build-rate stuck at 0 until operator marks `operator_action`.

**Known minor gaps:**
- `audit_links.sh` has no dedup — repeated audits accumulate duplicate `stripped` rows in `audit_link_failures` (v1.1).
- The popcorn-ceiling regression audit left 8 real `stripped` rows in `audit_link_failures` (accurate findings, not test pollution).

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-25_0935.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-24 17:58 (THREE parallel chats: internal-link lane [D066/D072/link-audit] + asbestos-cleanup lane [recon/push/D090/gitignore] + agent-build lane [domain-hunter/idea-agent-synth/Issues-Queue])  **This chat (agent-build lane):** Built **domain-hunter** v0.1 end-to-end (expireddomains/godaddy fetchers → no-LLM hard-filter → brandability + Haiku use-case tags → Wayback classify → Telegram digest; cron live 06:30; godaddy edge-blocked, graceful) then **refit to v0.1.5** — Haiku topical-fit gating against 3 sites (ust/asbestos/ssg), surface only `topical_fit_max ≥ 0.3`, brandability demoted to secondary, Ahrefs deep link in digest. Diagnosed + fixed **idea-agent synthesize** (Sonnet rambled 34K tokens / narrated multi-fence; brittle parser): hardened single-JSON prompt + 3-tier parser (exit 4) + retry-once; **removed the budget cap** after a live run proved healthy synth costs $0.35–0.45 (not ~$0.10); cadence **weekly→daily 07:10**. Built the **Issues Queue**: `agent_issues` table + `*/5` capture daemon (fresh-start anchored at audit_log.id 2991, no backfill, `new_issues.log`) + dashboard `/issues` (filters, status/notes, one-click CC-prompt modal). 3 crons installed live (domain-hunter 06:30, idea-agent 07:10, issues-capture */5). Commits: domain-hunter `ee9e34c…044d619`, idea-agent `ca21007…ce46867`, issues `bdc1909…0d099ab`.  
