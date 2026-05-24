# Notes — Telegram capture + weekly sort

Working memory captured via Telegram, auto-categorized by Haiku, sorted by Sonnet weekly.

## Triggers (Telegram)

- `/note <text>` — slash command
- `note <text>` / `note: <text>` / `n: <text>` — prefix triggers (case-insensitive)
- Any photo with empty caption OR caption matching the above triggers
- Photo with non-note caption → routes to orchestrator-chat (vision-enabled chat), not capture

## File layout

```
notes/
├── inbox_<YYYY-MM-DD>.md     — today's accumulating capture (one file per day)
├── ideas.md                  — sorted category file (grows over time)
├── crypto.md
├── websites.md
├── business.md
├── reminders.md
├── tasks.md
├── misc.md
├── <improvised>.md           — any new category Haiku proposed during capture (normalized at sort time)
├── archive/
│   └── inbox_<YYYY-MM-DD>.md — processed inbox files after weekly sort
└── images/
    └── <YYYY-MM-DD>/
        └── <HH-MM-SS>_<hash>.jpg
```

Inbox and category files are at the **same directory level**, so image markdown links use the relative path `images/<date>/<file>.jpg` (no `../` prefix needed). Obsidian renders inline.

## Entry format

Text:
```markdown
## HH:MM <category>
<note text>
```

Image:
```markdown
## HH:MM <category> [image]
![<short_hash>](images/<YYYY-MM-DD>/<HH-MM-SS>_<hash>.jpg)

**Description:** <one-line factual description>
**Key data:** <numbers, tickers, prices, usernames, transcribed text>

**Caption:** <operator's caption if any>
```

## Cadence

- **Capture** — instant, on every trigger
- **Sort** — Sunday 10:00 local (`0 10 * * 0`), reads any `inbox_<date>.md` older than today, organizes into category files, moves the inbox file to `archive/`

## What this is NOT

- Not an action queue. Operator decides what to do with sorted notes manually.
- Not a chat partner. The agent never asks follow-up questions.
- Not connected to `drafter_queue.txt`, `idea_proposals`, or any downstream pipeline.

## Kill switches

- `~/agents/notes-agent/halt.flag` — pause both capture and sort
- `NOTES_AGENT_ENABLED=false` in `~/agents/config/.env` (if added)
- `SYSTEM_PAUSED` / `LLM_SPAWN_ENABLED` — global cascade
