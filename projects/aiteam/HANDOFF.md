# AITEAM Project Handoff
**Last updated:** 2026-05-25 09:40 (TWO parallel chats merged: **hardening lane** [D092 issues nested-logs+TG pager / D026 dashboard launchd / D094 lockfile+timeout] + **verification/build lane** [outstanding-work audit → DEFERRED bookkeeping / internal-link SSG-fy v2 + D091 / orchestrator memory / typing indicator / dropzone + D025])
**Last session summary:** Reliability + operator-facing hardening across the fleet — cron hybrids now self-serialize and bound hung claude calls, the dashboard auto-restarts under launchd, nested-agent failures surface correctly on /issues and page Telegram, and the orchestrator gained conversation memory + a typing indicator + a safe write/photo dropzone.

**Hardening lane:** Three recon-gated builds. **D092** — issues-capture resolves nested-agent log paths (recon found actor_ids are the BARE child name `scribe`, not `market/scribe`, and nested agents write `*.log` in-dir not under `log/`): `resolve_agent_dir` globs one level deep; same `cd` bug fixed in the dashboard CC-prompt route. `notify_telegram.sh` pages `new_issues.log` to Telegram (high-water on `audit_log_id`, 10/tick, **no dashboard link** per operator), `*/5` cron live. **D026** — dashboard (last always-on process on nohup) → launchd `com.aiteam.dashboard`; `lib/dashboard.sh` wraps `launchctl`; logs → `dashboard/log/`. **D094** — lockfile + timeout, **zero new deps** (`shlock`+`perl alarm`): timeout centralized in the 3 `ai-*.sh` wrappers (`AI_TIMEOUT_SEC` default **1200s**), shlock locks on drafter + market chain + idea-agent. Commits: agents `20611fd/24b7cd7/eaa4fe9/fb33435/633f6dc`; brain `1d89771/adfe1e5/904992f`. ctx: `context-saves/AITEAM_Context_Save_2026-05-25_0935_hardening.md`.

**Verification/build lane:** Outstanding-work audit (15 items) → DEFERRED bookkeeping (`1c35674`): closed F17/F18, D045→D050, D087 weekly→daily (`6192499`), +D093. Internal-link SSG-fy v2 + D091 (`934dc76`/`ff63975`, brain `e4d132e`). Orchestrator conversation memory (`9f7c5c4`/`dc1dfc4`), typing indicator (`1627643`), dropzone (swept into `74c1cde`) + D025. ctx: `context-saves/AITEAM_Context_Save_2026-05-25_0935.md`.

---

## Current phase
**Fleet reliability + operator-facing orchestrator hardening — all shipped+pushed; awaiting operator-side live smoke and first organic fires.**
- Hybrid agents self-serialize (shlock) and bound hung `claude -p` calls (1200s → `timeout_kill` on /issues). Verified synthetically; awaiting first organic fire.
- Dashboard crash/reboot-resilient under launchd `KeepAlive`+`RunAtLoad` — verified via `kill -9` respawn, NOT yet via a real reboot.
- Orchestrator has conversation memory + typing indicator + safe write/photo dropzone; internal-link audits asbestos + SSG. One forward-photo path awaits a live forward.

## Completed (cumulative — this period)
- [x] **D092 — issues nested log paths + Telegram pager** (hardening). `resolve_agent_dir` (bare-name→one-level glob); `notify_telegram.sh` (`*/5`, high-water, no TG link); dashboard `cd` fix. E2E-verified: synthetic `scribe` failure → real TG send (HTTP 200). `20611fd/24b7cd7/eaa4fe9`, brain `1d89771`.
- [x] **D026 — dashboard launchd auto-restart** (hardening). `com.aiteam.dashboard.plist`; `lib/dashboard.sh`→launchctl wrap; logs `dashboard/log/`. Verified bootstrap→200, `kill -9`→respawn ~1s, bootout→gone. `fb33435`, brain `adfe1e5`.
- [x] **D094 — hybrid lockfile + timeout** (hardening). `shlock` locks (drafter + market chain + idea-agent) + `perl-alarm` timeout in the 3 `ai-*.sh` wrappers. Verified: lock-held→exit 0 + `lock_contention` + no run-batch; `AI_TIMEOUT_SEC=10` sleep-stub→SIGALRM@10s→rc142→`timeout_kill` on /issues. `633f6dc`, brain `904992f`.
- [x] **D091 closed** (verification) — `audit_links.sh` parses `APPROVED_GUIDE_SLUGS` from render component; asbestos non-approved=stripped, SSG=broken. `ff63975`, brain `e4d132e`.
- [x] **Internal-link SSG-fy v2** (verification) — multi-site schema migration (`.backup`-tested, 32/0/5 preserved), `site_lib.sh`, all scripts `--site`-aware; SSG sourced from live `smartsourceguide/data/guides`. `934dc76`.
- [x] **F17/F18 closed** — daily cost-cap already shipped (`54a69bd`); D045→**D050** rename; **D087** weekly→daily (`6192499`); **D093** added. brain `1c35674`.
- [x] **Orchestrator conversation memory** — `build_conversation_context.sh` (30 msgs/72h, current turn excluded) wired into `run_agent.sh`, gated by `agents_with_memory.txt`. Live-verified (input 242→3765 tok, cache_read 12963 on 2nd call). `9f7c5c4/dc1dfc4`.
- [x] **Telegram typing indicator** (`1627643`) — 4s pulse in `routeToOrchestrator`, `finally` cleanup. Live-verified.
- [x] **Orchestrator dropzone** (swept into `74c1cde`) — `~/orchestrator_inbox/` + `write_to_dropzone.sh` + photo archive in both photo paths + cwd-local permission grant (D025). Text + direct-photo live-verified.
- [x] **Prior (carried):** idea-agent v1 (daily), notes-agent v1 (+forward capture), /model picker, market pipeline cron (D033), watchdog, Issues Queue v1, domain-hunter v0.1.5, internal-link v1, SSG pipeline, ship-to-site, orchestrator/telegram bot, D066/D072 closes.

## In progress
- [ ] **Operator-side live smoke of the three new agents** (carried) — `/model` toggle round-trip; text+image notes + forward /yes /no; spot-check idea-agent proposals + mark `operator_action` in `idea_proposals` (build-rate 0 until then).
- [ ] **Forwarded-photo dropzone archive** (verification) — committed + syntax-checked, never run live; confirm `image_received forwarded:true` + `telegram-photo-*.jpg` on next forward.
- [ ] **First production fires** — market pipeline; domain-hunter 06:30; issues-capture/notify_telegram `*/5`. Plus operator browser-smoke of `/issues` (Generate CC Prompt → clipboard) + tap-test domain-hunter Ahrefs link.

## Blocked
- none

## Known bugs
| Bug | Severity | File(s) | Notes |
|---|---|---|---|
| Memory: vision/photo turns get no conversation context | minor (v1.1) | `lib/run_agent.sh`, `telegram/bot.js` | Text path only — vision left unwired to avoid breaking `@path` image attach. A later text turn still sees the prior photo turn. |
| `audit_links.sh` no dedup | minor (v1.1) | `internal-link/audit_links.sh` | Repeated audits accumulate duplicate `stripped` rows (8 real popcorn-ceiling rows already present). |
| `idea_calibration` duplicate per-week rows under daily cadence | minor | `idea-agent/calibrate.sh` | Daily runs insert one row each under same `week_of`. D087. |
| D044 `log_to_audit.sh` apostrophe escape | live workaround in place | `lib/log_to_audit.sh` | APOS scrub works; parameterized rewrite deferred. |
| Photo album `media_group_id` not in inbox markdown | minor (v1.1) | `notes-agent/capture.sh`, `bot.js` | D089. |
| `pending_forwards` rows never deleted | future risk | schema | Forensic by design. Hard cap if >1000. D089. |
| approved-index drift on escalated slugs | medium | `content/run-batch.sh`, `approved-index-asbestos.md` | Root cause open as **D090**. Check before next batch. |
| `asbestos-encapsulation-vs-removal` stuck in needs-review/ | minor | `content/asbestos/needs-review/` (gitignored) | Failed audit rounds 2+3. Operator: recover/re-run/drop. |
| domain-hunter godaddy fetcher non-producing | by design | `domain-hunter/fetch/godaddy.py` | Akamai 403; exits 4, continues on expireddomains. |
| `/issues` clipboard copy unverified headlessly | unknown | `dashboard/server.js` | Needs an operator browser tap. |
| D094 1200s timeout may clip out-of-scope pipeline writer | latent | `content/run-batch.sh` (routes through `ai-do.sh`) | Writer single-call duration unmeasured; if it `timeout_kill`s, raise `AI_TIMEOUT_SEC` for that path. |
| ~~`/issues` CC-prompt log-path/`cd` assumed top-level~~ | FIXED (D092) | — | Nested resolution + `cd` fix shipped. |
| ~~internal-link audit blind to body links/render~~ | FIXED (D091) | — | render-aware audit shipped. |

## Architecture decisions (recent)
| Decision | Reasoning | Date |
|---|---|---|
| Nested-agent log resolution = bare-name → glob `~/agents/*/<id>` one level deep | actor_id is the bare child name (`scribe`); agent lives at `market/scribe`; logs are `*.log` in-dir | 2026-05-25 |
| Telegram issue pager carries NO dashboard link | operator: avoid token-in-Telegram; open `/issues` manually | 2026-05-25 |
| AI-call timeout centralized in the 3 `ai-*.sh` wrappers | one change covers every agent + future ones; fixes infinite-hang mode globally | 2026-05-25 |
| `AI_TIMEOUT_SEC` default 1200s (not briefed 600) | idea-agent-synth legitimately hits ~680s (34K tok); 600 would clip it | 2026-05-25 |
| Lock+timeout via `shlock`+`perl alarm`, zero new deps | gtimeout/flock/coreutils absent on this Mac; both primitives built-in | 2026-05-25 |
| `timeout_kill`→/issues (payload `error` key); `lock_contention`→audit-only | timeout is a real failure; contention is expected | 2026-05-25 |
| Dashboard managed by launchd; `lib/dashboard.sh` wraps launchctl | only always-on process not auto-restarting; KeepAlive fights manual `kill` | 2026-05-25 |
| Agent perm allow-rules in cwd-local `<agent>/.claude/settings.json` | `run_agent.sh` cd's into agent dir; git-root settings silently ignored (D025) | 2026-05-25 |
| Orchestrator memory = last 30 msgs/72h `conversation_log` replay, current turn excluded | cheap, bounded; fixes Mode-2 (history saved, never replayed) | 2026-05-25 |
| internal-link audit parses `APPROVED_GUIDE_SLUGS` from render file; per-site verdict asymmetry | mirrors render truth, not curl raw URLs (D091) | 2026-05-25 |
| SSG sourced from live `smartsourceguide/data/guides`, not ssg-content | ssg-content pipeline dir is empty | 2026-05-25 |

## Deferred items
(See `DEFERRED.md` for full context.)
**Closed this period:** D092, D026, D094, D091, F17/F18. D045→D050 (renumber). **D083 → PARTIAL** (propose_backlinks SSG-fied ✅; `keyword-registry update_registry.py` SSG handling + `deploy_batch` SSG-branch wiring still open, gated on SSG first batch).
**Logged this period:** **D025** (agent perm allow-rules cwd-local). **D093** (domain-hunter v0.2 ExpiredDomains Pro — gated on 2-4wk signal + ≥1 domain acquired; CLAUDE.md still mislabels v0.1.5 as "v0.2").
**Open/carried:** F2 (drafter pre-flight keyword-config), D044 (log_to_audit apostrophe rewrite), F14 (pre-ship approval gate), D087 (idea-agent daily-cadence calibration), D088 (/model v1.1), D089 (notes-agent v1.1), D090 (approved-index drift), D073/D074 (editor rubric + live-gate).
**Hardening-lane follow-ups (not D-numbered):** watchdog could monitor `com.aiteam.dashboard` via its existing `probe_launchctl_label` (noted in `dashboard/CLAUDE.md`); light daily/weekly hybrids (domain-hunter, backlink-prospector, research-opportunity) left unlocked by design — revisit only if a hang is observed.

## Next session should
1. **Live-smoke the three new agents from the phone** (carried) — /model toggles → token_usage; notes triggers → inbox; forward flow; orchestrator memory + dropzone over a few real turns.
2. **Spot-check idea-agent proposals** (carried) — mark `operator_action` in `idea_proposals`.
3. **Confirm D026 reboot persistence** after the next real reboot (`com.aiteam.dashboard` up on `:3141`).
4. **Verify forwarded-photo dropzone** archive on the next real forward.
5. **Watch for first organic `timeout_kill`/`lock_contention`** — `timeout_kill` should appear on `/issues`.
6. **D083 remainder** (if touching SSG) — site-aware `update_registry.py` + `deploy_batch.sh` SSG-branch (once SSG ships).

## Files to reference
**Hardening lane:**
- `~/agents/lib/{ai-do,ai-think,ai-cheap}.sh` — perl-alarm timeout block (`AI_TIMEOUT_SEC`), `timeout_kill` audit with `error` key.
- `~/agents/{assignment-drafter/drafter.sh, market/scribe/process_queue.sh, market/curator/curate.sh, market/briefer/brief.sh, idea-agent/run_weekly.sh}` — shlock lock + `export AI_CALLER`.
- `~/agents/issues-capture/{capture.sh,notify_telegram.sh}` — nested resolver + TG pager (`state/notify_last_audit_id`).
- `~/agents/dashboard/{com.aiteam.dashboard.plist,CLAUDE.md}` + `lib/dashboard.sh` (launchctl wrap); `dashboard/server.js` (generate-prompt `cd`).
**Verification lane:**
- `~/agents/lib/{build_conversation_context.sh,agents_with_memory.txt,run_agent.sh}` (memory); `lib/write_to_dropzone.sh` + `orchestrator/.claude/settings.json` (D025); `telegram/bot.js` (typing + dropzone photo paths).
- `~/agents/internal-link/{site_lib.sh,audit_links.sh}` + `migrations/2026-05-24_multisite*.sql`.
- `~/projects/asbestoshq-site/src/components/GuideArticle.tsx` + `~/projects/smartsourceguide/lib/{inline-md.tsx,site-config.ts}` — render-link sources of truth.

## Gotchas for future me
- **shlock + perl-alarm are the macOS zero-dep lock/timeout** — `gtimeout`/`flock`/coreutils NOT installed. `shlock -f <lock> -p $$` (PID-aware, steals stale locks); `perl -e 'alarm shift @ARGV; exec @ARGV' N cmd…` (alarm survives exec, SIGALRM→142). The 3 `ai-*.sh` wrappers now carry this guard (`AI_TIMEOUT_SEC` default 1200).
- **`claude -p` has no `--max-tokens` and no timeout flag** — only `--max-turns` + `--max-budget-usd` (soft). A stalled call hangs forever.
- **`/issues` surfacing is payload-shaped** — capture matches `action LIKE '%_failed'/'%error%'` OR `payload_json LIKE '%"error"%'/...`. Give an action an `error` payload key to surface it; omit to keep audit-only.
- **launchd `KeepAlive` fights `kill`** — stop the dashboard with `dashboard.sh stop` (bootout), apply plist edits with `dashboard.sh restart`; kill the nohup PID before first bootstrap or `:3141` collides. Bot restart: `launchctl kickstart -k gui/$UID/com.aiteam.grammy-bot` (session PID was 91499).
- **Agent perm allow-rules are cwd-local** — `~/agents/<agent>/.claude/settings.json`, NOT git-root (silently ignored). `run_agent.sh` cd's into the agent dir [D025].
- **D-number race + shared-file commit sweep** — `grep -oE 'D0[0-9]{2}'` immediately before writing DEFERRED.md (D093 was claimed mid-session → used D094); committing DEFERRED.md sweeps a sister's uncommitted hunks under your message. Stage exact paths, never `git add -A`.
- **Same-minute `ctx.sh` collision happened again today** — two chats at `_0935` clobbered each other's skeleton; hardening save is `_0935_hardening.md`, verification save is `_0935.md`. If your skeleton's topic is someone else's, author yours under `_HHMM_<topic>.md`.
- **`log_to_audit.sh` is positional + echoes the corr id** — `actor_type actor_id action target payload_json [corr]`; flag-style writes a malformed row silently; `audit_log` uses `action`/`payload_json` (no `event_type`); redirect both streams in any path-returning helper.
- **Verify against hard evidence (audit rows + files), not "it worked"** — a forwarded photo silently skipped the dropzone archive while the bot replied fine; only the missing `image_received` row exposed it.
- **`token_usage` has no duration column** — measure single-call wall-clock via `LAG(ts) OVER (PARTITION BY correlation_id ORDER BY ts)`.
- **`bot.js` is ESM** — reuse `logAudit()`/`file.download()`/`node:fs/promises`; no `require()`/`execSync`. Grep for an existing `bot.on("message:photo")` before adding one.
- **23:50 PT auto-commit sweeps uncommitted work** under `agents: auto-commit <date>`; commit before then for descriptive messages (no force-push to fix).
- **macOS cron does NOT catch up on missed firings** after sleep/reboot — but launchd `RunAtLoad` does (dashboard + grammy-bot now covered).
- **dotenv reads `.env` from CWD, not script dir** — dotenv scripts need `cd "$(dirname "$0")"`.
