# AITEAM Context Save — Friend-Bot Build Session
## Date: 2026-05-14 | Topic: Telegram bot for friend in Italy, powered by Claude Max sub via CLI subprocess

---

## TL;DR

In one ~3-hour session (roughly 11pm to 3am local), Stonecreed went from zero working code to a deployed, internet-enabled, memory-equipped Telegram bot running on the Mac Mini M4 and serving a friend traveling in Italy. The bot uses Claude Code CLI as a subprocess (bills against Max subscription, not API), supports text + photos + web search + 48-hour SQLite memory, and is locked-down with an allowlist pattern that ships open by default and tightens after first contact.

This is **agent #0** in the Stonecreed fleet. The patterns established here (.env with chmod 600, venv activation, subprocess-safe Claude calls, allowlist gating, structured logging, prompt-as-file, SQLite per-key isolation, caffeinate for always-on) all carry forward directly to Scout/Writer/Editor/etc.

---

## What was actually built

### Location on disk
```
~/agents/friend-bot/
├── .env                  (TELEGRAM_BOT_TOKEN, allowlist) — chmod 600
├── bot.py                (436 lines)
├── system_prompt.txt     (~3 KB Italy travel companion persona)
├── requirements.txt      (python-telegram-bot, python-dotenv)
├── memory_prompt.txt     (53 lines, the spec used to add memory)
├── bot.log               (rolling log)
├── bot.db                (SQLite, per-chat conversation history)
└── venv/                 (isolated Python env, python-telegram-bot 22.7)
```

### Architecture (final state after all three feature waves)
```
Telegram → polling loop (python-telegram-bot 22.7)
  → handle_message / handle_photo / cmd_forget / cmd_start / cmd_whoami
    → allowlist check (chat_id gate)
      → fetch last 48hr / 20 msgs from bot.db for this chat_id
        → subprocess: claude -p <prompt with <conversation_history> tags>
                              --permission-mode bypassPermissions
                              --allowed-tools "WebSearch,WebFetch,Read"
                              --append-system-prompt <system_prompt.txt>
          → split reply into Telegram-sized chunks
            → save user turn + assistant turn to bot.db
              → reply back via Telegram
```

### Key technical decisions

| Decision | Rationale |
|---|---|
| Claude Code CLI subprocess over Anthropic API | Uses Max sub, no per-token billing |
| `bypassPermissions` mode | `-p` (print mode) can't show approval prompts; subprocess would hang otherwise |
| Narrow `--allowed-tools` to WebSearch/WebFetch/Read | Bypass mode + narrow tool list = safe; no Bash/Write access |
| Allowlist starts OPEN, tightens after first contact | Lets friend's first message log their chat_id; trade for one night of risk |
| SQLite for memory, not in-memory dict | Survives bot restarts, easy to inspect, scales fine |
| Per-chat_id isolation in DB | Same DB file, but queries always WHERE chat_id = ? |
| stdin=DEVNULL on subprocess | Prevents `claude -p` hang on missing stdin |
| Strip ANTHROPIC_API_KEY from subprocess env | Belt-and-suspenders against accidental API billing |
| `caffeinate -i python bot.py` for runtime | Prevents Mac sleep killing the bot |

### Feature waves shipped this session

1. **v1 — Core chat relay.** Telegram polling → claude -p subprocess → reply. ~258 lines bot.py. Italy travel system prompt. Allowlist scaffolding (defaults OPEN).
2. **v2 — Web access.** Added `--permission-mode bypassPermissions` and `--allowed-tools WebSearch,WebFetch`. Updated system prompt to encourage tool use over disclaimers. Unlocked: live train times, weather, hours, news, strikes, exchange rates.
3. **v3 — Image input.** Added `handle_photo` handler. Downloads largest resolution photo to temp file, passes path to Claude with Read tool. Caption optional. Temp file cleanup in finally block. Unlocked: menu translation, sign reading, dish identification, monument recognition, transit signage decoding.
4. **v4 — Memory.** SQLite `bot.db` with messages table keyed on (chat_id, timestamp, role, content). `get_recent_history` fetches last 48hr / 20 messages. History prepended to each user message inside `<conversation_history>` tags. `/forget` command wipes per-chat. Photo turns save as `[user sent a photo]` for the user role.

---

## Critical pitfalls hit and solved

### 1. Smart quotes corrupting `.env`
**Symptom:** `wc -c .env` showed 53 bytes (impossible for a real token line), `head -c 25` showed `"8"702486621:AAEaJGpT` — curly quotes around the leading digit.
**Cause:** Pasting through some app or text editor that auto-replaces straight quotes with typographic ones. Python's dotenv parser treats the smart-quoted line as garbage and reports the variable as missing.
**Fix:** Rewrite `.env` from terminal with `echo 'TELEGRAM_BOT_TOKEN=...' > .env` — single quotes are literal in zsh.
**Lesson:** For any file containing secrets, prefer `echo` over `nano` — terminal echo can't introduce typographic substitutions.

### 2. Missing newline between `.env` lines
**Symptom:** `InvalidToken: The token \`8702486621:AAE...8eU ALLOWED_CHAT_IDS=\` was rejected by the server.`
**Cause:** Token and the `ALLOWED_CHAT_IDS=` line got concatenated because the `cat > .env <<EOF` heredoc didn't end the first line with a real newline.
**Fix:** Keep `.env` to one line (just `TELEGRAM_BOT_TOKEN=...`) — script defaults allowlist to empty if missing.
**Lesson:** Fewer lines = fewer ways to break formatting. Add the second line only after the first one is verified working.

### 3. `dquote>` infinite prompt
**Symptom:** Terminal got stuck at a `>` or `dquote>` prompt and ignored subsequent commands.
**Cause:** Unclosed `"` in a pasted command. zsh waits for the closing quote.
**Fix:** `Ctrl+C` always escapes. Going forward, use single-quoted strings or write prompts to a file first.
**Lesson:** Nested quotes + paste + zsh = guaranteed pain. For any complex prompt being passed to `claude -p`, write it to a `.txt` file first and tell Claude Code to read the file.

### 4. Pasting terminal output back INTO terminal
**Symptom:** `zsh: parse error near 'mmm2@MMM2s-Mac-mini'`
**Cause:** Builder pasted my chat response (which contained example terminal output with prompts) into Terminal; Terminal tried to execute the prompt prefix as a command.
**Fix:** Only paste content from inside code blocks, never the surrounding prompt prefixes.
**Lesson:** Future docs and prompts should make explicit which lines are pasteable. The instinct of "copy everything I see" is strong in non-coders.

### 5. CC reported "already done" suspiciously
**Symptom:** After running the memory-add prompt, Claude Code said `bot.py` "already contained" the implementation — even though we knew it hadn't 30 seconds prior.
**Cause:** Most likely CC actually did the work but described it post-hoc as if it had already existed.
**Fix:** Always verify with `wc -l` and `grep -n "def function_name"` — quick sanity check that the actual code is in the file.
**Lesson:** Don't trust agent summaries blindly. Build a verify-step into every CC-mediated edit before moving on.

### 6. I vs l vs 1 in tokens
**Symptom:** Bot returned `InvalidToken: Not Found` even after correct-looking `.env`.
**Cause:** Human transcription of token included a wrong character (probably `I`/`l`/`1` ambiguity).
**Fix:** Don't transcribe tokens by eye. Copy directly from BotFather → email/AirDrop/Universal Clipboard → paste into terminal with `pbpaste` or `Cmd+V`.
**Lesson:** Tokens are not human-readable strings. Pretending otherwise costs revoke cycles.

### 7. Builder asked Claude in chat for Italian sentences instead of bot
**Symptom:** Builder typed "write me 3 sentences in Italian" into the project chat with Claude.
**Cause:** Old reflex — "ask Claude" means open the chat.
**Lesson:** Once you've built a thing for a use case, retraining the reflex to use the thing is its own muscle. Test the bot for "Italian sentences" use case to discover what's missing.

---

## What the bot can now do (final feature set)

- Text Q&A in Italian/English with travel-companion persona
- Live web data (train schedules, weather, hours, news, strikes, exchange rates) via WebSearch/WebFetch
- Photo input (menu, sign, label, dish, monument identification + translation)
- 48-hour conversation memory per user (SQLite, 20-message cap)
- `/start`, `/whoami`, `/forget` commands
- Allowlist gating by chat_id (blocked users get told their ID for self-service onboarding)
- Long-reply chunking at paragraph/sentence boundaries for Telegram's 4096-char limit
- Typing indicator refresh every 4 seconds during slow replies
- Structured logging to `bot.log` with IN/OUT/BLOCKED markers

---

## Cost / quota data points

- Model in use: **Claude Opus 4.7** (Max default; verified via `claude -p "What model are you?"`)
- No quota measurement yet — bot has been running <1 hour as of save time
- Builder's stance: "if Max gets rate-limited for a week, that's an acceptable worst case" — risk-tolerated choice, not unaware
- Allowlist still OPEN as of save time — friend hasn't messaged yet, so locking deferred until morning

---

## Open items for tomorrow / next session

1. **Confirm friend's first message + lock allowlist.** Run `grep "IN  chat=" bot.log | awk '{print $5}' | sort -u` to get all chat IDs that have messaged. Add friend's to `ALLOWED_CHAT_IDS` in `.env`, restart bot.
2. **launchd plist for auto-restart.** Current setup: bot dies if Mac restarts. Tomorrow's task is a launchd plist at `~/Library/LaunchAgents/com.stonecreed.friend-bot.plist` that auto-starts the bot at login and restarts on crash. Wraps `caffeinate -i` + `python bot.py` from inside `~/agents/friend-bot/venv`.
3. **Verify memory actually works in production.** Builder hasn't yet run the Bologna test (send "I'm in Bologna" → "what should I eat" → verify city-specific response). Should be confirmed first thing.
4. **Glance at `bot.log` to see friend's usage pattern.** Once friend has been using it a day, the log will reveal: rough volume, kinds of questions, latency patterns, any crashes. Best signal for what to improve in v5.
5. **Photo input not yet tested.** Verify by sending the bot a menu photo or sign and checking the response.

---

## Patterns established that carry to Stonecreed

| Pattern | Why it matters for the fleet |
|---|---|
| `~/agents/<name>/` folder convention | Already in Master Plan; this is the first real instance |
| `.env` + `chmod 600` for secrets | Every agent will have credentials — same lockdown |
| `python3 -m venv venv` + `source venv/bin/activate` | Per-agent isolation prevents dependency conflicts |
| `claude -p` subprocess + ANTHROPIC_API_KEY stripping | All Stonecreed agents on Max sub use this same pattern |
| `bypassPermissions` + narrow `--allowed-tools` | Same posture for Scout, Writer, etc — automated tool use without runaway access |
| Prompt-as-file (`memory_prompt.txt`) → `claude -p "Read the file..."` | Avoids quote escaping; cleaner for complex specs |
| Verify edits with `wc -l` + `grep` before moving on | Don't trust CC summaries blindly |
| Allowlist starts open, tightens after first contact | Will scale to any "let one user in to bootstrap" pattern |
| SQLite per-key isolation | Same model for Scout's niche data, Writer's draft history, etc |
| Structured INFO logging with chat=/role= prefixes | Easy to grep, count, debug — foundation for analytics later |
| caffeinate-wrapping for always-on | Same for any long-running orchestrator on the Mac |

---

## Builder behavior notes (for future Claude sessions)

- Strongly prefers single bash commands over multi-step instructions
- Will paste me chat output that includes terminal prompts; explicit "only paste content inside code blocks" disclaimers help
- Asks security questions proactively ("is it ok to paste this in chat?") — good instinct, encourage
- Risk-tolerated: chose to leave bot OPEN and ship to friend rather than wait + lock; "Max rate-limited for a week is an acceptable worst case"
- Pushed back when I said the bot couldn't do something it actually could (web, images) — kept me honest, kept improving the product
- Asked CC to do edits via prompt-as-file approach after first manual nano walkthrough — pattern recognition for what's easier
- Stayed up until 3am to ship — high motivation when in flow, capitalize on it

---

## What was LEFT in mid-state at save time

- Bot is **running** with web + photo + memory enabled
- Allowlist is **still OPEN** — pending friend's first message + chat ID capture
- Mac sleep settings **not yet configured** — builder hasn't picked path (GUI vs caffeinate vs both); going to bed without doing this means risk that Mac sleeps overnight and bot dies
- Memory tests (Bologna, /forget) **not yet run** — code is in but unverified end-to-end
- Photo input **not yet tested**

---

## Bigger picture

Tonight transitioned the project from "Phase 0: planning and hardware" to **"Phase 1: agent #0 running in production."** The Master Plan's Getting Started Checklist (Section 10) is no longer hypothetical — the patterns it specifies (`~/agents/`, `.env`, venv, structured logs, SQLite, Claude Code subprocess) are now battle-tested through real iteration. Builder now has muscle memory for: editing files in nano, fixing zsh quote disasters, running `claude -p` for code edits, verifying with grep, restarting via Ctrl+C + python, debugging an InvalidToken error.

The next Stonecreed agent (Scout) will reuse 70%+ of this codebase's structure verbatim. The hard part of going from zero to one is done.
