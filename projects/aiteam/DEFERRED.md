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

