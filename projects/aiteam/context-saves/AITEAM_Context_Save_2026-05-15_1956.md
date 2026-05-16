# AITEAM Context Save — 2026-05-15_1956
**Generated:** 2026-05-15T19:56:31-0700
**Since last save:** 2026-05-15 18:15:53
**Session topic:** Market Scribe + Analyst session 1 — built and signed off the YouTube polling + transcript-fetching + brief-generating pipeline for two channels (Cowen rich, Casper lean). 10 steps shipped end-to-end with live smoke test on real videos. `session_1_complete` audit row #140 at 19:55:56. Total session cost $0.33.

---

## Mechanical record

### Git activity since last save
```
df19fa3 Market scribe (session 1 steps 8-9): analyze.sh + prompt template + README
1c070df Market scribe (session 1 steps 6-7): fetch_transcript.sh + brief templates
6ce2c1b Market scribe (session 1 steps 1-5): API key wiring + watchlist + poll.sh
```

### Files changed
```
A	market/analyst/analyze.sh
A	market/analyst/brief_casper.template.md
A	market/analyst/brief_cowen.template.md
A	market/analyst/prompt_template.md
A	market/README.md
A	market/scribe/fetch_transcript.sh
A	market/scribe/poll.sh
A	market/scribe/watchlist.yaml
M	.gitignore
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
| id  |         ts          | actor_id |             action             |        target        |
|-----|---------------------|----------|--------------------------------|----------------------|
| 140 | 2026-05-15 19:55:56 | claude   | session_1_complete             | aiteam_market_scribe |
| 139 | 2026-05-15 19:23:51 | analyst  | brief_generated                | casper:zyHJRBIEVcs   |
| 138 | 2026-05-15 19:23:26 | analyst  | brief_generated                | cowen:FgxAe_NAh5c    |
| 137 | 2026-05-15 19:19:48 | claude   | scribe_readme_written          | aiteam_market_scribe |
| 136 | 2026-05-15 19:11:45 | claude   | scribe_brief_templates_written | aiteam_market_scribe |
| 135 | 2026-05-15 19:10:55 | scribe   | transcript_fetched             | casper:zyHJRBIEVcs   |
| 134 | 2026-05-15 19:10:50 | scribe   | transcript_fetched             | cowen:FgxAe_NAh5c    |
| 133 | 2026-05-15 19:08:32 | scribe   | transcript_fetched             | casper:zyHJRBIEVcs   |
| 132 | 2026-05-15 19:08:27 | scribe   | transcript_fetched             | cowen:FgxAe_NAh5c    |
| 131 | 2026-05-15 19:06:26 | scribe   | scribe_poll                    | aiteam_market_scribe |
| 130 | 2026-05-15 19:06:25 | scribe   | scribe_poll                    | aiteam_market_scribe |
| 129 | 2026-05-15 19:04:42 | claude   | scribe_ytdlp_installed         | aiteam_market_scribe |
| 128 | 2026-05-15 19:03:57 | claude   | scribe_folder_structure        | aiteam_market_scribe |
| 127 | 2026-05-15 19:03:40 | claude   | scribe_watchlist_written       | aiteam_market_scribe |
| 126 | 2026-05-15 19:00:13 | claude   | youtube_api_key_provisioned    | aiteam_market_scribe |

### Token usage
|           model           | in_tok | out_tok | cost_usd |
|---------------------------|--------|---------|----------|
| claude-sonnet-4-6         | 6      | 13296   | 0.313    |
| claude-haiku-4-5-20251001 | 16495  | 33      | 0.0167   |

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

- **Built the Market Scribe (poll + transcript fetch) and Analyst (per-video brief) end-to-end in one session.** Two channels live: Cowen (rich treatment, year/months/weeks horizons) and Casper (lean treatment, months/weeks/days, no trading-technique deep-dives). 10 steps signed off; live smoke test produced one brief for the most-recent video from each channel.
- **Cowen handle is `@benjaminjcowen`, not `@intothecryptoverse`** (the build spec was stale). Real Cowen channel resolved via `search.list` lookup + subscriber-count verification; the stale handle now resolves to 2-sub squatter accounts. Channel rename pattern captured as a lesson — future channel onboarding should use `search.list + verify`, not `forHandle`.
- **Models on internal SSD permanently confirmed (no new decision; reinforces last session).** The YouTube transcripts and briefs live in `~/brain/channels/<slug>/{raw,transcripts,briefs}/`, indexed by date+video_id.
- **Soft-cap cost attribution per channel.** Analyst LLM calls tag `agent_id` as `analyst_cowen` / `analyst_casper` (via the AGENT_ID_OVERRIDE pattern from Phase 2) so the dashboard rolls up per-channel spend. Cost-after audit, not pre-cap. Cowen smoke test = $0.26 (mostly internal multi-turn reasoning; $0.07 of which was Haiku for routing), Casper = $0.07.
- **Pulled the analyst prompt out of `analyze.sh` into a separate `prompt_template.md`** with `__PLACEHOLDER__` tokens substituted via bash `${VAR//pattern/replacement}`. Forced by a real bash 3.2 parser bug (unquoted heredoc + variable expansion + apostrophe in body → cascading phantom missing-quote errors). The separation is also a cleanliness win — prompt is now editable without touching code.
- **VTT rolling-caption dedup via prefix/suffix overlap detection** (min 5-char threshold) — `fetch_transcript.sh`'s awk block. 3x size reduction on YouTube auto-captions (Cowen 24-min: 71 KB → 26 KB; Casper 10-min: 35 KB → 13 KB). Falls out to clean readable transcripts with proper [MM:SS] timestamps.
- **Acceptance of two analyst-output quirks** (operator-stated):
  - **Timestamp behavior**: analyst concatenates VTT cues to sentence boundaries and attributes the quote to the *start* of the spoken thought. Desired behavior — quotes are complete sentences, not fragments. Future iterations preserve this.
  - **Casper template overreach**: analyst added Quotes / Tools / Topics sections beyond the lean Casper template. Accepted because content stays at list-level (no technique explanations). Session 2 briefer should know these fields are available on Casper briefs.
- **Telegram outlook-format contract for session 2 briefer** (decided this session, not built):
  - On-disk: markdown tables (renders cleanly in Obsidian; no change)
  - In Telegram: monospace code-block grid (markdown tables don't render in Telegram)
  - Em-dash (—) for blank/unaddressed cells (NOT empty space)
  - Asset names left-aligned, arrow+conviction centered
  - Wrapped in triple-backtick for MarkdownV2
- **Cron NOT activated.** `poll.sh` documented in `~/agents/market/README.md` with the `*/15 * * * *` line commented out. Activate after a few more manual runs build confidence.
- **`seen_videos.db` gitignored** (same policy as `~/store/aiteam.db`) — local mutable state, not source. Added `market/shared/seen_videos.db`, `market/shared/seen_videos.db-journal`, `market/scribe/workspace/`, `market/scribe/poll.log` to `.gitignore`.

## Lessons learned

- **YouTube channel handles drift over time.** Benjamin Cowen's channel was `@intothecryptoverse` historically; now `@benjaminjcowen`. The freed handle was taken by squatter accounts (2-sub orphans). `forHandle=<old_handle>` returns 0 results silently — no error. The resilient resolution pattern is `search.list?q="<channel name>"&type=channel`, then verify the top result by subscriber count / video count / `customUrl`. Always verify, don't trust the spec-stated handle. **Always capture both `id` AND current `customUrl` at watchlist creation time** so future spec-doc audits can spot drift.
- **`forHandle` is case-insensitive in YouTube URLs but the API returns 0 silently** when the handle doesn't currently belong to any active channel. Not a 404 — a 200 with empty `items[]`. Easy to mistake for "channel doesn't exist" when really "channel renamed." Check `pageInfo.totalResults` and treat 0 as a signal to search-fallback.
- **macOS bash 3.2 bug — unquoted heredoc + variable expansion + apostrophe in body cascades into phantom quote-matching errors.** `PROMPT=$(cat <<EOF ... ${VAR} ... what's ... EOF)` triggers `unexpected EOF while looking for matching` errors that report at lines FAR from the actual offense. Fix: pull static text to a separate file and substitute via `${VAR//pattern/replacement}` — that's the cleanest workaround. Quoted heredoc (`<<'EOF'`) also works but you lose variable expansion in the body. Add to the macOS-bash-3.2 gotchas list alongside `${var,,}` / associative arrays / `mapfile`.
- **YouTube auto-VTT rolling captions need real dedup, not exact-match dedup.** YouTube delivers each phrase across 2-3 overlapping cues with sliding boundaries. Exact-match `if (cue_text == last_emit)` only catches identical lines — misses the rolling pattern entirely and leaves the transcript at 3x its real size. Correct approach: for each new cue, find the longest tail of the prior cue that matches the head of the current cue, emit only the suffix beyond that overlap. Threshold of 5 chars avoids false-positive matches on trailing punctuation.
- **YouTube descriptions contain raw control characters (`\r`, others in U+0000-U+001F).** This breaks jq's strict JSON parsing with `Invalid string: control characters from U+0000 through U+001F must be escaped`. Pre-clean with `tr -d '\000-\037'` before piping to jq. The API returns HTTP 200 with this malformed JSON — jq's strict mode rejects it, Python's `json` module is more lenient.
- **Telegram voice notes are `.oga`, YouTube auto-captions are `.vtt`** — both need format-aware handling. yt-dlp output filename is templated as `<base>.<lang>.vtt`, not a fixed name; use a glob to find it.
- **Sonnet via `claude -p` with default `--max-turns 30` will do internal multi-turn reasoning even for pure text-in/text-out tasks.** Cowen brief: 12.5K output tokens generated for a 1.5K-token visible brief — the rest was internal planning. For non-agentic transcript-analysis style tasks, `--max-turns 1` should cut cost ~70%. Worth a session 2 ai-do.sh enhancement (perhaps a sibling `ai-do-single.sh` for single-shot use cases).
- **bash `${VAR//pattern/replacement}` handles newlines and special chars in $VALUE cleanly** — better than sed for template-substitution when the replacement is multi-line content (markdown transcripts, brief templates). Bash 3.2 supports this. Use it instead of sed/awk for multi-line variable insertion.
- **The analyst's quote-attribution pattern: start-of-spoken-thought timestamp.** Quotes span multiple VTT cue lines, attributed to the timestamp of the *first* line of the spoken sentence. Operator confirmed this is desired (complete sentences > strict line-by-line fragments). Future iterations should preserve this even if the prompt could be read as "one cue per quote."
- **Per-channel cost attribution via AGENT_ID_OVERRIDE = `analyst_<slug>`** is the cleanest pattern for downstream dashboards. The Phase 2 telegram_bot_<purpose> tagging generalises here — each pipeline gets a tag prefix, each per-channel/per-call gets a suffix.

## Operator corrections

- **Brief-quality acceptance (with notes for session 2):** verbatim — *"Accept both briefs as-is. Sign off step 10 — log session_1_complete."* Operator chose Option (a) — accept the analyst's slightly-overreaching Casper output and informative-but-imprecise quote timestamps — over iterating the prompt now. Reasoning: content quality is high; the deviations are additive (more info, not less); future tightening can happen if patterns of bad output emerge.
- **Telegram outlook-format decision came from the operator, not from me.** I had not surfaced this question at all — operator's correction was to specify the contract proactively for session 2's briefer. The contract (Option A: code-block monospace grid with em-dash for blanks) is captured both in the `session_1_complete` audit-row payload and here.
- **"How can I read everything these agents produced by watching the videos? Can you put it all in my downloads."** Operator wanted phone/desk-side grading material in `~/Downloads/`, not buried in `~/brain/channels/`. Reinforces the **phone-first** principle from Phase 2: artifacts should be where the operator is, not where the agents are.
- **No corrections of my technical reasoning this session.** The TCC/EPERM diagnosis from yesterday was the last big "you got the shape right but the name wrong" correction. This session's bug-finds (bash 3.2 heredoc, VTT rolling dedup, jq control-char) were mine to surface and fix without operator pushback.

## What's next

- **Session 2 — Curator + Briefer + cron activation.** Spec to be written by operator. Known scope from session 1:
  - **Curator** agent: read Cowen briefs (treatment=rich, `expertise_enabled: true` in `watchlist.yaml`) and accrete knowledge into `~/brain/expertise/cowen/`. Casper is `expertise_enabled: false` so curator skips it.
  - **Briefer** agent: morning Telegram roll-up of yesterday's briefs. Contract from session 1: outlook tables on-disk stay as markdown; in Telegram, briefer converts them to **monospace code-block grids** with em-dash (—) for blanks, asset names left-aligned, arrow+conviction centered, wrapped in triple-backtick for MarkdownV2.
  - **Cron activation**: enable `*/15 * * * * /Users/mmm2/agents/market/scribe/poll.sh` once we've done a few more manual runs.
  - **Whisper fallback** for videos with no auto-captions (rare on Cowen/Casper but possible on smaller channels we may add).
  - **Prediction scoring**: track each `Predictions made` entry against actual price data over time; produce a per-channel hit-rate metric. Needs a price data source (CoinGecko free tier likely).
  - **`ai-do-single.sh` cost-optimization wrapper**: `--max-turns 1` variant for pure text-in/text-out tasks like the analyst. Could cut analyst spend ~70%.
- **Open backlog from prior sessions (still not in this session's commits):** 8 modified + 6 untracked files from Phase 2 telegram stack work sit uncommitted in `~/agents/`. Surfaced twice now. Either (a) retro commit before session 2 ships, (b) operator decides it's acceptable as long as the working tree state is reproducible, or (c) operator does it themselves with a personal git workflow.
- **Memory updates pending:** add the bash-3.2 heredoc+apostrophe quirk to the gotcha list. Add the YouTube channel-handle-drift / resilient-resolution lesson. Both belong in `HANDOFF.md` gotchas section after this /ctx run finishes.
- **No active bugs, no in-progress build threads.** Market scribe pipeline is shippable; awaiting curator + briefer + cron activation in session 2 to close the full loop.
- **D028 candidate (not yet logged to DEFERRED.md):** YouTube channel handle drift — resilient resolution pattern needed for future channel additions. Trigger to log it formally: when a third channel is added to the watchlist OR when one of the existing two renames again.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_1813.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-15 18:13 **Last session summary:** Phase 2 Telegram stack built and signed off in one sitting — text bot (A), Whisper voice transcription with confirm-flow (B), full /ctx narrative pipeline, destructive-verb safety gate (D). launchd auto-restart (C) deferred. `phase_2_telegram_complete` audit row #121 fired at 18:00:22. Three engineering items deferred to `DEFERRED.md` (D025/D026/D027).  --- 
