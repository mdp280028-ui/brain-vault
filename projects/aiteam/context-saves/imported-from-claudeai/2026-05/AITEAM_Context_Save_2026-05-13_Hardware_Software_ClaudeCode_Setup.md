# AITEAM Context Save — Hardware, Software, & Claude Code Setup
**Date:** 2026-05-13
**Session length:** ~6 hours (afternoon + evening)
**Phase:** Pre-Launch (Phase 0) — Day 1 with new hardware
**Status at end:** Mac Mini fully operational, foundation complete, ready for API signups + first agent build

---

## TL;DR — What Got Done This Session

Hardware day. Mac Mini M4 arrived; over the course of one extended session, completed the entire foundation layer:

1. Storage architecture finalized (different from original plan due to new info)
2. All drives formatted, partitioned, named
3. Time Machine running
4. Folder structure created on ActiveSSD
5. Homebrew, Python 3.12, Git, Node installed and configured
6. Git identity set with existing GitHub noreply email
7. Claude Code installed, authenticated against Max sub, file system access granted
8. End-to-end verified: Claude Code can list/read/write in all needed folders

Builder is at clean stopping point with everything ready for next session (API signups + first agent).

---

## Hardware Decisions Made This Session

### What Stonecreed actually has on the desk
- Mac Mini M4 (24GB / 512GB) — confirmed has 5 USB-C ports, **zero USB-A**
- SanDisk Extreme Go 4TB SSD #1 — active
- SanDisk Extreme Go 4TB SSD #2 — sealed in box, in storage
- **WD Elements HDD: 14TB** (bigger than original 4-8TB plan — significantly changes backup architecture)
- Anker USB-C data hub, 4-in-1, 5Gbps
- UPS, ethernet cable
- (Note: Claude initially said Mac Mini had USB-A ports — wrong. Builder caught this. Apple removed USB-A from M4 redesign.)

### Final storage architecture (changed from original plan)

**Original plan from earlier sessions:**
- SSD #1 active, SSD #2 in drawer for Phase 2, WD HDD as nightly mirror

**Decision point this session:**
With 14TB HDD (vs original 4-8TB), one HDD can fold three jobs:
- Time Machine (4TB partition, hard-capped)
- Mirror of SSD #1 (up to 4TB)
- 30-day snapshots (1-3TB additional)
- Cold archive (whatever's left)

**Settled architecture:**
- **Mac Mini internal (512GB):** OS, agent code, SQLite DB
- **SSD #1 (4TB):** Active drive, all sites/content, mounted as `ActiveSSD`
- **WD HDD (14TB):** Partitioned into `TimeMachine` (4TB) + `Backups` (10TB)
- **SSD #2:** Sealed in original packaging, marked "Sealed: May 13, 2026 / For: Phase 2 second Mac Mini", in drawer

**SSD #2 logic:** Originally plan was to use it as live mirror today. Reverted to "keep sealed for Phase 2" because:
- 14TB HDD covers all backup needs
- Sealing preserves SanDisk warranty (5yr from purchase) for when 2nd Mac Mini arrives
- Avoids hot-swap mirror script complexity Stonecreed doesn't need yet

### Connection routing
- SSD #1 → Mac Mini USB-C port (back, full Thunderbolt speed)
- Anker hub → Mac Mini USB-C port (back)
- WD HDD → Anker hub USB-A port (HDD only does 150MB/s so hub's 5Gbps is plenty)
- Keyboard/mouse → Anker hub USB-A ports

---

## Software Installed This Session

| Tool | Version | Status |
|------|---------|--------|
| Homebrew | 5.1.11 | Installed, on PATH |
| Python | 3.12.13 | Installed, set as default `python3` |
| pip | 26.1 (from py3.12) | Working |
| Git | 2.54.0 | Installed (replaced Apple's 2.50.1) |
| Node | 26.0.0 | Installed via Homebrew |
| npm | 11.12.1 | Working |
| Claude Code | 2.1.141 | Installed globally, authenticated |

### Key gotcha resolved
After installing Python 3.12 via Homebrew, `python3 --version` still showed Apple's 3.9.6. Fixed by adding to `~/.zprofile`:
```
export PATH="/opt/homebrew/opt/python@3.12/libexec/bin:$PATH"
```
Same issue partly applies to git (Apple ships old version at `/usr/bin/git`) but Homebrew's `/opt/homebrew/bin` was already early enough in PATH that `git --version` correctly shows 2.54.0.

### Git identity
- user.name: `mdp280028-ui`
- user.email: `mdp280028-ui@users.noreply.github.com` (GitHub noreply format)
- init.defaultBranch: `main`
- Uses existing GitHub identity from prior projects (not a new one)

---

## Claude Code Setup

### Authentication
- Authenticated against Stonecreed's Max sub (NOT API key)
- Means Claude Code runs on existing $200/month allocation, no separate billing
- Logged in via browser flow, picked up existing claude.ai session
- Account email visible in welcome banner: `ruralpropertyguide@gmail.com`

### macOS permissions journey (took longer than expected)
The macOS Privacy & Security panel for Files & Folders behaved inconsistently. Sequence:
1. First prompt: "Terminal would like to access Desktop" → Allowed (correct)
2. Second prompt: "Terminal would like to access data from other apps" → Denied (correct — this is App Management, not what was needed)
3. Documents access silently failed with "Operation not permitted" — no popup ever triggered
4. Tried having Claude Code list folders to force prompt — listing was blocked but Write/Read/rm on specific paths worked (TCC quirk: listing requires different permission than file ops)
5. System Settings → Privacy & Security → Files & Folders showed Terminal in list but with no expandable per-folder toggles
6. Resolution: granted **Full Disk Access** to Terminal (in same Privacy panel, separate section). This was the right call for this specific use case despite Claude's original advice to avoid Full Disk Access:
   - Single user, single trusted machine
   - Agent system needs to read/scan many folders anyway
   - Avoids whack-a-mole permission grants for each new folder agents touch
   - Granted ONLY to Terminal.app, not other apps

### Final verification
End-of-session test in Claude Code:
```
list the files in ~/Documents, ~/Downloads, and ~/Desktop and also in /Volumes/ActiveSSD
```
All four listed cleanly. Foundation complete.

---

## Architectural Decisions Made This Session

### 1. Claude Code over Hermes for Phase 1
Major reversal of earlier project-doc recommendations. The case for Claude Code:
- Already paid for via Max sub (zero additional cost vs Hermes burning API credits)
- Built-in file/command/MCP capabilities — no plumbing to write
- Native macOS scheduling (Desktop scheduled tasks)
- Skills system can wrap future agents (Editor, Writer, Intelligence)
- Hermes was right when Claude Code didn't exist in current form

**Plan:** Use Claude Code as the runtime for everything in Phase 1. Revisit Hermes only in Phase 2 if specific "persistent ambient watcher" pattern proves necessary.

### 2. Personal assistant / chief-of-staff agent — architecture
Stonecreed wants a Telegram-driven assistant they can message naturally to manage the agent fleet. Decided architecture:

```
Stonecreed (anywhere via phone)
  ↓ Telegram DM to secondary account
Python Telegram bot (~100 lines, runs as background service on Mac Mini)
  ↓ relays message to
Claude Code via `claude -p` headless invocation
  ↓ executes (read logs, run scripts, query SQLite, edit files)
  ↓ result returned via Telegram bot
```

**Setup time estimate:** 3-5 hours for v1.
**Cost:** Effectively free (Telegram free, Claude Code on Max sub, only Telegram bot's hosting overhead).

### 3. Telegram monitoring strategy
- **Userbot** (not Bot API) — runs as user on a secondary Telegram account
- Secondary account does double duty: monitors channels AND is the chief-of-staff interface
- Userbot is invisible to other group members (looks identical to user reading on second device)
- No risk of getting kicked from groups (unlike adding a Bot account)
- Telegram library: Telethon or Pyrogram
- Stonecreed has spare TG account ready to use

### 4. Twitter monitoring deferred
- Original plan included Twitter
- X API now pay-per-use ($0.005/read, ~$15-50/mo for light monitoring)
- Scraping = fragile (Twitter changes HTML constantly, IP bans, ToS issues)
- Grok is NOT a shortcut (it's an LLM, still needs Twitter API to see tweets)
- **Decision:** Telegram-only for v1. Add Twitter later if/when Telegram monitor proves useful.

### 5. First agents — priority order
Reaffirmed and refined from project files:
- Official minimum viable team is 4: Intelligence, Writer, Editor, Revenue Attribution
- **Refined for Stonecreed:** Build only 3 first (skip Revenue Attribution until there's revenue to attribute)
- **Build order within Phase 1:**
  1. Editor (simplest, lowest risk, fastest hands-on win with API)
  2. Writer (medium complexity)
  3. Intelligence (most complex, biggest payoff)
  4. Wire them together
  5. Manual publish of first article
  6. Then Telegram bridge / chief-of-staff
- Counterintuitive but deliberate: Editor first to learn the toolchain on simple work before tackling Intelligence

---

## Specific User Preferences/Patterns Observed This Session

1. **Caught Claude's USB-A mistake.** Builder physically looked at the Mac Mini, didn't see USB-A ports, asked. Critical habit — Claude's port/spec info can be outdated.
2. **Asks why before doing.** Multiple times asked "what is this for" / "should I" before clicking. Good. Continue explaining trade-offs, not just commands.
3. **Asked for TTWC to stop mid-session.** Builder said "no more TTWC this session." Honored for the rest of the session.
4. **Already had Git identity from prior projects** — used `mdp280028-ui@users.noreply.github.com`. Good hygiene.
5. **Made the right call on permissions complexity.** Asked for nuclear option (`--dangerously-skip-permissions`) when frustrated with macOS Privacy panel — Claude pushed back and explained what the flag actually does. Builder accepted explanation and moved on without the flag. Eventually solved with Full Disk Access at Terminal level instead.

---

## Open Questions / Things to Decide Next Session

1. **Backup script** — Still need to write nightly `rsync` + snapshot rotation script (SSD #1 → WD `Backups`). Was deferred until software install was done. Should be early in next session.
2. **API account signups:**
   - Anthropic API (for agents using Claude — separate billing from Max sub)
   - Together.ai (for Llama 70B bulk content)
   - $20 credit on each to start
3. **First agent build:** Editor agent. Decision needed: pure Python script, or build it as a Claude Code skill from day 1?
4. **Telegram secondary account:** Confirm phone number, API credentials at my.telegram.org. Set up before bridge build.
5. **Mac Mini's local IP** for future SSH access — never noted. Should record next session.

---

## Lessons Learned This Session

### For Claude in future sessions

1. **Apple ports change.** Don't trust training data on port layouts — search or ask builder to verify physically before giving cable/connection instructions.
2. **macOS Privacy & Security UI is unreliable across versions.** When granular Files & Folders toggles don't appear, Full Disk Access for Terminal is the practical answer for an agent operator (not a regular user). Adjust the earlier "never grant Full Disk Access" advice for this specific context.
3. **Homebrew Python doesn't replace Apple Python automatically.** Always include the PATH export step in instructions or builder will hit "Python 3.9.6" confusion.
4. **Claude Code's `--dangerously-skip-permissions` flag is ONLY for headless/cron use.** Never recommend for interactive sessions, even when builder is frustrated with prompts.
5. **The "second prompt" gotcha:** macOS asks for permissions in confusing waves. The "access data from other apps" prompt is App Management permission and should be DENIED. Files & Folders prompts are the ones to ALLOW. They look nearly identical.

### For Stonecreed (the builder)

1. **The physical-verify habit is critical.** Caught the USB-A mistake by looking at the actual hardware. Keep doing this.
2. **Single sessions, single focus.** This session was hardware → software → Claude Code. Three chapters, all done, clean stop. Don't try to do API signups + backup script + first agent in one session — those are next session's chapter.
3. **Default macOS settings are not your settings.** Several decisions this session involved overriding defaults (partition split 4/10 not 7/7, Low Power Mode off, never-sleep, full disk access to Terminal). Defaults are for casual users running one app at a time. You're running 24/7 infrastructure.

---

## Where We Are vs Original Project Status Doc

| Metric | At start of project | End of this session |
|--------|---------------------|---------------------|
| Hardware purchased | NO | YES (all set up) |
| Software installed | NO | YES (Homebrew, Python, Git, Node, Claude Code) |
| First agent built | NO | NO (next session) |
| First site live | NO | NO |
| Total sites | 0 | 0 |
| Total agents | 0 | 0 |
| Total articles | 0 | 0 |
| Monthly revenue | $0 | $0 |
| Monthly API costs | $0 | $0 |

**Phase 0 (Pre-Launch) is now effectively complete on the hardware/software axis.** Phase 1 begins next session with API signups + first agent build.

---

## Next Session Starting Point

When Stonecreed comes back, the natural opener is:

> "Ready for API signups and first agent."

Tasks for next session, in order:
1. Quick sanity check — open Claude Code, verify still working
2. Sign up for Anthropic API (different from Max sub — load $20 credit)
3. Sign up for Together.ai (load $20 credit for Llama 70B access)
4. Set up API key storage on Mac Mini (`~/agents/config/` with proper permissions)
5. Write nightly backup script (rsync + snapshot rotation)
6. Test backup script manually before scheduling
7. Schedule backup script via cron
8. Decide: build Editor as standalone Python script OR as Claude Code skill
9. Build the first version of the Editor agent
10. Test Editor on a manually-written sample draft

Realistic scope for one next session: items 1-6 plus initial discussion of Editor architecture. The actual Editor build is probably its own session after that.

---

*End of context save. File this in project as `AITEAM_Context_Save_2026-05-13_Hardware_Software_ClaudeCode_Setup.md`.*
