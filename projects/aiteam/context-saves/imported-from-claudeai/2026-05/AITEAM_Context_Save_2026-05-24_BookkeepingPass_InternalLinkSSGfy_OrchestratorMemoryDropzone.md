# AITEAM Context Save — Bookkeeping Pass + Internal Link SSG-fy + Orchestrator Memory/Dropzone

**Date:** 2026-05-24 (evening session)
**Session topic:** Outstanding-work audit → Phase A bookkeeping → Internal Link Agent SSG-fy + D091 → Orchestrator memory wiring → Typing indicator → Orchestrator dropzone (text + image archive)
**Companion chats:** Sister chat shipped D092 (Issues Queue v1.1) and D094 (lockfile) concurrently. Two parallel-chat collisions resolved in-flight.

---

## TL;DR

Five distinct work items shipped in one session:

1. **Outstanding-work audit** — CC verified 15 candidate items against disk. 6 were already shipped but uncatalogued, 1 was shipped-but-not-closed, 1 was mischaracterized. Cleanup pass shrunk the real backlog substantially.
2. **Phase A bookkeeping (DEFERRED.md)** — F17/F18 closed inline (54a69bd), D045 collision resolved (Cowen item → D050), D087 reframed weekly→daily, D093 added (domain hunter v0.2 real scope = ExpiredDomains Pro). Sister chat owned D092 (Issues Queue v1.1 — shipped same session). Final commit `1c35674`.
3. **Internal Link Agent SSG-fy + D091** — multi-site schema migration (reversible, round-tripped on backup), per-site JSON shape/path handling, audit_links.sh now uses render-component APPROVED_GUIDE_SLUGS Sets (not file existence — direct fix for 2026-05-24 false-positive incident). Commits `934dc76`, `ff63975`, `e4d132e`.
4. **Orchestrator memory** — Mode-2 diagnosis (history persisted but never replayed). Wired `conversation_log` SELECT into `run_agent.sh` via `build_conversation_context.sh`. 72h window, 30-turn cap, human-chat filter. Live-verified via phone test (orchestrator recalled GTA-6 screenshot reference).
5. **Typing indicator + Orchestrator dropzone** — pulse-every-4s typing action during orchestrator runs. Then dropzone: `~/orchestrator_inbox/` with `write_to_dropzone.sh` helper (scoped allowlist via cwd-local `.claude/settings.json`), photo auto-archive for both direct AND forwarded photos. Final commit `74c1cde`.

Net: 4 explicit commits this chat + 1 from sister chat + ~26 commits during the dropzone build. Everything pushed. Zero rollbacks.

---

## Session arc

1. **Outstanding-work audit kicked off the session.** Operator asked for verification of 15 candidate items. CC's verification pass found significant doc-vs-reality drift: D045 used twice, F17/F18 fully shipped but Open in DEFERRED, domain-hunter v0.2 mislabel in CLAUDE.md, multiple "open" items already shipped.
2. **Phase A bookkeeping** ran into the textbook parallel-chat collision: sister chat shipped Issues Queue v1.1 and claimed D092 mid-edit. CC's pre-flight had verified D091 was highest at start; sister chat sprinted past us. CC caught the collision via `git diff` on file-modified-since-read error. Dropped my D092 (OPEN) block, committed D093 only. Side-effect: sister chat's `git add DEFERRED.md` swept my edits 1/2/3 into their commit (1d89771) — work landed correctly but under a misleading commit message. Flagged for LESSONS.md.
3. **Internal Link SSG-fy** pre-flight surfaced three real disk-vs-prompt deviations: SSG content lives in site repo (not pipeline dir, which is empty), SSG has NO render-side stripping safety net (broken means actually 404), and the render truth is hardcoded APPROVED_GUIDE_SLUGS Sets in TS components — not file existence. Last point is the direct fix for the 2026-05-24 false-positive incident. CC built a shared `parse_approved_slugs.sh --site` helper so both audit_links.sh and propose_backlinks.sh use one source of truth.
4. **Schema migration** ran on a `.backup` copy first, round-trip verified (forward + revert, 32/0/5 rows preserved), then applied to live DB. Safety pattern worth standing.
5. **D083 partial-close** decision: 2/3 sub-parts still open (keyword-registry update_registry.py + deploy_batch SSG-branch wiring = deferred item 7 per morning's context save). Closing it fully would've tripped the "shipped but not closed / contradiction" gate that started today's bookkeeping pass. Honored with 🟡 marks.
6. **Orchestrator memory diagnosis** found Mode 2 cleanly: conversation_log faithfully written (105 rows, both roles, current) but never read back. Pure wiring gap. Sonnet caching makes replay nearly free on warm calls.
7. **Memory fix held uncommitted** awaiting phone-test confirmation. Operator moved on without testing. Next session (typing indicator) caught the uncommitted memory entanglement on pre-flight — bot.js had both pending changes. Order forced: phone-test memory first, then commit, then build typing on clean base.
8. **Phone-test memory passed.** Memory committed. Typing built and shipped.
9. **Dropzone session** ran into THREE auto-fix-able issues on pre-flight: existing photo handler at bot.js:1276 (D089 notes-agent + vision-chat), log_to_audit.sh is positional not --flag, audit_log column is `action` not `event_type`, bot.js is ESM not CommonJS. CC fixed all four without asking. Photo-handler coexistence decision: option 1 (save every photo to dropzone AND keep existing notes/vision behavior).
10. **First photo test missed.** Block sat in main photo handler AFTER the `if (isForward) return handleForwardedMessage(...)` early-return. Operator's test photo was a forward → archive block never executed. CC caught it via audit log evidence and refused to commit as "working." Extended `handleForwardedMessage` to archive too, restarted, re-tested direct path (verified), left forward path syntactically proven but unexercised by operator's call.
11. **Dropzone commit bundled into 23:50 auto-commit** as `74c1cde` rather than CC's two descriptive commits. Code is safe; no force-push to "fix" history.

---

## What got built / committed

### ~/brain/projects/aiteam/

| SHA | What |
|---|---|
| `1d89771` | (sister) chore(deferred): close D092 — Issues Queue v1.1 — swept my edits 1/2/3 into this commit unintentionally |
| `1c35674` | chore(deferred): close F17/F18, resolve D045 collision, reframe D087, add D093 |
| `e4d132e` | chore(deferred): close D091 + partial-close D083 |

### ~/agents/

| SHA | What |
|---|---|
| `934dc76` | feat(internal-link): SSG-fy v2 (items 1-5): migration + revert, site_lib.sh, build_inventory.sh, propose_backlinks.sh, CLAUDE.md |
| `ff63975` | fix(internal-link): D091 audit upgrade: audit_links.sh body-link + render-path mirroring |
| `1627643` | (memory + typing — exact SHA verified mid-session as HEAD before dropzone work) |
| `74c1cde` | dropzone bundle: write_to_dropzone.sh + orchestrator/.claude/settings.json + orchestrator/CLAUDE.md + bot.js direct+forward archive |
| `633f6dc` | (sister) D094 lockfile (interrupted my dropzone pre-flight; bot was running uncommitted memory code at this point) |

---

## Decisions made (and why)

### Phase A bookkeeping
- **F17/F18 closed inline** (not moved to a closed section) — matches authoring rule at DEFERRED.md line 1114
- **D045 → D050** for Cowen item — per file's own authoring note at line 1112
- **D093 = real domain-hunter v0.2** — ExpiredDomains.net Pro ($10/mo), gated on 2-4 weeks signal AND ≥1 domain acquired. Three items previously called "v0.2" (Wayback gate, UST threshold, calibration) were actually shipped in v0.1.5.
- **Dropped my D092 block** — sister chat shipped+closed it concurrently; keeping mine would create a duplicate
- **D083 partial-close not full close** — 2/3 sub-parts open; honest accounting over premature ✅

### Internal Link SSG-fy
- **SSG GUIDES_DIR = ~/projects/smartsourceguide/data/guides/** — SSG pipeline content dir is empty (ssg hasn't run pipeline); only real SSG guides live in site repo
- **APPROVED_GUIDE_SLUGS from render components** — not file existence. Asbestos parses GuideArticle.tsx Set, SSG parses site-config.ts Set. Single shared parser `parse_approved_slugs.sh --site`.
- **Bare slug + separate `category` column** for SSG (vs collapsed `category/slug` slug). Cleaner schema.
- **Per-site render-path logic in audit:**
  - asbestos: bare `/<slug>/` → if in APPROVED → resolves to `/guides/<slug>/`; else stripped to plain text (flag as "stripped", not "broken")
  - SSG: NO stripping safety net — broken means actually 404, no soft-fail category
- **Migration on .backup first, then live** — round-trip verified before touching production DB. Safety pattern worth standing.
- **D091 fully closed; D083 partial** — propose_backlinks SSG-fy done (934dc76); keyword-registry update_registry.py + deploy_batch SSG-branch wiring still open

### Orchestrator memory
- **Mode 2 diagnosis (history persisted, not replayed)** — diagnose-first paid off, smallest possible fix
- **72h window, 30-turn cap, human chat filter** — matches operator's "24h+/few days" ask with hard upper bound on cost
- **Hybrid window strategy** — time-bounded AND turn-bounded, not OR. Either one tripping ends the window.
- **`build_conversation_context.sh` as shared helper** — opt-in via `agents_with_memory.txt` list. Only orchestrator opts in this session.
- **Wired at run_agent.sh layer** — benefits any agent that opts in, not just orchestrator. Anti-instruction-injection note added to CLAUDE.md.
- **No summarization, no /reset, no auto-expire** — explicitly deferred. Current volume (7 msgs/day) is months from needing summarization.

### Typing indicator
- **Pulse every 4s on setInterval** — Telegram's typing action auto-expires ~5s; existing one-shot calls were the visible bug
- **`ctx` threaded as parameter** into routeToOrchestrator — exists in callers (routeAndLog:257, vision:1316) but wasn't in scope inside the function
- **finally-guaranteed cleanup** — clearInterval is idempotent so calling stopTyping twice is fine
- **Orchestrator-only scope** — slash commands return instantly, don't need indicator

### Dropzone
- **`~/orchestrator_inbox/` top-level** — easier to find than nested in brain
- **Helper script enforces dest** — safety in code, not in prompt that conversation context can shift
- **Allowed text extensions:** .md, .txt, .json, .csv. **Allowed image extensions:** .jpg, .jpeg, .png, .webp, .gif. No executables.
- **Auto-suffix timestamp on collision** — no overwrites possible
- **Scoped allowlist via cwd-local `.claude/settings.json`** — not skip-permissions. Narrow grant for one helper. **GOTCHA: orchestrator launched via run_agent.sh honors allow-rules from cwd-local `~/agents/<agent>/.claude/settings.json`, NOT the git-root settings.** Saved to memory.
- **Save every photo to dropzone AND keep existing notes/vision behavior** — non-destructive. Photos in BOTH places by design (dropzone = universal archive).
- **Forward path wired** — extended handleForwardedMessage to archive too. Operator's actual use case is forwards, not direct camera-roll photos.

---

## What was left mid-task

### Forward-path archive — code shipped, not exercised
`handleForwardedMessage` extension archives forwarded photos to dropzone, syntax-checked, mirrors proven direct path. Not yet tested by a real forwarded photo. If operator forwards a photo in the next day or two and it doesn't land in ~/orchestrator_inbox/, that's where to look.

### Auto-Refresh Agent
Was the next planned item in this chat's lane. Deferred when CC pre-flight made clear it has nothing to act on for ~22 days (needs ≥30d GSC data per page, asbestos first slug shipped 2026-05-16). Build later when data exists.

### Sister chat's launchd auto-restart (D026)
Still owed by sister chat lane. This chat doesn't touch it.

### audit_links.sh dedup (potential D094 — but D094 already claimed)
CC flagged that audit_links.sh has no dedup — repeated runs accumulate duplicate stripped rows in `audit_link_failures`. Sister chat already took D094 for a lockfile commit, so this needs **D095** when picked up. Logged in CLAUDE.md coverage notes; not blocking.

---

## Open decisions (live design space)

Carrying forward from prior sessions:
- The 7 operator-policy questions from cross-agent failure modes audit §7 (still blocking D056 editor production runner status closure + others)

New from this session:
- **D025 cwd-local settings.json pattern** — worth promoting from gotcha to documented convention for any future scoped-permission agent

---

## Lessons learned

### Parallel-chat collision via `git add <file>` sweep
When two chats edit the same file, the second chat's `git add <file>` stages the WHOLE file including the first chat's uncommitted edits. The first chat's work then lands under the second chat's commit message. Today: sister chat's `1d89771` commit unintentionally swept my F17/F18 + D045 + D087 edits. Work landed correctly but attribution is wrong. Mitigations: (a) `git stash` before pre-flight ends to isolate state, (b) shorter edit→commit windows.

### Diagnose-first paid off (orchestrator memory)
The "remembers everything" build could've been a multi-hour memory architecture. Diagnosis confirmed Mode 2 — pure wiring, ~50 lines of code. Pre-flight saved an evening of misdirected work.

### CC's pre-flight discipline caught real bugs (again)
Internal Link SSG-fy: 3 disk-vs-prompt deviations caught before any code. Dropzone: 4 auto-fixable issues (existing photo handler, positional log_to_audit.sh, action vs event_type, ESM vs CommonJS). Each one would've been a wasted iteration without pre-flight. Never relax pre-flight requirements to "speed up" a build.

### Honest partial-close beats premature ✅
D083: 1 of 3 sub-parts shipped. Tempting to mark closed since the prompt said "close D083". CC refused and asked for direction. Right call — closing it fully would've tripped the same gate that started today's bookkeeping pass.

### Render-path is the truth, not file existence
The 2026-05-24 false-positive incident lesson now baked into code: audit checks against APPROVED_GUIDE_SLUGS Sets parsed from render components, not against file existence in the approved/ directory. The two CAN diverge silently.

### Schema migration on .backup first
~/store/aiteam.db.pre-multisite-*.backup saved before applying schema changes. Round-trip (forward + revert) verified row preservation before live apply. Worth making this standing convention for any schema work.

### Scoped allowlist > skip-permissions
Operator instinct was "give it write permission." Reality: orchestrator already had skip-permissions globally — the missing piece was telling it the helper was safe to call. cwd-local `.claude/settings.json` with a narrow Bash rule = exact-fit safety scaffolding. Skip-permissions would've been a silent security downgrade.

### "Forward" vs "direct" photo path matters
Operator's actual use case is forwarding photos (screenshots from other chats, social media saves, etc.). Direct camera-roll photos are edge case. Forwarded photos go through handleForwardedMessage which is a separate code path from `bot.on('message:photo')`. Don't assume one handler covers both.

### `audit_log` column is `action` not `event_type`
Documented schema mismatch. Burned twice this month. Worth a one-line note in CLAUDE.md (or skill file) for future agents writing audit rows.

### `log_to_audit.sh` is positional, not --flag-based
Has D064 validation per CC. Signature: `<actor_type> <actor_id> <action> <target> <payload_json> [corr]`. --flag invocations write malformed rows silently. Same caveat.

---

## DEFERRED items captured / status

### Closed this session
- **F17/F18** — daily cost-cap enforcement shipped 2026-05-17 (54a69bd)
- **D045 (operator's ai-do.sh rewire)** — already CLOSED-OBSOLETE, just acknowledged in bookkeeping
- **D087** — reframed weekly→daily (not closed, reframed)
- **D091** — internal-link audit upgrade shipped (ff63975)

### Renumbered
- **D045 (Cowen prompt-template) → D050** — collision resolved per authoring note

### New (added to DEFERRED)
- **D093** — Domain hunter v0.2 = ExpiredDomains.net Pro membership ($10/mo). Gated on 2-4 weeks v0.1.5 signal + ≥1 domain acquired.
- **D092** — Issues Queue v1.1 (sister chat shipped+closed same session)
- **D094** — sister chat (lockfile)

### Still open / new from this session
- **D083 partial** — propose_backlinks.sh SSG-fy done; keyword-registry update_registry.py + deploy_batch SSG-branch wiring (= morning's deferred item 7) remain
- **D074** — editor + GEO live verification — still passive, waits for next organic slug ship
- **D025 cwd-local settings.json gotcha** — worth promoting to documented convention
- **D095 (next available)** — audit_links.sh dedup (repeated runs accumulate stripped rows in audit_link_failures)
- **Auto-Refresh Agent build** — deferred until ~2026-06-15 when 30d GSC data exists per page
- **CLAUDE.md cleanup for domain-hunter** — v0.1.5 mislabel; ~15 min, do at next domain-hunter touch

### Standing open (no change)
- The 7 operator-policy questions from cross-agent failure modes audit §7

---

## Coordination notes with the other chat

### What sister chat did
- Issues Queue v1.1 — shipped + closed as D092 (commits 20611fd / 24b7cd7 / eaa4fe9; audit row 3049)
- D094 lockfile work — commit 633f6dc (interrupted my dropzone pre-flight; no functional impact, just noted)

### Collisions encountered
1. **D092 collision** — both chats independently claimed for different items. Resolved: I dropped my D092 (Issues Queue v1.1 was already done by sister), kept D093.
2. **File sweep collision** — sister's `git add DEFERRED.md` staged my pending edits 1/2/3. Resolved: accepted the attribution drift; work landed correctly.
3. **HEAD-ahead** — sister's 1d89771 was 1 commit ahead of origin/main at my push time. My push published it. Expected git behavior, flagged for awareness.

### No collisions
- ~/agents/internal-link/ work — sister chat untouched
- ~/agents/orchestrator/, telegram/, lib/ for memory/typing/dropzone — sister chat untouched

---

## Files created/modified

### ~/agents/
- `internal-link/audit_links.sh` (D091 upgrade — body links + render-path mirroring)
- `internal-link/build_inventory.sh` (per-site JSON shape + path handling)
- `internal-link/propose_backlinks.sh` (--site arg)
- `internal-link/CLAUDE.md` (neutralized + per-site behavior table)
- `internal-link/site_lib.sh` (new — site helpers)
- `internal-link/parse_approved_slugs.sh` (new — shared parser for both sites)
- `internal-link/migrations/` (new — forward + revert SQL)
- `lib/build_conversation_context.sh` (new — memory replay helper)
- `lib/agents_with_memory.txt` (new — opt-in list, contains: orchestrator)
- `lib/run_agent.sh` (memory injection wiring)
- `lib/write_to_dropzone.sh` (new — safe file writer)
- `orchestrator/CLAUDE.md` (Conversation context + File writes sections)
- `orchestrator/.claude/settings.json` (new — scoped Bash allowlist for write_to_dropzone.sh)
- `telegram/bot.js` (memory chat_id passthrough + typing pulse + photo archive direct + forward)

### ~/brain/projects/aiteam/
- `DEFERRED.md` (F17/F18 closed inline, D045 collision resolved, D087 reframed, D093 added)
- LESSONS.md (carried entries from earlier session; no new edits this session — pattern of lessons captured here in context save instead)

### ~/store/aiteam.db
- `audit_link_failures`: added `site` column
- `backlink_proposals`: added `site` column, replaced UNIQUE constraint with `UNIQUE(site, new_slug, source_slug)`
- `internal_link_inventory`: added `category` column (NULL for asbestos, populated for SSG)
- `conversation_log`: no schema change, now read by orchestrator
- `audit_log`: new event types — `dropzone_write`, `image_received`, `link_audit_broken`, `link_audit_stripped`, `link_audit_self`
- Pre-migration backup at `~/store/aiteam.db.pre-multisite-*.backup`

### ~/orchestrator_inbox/ (new)
- README.md
- Live test files from session (perm-test2.txt, telegram-photo-* from photo test)

---

## Cost data points

| Workload | Cost (Max-sub burn, NOT real $) |
|---|---|
| Outstanding-work audit | ~$0.50-1.00 |
| Phase A bookkeeping | <$0.50 |
| Internal Link SSG-fy + D091 (2.5h CC) | ~$2-3 |
| Orchestrator memory diagnosis + build | ~$1-2 |
| Typing indicator | <$0.25 |
| Dropzone build (with iterations) | ~$2-3 |
| **Session total** | **~$7-10 Max-sub burn** |
| **Real $ out of pocket** | **$0** |

Memory replay observed: input grew 242 → 3,765 tokens on first warm call, cache_read_tokens 12,963 on second internal call — caching working as expected. Per-message cost increase trivial against F17/F18 daily caps.

---

## Where we left off

**This chat's lane status:**
1. ✅ Phase A bookkeeping
2. ✅ Internal Link Agent SSG-fy + D091
3. ✅ Orchestrator memory wiring
4. ✅ Typing indicator
5. ✅ Orchestrator dropzone (text + image archive)
6. ⏳ Auto-Refresh Agent — deferred ~22 days for GSC data accumulation

**Sister chat's lane status:**
1. ⏳ Phase B launchd auto-restart (D026) — still owed
2. ✅ Issues Queue v1.1 — shipped concurrent with our session

**Forward-path photo archive** is the one piece of tonight's work not exercised by a live test. Watch for it if forwarded photos don't land in ~/orchestrator_inbox/.

**Next session candidates** (ranked):
1. Phone-test forward-path photo archive (operator-only, no CC needed)
2. Domain hunter CLAUDE.md cleanup (~15 min, trivial)
3. The 7 operator-policy questions (unblocks multiple deferred items — operator-only)
4. Launchd auto-restart (D026) — sister chat's lane, could be pulled if sister idle
5. Auto-Refresh Agent — wait ~22 days for data

---

## Mission-bar nuance reminder

Dashboard "API cost" figures are Max-sub burn, not real $. Tonight's ~$7-10 of CC work cost $0 in dollars. Only Max-subscription capacity consumed. F17/F18 caps ($25/$50 daily) absorbed everything with room to spare.
