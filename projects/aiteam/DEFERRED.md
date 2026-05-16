# AITEAM — Deferred items

Engineering decisions that we've consciously kicked down the road. Each item names a trigger ("when to revisit") so they don't rot quietly.

---

## D025 — Orchestrator permission policy for Telegram-spawned sessions

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** When `ai-do.sh` runs `claude -p` non-interactively (as the Telegram bot does for every routed message), no `~/agents/.claude/settings.json` exists, so the spawned session falls back to default policy. Default policy denies `Read` on scripts in `~/agents/lib/` and on `~/store/aiteam.db`, which the orchestrator needs for operational meta-queries (e.g. "what's our token spend today?", "any unauthorized access in the audit log?"). The model can't get permission interactively, so it surfaces the denial in chat as "I need read permission for the lib directory…" — confusing UX, but the bot plumbing is correct.

**Discovered:** Phase B live sign-off voice tests (2026-05-15). Phase A text tests dodged it because "what's our SSD mount path?" was answerable via `Bash(df)`, which is allowed by default.

**Two candidate fixes:**

- **(a) Allowlist.** Write a `~/agents/.claude/settings.json` with `permissions.allow` covering the orchestrator's working scope: `Read(./lib/**)`, `Read(~/store/**)`, `Read(~/brain/projects/aiteam/**)`, `Bash(sqlite3:*)`, etc. Fast (~10 min) but fragile — every new operational query risks hitting another denied tool.
- **(b) Deterministic slash commands** for operational meta-queries: `/spend [period]`, `/audit_recent [n]`, `/tasks_open`, `/kill_switches`, etc. Each runs a sqlite query (or shell) and returns the answer without an LLM call. Architecturally cleaner, aligns with the [[project-phase2-phone-first]] principle, costs nothing per query, and never dead-ends on permissions. Slower to build (~30–45 min for a small starter set).

**Recommendation:** (b). Burns no tokens for queries that are deterministic, doesn't grow an ever-leakier allowlist, and operator gets faster responses.

**Trigger to revisit:** after Phase 2 closes (all four sub-phases A–D signed off), before Phase 3 begins. **OR** sooner if the next routed Telegram operational query dead-ends on the same permission wall and operator wants it answered.

**Workaround in the meantime:** for Phase B sign-off, operator uses voice prompts the orchestrator can already handle with default tools (git, web search, files in orchestrator's own working dir).

---

## D026 — Phase C launchd auto-restart for Telegram bot

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** Bot currently runs in foreground (started via `node bot.js` from a terminal). Dies on terminal close, Mac sleep, crash, or reboot. No supervisor process.

**Operator's stated reason for deferring:** "computer hardly ever shuts off."

**Real risk:** silent failure when the bot dies and operator doesn't notice until a voice note vanishes into nothing or a friend reports the bot stopped responding. The longer between "bot died" and "operator notices," the more confused-looking the symptom (operator thinks the message system is broken, when actually just the bot exited).

**Same exposure on related stack:** Friend-bot at `~/projects/friend-bot/` (which I can see running as `caffeinate -i python bot.py`, PID was 33238 during Phase B) has identical lack-of-supervisor risk. Captured as D023 (or whatever the friend-bot equivalent is — flag if not already).

**Planned scope when un-deferred:**
- `~/Library/LaunchAgents/com.aiteam.telegram.plist` with `RunAtLoad: true`, `KeepAlive: true`, log paths to `~/agents/telegram/logs/stdout.log` & `stderr.log`, `EnvironmentVariables` including Homebrew + python paths.
- `launchctl bootstrap gui/$(id -u)` (the modern API; `launchctl load -w` is deprecated on recent macOS).
- Verify `kill -9 <pid>` triggers auto-restart within seconds.
- Operator-verified reboot survival.
- Manual start/stop/restart docs at `~/brain/projects/aiteam/docs/restore.md`.

**Trigger to revisit:**
- (a) first time the bot dies silently and operator notices something broke, OR
- (b) before any agent that publishes externally (Twitter, Pinterest, etc.) goes live — those *must not* miss a beat, OR
- (c) next working session if operator wants to fully close Phase 2 with all four sub-phases signed off.

**Estimate when revisited:** 1–1.5 hrs build + operator-time for the reboot test.

---

## D027 — Add `*.bak` to `~/brain/.gitignore`

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** The `/ctx` pipeline writes a pre-narrative skeleton backup at `<save_path>.bak` before splicing the orchestrator's narrative into the final save file. These `.bak` files get picked up by the nightly brain-autocommit and committed to the brain repo, bloating the vault with content that's already represented (and superseded) by the final save.

**Observed:** Commit `87d9f1a` (2026-05-15 17:26) included `AITEAM_Context_Save_2026-05-15_1614.md.bak` alongside the real save. Not harmful — it's just a stale skeleton — but pure noise in `git log` diffs.

**Fix:** Add a single line to `~/brain/.gitignore`:
```
*.bak
```

Then `git rm --cached` the existing `.bak` files so they stop being tracked.

**Estimate:** 2 min.

**Trigger to revisit:**
- (a) next time `~/brain/.gitignore` is being edited for any other reason — fold this in.
- (b) when `.bak` files start showing up in `git log` diffs frequently enough to be in the way.
- (c) before any consumer (a future dashboard, search index, anyone scrolling the vault) starts pulling junk results from `.bak` files.

---

## D028 — Diary data source incomplete — non-AITEAM agents invisible to hive_mind

**Status:** Deferred (Phase 2, 2026-05-15)

**Problem:** `write_diary.sh`'s `AGENT_RUNS` query reads from the `hive_mind` table, but Briefer, Market Scribe (scribe/analyst), Curator, and other CC-built agents don't write to `hive_mind` — only their token spend lands in `token_usage` and their state-transition rows land in `audit_log`. As a result, the diary's "What ran" section undercounts (or zeroes out) daily activity.

**Observed:** Backfilled diary for 2026-05-15 reported *"No agent runs recorded today. All activity was direct orchestrator/operator interaction."* despite the briefer running twice (21:27, 21:40) and the Market Scribe pipeline shipping 10 steps earlier in the day. Spend was visible ($1.91) but the agents producing it were invisible to the diary.

**Three candidate fixes:**

- **(a) Instrument the agents.** Have briefer/scribe/analyst/curator each write a `hive_mind` row per run. Smallest change to `write_diary.sh`; new agents must remember to do this, so the gap reopens with every new agent unless the convention is codified.
- **(b) Expand the diary's source.** Drop the `hive_mind` query; derive agent activity from `token_usage` (group by `correlation_id` or by `agent_id`/`model+ts-bucket`) and/or `audit_log` (group by `actor_id` with action `^_run$` or similar). Single source of truth; cleanly captures historical runs even for agents that pre-dated the fix.
- **(c) Scope the diary explicitly.** Rename the section from "What ran" to "AITEAM-fleet — what ran" and accept that off-fleet agents (built ad-hoc via CC) are intentionally invisible. Smallest change; punts the visibility question.

**Trigger to revisit:** When building **SSG Agent 1 (Keyword Screener)** — set the precedent for whether new agents write to `hive_mind`. The decision made there carries forward to every subsequent SSG-stack agent and locks in (a), (b), or (c) by implication.

**Date observed:** 2026-05-15

