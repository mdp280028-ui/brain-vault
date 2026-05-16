# AITEAM Context Save — 2026-05-15 Market Agents Session 1
**Date:** May 15, 2026 (evening Pacific)
**Session topic:** Designed and built the first half of the YouTube market-watching agent pipeline. Scribe + analyst shipped end-to-end with smoke test verified on real Cowen and Casper videos.

---

## TL;DR

Operator asked if an agent could watch YouTube videos for him automatically. Conversation evolved from "is this possible" through architecture design (4-agent pipeline: scribe → analyst → curator → briefer) into a built and verified session 1: scribe (YouTube poll + transcript fetch) and analyst (per-video brief generation) are live. Smoke test produced clean briefs for one Cowen video (24min, $0.26) and one Casper video (10min, $0.07). Operator confirmed brief quality acceptable. Session 1 closed. Comprehensive handoff file written for session 2.

---

## HOW THIS SESSION STARTED

Operator opened with: "I want a ai agent that can watch YouTube videos for me and become an expert on a YouTube channel..."

The conversation evolved through these turns:
1. Confirmed feasibility — yes, 3-tier approach (transcripts, Whisper fallback, Gemini video as overkill)
2. Operator clarified use case: "this is one of the most important agents — lets me know when the right time to invest in the markets is"
3. Pushed back honestly: this is a personal-utility agent, not an AITEAM revenue agent, competes with SSG build time. Operator chose to build anyway.
4. Two channels locked: Benjamin Cowen (@benjaminjcowen) and Jayson Casper (@JaysonCasper)
5. Asymmetric treatment decided: Cowen rich (wide knowledge harvest), Casper lean (outlook + levels only, defer trading techniques)
6. Hybrid topic taxonomy (seeded + auto-discovered + operator-editable)
7. Light dupe guardrail (≥0.90 cosine similarity)
8. Per-channel timeframe semantics: Cowen (weeks/months/year), Casper (days/weeks/months)
9. Walked operator through Google Cloud Console for YouTube Data API v3 key
10. Wrote CC build prompts in 2 phases: (a) API key + key verification, (b) full scribe + analyst build through smoke test
11. Operator ran builds on CC; both passed
12. Operator read smoke-test briefs; quality confirmed
13. Cost question came up — Sonnet vs Opus. $19/mo Sonnet vs $94/mo Opus. Decision: stay Sonnet, add `/important` flag for Opus override on key videos in session 2.
14. Confusion about reading markdown tables resolved (operator reads CC raw output, not Obsidian-rendered). Identified that Telegram doesn't render tables at all → Option A code-block monospace format decided for briefer.
15. Comprehensive handoff written for session 2

---

## DECISIONS MADE

### Decision 1 — Four-agent market pipeline architecture
Scribe (YouTube monitor + transcript fetch) → Analyst (per-video brief) → Curator (Cowen knowledge accretion) → Briefer (morning Telegram roll-up). Sessions 1 ships first two, session 2 ships last two.

### Decision 2 — Asymmetric channel treatment
- Cowen: rich template (10 sections), expertise folder enabled, broad topic auto-creation. Goal: personal Cowen-pedia.
- Casper: lean template (outlook + levels only), no expertise folder, trading techniques excluded via config flag. Goal: market timing signal.

### Decision 3 — Hybrid topic taxonomy with light dupe guardrail
- Operator can seed topics manually (or skip)
- Curator can auto-create new topics
- Cosine similarity ≥0.90 blocks fragmented dupes ("fed-policy" + "fed-policy-impact" + "powell-policy")
- Operator can delete/trim folders freely on disk

### Decision 4 — Per-channel timeframe labels
- Cowen: short=weeks, mid=months, long=year (cycle theorist)
- Casper: short=days, mid=weeks, long=months (swing trader)

### Decision 5 — Telegram outlook format = Option A (monospace code block)
Markdown tables don't render in Telegram. Briefer converts at send-time:
- Em-dash (—) for blank cells
- Aligned columns inside triple-backtick code block
- On-disk markdown table preserved (renders fine in Obsidian)

### Decision 6 — Sonnet 4.6 default, Opus available via override
- Default: ai-do.sh (Sonnet) for all analysis. Cost: ~$19/mo at 2 videos/day.
- Override: `/important <video_id>` Telegram slash command writes 'opus' to `seen_videos.model_override` column. analyze.sh reads override, uses ai-think.sh (Opus) when set.
- Side-by-side experiment ($1.30 one-time) scheduled for session 2 to validate Opus is worth flagging at all.

### Decision 7 — Sign-off discipline modeled correctly by CC
CC caught its own template overreach on Casper brief (added Quotes/Tools/Topics beyond lean template), flagged honestly, asked operator to accept-as-is vs iterate. Operator accepted as-is, noted in audit payload. Continue crediting CC pushback.

### Decision 8 — Cron not activated until session 2
poll.sh + analyze.sh run manually for 1-3 more days. Cron lines documented (commented) in `~/agents/market/README.md`. Activate after curator + briefer ship.

---

## FILES CREATED/MODIFIED ON MAC MINI

### Configuration
| File | Status |
|---|---|
| `~/agents/config/.env` | Added `YOUTUBE_API_KEY=<value>` (mode 600, gitignored) |
| `~/agents/market/scribe/watchlist.yaml` | Cowen + Casper channel IDs + treatment configs |
| `~/agents/market/README.md` | Documents (commented) cron lines for future activation |

### Database
| Object | Status |
|---|---|
| `~/agents/market/shared/seen_videos.db` | New SQLite, tracks processed video IDs |
| `seen_videos` table | Schema includes video_id, channel_slug, published_at, title, duration_seconds, first_seen_ts, transcript_fetched, analyzed |

### Code
| File | Status |
|---|---|
| `~/agents/market/scribe/poll.sh` | Polls YouTube API every 15min (planned), idempotent re-runs verified |
| `~/agents/market/scribe/fetch_transcript.sh` | yt-dlp + VTT cleaner (dedups rolling captions, strips styling tags, formats [MM:SS]) |
| `~/agents/market/analyst/analyze.sh` | Loads channel template, calls ai-do.sh (Sonnet), writes brief, updates seen_videos |
| `~/agents/market/analyst/brief_cowen.template.md` | Rich template (10 sections) |
| `~/agents/market/analyst/brief_casper.template.md` | Lean template (outlook + levels only) |

### Output (smoke test artifacts)
| File | Size | Status |
|---|---|---|
| `~/brain/channels/cowen/transcripts/2026-05-15_FgxAe_NAh5c.md` | 26 KB | Powell Steps Down transcript |
| `~/brain/channels/cowen/briefs/2026-05-15_FgxAe_NAh5c.md` | 6 KB | Sonnet-generated brief |
| `~/brain/channels/casper/transcripts/2026-05-15_zyHJRBIEVcs.md` | 13 KB | Bitcoin sideways range transcript |
| `~/brain/channels/casper/briefs/2026-05-15_zyHJRBIEVcs.md` | 2 KB | Sonnet-generated brief |

### Google Cloud Console
- Project: `aiteam-market-scribe` created
- YouTube Data API v3 enabled
- API key created, restricted to YouTube Data API v3 only

### Audit log
- ~10 new rows this session (API key provisioned, channel IDs resolved, each step of scribe/analyst build, session_1_complete)

---

## LESSONS LEARNED

1. **Walking the operator through Google Cloud Console step-by-step works better than dumping all the steps at once.** Screenshot-driven progression let operator catch UI changes (Google now combines key creation + restriction into one modal) without breaking flow.

2. **Cowen's actual handle (@benjaminjcowen) is different from his brand (@intothecryptoverse).** YouTube's forHandle endpoint is lenient with channel aliases — both resolve to same channel ID. CC caught the spec/reality mismatch and logged correctly in audit. Worth knowing for any future channel resolution work.

3. **Sonnet's $0.26 Cowen brief includes ~12K internal reasoning tokens via Claude Code's `--max-turns 30` default.** Tightening to `--max-turns 1` would cut cost significantly. Logged but not changed — out of scope for session 1.

4. **VTT cue-level extraction would produce unusable fragments.** Auto-captions break sentences mid-word at 2-3 second boundaries. The analyst concatenates to sentence boundaries with start-TS attribution — this is the right behavior and was logged as such in audit. Don't "fix" in future iterations.

5. **Operator reads raw markdown in CC terminal, not rendered Obsidian.** When asking format questions, defaults to seeing source code (pipes/dashes). Explanations need to show both raw vs rendered. Specifically for Telegram delivery: tables don't render at all → Option A monospace code-block format is the answer.

6. **Asymmetric channel treatment was the right call.** Operator wanted everything Cowen says stored for later trimming, but only outlook from Casper. Single-template approach would have forced compromises in both directions. Per-channel templates + per-channel expertise flags isolate the asymmetry cleanly.

7. **Cost math for "should I use Opus" needs concrete numbers, not vibes.** Showed operator the $19/mo Sonnet vs $94/mo Opus calculation at 2 videos/day. Real numbers led to the right decision (default Sonnet + Opus override for flagged videos) without going in circles.

8. **The Casper template overreach (analyst added unrequested sections) was correctly flagged by CC and correctly accepted by operator.** The extra fields were informative not harmful, and noted in audit so session 2 briefer knows they're available. Project's sign-off discipline keeps working.

9. **Operator gets confused by outlook grid format on first read.** Took multiple explanation passes (what arrows mean, what conviction numbers mean, what the column headers mean, why blanks ≠ neutrals). Briefer in session 2 must produce something self-explanatory at first glance. Worth a footer legend on the first few morning briefs.

10. **Conversation drifted toward research mode after the build completed.** Operator's open question about Opus cost could have spiraled into a long detour. Recognized the pattern, gave concrete recommendation (stay Sonnet, run side-by-side later, use `/important` for overrides), moved forward. The research-before-building flag from project instructions is real and worth catching in real-time.

---

## OPERATOR CORRECTIONS

1. **"Cc hasn't done option A yet include that fix in the prompt"** — operator initially thought Option A was something CC needed to do this session. Clarified: Option A is a session 2 deliverable (briefer agent doesn't exist yet), but the spec for it needed to be locked into the audit trail this session. Updated close-out prompt to capture as session 2 contract rather than active work item.

2. **"What model is watching these videos?"** — operator wanted to know mid-build what was doing the analysis. Confirmed Sonnet 4.6 via ai-do.sh. Distinguished from the Haiku tokens in CC's report (those are CC's own internal routing, not the analyst work). Worth keeping the distinction explicit in future cost reports.

3. **"How am I supposed to tell what timeframe its talking about?"** — operator's first read of the outlook grid. Needed explicit per-channel column-header mapping. Indicated briefer's Telegram delivery needs to be obvious at glance. Folded into D034 (legend in first morning briefs).

---

## DEFERRED ITEMS (new this session)

| ID | Item | Trigger |
|---|---|---|
| D028 | Whisper fallback in fetch_transcript.sh | First time a transcript fetch fails on a video operator cares about |
| D029 | Prediction-scoring layer with CoinGecko price data | 30+ days of predictions accumulated in evidence-log.md |
| D030 | Heuristic auto-routing to Opus (duration/keywords) | Only if `/important` flagging pattern reveals a learnable heuristic |
| D031 | Casper trading-technique extraction | When operator wants this signal — toggle flag in watchlist.yaml |
| D032 | Side-by-side Sonnet vs Opus experiment ($1.30) | Session 2 or 3 |
| D033 | Cron activation for market pipeline | After curator + briefer ship + 1-3 days manual validation |
| D034 | Telegram morning brief readability legend | After operator reads first 3-5 morning briefs |
| D035 | Update project instructions to reflect market agent track | Next project-instructions edit cycle |

Open from prior sessions still standing: D025 (orchestrator perms), D026 (launchd), D027 (.bak gitignore), all the D001-D024 from previous handoffs.

---

## WHAT'S NEXT (Session 2)

### Build plan
1. **Curator agent** — Cowen knowledge accretion to `~/brain/expertise/cowen/`, light dupe guardrail (≥0.90 cosine), nightly batch
2. **Briefer agent** — morning Telegram roll-up at 6:00 AM in Option A monospace format
3. **`/important` slash command + `model_override` column** — enables Opus override per video
4. **Opus side-by-side experiment** — $1.30 one-time, decides whether `/important` becomes frequent or rare
5. **Cron activation** — poll.sh every 15min, curator nightly, briefer 6 AM

### Pre-session-2 optional task for operator
Provide topic seed list for Cowen expertise (8-15 topics across crypto/macro), or skip and let curator start empty.

### First moves at session 2 start
1. Confirm both context saves (this file + handoff file) loaded in project knowledge
2. Ask about topic seed list
3. Confirm session 2 scope still applies (operator may want to reorder)
4. Confirm Option A monospace format still the call
5. Start with curator (harder; briefer depends on it)

---

## WHERE TO RESUME

**Last context save before this one:** `AITEAM_Context_Save_2026-05-15_Phase2_Complete.md` (Phase 2 Telegram bot closure earlier today)

**Active state:**
- Mac Mini M4 Pro live, all peripherals operational
- 5 agents total: orchestrator, librarian, editor, scribe (NEW), analyst (NEW)
- Telegram bot live with text + voice + slash commands + destructive verb gate
- Brain vault git-tracked with nightly GitHub auto-commit
- Dashboard at :3141 fully functional
- Cron NOT activated for market pipeline — manual runs only until session 2 ships curator + briefer

**Project files updated this session (Claude.ai project):**
- `AITEAM_Session_Handoff_2026-05-15_MarketAgents_Session1_Complete.md` — comprehensive handoff for session 2
- This context save (will be auto-harvested from ~/Downloads/ if dropped there + run /ctx)

**Operator next moves:**
1. Drop the handoff file into Claude.ai project knowledge
2. Drop this context save file into ~/Downloads/ (or skip — CC's /ctx will generate its own version from the audit_log)
3. Step away from the terminal. Good stopping point.

---

*End of context save. Two new agents shipped, smoke test verified, costs trivial, handoff comprehensive. Session 1 closed cleanly.*
