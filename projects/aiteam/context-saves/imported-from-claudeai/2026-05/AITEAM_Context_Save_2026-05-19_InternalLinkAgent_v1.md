# AITEAM Context Save — Internal Link Agent v1 Shipped + D066 ~/agents/ Closed

**Date:** 2026-05-19 (afternoon session, ~16:00–19:00 PT)
**Session length:** ~3 hours
**Chat role:** Secondary chat, queue swap with sister mid-session
**Session character:** Two closures (D066 ~/agents/ half + Internal Link Agent v1). Queue re-shuffle resolved cleanly. Four CC HALTs caught real decisions. One bot-off blocker handled by partial-pass close. $0.07 build spend.

---

## TL;DR

**D066 ~/agents/ half closed.** Recon found working tree clean — earlier 2026-05-17 audit headline ("15+ uncommitted files") was swept by intervening commit discipline (D068, D056, D070 backup). No dedicated audit session needed. DEFERRED.md follow-up validation block appended in brain commit `c9e83f4`. Audit row `E5C322E4-2D48-40EB-AC92-5FC6622EEED4`.

**Internal Link Agent v1 shipped.** 4 commits across 3 repos. Inventory builder + audit-links (advisory mode) + backlink-proposal (Telegram approval per proposal) + dual-tier relatedLinks/suggestedLinks UI. Wired into pipeline at three points: writer audit (advisory), post-deploy backlink proposals, updated_slugs_queue.txt consumed by deploy_batch.sh. Build cost $0.07 vs $5.00 cap.

**Mission bar:** $0 / $200/mo unchanged. Foundation stronger — every new deploy now triggers backlink proposals from existing inventory. Compounds with sister chat's GEO Optimizer when that lands.

---

## SESSION ARC

1. Opened with chat-queue confusion. Original 2026-05-19 save had this chat owning D056/Backlink/GEO. Sister chat clarified mid-session: queues swapped wholesale. This chat now owns D066 ~/agents/ half + Internal Link Agent + Research/Opportunity Agent.

2. D066 ~/agents/ half recon-only fired. Working tree clean. Two unpushed sister commits (`1323040` D056 editor gate, `dc05c51` threshold tune) — autocommit at 23:50 handles. No commit-grouping needed.

3. D066 close-out: brain commit `c9e83f4` appended follow-up validation block to existing D066 entry (closed 2026-05-17). Audit row logged with 5 positional args (D064 validation).

4. Internal Link Agent v1 pre-flight surfaced 5 findings:
   - JSON schema lacks `topics`/`mentioned_entities` fields — synthesize at backlink-proposal time from h1+h2s+first paragraph
   - `relatedLinks` already human-curated — dual-tier with separate `suggestedLinks` array
   - Migration lane sister-coordination — sister GO'd shared lib/migrations/ convention
   - Schema migration filename used 2026-05-17 (real wall-clock, not session date drift)
   - Pre-existing ship-to-site "no per-ship approval" convention — divergence intentional for backlink proposals (different stage, different gate)

5. Build prompt fired with v1 scope baked in: inventory + audit-links + backlink-proposal. Suggest-for-draft deferred to v2.

6. Four CC HALTs during build, all resolved with operator decisions:
   - **HALT 1 (deploy path):** Option 2 — queue + 23:00 batch via `updated_slugs_queue.txt`
   - **HALT 2 (bot.js handlers):** Add step 10.5 for `/approve` + `/reject` handlers
   - **HALT 3 (advisory vs hard gate):** Option 1 — advisory mode, log to `audit_link_failures` table, never block
   - **HALT 4 (failure routing):** Option 1 — log only, no file moves
   - **HALT 5 (Step 14 close):** Option 2 — visual render verification via local `npm run build`, defer live Telegram + re-ship to next session

7. CC closed Step 16 with 16/16 steps complete (2 documented partial-passes for bot-off state).

---

## WHAT GOT SHIPPED

### D066 ~/agents/ half closure
- Brain commit `c9e83f4` — DEFERRED.md follow-up validation block
- Audit row `E5C322E4-2D48-40EB-AC92-5FC6622EEED4`

### Internal Link Agent v1

**Commits:**
| Repo | SHA | Subject |
|---|---|---|
| `~/agents/` | `928671a` | feat(internal-link): v1 agent — inventory + audit-links + backlink-proposals |
| `~/agents/` | `ec3f087` | feat(deploy-batch+telegram): wire internal-link agent into pipeline |
| `~/projects/asbestos-contractors/` | `370f62c` | feat(run-batch+chrysotile): advisory audit gate + first approved backlink |
| `~/projects/asbestoshq-site/` | `41c5429` | feat(GuideArticle): render suggestedLinks as 'Related Reading' section |

**New SQLite tables (migration `2026-05-17_internal_link_inventory.sql`):**
- `internal_link_inventory` — per-slug snapshot of h1/h2s/first_para/relatedLinks/suggestedLinks (32 rows)
- `backlink_proposals` — proposed/approved/rejected/shipped status per proposal
- `audit_link_failures` — broken `relatedLinks` references (advisory)

**New scripts in `~/agents/internal-link/`:**
- `build_inventory.sh` — scans approved/guides/*.json, populates inventory (idempotent)
- `audit_links.sh` — verifies relatedLinks targets exist (advisory)
- `propose_backlinks.sh` — Sonnet picks 3-5 existing slugs that should add inbound link to new slug
- `apply_approval.sh` — writes approved suggestedLinks back to JSON + appends slug to updated_slugs_queue.txt

**New bot.js handlers (deferred live test):**
- `/approve <id>` — spawn apply_approval.sh approve
- `/reject <id>` — spawn apply_approval.sh reject

**Pipeline wiring:**
- `run-batch.sh` (asbestos-contractors): advisory audit gate after audit_guide.py passes
- `deploy_batch.sh` (~/agents/ship-to-site/): post-deploy `propose_backlinks.sh` hook + `updated_slugs_queue.txt` consumer with new/refresh column

**UI:**
- `GuideArticle.tsx` renders "Related Reading" section for `suggestedLinks` (separate from existing relatedLinks visual treatment)

### Audit row UUIDs
| Event | UUID |
|---|---|
| inventory_built | `331CD649-C211-479C-BA4A-C95FB053B2BC` |
| build_v1_shipped | `99BCB914-0000-4EC0-A60B-A24EAF97A7B3` |
| D066 follow-up close | `E5C322E4-2D48-40EB-AC92-5FC6622EEED4` |

### Cost
$0.0746 — one real Sonnet call (Step 9 smoke on white-asbestos-vs-blue-asbestos, 5 proposals). 1.5% of $5.00 build cap.

---

## KEY FINDINGS / DECISIONS

### Queue ownership swap mid-session
2026-05-19 morning save assigned this chat D056/Backlink/GEO. Sister chat re-assigned wholesale: this chat now owns D066 ~/agents/ + Internal Link + Research/Opportunity. Pattern: when queues drift in parallel chats, surface explicitly and re-anchor. Operator's word is ground truth.

### D066 closed twice (intentional)
Original closure 2026-05-17 (audit row `F9D65187-…`). Today's session appended a "Follow-up recon ~/agents/ half (2026-05-19)" validation block inside the existing entry. Structurally clean — doesn't re-open the original closure, just records that the ~/agents/ half got verified.

### Approval-pattern divergence from ship-to-site
Ship-to-site README: "No per-ship Telegram approval (auditor is the gate)." Internal Link Agent v1: per-proposal Telegram approval. Documented in `~/agents/internal-link/CLAUDE.md` as intentional — backlink proposals are NEW unreviewed agent output editing already-approved content. Different stage, different gate.

### Topic-source decision: synthesize, don't add to schema
Adding `topics` / `mentioned_entities` to guide JSONs would ripple to writer prompt + audit_guide.py. Synthesizing at propose-time from h1+h2s+first_para via Sonnet is cheaper, no schema touch, no backfill. Upgrade to embeddings later if 500+ slugs makes per-propose calls expensive.

### Advisory audit gate (not hard gate)
Cross-batch `relatedLinks` references (writer points at sibling slug still in drafts/) would false-positive on a hard gate. Advisory mode logs to `audit_link_failures` table, doesn't block approval. Observe false-positive rate before deciding whether to enforce.

### CC labeled the build "D070" in sign-off
Internal Link Agent doesn't have a D-number (it's a build, not a deferred item). D070 is the agents-vault backup, closed 2026-05-17. Cosmetic label drift — flagged for cleanup if relevant.

### Date drift between sessions
This chat's system clock reported 2026-05-17, but session has been using 2026-05-19. Sister chat is on 2026-05-17. Migration filename uses 2026-05-17 (real wall-clock). Note for hygiene pass — the D066 close-out used 2026-05-19 in the closure block at operator's instruction. May want correction later.

---

## FILES CREATED / MODIFIED

### In `~/agents/`
- `internal-link/agent.yaml` — new
- `internal-link/CLAUDE.md` — new (with HALT-3 rationale + HALT-4 ongoing cost note)
- `internal-link/build_inventory.sh` — new
- `internal-link/audit_links.sh` — new
- `internal-link/propose_backlinks.sh` — new
- `internal-link/apply_approval.sh` — new
- `internal-link/workspace/responses/` — directory for raw Sonnet response persistence
- `lib/migrations/2026-05-17_internal_link_inventory.sql` — new
- `ship-to-site/deploy_batch.sh` — modified (updated_slugs_queue.txt consumer + propose_backlinks hook)
- `ship-to-site/state/updated_slugs_queue.txt` — new (chrysotile queued)
- `grammy-bot/bot.js` — modified (/approve + /reject handlers added)

### In `~/projects/asbestos-contractors/`
- `content/run-batch.sh` — modified (advisory audit gate after audit_guide.py pass)
- `content/asbestos/approved/guides/chrysotile.json` — modified (first approved suggestedLinks entry)

### In `~/projects/asbestoshq-site/`
- `src/components/GuideArticle.tsx` — modified (suggestedLinks type + "Related Reading" render section)

### In `~/store/aiteam.db`
- 3 new tables (internal_link_inventory, backlink_proposals, audit_link_failures)
- 5 backlink_proposals rows (1 approved, 1 rejected, 3 still proposed)
- 32 internal_link_inventory rows

### In `~/brain/projects/aiteam/`
- `DEFERRED.md` — D066 follow-up validation block appended (commit `c9e83f4`)

---

## DEFERRED ITEMS

### New this session
- **Live `/approve` + `/reject` Telegram round-trip** — bot was off, handler pattern matches existing handlers exactly, `apply_approval.sh` proven end-to-end directly. Trigger: next session with bot up.
- **Actual re-ship of chrysotile via deploy_batch.sh** — `updated_slugs_queue.txt` has chrysotile queued; next 23:00 cron consumes it, or operator can fire manually. Visual render verified locally via `npm run build`. Trigger: next 23:00 deploy_batch.sh cron OR operator manual fire.
- **Cost-monitoring after 5 real deploy cycles** — propose_backlinks.sh runs $0.07-0.80 per cycle. Trigger: after first 5 real deploys, review token_usage table for actual cost distribution.
- **Batch approval flow (v1.1)** — per-proposal Telegram approval is v1 shape; batch (3-5 proposals/day in one message) deferred until burn-in completes. Trigger: after first 10 proposals burn-in successfully.
- **suggest-for-draft (v2)** — writer-phase internal-link suggestion not implemented in v1. Trigger: after v1 produces useful backlink graph (~30 days).
- **D-number assignment for these items** — none of the above got D-numbers in DEFERRED.md this session. Logged here in context save only.

### Carried forward, no change
D025, D026, D028, D033, D039, D040, D041, D042, D043, D052, D053, D055, D057, D058, D060, D063, D065, D069, D071, D072, D073, D074, D-SSG-01 through D-SSG-09 (D-SSG-05 closed earlier).

### Closed this session
- D066 ~/agents/ half — follow-up recon found tree clean, swept by D068/D056/D070

### Outstanding DB state for operator triage
- `backlink_proposals` has 3 proposals still in `proposed` status (ids #2, #3, #4 — siblings of approved #1 against white-asbestos-vs-blue-asbestos). Sensible Sonnet picks; operator can approve/reject via Telegram (once bot is up) or manual sqlite3.

---

## HANDOFF — NEXT CHAT

### Read in this order
1. **This context save** (you're reading it)
2. `~/brain/projects/aiteam/PRIME.md` — points at HANDOFF / POLICY / DEFERRED
3. `~/brain/projects/aiteam/POLICY.md` — 7 locked operator policies
4. `~/brain/projects/aiteam/HANDOFF.md` — current state
5. `~/brain/projects/aiteam/DEFERRED.md` — D072/D073/D074 are sister-chat additions worth scanning

### Current state at session close

| Surface | State |
|---|---|
| `~/agents/` | HEAD post-build, 2 unpushed commits from earlier sister work + this session's 2 new commits |
| `~/projects/asbestos-contractors/` | HEAD `370f62c` |
| `~/projects/asbestoshq-site/` | HEAD `41c5429` |
| `~/brain/` | `c9e83f4` from this session + pre-existing dirty state, brain-autocommit at 23:55 handles |
| Sister chat | Building GEO Optimizer (their queue: D067 → D068 → D056 → Backlink → GEO) |
| Mission bar | $0 / $200 mo. Revenue still gated on operator-side work (slugs into drafter_queue.txt + AdSense enrollment). |

### Opening moves for next chat

**1. Confirm overnight crons fired.** 
- `cd ~/agents && git log origin/main..HEAD --oneline | wc -l` — expected ~0
- `cd ~/brain && git log origin/main..HEAD --oneline | wc -l` — expected ~0
- Check 23:00 deploy_batch.sh consumed `updated_slugs_queue.txt` — chrysotile should be live with Related Reading section visible at asbestoshq.com/guides/chrysotile

**2. Bot-up + Telegram round-trip test.** If operator turns bot on, fire `/approve 2` (or `/reject 2`) against existing proposed backlink. Verifies handler end-to-end.

**3. Remaining queue for this chat:** Research/Opportunity Agent (#10 in master rankings, 4-6h CC). Reddit pain-point mining, surfaces conversational queries for drafter queue. Last item before this chat's queue empties.

### Critical context for next Claude

- **Internal Link Agent v1 is LIVE.** Every new asbestos slug deploy now triggers `propose_backlinks.sh` automatically. First production cycle will fire on the next real (non-chrysotile-refresh) deploy.
- **Advisory audit gate is LIVE in run-batch.sh.** Failures log to `audit_link_failures` table, do NOT block approval. Observe false-positive rate before deciding to enforce.
- **`updated_slugs_queue.txt` is a NEW state file.** Different from `drafter_queue.txt` (new authoring) — this queue is for re-deploys (suggestedLinks edits to existing approved guides). deploy_batch.sh consumes both at 23:00.
- **Per-proposal Telegram approval pattern is intentional divergence** from ship-to-site's "auditor is the gate" rule. Documented in `~/agents/internal-link/CLAUDE.md`. Don't "fix" the inconsistency.
- **The 3 outstanding proposals** (ids #2, #3, #4) are real — sensible Sonnet picks against white-asbestos-vs-blue-asbestos. Operator can approve/reject when bot is up.
- **D-number cleanup may be wanted** — Internal Link Agent build items above don't have D-numbers, only context-save entries. Next brain-touching session can assign if hygiene matters.

### Do NOT

- Re-derive any of the 4 baked-in decisions (synthesize topics / dual-tier links / shared migration lane / v1 scope without suggest-for-draft).
- Touch `~/agents/internal-link/` without checking sister chat isn't also building there.
- Force-push to either remote — both have nightly auto-push.
- Try to "fix" the ship-to-site vs internal-link approval-pattern divergence — it's intentional.

---

## LESSONS LEARNED

### Queue confusion in parallel chats is real
This chat opened thinking it owned D056/Backlink/GEO per 2026-05-19 morning save. Sister had picked those up. Resolved by surfacing the mismatch explicitly and asking sister "what are you working on?" Pattern: when timelines feel off between parallel chats, ask before acting.

### Pre-flight HALTs caught real decisions, not just bugs
Five HALTs in one build, all surfaced real operator decisions (deploy path, approval gate strictness, failure routing, etc.) that the build prompt didn't pre-resolve. Pattern continues: pre-flight is the cheapest place to surface forks.

### `g` rule pays off
Operator flagged mid-session: "you are writing way too much, limit to 300 words." Adjusted, sessions ran faster. Note for future: default to short, expand only when depth would change the decision.

### Bot-off state requires its own protocol
"Test each command before proceeding" is sound spec but doesn't account for bot-off state. Partial-pass with documented gap is the right close — don't start the bot just to satisfy a test step.

### Visual render check ≠ type check
GuideArticle.tsx type-checked clean, but visual render only verified via `npm run build` + eyeball. Type system catches structural bugs, not CSS/layout. Worth a default for any UI-touching agent build.

---

## SIGN-OFF NOTE

Two closures (D066 ~/agents/ half, Internal Link Agent v1). 5 commits across 3 repos + 1 brain commit. $0.07 build spend. Five CC HALTs caught real decisions. Two documented partial-passes (bot-off state).

Mission bar: $0 / $200 mo. Foundation harder — autonomous loop now has authority-compounding via backlink proposals on every new deploy. Revenue still gated on operator-side work (slugs + AdSense).

Next chat opens by reading this file, verifying overnight crons, then picking Research/Opportunity Agent (~4-6h CC) as the last item before this chat's queue empties.

---

*End of context save. Resume next session by reading this file first, then PRIME.md (which points at POLICY/HANDOFF/DEFERRED). Confirm overnight crons fired + check chrysotile is live with Related Reading section before any new work.*
