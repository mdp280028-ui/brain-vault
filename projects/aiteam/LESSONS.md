# AITEAM Lessons Learned

Growing log. Each ctx save appends lessons under a date heading.

---

## 2026-05-15

- Verify wrapper model attribution BEFORE shipping cost-sensitive decisions. `ai-do.sh` silently used Opus (Claude Code default inheritance) and invalidated step 17 v1's "Sonnet wins" tier test. Pin `--model` explicitly in every wrapper.
- Bash heredoc `${5:-{}}` has brace-matching ambiguity — corrupted 16 audit_log payloads with a literal trailing `}`. Use `PAYLOAD="${5-}" ; [ -z "$PAYLOAD" ] && PAYLOAD="{}"`.
- `claude -p --output-format json` returns `total_cost_usd` and per-model `modelUsage` directly. No need to maintain a pricing map. Trust Claude Code's own numbers.
- Claude Code routes some work to Haiku even on Sonnet/Opus calls. Per-tier cost logging requires one row per model in `modelUsage`, not per call.
- Operator-suggested fixes deserve verification before literal implementation. The "export AGENT_ID env var" suggestion wouldn't have stuck — wrappers do `AGENT_ID="${2:-shell}"` which shadows env. Pass as `$2` instead.
- Schema mismatches in specs vs reality silently break SQL. The ctx-system spec used `timestamp`/`cost_usd`/`title`; real schema is `ts`/`estimated_cost_usd`/`description`. Schema-check before writing queries.
- Empty-data graceful handling is load-bearing for credibility. `/weekly_review` early-exits when <2 diary days. Better to honestly say "not enough data" than hallucinate.
- macOS bash is 3.2.57. No `${var,,}`, no assoc arrays, no `mapfile`/`readarray`. `${var/pattern/}` works. Process substitution works. `case` patterns work.
- Append-only audit logs are recoverable; mutable ones aren't. When a bug corrupts history, write a corrective row pointing back. Never UPDATE/DELETE. The corruption itself is a fact worth recording.
- Round-2+ discussion prompts MUST explicitly require engagement with a specific prior claim. Without this, multi-round devolves into parallel monologues — each agent restating its position.
- Document gaps only after verification. Initially documented `rubric.md` as not git-tracked; it actually was (commit `8ffd923`). Verify before claiming.
- Cost cap discipline: budget for retries. A client-side JSON parse error in step 18 burned a 3rd run on a 2-run cap. Save to file first, parse second.

### Session 2 — Curator + Briefer + /important (evening)

- all-MiniLM-L6-v2 sentence-transformers scores cluster low (0.3-0.6 range) for short-query-vs-long-anchor matches. Don't set a flat 0.50+ threshold without calibrating against real query/anchor pairs. Working values for curator: 0.35 slug→thesis matching, 0.15 group-mean placement. Top-1 was correct in 5/5 off-threshold cases — ranking discriminates well even when absolute scores look "weak."
- `close` is a reserved word in awk (builtin function). Using it as a `-v` variable name produces a misleading "syntax error" at the WRONG line. Cost the curator $0.28 on responses we couldn't parse. Use `startmark`/`endmark` or any non-builtin name.
- `paste -sd ", " -` on BSD/macOS treats the delimiter argument as a cycling list of single chars, not a multi-char separator. To join with ", " use `tr '\n' ',' | sed 's/,/, /g'`.
- Persist LLM responses to disk BEFORE parsing for any agent with non-trivial per-call cost. A parse-only bug becomes free to retry. Pattern: write `workspace/responses/<correlation_id>.md` first, parse second.
- `log_to_audit.sh` embeds the payload in a single-quoted SQL string with no escaping. Any `'` in the payload breaks the INSERT. Workaround: pipe payload through `sed "s/'/''/g"` before passing. Real fix: D044.
- Opus on `claude -p` does NOT internal-multi-turn like Sonnet at max-turns=30. Opus wrote 1.7K output tokens and stopped on the analyst prompt; Sonnet writes 12-13K. Result: Opus run cost LESS than Sonnet ($0.20 vs $0.26) on identical inputs. Don't assume "Opus is always more expensive" — benchmark on the actual prompt.
- Idempotent setup scripts must distinguish "seed" from "reset." `seed_taxonomy.sh` unconditionally overwrites; wiped Casper btc.md mid-checkpoint-4 (the curator had already accreted rows there). Fix at D042: write Casper outlook-history files only if missing or `row_count: 0`.
- Spec text can drift from spec body. The curator taxonomy spec said "24 topics" header / 27 in body / 28 after addition. When this happens, the body is authoritative.
- Model-override briefs (`_opus.md`, future `_haiku.md`) are operator spot-check artifacts, not pipeline inputs. Discovery globs across the pipeline must exclude model-suffix files explicitly. Tonight's filter: `grep -Ev '_(opus|haiku)\.md$'`.
- Telegram outbound via `parse_mode=HTML` is the lightest path for monospace grids: `<pre>...</pre>` for the grid, escape only `<>&` in regular text. Avoid MarkdownV2 — every `_*[]()~>#+-=|{}.!` would need escaping.
- Cron-spawned outbound messages need direct Bot API calls via curl. grammy's `ctx.reply` only works inside inbound handlers. Don't try to bridge through the long-polling bot process.
- `AGENT_MAX_TURNS_OVERRIDE` pattern (mirrors `AGENT_ID_OVERRIDE`): set inline before invocation, wrapper applies after `source .env`. Use for non-agentic text-in/text-out callers (curator topic updates, briefer compose) to pin max-turns=1 without globally editing .env.
- MEMORY-stored operator preferences need active recall. Operator's "put output files in ~/Downloads/" preference was relevant to D032's blind-comparison artifacts but I didn't apply until asked. For future similar phone-readable artifacts (D032-style spot-checks, comparison files, anything for grading), copy to `~/Downloads/` proactively.
- Seed-description embedding style coaching from operator: descriptive ("Cowen covers X using Y framework") embeds better than positional ("Cowen believes X means Y") for similarity-index purposes. Apply to future seed taxonomies.
- When claiming a D-number for a deferred item, check whether the operator may have intended that number elsewhere — especially when D-numbers haven't been ratified in HANDOFF/DEFERRED yet. I claimed "D043" for log_to_audit SQL-escape; operator had intended D043 for ETH/SOL convergence. Renumbered to D044.
- Bash `set -uo pipefail` (no `-e`) lets compound conditions like `[ ... ] && { ... ; exit; }` work without surprise exits, BUT a chained `&& [ ... ] && [ ... ]` evaluates left-to-right and can silently return non-zero from the last test. Use explicit `if` blocks for safety.
- macOS `sed -i` requires an argument: `sed -i '' -e '...' file`. The empty backup-suffix string is required; GNU sed pattern `sed -i -e '...'` fails on macOS. (Hit during curator's row_count frontmatter update.)
- jq's `--arg` escapes content correctly for JSON values but the resulting JSON, when embedded in single-quoted SQL by `log_to_audit.sh`, still breaks on literal `'`. The escape boundaries are: jq → safe JSON; sed `s/'/''/g` → SQL-safe-via-SQLite-double-single-quote convention.

### Brain vault git-init (later that day)

- Pre-flight checks catch blockers before they're expensive. Scoping `~/brain/` size, existing git state, existing SSH state, and crontab state up front flagged the missing `~/.ssh/` directory BEFORE we got to step 6 (push) where it would have stalled the chain.
- Verify operator-stated assumptions before relying on them. The build prompt said "the ust-contractors repo uses SSH on this machine, so the key should already be there." It wasn't. Don't pattern-match to "this should already work" — check.
- Pause points are leverage. Reporting the staged file list before the initial commit gave the operator a chance to opt `dashboard.log` and `inbox/` out of permanent history.
- `ssh -T git@github.com` exits 1 on success. GitHub doesn't provide shell access, so SSH disconnects with exit 1 after authenticating. Look for `Hi <user>! You've successfully authenticated` in stdout, not exit code 0.
- `ssh-keyscan -t <key-type> <host> >> ~/.ssh/known_hosts` is the right way to pre-populate host fingerprints for non-interactive SSH. Avoids the interactive prompt without disabling host verification.
- Specific token-pattern regex beats loose secret-keyword grep for pre-commit scans. `(sk-[a-zA-Z0-9]{20,}|ghp_[a-zA-Z0-9]{20,}|xoxb-[a-zA-Z0-9-]{30,}|AKIA[A-Z0-9]{16})` catches actual credentials; `secret|api_key|password` returns false positives from any doc mentioning secrets management.
- Per-repo `git config user.name` works without `--global` — useful for machines hosting repos under multiple identities. The `--local` flag is implicit when run inside a repo. Don't pollute global identity for one project.
- `git rm --cached <file>` (with `-r` for dirs) removes from index without touching the working tree. Correct unstage pattern when adjusting gitignore mid-build.

### Phase 2 Telegram stack (later same day)

- grammy ≥1.21 moved file-download helpers to `@grammyjs/files`. `ctx.getFile()` in core returns a bare `File` object — no `.download()` method. Wire `bot.api.config.use(hydrateFiles(bot.token))` after bot construction. Without it, voice download dies with `file.download is not a function`.
- faster-whisper on Apple Silicon does NOT use Metal/Neural Engine. CTranslate2 is CPU-only on macOS, using int8 AVX/NEON kernels. Still ~1.8x realtime for medium model on M4 Pro (22s audio → 11.7s). True Metal acceleration is whisper.cpp or mlx-whisper, not faster-whisper. Don't trust prose claims about ANE/Metal — verify with a real timing test.
- macOS TCC denial on external volumes presents as EPERM, not EACCES, even when Unix perms are correct. Errno discriminates: EPERM = something above filesystem (TCC/sandbox/SIP/file flags), EACCES = Unix perms. On macOS 13+ "Removable Volumes" is a separate TCC category from "Full Disk Access," and the responsible process for a CC child is the `claude` CLI binary, not Terminal — granting FDA to Terminal doesn't transfer.
- Non-interactive `claude -p` sessions inherit default permission policy and deny most Reads on script files. Phase A "what's our SSD mount path?" dodged this via `Bash(df)`; Phase B voice queries like "what's our token spend today?" hit it immediately. The orchestrator surfaces denial in chat as "I need read permission for…" — that's an LLM honestly reporting being blocked, not a bot bug.
- iOS autocorrect capitalizes the first letter of a sentence. Telegram delivers `Confirm` when user types `confirm`. Keep safety-critical accept paths strict (autocorrect → cancel is safe-fail); relax stray-detection to case-insensitive on the lone word (false-positive cost is just suppressing a reply, not auto-routing a destructive command).
- Verify audit_log against the bar list before requesting more test inputs. Phase B sign-off: operator's 3 voice notes already hit every bar including the 10-min timeout. I asked for 3 more — wasted their time. Same rule applies to sign-off claims: query the data, don't trust the claim. Both directions.
- Telegram voice notes are `.oga` (Opus-in-Ogg), not `.ogg`. faster-whisper handles via ffmpeg internally, so the wrapper just needs to accept either extension. ffmpeg is a hard runtime dep for non-WAV input — `brew install ffmpeg` is part of the install recipe.
- Soft budget caps with cost-after audit beat hard pre-caps for LLM pipelines. The /ctx pipeline runs ai-do.sh, then queries `token_usage` by correlation_id post-call, audits `ctx_over_budget` if exceeded, but always completes. Pre-estimating token cost is unreliable; complete-and-flag is more useful than refuse-with-guess.
- `.bak` files written by tooling will get auto-committed to git if not in `.gitignore`. The /ctx skeleton-backup ended up in the brain repo. Add to gitignore proactively when any pipeline writes sibling `.bak` / `.tmp` files.
- Single-chokepoint security gates beat parallel implementations. Phase D's destructive scan lives in `routeAndLog()`, called from text and voice-confirmed paths both. One scan, one audit, no risk of a future path forgetting to gate.
- The "responsible process" model in macOS TCC generalizes beyond removable volumes — whenever an EPERM appears with valid Unix metadata, ask: which app is TCC tracking as responsible for this child process? It's often not the parent terminal.

### Market Scribe session 1 (later same day)

- YouTube channel handles drift. Benjamin Cowen's channel was `@intothecryptoverse` historically; now `@benjaminjcowen`. The freed handle was taken by squatter accounts (2-sub orphans). `forHandle=<old_handle>` returns HTTP 200 with empty `items[]` — not a 404. The resilient pattern: `search.list?q="<channel name>"&type=channel`, then verify by subscriber count and `customUrl`. Always capture both `id` AND current `customUrl` at watchlist creation time so future audits can spot drift.
- macOS bash 3.2 bug — unquoted heredoc + variable expansion + apostrophe in body produces cascading phantom quote-matching errors. `PROMPT=$(cat <<EOF ...${VAR}... what's ... EOF)` triggers `unexpected EOF while looking for matching` at lines far from the actual offense. Fix: pull static text into a separate file and substitute via `${VAR//pattern/replacement}`. Quoted heredoc (`<<'EOF'`) also works but disables variable expansion.
- YouTube auto-VTT rolling captions need real overlap dedup, not exact-match. Each phrase appears across 2-3 cues with sliding boundaries. Algorithm: for each new cue, find the longest tail of the prior cue that matches the head of the current cue, emit only the suffix beyond that overlap. Threshold of 5 chars avoids false-positive matches on trailing punctuation. 3x size reduction on real video transcripts.
- YouTube API descriptions contain raw control characters in U+0000-U+001F that break jq strict mode. Pre-clean with `tr -d '\000-\037'` before piping to jq. HTTP 200 returns with malformed JSON; jq errors, Python's `json` is more lenient.
- Sonnet via `claude -p` with default `--max-turns 30` does internal multi-turn reasoning even for pure text-in/text-out tasks. A 1.5K-token visible brief can ring up 12K+ output tokens internally. For non-agentic tasks, a `--max-turns 1` variant (e.g. an `ai-do-single.sh` sibling) would cut cost ~70%.
- bash `${VAR//pattern/replacement}` handles multi-line $VALUE content cleanly — better than sed for template substitution when the replacement is a markdown transcript or filled template.
- Per-pipeline cost attribution generalises: `<pipeline>_<purpose>` agent_id naming (`telegram_bot_orchestrator`, `analyst_cowen`, `analyst_casper`) makes the dashboard's per-pipeline rollups trivial. The Phase 2 AGENT_ID_OVERRIDE env-var pattern is the right hook.
- `yt-dlp` filenames are templated as `<base>.<lang>.vtt`, not fixed — use a glob (`ls "$base".*.vtt | head -1`) to find the produced file, since the language code can vary.
- Telegram doesn't render markdown tables. The on-disk format and the Telegram format need different renderers for the same data. On-disk markdown (Obsidian rendering) stays as tables; Telegram presentation is monospace code-block grid with em-dashes for blanks. The presentation layer is downstream from the storage layer — keep them decoupled.
- The analyst's start-of-spoken-thought timestamp attribution for quotes (concatenating VTT cues to sentence boundaries) is *desired behavior*, operator-confirmed. Strict line-by-line verbatim produces fragments; sentence-level produces useful quotes. Future iterations of any transcript-to-quote pipeline should preserve this.

