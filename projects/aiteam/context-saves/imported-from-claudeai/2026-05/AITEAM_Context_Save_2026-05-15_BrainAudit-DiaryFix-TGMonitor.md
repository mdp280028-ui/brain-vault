# AITEAM Context Save — 2026-05-15
## Topic: Brain Audit + Diary Fix + TG Monitor Agent (Phase A + B + Archive)

**Session date:** May 15, 2026 (~8 PM – past midnight, into May 16)
**Filename:** AITEAM_Context_Save_2026-05-15_BrainAudit-DiaryFix-TGMonitor.md
**Session length:** ~5 hours
**Operator:** Stonecreed (Bonners Ferry, ID)

---

## TL;DR

Massive build session. Three interlocking pieces shipped end-to-end with full audit trail:

1. **Brain audit** — CC produced a full read-only map of `~/brain/`; surfaced two issues (no diary for today, undocumented new top-level folders)
2. **Diary fix** — diagnosed root cause (never scheduled in cron despite script header claiming so), backfilled today's diary, added cron at 23:45 staggered ahead of the 23:55 brain-autocommit, retro-committed 3 sessions of stale Step 19 work under accurate commit message
3. **TG monitor agent** — built from zero to running fleet: Phase A scaffold + Telethon auth, Phase B reader + analyzer + digest pipeline, plus archive extension for `@supershadowy` capped at 30/day

By session close, three systems are running on cron with no operator involvement: diary writer at 23:45, brain autocommit at 23:55, TG reader every 5 minutes, TG analyzer at 07:00 daily. First real autonomous AITEAM activity.

Eight audit_log rows added today. Zero real-money cost (everything ran through Claude Max sub, dashboard "API cost" = $0 actual).

---

## SESSION ARC

Started with strategic context (Claude.ai recovering project state, drawing system map). Pivoted into infrastructure diagnostics (brain audit, diary gap) when those surfaced naturally. Major build wave landed when operator chose to push through Phase A + B + archive of TG monitor agent in one session rather than splitting across nights. Operator explicitly overrode my "stop here, /ctx, sleep" recommendation at three separate handoff points and was right each time — Phase B build went cleanly with no rework needed.

---

## DECISIONS MADE

### Infrastructure / Diary

- **Diary cron locked at 23:45**, 10 min ahead of brain-autocommit (23:55) so today's diary lands on disk before the nightly push
- **Step 19 closure committed under accurate message** — date-arg backfill, DAY_END SQL bound, expanded audit payload, header comment fix. Caught when CC bundled into the cron-fix commit with a misleading subject line. Retro-fixed via soft-reset + re-commit
- **D028 captured: hive_mind data-completeness gap** — non-AITEAM agents (briefer, Market Scribe) invisible to write_diary.sh's data source. Three candidate fixes documented; decision deferred to SSG Agent 1 build (sets precedent for whether new agents write hive_mind)

### TG Monitor Agent Architecture

- **Telethon (Python) over gramjs (Node.js)** — more mature for monitoring patterns, separate stack from Phase 2 friend-bot (Node/grammy) avoids dependency tangle
- **Burner phone with dead SIM works fine** — Telegram delivers auth code in-app to existing logged-in Telegram on burner phone, SMS fallback not needed
- **Read-only by design** — no methods that modify state (send, react, mark-read, join, leave). Read-only Telethon use is the lowest-ban-risk userbot pattern
- **Polling scope: 5 monitored groups only** — NOT all ~50 burner groups. Polling all 50 every 5 min would look like scraping behavior to Telegram's anti-userbot detection
- **5-min cron with 0-30s random jitter** — bots poll on the metronome; humans don't. Tiny detail, real signal
- **Hot-reload config.yaml** — every reader/analyzer run reads it fresh. Operator can edit groups/users any time, no rebuild needed

### TG Monitor Scope

- **5 monitored groups:** Ouroboros Protocol (#3), Helios Ecosystem (#14), Horizon.win (#17), GOLD MINE (#19), BuilDeFi.win (#37)
- **8 tracked users (digest highlights):** @supershadowy, @DigitalNomad948503, @Brzina, @DudeIsGuy, @Allykane32, @richland100, @James88888888, @ThePrairieBanana
- **Tracking scope: global** — these 8 are tracked across all 5 monitored groups (not per-group pairs)
- **Digest format: by_group** — group-by-group summaries; tracked individuals surfaced within each group's section
- **Digest delivery: Phase 2 bot to personal Telegram at 07:00 local daily**
- **Digest cost target: ~$0.05-0.15/day at API rates (= $0 real, on Max sub)**

### TG Monitor Archive Extension

- **Archive target: @supershadowy only** — Li (@DigitalNomad948503) NOT archived initially. Proof-of-concept pattern; extend if/when archive proves useful
- **Daily cap: 30 messages per user per local day** — resets at local midnight. Real heavy posters average 10-25/day; 30 leaves headroom without scraping signal
- **Forward-only, no backfill** — backfill across many groups is exactly the userbot-ban-bait pattern. Archive starts from feature ship time
- **Separate DB at `~/agents/tg-monitor/archive.db`** — isolated from monitoring messages.db for performance + wipeability
- **Best-effort writes wrapped in try/except** — archive failures NEVER block messages.db (monitoring is load-bearing; archive is bonus data)

### Explicit Non-Builds

- No backfill of historical messages from any user
- No polling of non-monitored groups (i.e., the other 45+ burner groups go unread by the agent)
- No archive of Li or anyone else beyond shadowy initially
- No daily/weekly archive summary — raw storage only, operator queries SQLite manually
- No launchd service (cron is sufficient)
- No web dashboard view for tg-monitor

---

## FILES CREATED OR MODIFIED

### On Mac Mini

**TG monitor (new agent):**
```
~/agents/tg-monitor/
├── .env                       (chmod 600, 4 secrets filled by operator)
├── .gitignore                 (venv/, session/, *.db, *.log, __pycache__/)
├── config.yaml                (5 groups, 8 tracked, 1 archive user, cap=30)
├── requirements.txt           (telethon, python-dotenv, pyyaml)
├── auth_test.py               (Phase A — auth + dialog list, also reusable for re-listing)
├── reader.py                  (Phase B — polls 5 groups every 5 min, archives shadowy)
├── analyzer.py                (Phase B — daily 07:00 digest, Haiku triage + Sonnet summary)
├── init_db.py                 (one-shot schema initializer for messages.db)
├── init_archive_db.py         (one-shot schema initializer for archive.db)
├── archive_helpers.py         (pure cap-decision function for testing)
├── test_cap.py                (verifies cap logic at 29/30/31)
├── README.md                  (operations + archive query examples)
├── session/burner.session     (Telethon auth, ~40KB, chmod 600)
├── messages.db                (WAL, 1000 bootstrap msgs, 8 tracked usernames matched)
├── archive.db                 (WAL, 0 rows initially — forward-only)
└── venv/                      (Python 3 + Telethon 1.43.2)
```

**Diary / cron fixes:**
- `~/agents/orchestrator/write_diary.sh` — header comment updated to `cron: 45 23 * * *`
- `~/agents/.git/` — commit `8560861` "diary: step 19 closure + cron schedule fix"
- `~/brain/projects/aiteam/diary/2026-05-15.md` — backfilled (will be overwritten by 23:45 cron tonight, which is correct)
- `~/brain/projects/aiteam/DEFERRED.md` — D028 appended
- `crontab` — added `45 23 * * * write_diary.sh` (already had `55 23 brain-autocommit`)
- `crontab` — added `*/5 * * * * reader.py` and `0 7 * * * analyzer.py`

### Audit log entries (8 today)

| ID | Action | Target | Significance |
|---|---|---|---|
| 160 | diary_written | 2026-05-15.md | Manual backfill |
| 165 | config_change | crontab | Cron line for diary added |
| 173 | agent_auth | tg-monitor | Telethon session persisted |
| 175 | agent_built | tg-monitor-phase-b | Reader + analyzer + digest live |
| 176 | agent_extended | tg-monitor-archive | Shadowy archive live |
| (others from earlier work — tuning_decision, slash_important_complete, etc.) | | | |

---

## TECHNICAL DETAILS WORTH PRESERVING

### TG Monitor Architecture (one-screen summary)

```
[5 monitored groups]
        ↓ every 5 min (cron + jitter)
   reader.py (Telethon)
        ↓
        ├─→ messages.db (all msgs from 5 groups)
        └─→ archive.db (only @supershadowy msgs, capped 30/day)

[messages.db]
        ↓ once daily at 07:00 (cron)
   analyzer.py
        ↓
        ├─→ Haiku triage (substantive vs noise per msg)
        ├─→ Sonnet summary (per-group)
        └─→ notify.sh → Phase 2 bot → Telegram → operator's phone
```

### TG Database Schemas

**messages.db tables:** `messages`, `poll_log`, `digest_log` (all WAL)
- Key indexes: `(chat_id, ts)`, `(is_tracked_user, ts)`, `(analyzed_at) WHERE NULL`
- UNIQUE: `(chat_id, tg_message_id)` prevents duplicate inserts

**archive.db tables:** `archive`, `daily_cap_log` (WAL)
- Key indexes: `(sender_username, ts)`, `(local_date, username)`
- UNIQUE: `(chat_id, tg_message_id)` on archive; `(local_date, username)` on cap_log

### Telethon Auth Flow Reality Check

- API ID + API hash obtained from my.telegram.org/apps (one-time, login via burner's existing Telegram app since SIM is dead)
- Burner's Telegram app on phone receives auth code via in-app notification from "Telegram" service account (NOT SMS, since SIM is dead)
- Session file written to `~/agents/tg-monitor/session/burner.session` after first successful auth
- Session persists indefinitely as long as: (a) file not deleted, (b) burner's Telegram session on burner phone stays active (Telegram inactivates after ~6 months of zero account activity), (c) API ID/hash not regenerated
- Emergency kill switch from burner phone: Settings → Devices → terminate "Telethon" session

### Phase 2 Bot Outbound IS Wired

The analyzer's test digest send succeeded via notify.sh (exit 0, no outbox fallback). This means the Phase 2 Telegram bot can both receive AND send. Worth noting — wasn't certain before this session.

### Cron State at Session Close

```
45 23 * * * /Users/mmm2/agents/orchestrator/write_diary.sh
55 23 * * * /Users/mmm2/agents/scripts/brain-autocommit.sh
*/5 * * * * cd /Users/mmm2/agents/tg-monitor && venv/bin/python reader.py
0 7 * * * cd /Users/mmm2/agents/tg-monitor && venv/bin/python analyzer.py
```

---

## LESSONS LEARNED

1. **CC reads real machine state more accurately than Claude.ai reads project files.** Trust CC's pushback on SQL schemas, git commands, file paths. Three real catches this session: `payload` vs `payload_json` column name; `ts` as unix-epoch integer not date string; bundled commit with misleading message.

2. **Burner safety > feature completeness.** First instinct was "archive shadowy across all 50 groups." That's exactly the userbot-ban-bait pattern. Scoping to monitored groups + daily cap is the right shape. Operator's "I just don't want my account banned" is the correct prioritization.

3. **Soft-reset + accurate re-commit is the right pattern when a commit bundles work under a misleading subject.** Don't accept inaccurate git history. Three sessions of uncommitted Step 19 work finally landed in git tonight under an honest message because we caught the bundle.

4. **CC making good unprompted decisions:** pulling cap logic into `archive_helpers.py` as a pure function + writing `test_cap.py` for boundary cases. Better engineering than I specified. The cap logic is now testable in isolation without Telethon.

5. **$0.07 dashboard "API cost" = $0 real cost** on Max sub. Dashboard label is misleading; D018 already captures this. Worth reframing all "expensive" decisions as Max-sub capacity decisions, not dollar decisions.

6. **Two-stage builds work better than monolithic specs.** Phase A (scaffold + auth) → operator gets API credentials → Phase B (reader + analyzer + digest). If auth had failed or Telegram had banned the burner on first connect, no reader code was wasted. This is the right pattern for any agent that talks to a third-party API.

7. **Hot-reload config beats rebuild-to-change.** All TG monitor params (groups, users, cap, schedule, format) live in config.yaml. Every cron run reads it fresh. Adding a 9th tracked user or 6th monitored group is a 30-second nano edit, not a rebuild.

8. **Cron entries that are documented but not installed are the silent gap.** The diary script's header said `cron: 55 23 * * *` for weeks; the cron line was never actually added. The script worked when manually invoked, so the gap went undetected. Phase 2 watchdog (D014) is the right detection layer, but the deeper lesson: documentation comments are not verification.

9. **Operator's "I don't want to" / "let's just" pushes are often right.** Three times tonight I recommended stopping; three times the operator pushed through and the work landed clean. The research-before-building risk in project instructions is real, but tonight was the inverse — momentum was real, fatigue hadn't compromised judgment yet.

---

## OPERATOR CORRECTIONS APPLIED

1. **"What was the amir guys name?"** — answered "Mark Kashef" (from project files). Trivial but worth noting as continuity check.

2. **SQL bug caught BEFORE I ran it** — operator-side via CC. My prompt used `WHERE ts >= '2026-05-15'`. CC noticed `ts` is INTEGER and the comparison would match every row. Fixed to `date(ts,'unixepoch','localtime')='2026-05-15'`.

3. **"I can't do it" on nano** — CC can't drive interactive editor. Pivoted to step-by-step manual instructions for the operator to fill .env directly.

4. **API hash format confusion** — operator initially looked at the RSA public key (long multi-line block) thinking it was the api_hash (32-char hex). Corrected before bad value got pasted. Worth knowing for future Telegram auth setups.

5. **"Phone starts with 1?"** — yes, US numbers need `+1` prefix in international format for Telethon. Caught before .env save.

6. **"Maybe we could just do shadowy and do a very small amount of messages per day or something?"** — better instinct than my "shadowy + Li, both archived" recommendation. Smaller scope = safer + proof-of-pattern before expanding.

7. **"Whatever you recommend, I just don't want my account banned"** — clear delegation with a hard constraint. Reframed every choice through that lens (polling scope, cap value, jitter, no-historical-backfill).

8. **"Add archive now"** — overrode my "do it tomorrow with fresh eyes" recommendation. Build went clean, no fatigue-induced bugs surfaced.

---

## DEFERRED ITEMS

| ID | Item | Trigger |
|---|---|---|
| D028 (new this session) | Diary data source incomplete — non-AITEAM agents invisible to hive_mind | When building SSG Agent 1 — sets precedent for whether new agents write to hive_mind |
| D018 | Project Instructions reflect Max-sub vs API spend distinction | Next instructions update |
| D013 | Budget enforcement hook (DAILY_API_BUDGET_USD) | When project moves to direct API instead of Max sub |
| D014 | Detection cron infrastructure (diary watchdog at 23:00, stuck-inbox watchdog, etc.) | Phase 2 (after launchd is established) |
| D015 | Per-agent switch bypasses via wrappers/slash commands | Phase 2 polish |
| D016 | PUBLISHER_DEPLOY_ENABLED enforcement | When publisher agent built |
| D017 | rubric.md change-detection / version pinning | When editor goes into production |
| D019 | Strategic: Phase 2 (infra) before or after Phase 3 (first site)? | Largely resolved by SSG adoption (SSG = first site, parts of Phase 2 are built/built-tonight) |
| D001 | Editor cascade pattern (Haiku triage → Sonnet final gate) | Publishing ≥100 articles/week |
| D002 | Kanban view (step 10b) | When any agent emits mission_tasks rows |
| D025 (from earlier sessions, unresolved) | Orchestrator permissions for Telegram operational queries (settings.json allowlist vs deterministic slash commands) | Operator decision before Phase 3, or earlier if Telegram queries dead-end |
| **Li archive consideration** | Whether to extend archive feature to @DigitalNomad948503 | When shadowy archive has 30+ days of data and operator has actually queried it |
| **Phase 2 telegram-stack retro-commit** | 8 modified + 6 untracked files in ~/agents/ per HANDOFF.md | Next CC session, end-of-day cleanup |

---

## WHAT'S NEXT

### Tomorrow morning (May 16)

1. **Wake up, check personal Telegram** — first cron-fired digest should land at 07:00 local
2. **If digest landed cleanly** → TG monitor is verified end-to-end; next move is SSG Agent 1
3. **If digest didn't land** → diagnostic:
   ```
   sqlite3 ~/agents/tg-monitor/messages.db "SELECT digest_date, delivery_method, error_text, length(digest_text) FROM digest_log;"
   tail -50 ~/agents/tg-monitor/analyzer.log
   ```
4. **Glance at reader.log** for any errors overnight: `tail -50 ~/agents/tg-monitor/reader.log`
5. **Check shadowy archive populated:**
   ```
   sqlite3 ~/agents/tg-monitor/archive.db "SELECT COUNT(*), MIN(ts_local), MAX(ts_local) FROM archive WHERE sender_username='supershadowy';"
   ```
6. **Verify diary cron fired** — `ls -la ~/brain/projects/aiteam/diary/2026-05-15.md` should show a much larger file than the 18-line backfill (overnight overwrite at 23:45 covers full day)

### Next real build (after verification)

**SSG Agent 1 — Keyword Screener.** This is the unblocked next move per the May 15 strategic session:
- Haiku tier, cheap and mechanical
- Validates Haiku-for-bulk pattern
- Pairs with SSG (existing site with 17 pages indexing) — first real revenue-adjacent workload
- D028 (hive_mind visibility) gets decided as part of Agent 1 scoping

### Tomorrow's session opening line

If next Claude.ai session is "verify TG monitor + start SSG Agent 1":
- Read this context save
- Confirm 07:00 digest delivery
- Write Agent 1 build prompt (Keyword Screener for SSG, Haiku tier, Ahrefs CSV → ranked candidates)

---

## OPEN LOOPS / FLAGS

1. **Phase 2 telegram-stack retro-commit** — 8 modified + 6 untracked files in ~/agents/ per HANDOFF.md. Surfaced 3× now. Needs a focused commit-pass to clean up.

2. **23:45 diary cron will OVERWRITE today's backfill tonight** — expected behavior, not a bug, but operator was warned. Tomorrow's `2026-05-15.md` file will be a fresh comprehensive write covering full day, not the truncated 18-line manual backfill made at 21:56.

3. **953 bootstrap messages marked `analyzed_at IS NULL` permanently** in messages.db — spec'd behavior (bootstrap → forward-only). They're searchable via ad-hoc SQL queries but excluded from digest analysis.

4. **First real `mission_tasks` row still pending** — kanban view (D002) defers until then. SSG Agent 1 will likely be the first agent that writes mission_tasks.

5. **Channels not monitored** — only 5 groups in current scope. Worth revisiting whether 1-2 channels (GuU's Clues, CF Market Updates, don't bet the house) deserve monitoring after first week of digests reveals signal-to-noise patterns.

---

## QUOTES WORTH PRESERVING

> "im not really using api right? I'm on a laude sub max so what was the real cost? no .07 right?"
*(Right framing — caught the Max-sub vs API distinction. Reinforces D018.)*

> "I just don't want my account banned whatever you think is safe"
*(Clear delegation with one hard constraint. Reframed every TG monitor design choice through this lens.)*

> "build it"
*(After full safety walkthrough. Cleanest "yes" of the session.)*

> "no we finish setting up the groups."
*(Override #1 on my "stop here" recommendation. Right call.)*

> "Add archive now"
*(Override #2 on my "do it tomorrow with fresh eyes" recommendation. Build went clean, no rework.)*

---

## OUTPUT FILES FROM THIS SESSION

- This file: `AITEAM_Context_Save_2026-05-15_BrainAudit-DiaryFix-TGMonitor.md`

---

## CONTEXT FOR NEXT CLAUDE

- This context save covers a long single session that wove together infrastructure cleanup (diary fix) and new agent build (TG monitor). Both shipped.
- Phase 1 of AITEAM remains complete from previous sessions; this session added a parallel-track agent (tg-monitor) that doesn't formally close any Phase 1 step but is operating on its own
- The 5 monitored TG groups are all crypto/TitanX ecosystem — that's where operator's networks live. Treat as locked unless operator specifically revisits
- Burner-account safety is the load-bearing constraint for tg-monitor. Any future modifications must preserve: read-only, ≤5 polled groups, daily cap on archive, no historical backfill
- Operator successfully overrode my "stop and rest" recommendations 3× in this session — momentum was real, no rework happened
- SSG Agent 1 (Keyword Screener) remains the next revenue-adjacent build — unchanged from previous strategic session
- D028 is the new high-priority deferred item; resolves at SSG Agent 1 build time
- Project instructions still describe a 14-agent BRAIN v2 architecture; what's actually live is: orchestrator, librarian, editor, tg-monitor (4 agents). SSG Agent 1 would be 5th. The "fleet" is smaller than the planning docs but is shipping real systems

— End of context save —
