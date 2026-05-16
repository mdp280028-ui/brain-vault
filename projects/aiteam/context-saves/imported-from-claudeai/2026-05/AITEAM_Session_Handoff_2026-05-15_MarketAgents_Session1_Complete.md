# AITEAM Session Handoff — Market Agents Session 1 Complete
**Created:** May 15, 2026 (late evening Pacific)
**Operator:** Stonecreed (Bonners Ferry, ID)
**Session topic:** Built the first half of the YouTube market-watching agent pipeline (scribe + analyst). Smoke test passed end-to-end on real Cowen and Casper videos.
**Project:** AITEAM (separate from PayrollInsider/UST/SMART)

---

## ONE-PARAGRAPH SUMMARY

Built and shipped two new agents (`scribe`, `analyst`) under `~/agents/market/`. They form the ingestion half of a 4-agent pipeline for watching YouTube market commentary channels (Benjamin Cowen + Jayson Casper), extracting structured signals from each new video, and accreting an expert knowledge base over time. Session 1 closed cleanly: real briefs generated for one Cowen and one Casper video, costs were trivial (~$0.33 combined), and operator confirmed brief quality is good. Session 2 builds the second half: curator (Cowen knowledge accretion), briefer (morning Telegram roll-up in monospace format), and `/important` slash command for Opus model override on key videos. The earlier deferred Opus-vs-Sonnet side-by-side experiment ($1.30 one-time) also moves to session 2.

---

## WHY THIS PROJECT EXISTS

Operator wanted an agent that watches YouTube videos for him and surfaces market timing signals. Specifically:

- "When to invest" decisions — Cowen (cycle/macro) + Casper (short-term levels) are his two primary information sources
- Doesn't want to drop URLs manually — agent should monitor channels automatically
- Wants a distilled brief per video, not the full transcript
- Wants a permanent knowledge base for Cowen ("everything he talks about, stored somewhere we can trim later")
- Wants outlook arrows: ↑/↓/→ with conviction 1-5 across short/mid/long timeframes
- Different timeframe definitions per analyst:
  - Cowen: short=weeks, mid=months, long=year (he thinks in cycles)
  - Casper: short=days, mid=weeks, long=months (he thinks in swings)
- Telegram is the primary delivery surface (operator is most likely to read briefs on his phone)

**This is NOT an AITEAM revenue agent.** It uses the Max subscription but doesn't generate revenue toward the $200/mo mission bar. Operator was explicit it's one of the most important agents *to him personally* for market timing decisions. Flagged the trade-off: this competes for build time with SSG agent work that does move revenue. Operator chose to build it anyway. Don't re-litigate.

---

## ARCHITECTURE — THE 4-AGENT PIPELINE

```
~/agents/market/
├── scribe/       ✅ BUILT  — polls YouTube every 15min, fetches transcripts via yt-dlp
├── analyst/      ✅ BUILT  — generates per-video structured brief (Sonnet via ai-do.sh)
├── curator/      ⏳ SESSION 2  — Cowen knowledge accretion to ~/brain/expertise/cowen/
└── briefer/      ⏳ SESSION 2  — morning Telegram roll-up in monospace format
```

### Data flow

```
poll.sh (cron, every 15min)
  → detects new videos via YouTube Data API
  → writes job file per new video to scribe/workspace/
  → INSERT into seen_videos table

fetch_transcript.sh (triggered by job file)
  → yt-dlp pulls auto-captions as VTT
  → cleans VTT (strips styling, dedups rolling overlaps, formats [MM:SS])
  → writes to ~/brain/channels/<channel>/transcripts/<date>_<videoid>.md
  → UPDATE seen_videos.transcript_fetched=1

analyze.sh (triggered after transcript)
  → loads channel-specific template (Cowen rich, Casper lean)
  → loads previous brief for thesis-shift detection
  → ai-do.sh (Sonnet 4.6) fills template from transcript
  → writes to ~/brain/channels/<channel>/briefs/<date>_<videoid>.md
  → UPDATE seen_videos.analyzed=1

curator (session 2, nightly batch)
  → reads new briefs since last run
  → for Cowen only: updates ~/brain/expertise/cowen/<topic>/synthesis.md
  → appends to evidence-log.md and predictions-log.md
  → light dupe guardrail (≥0.90 cosine similarity blocks fragmented topic creation)
  → autonomous topic creation enabled — operator can trim later

briefer (session 2, daily 6am)
  → reads briefs from last 24h
  → reformats outlook tables to monospace code-block grid (Option A)
  → composes morning brief, sends via existing Telegram bot
```

---

## WHAT GOT BUILT THIS SESSION (Session 1)

| Component | Location | Notes |
|---|---|---|
| YouTube Data API v3 key | `~/agents/config/.env` `YOUTUBE_API_KEY=` | Restricted to YouTube Data API v3 only, mode 600, gitignored. Project: `aiteam-market-scribe` in Google Cloud Console. |
| Watchlist config | `~/agents/market/scribe/watchlist.yaml` | Cowen + Casper channel IDs, treatments (rich/lean), timeframe labels |
| Folder structure | `~/agents/market/`, `~/brain/channels/` | scribe, analyst, shared, channels/cowen, channels/casper |
| yt-dlp | brew installed | For transcript fetching |
| seen_videos.db | `~/agents/market/shared/seen_videos.db` | SQLite, tracks processed video IDs to prevent re-processing |
| poll.sh | `~/agents/market/scribe/poll.sh` | YouTube monitor, idempotent re-runs verified |
| fetch_transcript.sh | `~/agents/market/scribe/fetch_transcript.sh` | yt-dlp + VTT cleaner |
| Brief templates | `~/agents/market/analyst/brief_cowen.template.md`, `brief_casper.template.md` | Cowen rich (10 sections), Casper lean (outlook + levels only) |
| analyze.sh | `~/agents/market/analyst/analyze.sh` | Calls ai-do.sh (Sonnet), fills template, detects thesis shifts |
| README with cron lines | `~/agents/market/README.md` | Cron documented but NOT activated yet — manual runs only |
| Smoke test briefs | `~/brain/channels/cowen/briefs/2026-05-15_FgxAe_NAh5c.md` (6 KB), `~/brain/channels/casper/briefs/2026-05-15_zyHJRBIEVcs.md` (2 KB) | Both human-quality verified |

### Channel IDs (resolved during build)

| Slug | Handle | Channel ID |
|---|---|---|
| cowen | @benjaminjcowen | UCRvqjQPSeaWn-uEx-w0XOIg |
| casper | @JaysonCasper | UCWfHVQvljwmIivf6dSI8Vzw |

Note: original spec said `@intothecryptoverse` for Cowen — that's his brand name. Actual YouTube handle is `@benjaminjcowen`. Channel ID resolution worked either way (YouTube's forHandle is lenient), but actual handle is captured for the audit trail.

---

## DECISIONS MADE THIS SESSION (KEEP THESE LOCKED)

### Decision 1 — Two-output pattern (per-video brief + cumulative expertise)
Each video generates a brief (what was said in this video). Cowen videos also contribute to a topic-organized expert knowledge base (what we know about each topic across all videos). Casper does NOT generate expertise files — operator wants outlook-only from him, no knowledge accretion.

### Decision 2 — Asymmetric treatments per channel
- **Cowen = rich**: 10-section template, broad knowledge harvest to `~/brain/expertise/cowen/`, autonomous topic creation
- **Casper = lean**: outlook + price levels only, no expertise folder, trading techniques explicitly excluded (toggle `include_trading_techniques: false` in watchlist.yaml — can flip to true later)

### Decision 3 — Topic taxonomy is hybrid (seeded + auto-discovered + operator-editable)
- Curator can create new topics autonomously
- Operator can create/delete topic folders manually on disk
- Light dupe guardrail (≥0.90 cosine similarity) prevents fragmentation ("fed-policy" + "fed-policy-impact" + "powell-policy" emerging separately)
- Pre-defined seed list optional — operator may provide one before session 2, otherwise curator starts from zero

### Decision 4 — Telegram outlook format = Option A (monospace code block)
Markdown tables don't render in Telegram. Briefer (session 2) converts at send-time:
- Asset names left-aligned, arrow+conviction centered
- Em-dash (—) for blank/unaddressed cells (NOT empty space)
- Whole grid wrapped in triple-backtick code block
- On-disk brief format unchanged (markdown tables render fine in Obsidian)

Example of target Telegram output:
```
Asset   Year     Months   Weeks
BTC     ↓ 4      ↓ 3      —
ETH     —        —        —
SPX     ↓ 3      ↓ 3      —
Gold    —        —        —
```

### Decision 5 — Sonnet 4.6 is the default model tier
Per-video cost: Cowen ~$0.26, Casper ~$0.07. Daily cost at 2 videos/day ≈ $19/mo — well under the $200 Max bar. Opus would be 5x more expensive (~$94/mo) for marginal-to-zero quality improvement on structured extraction tasks (editor test from May 14 showed Sonnet ≈ Opus at r=0.995 on similar work).

Operator wants to experiment with Opus on "important videos" (session 2):
- `/important <video_id>` Telegram slash command → flags video for Opus reprocessing
- `model_override` column on `seen_videos` table (TEXT, default NULL)
- analyze.sh reads override, uses ai-think.sh (Opus) if set

### Decision 6 — Cron not activated until more manual validation
Cron lines documented in `~/agents/market/README.md` but commented out. Reason: want 1-3 more days of manual poll.sh + analyze.sh runs to confirm behavior holds before unattended automation. Activate in session 2 after curator + briefer are also live.

---

## SMOKE TEST RESULTS

Both briefs read by operator. Quality acceptable.

### Cowen brief (24-min "Powell Steps Down as Chair of the Federal Reserve")
- TL;DR: Captured headline (Powell → Warsh transition)
- Outlook grid: 4 cells filled (BTC year ↓4, BTC months ↓3, SPX year ↓3, SPX months ↓3), 8 blank (correct — he didn't address those)
- Topics: 6 kebab-case slugs (primary + secondary properly split)
- Key points: 10 substantial, distillation-quality bullets
- Quotes: 6 quotes with [MM:SS] start timestamps
- Thesis shifts: "No previous brief." ✅
- Cost: $0.26

### Casper brief (10-min "WHAT JUST HAPPENED?! Bitcoin!")
- TL;DR: Captured sideways thesis
- Outlook: BTC weeks → 3, days → 2, ETH all blank ✅
- Levels: explicit and useful (support 79.2K/78.5K, resistance 82.3K)
- No technique deep-dives (MarketCipher/Fib/Volume listed under tools section, not explained) ✅
- Cost: $0.07
- **Template overreach noted in audit:** analyst added Quotes/Tools/Topics sections beyond the lean template. Content stays at list-level (no technique explanations) so accepted. Session 2 briefer should know these fields are available on Casper briefs.

### Observation: Timestamp behavior
Analyst concatenates VTT cues to sentence boundaries with start-TS attribution. VTT auto-captions break sentences mid-word at 2-3 second boundaries; strict cue-level extraction would give unusable fragments. Concatenating to sentence boundaries and attributing to start TS is the right move. **This is desired behavior, not a deviation.** Logged in audit payload so future iterations don't "fix" it.

---

## OPERATOR CONFUSION POINTS (worth knowing about for session 2)

Operator hadn't seen markdown tables rendered before — read raw `cat` output and was confused by the pipes/dashes. We walked through:

1. Markdown tables render to actual grids in Obsidian (they look fine there)
2. The pipes/dashes in raw source = column scaffolding, not what the user sees
3. **But Telegram doesn't render tables at all** — they come through as raw text with broken alignment on mobile
4. Hence the Option A code-block monospace decision for the briefer

Operator also needed examples of how to read the outlook grid. Worth keeping in mind for session 2's briefer — the morning Telegram message should be **self-explanatory at a glance**. Don't assume the operator remembers the conviction scale or the timeframe-label mapping. Either include a one-line legend at the bottom or use very visually obvious formatting.

---

## NEXT SESSION (Session 2) — BUILD PLAN

### What gets built
1. **Curator agent** (`~/agents/market/curator/`) — Cowen knowledge accretion
   - Nightly batch (cron at e.g. 11pm)
   - Reads briefs published in last 24h
   - For each Cowen brief: identify touched topics, update synthesis.md for each, append to evidence-log.md
   - Light dupe guardrail (cosine similarity ≥0.90 blocks new topic creation, proposes filing under existing instead)
   - Cost target: ~$0.30-0.50/day
   - Local embedding model (sentence-transformers via Python) for similarity check — keeps embedding cost at zero

2. **Briefer agent** (`~/agents/market/briefer/`) — morning Telegram roll-up
   - Cron daily 6:00 AM
   - Reads briefs from last 24h
   - Composes morning message:
     - Headline section: thesis shifts since yesterday (most important signal)
     - Convergence section: where Cowen and Casper agree/disagree on BTC
     - Per-channel briefs: TL;DR + outlook grid (Option A monospace code-block format)
     - Footer: links to full briefs (path on disk, since Telegram can't open them but operator can later)
   - Uses existing Telegram bot's `notify.sh` for delivery
   - Cost target: ~$0.05-0.10/day

3. **`/important` Telegram slash command + `model_override` column**
   - ALTER TABLE seen_videos ADD COLUMN model_override TEXT DEFAULT NULL
   - Bot slash command: `/important <video_id>` writes 'opus' to that column, kicks off re-analysis
   - analyze.sh reads model_override, uses ai-think.sh (Opus) if set, otherwise ai-do.sh (Sonnet)

4. **Opus side-by-side experiment** ($1.30 one-time)
   - Reuse the Cowen smoke-test video from session 1
   - Run through Opus, store output at `<date>_<videoid>_opus.md`
   - Operator reads both briefs blind, decides whether Opus quality difference justifies the cost
   - Result determines whether `/important` becomes a frequent-use command or remains rare exception

5. **Cron activation**
   - poll.sh every 15min
   - curator nightly at 23:00
   - briefer daily at 06:00
   - Only after curator + briefer pass smoke tests

### What NOT to build in session 2
- Whisper fallback for missing captions (both channels have working auto-captions; defer until one breaks)
- Prediction-scoring layer against actual price data (CoinGecko API integration, ~$0 free tier, but premature without 30+ days of prediction data accumulated)
- Telegram group monitoring agent (D022, separate burner-account project — different scope)
- Anything for Casper trading techniques (operator explicitly deferred; one-line config flag flip when ready)

### Optional pre-session-2 task for operator
Provide an initial topic seed list for Cowen expertise — 8-15 topics across crypto/macro that he consistently covers. If not provided, curator starts with empty taxonomy and topics emerge organically. Either works. Examples to consider:
- btc-cycle-theory
- bull-market-support-band  
- fed-policy-impact
- altcoin-rotation-patterns
- eth-vs-btc-ratio
- on-chain-metrics
- equities-correlation
- gold-vs-crypto
- inflation-outlook
- pi-cycle-top-indicator
- logarithmic-regression-bands
- stablecoin-flows

---

## DEFERRED ITEMS — UPDATED FROM PHASE 2 LIST

Adding/updating items from prior sessions:

| ID | Item | Status |
|---|---|---|
| D025 | Orchestrator permission policy for Telegram-spawned sessions | Still open; doesn't block session 2 (curator + briefer run from cron, not orchestrator routing) |
| D026 | Phase C launchd auto-restart for Telegram bot | Still deferred; not blocking |
| D027 | Add `*.bak` to `~/brain/.gitignore` | Still open, 2-min fix |
| **D028 NEW** | **Whisper fallback in fetch_transcript.sh** | Both channels currently have auto-captions. Add Whisper-fallback path when first transcript fetch fails on a video operator cares about. |
| **D029 NEW** | **Prediction-scoring layer with CoinGecko price data** | Build after 30+ days of predictions accumulated in evidence-log.md. Adds analyst-scorecard.md per channel. Highest long-term value but premature now. |
| **D030 NEW** | **Heuristic auto-routing to Opus (duration >30min, title keywords)** | Only build if operator's `/important` flagging pattern reveals a learnable heuristic. May never need this. |
| **D031 NEW** | **Casper trading-technique extraction** | Toggle exists in watchlist.yaml (`include_trading_techniques: false`). Flip to `true` when operator wants this signal. ~5-min change. |
| **D032 NEW** | **Side-by-side Sonnet vs Opus experiment** | Schedule for session 2 OR session 3. $1.30 one-time. Decides whether Opus has measurable quality advantage on extraction tasks. |
| **D033 NEW** | **Cron activation for market pipeline** | Wait until curator + briefer ship + 1-3 days of manual runs validate behavior. |
| **D034 NEW** | **Telegram morning brief readability legend** | First few morning briefs may need a footer legend explaining conviction scale + timeframe mapping per channel. Decide after operator reads the first 3-5 briefs. |
| **D035 NEW** | **Update project instructions to reflect market agent work** | Operator's open decision #1 ("first site target") effectively answered: SSG remains the revenue-bearing first site, but market agents are a parallel personal-utility track. Worth updating project instructions to reflect both tracks. |

---

## STATE OF THE FULL AITEAM PROJECT

### What's live and running
- Mac Mini M4 Pro (24GB/512GB) — all peripherals operational
- 3 original agents: orchestrator (Sonnet), librarian (Haiku), editor (Sonnet, r=0.933)
- 2 new agents (session 1 today): scribe, analyst
- Dashboard at :3141 (KPI cards, cost chart, diary view, war room with @mentions and discuss mode)
- Brain vault git-tracked, nightly auto-commit at 23:55 to private GitHub
- `/ctx` slash command on CC for context saves
- Telegram bot live with text + voice (Whisper) + slash commands + destructive verb gate
- Token usage logging through all wrappers
- Audit log: ~130+ rows

### What's still NOT live
- Curator + briefer (session 2)
- Cron activation for market pipeline
- Phase C launchd (bot still dies on terminal close — D026)
- D025 orchestrator perms fix (blocks LLM-routed operational queries via Telegram)
- Any AITEAM revenue work (SSG agents not yet started — D005 affiliate enrollment for SSG also outstanding)
- Telegram group monitoring agent (D022 — separate burner account project)

### Cost accounting
- Session 1 today: ~$0.33 (Cowen brief $0.26 + Casper brief $0.07)
- Phase 2 total (May 15 earlier sessions): ~$1.50
- Cumulative AITEAM Max-sub burn through today: ~$5 estimated
- All numbers are Max-subscription value consumed, NOT real dollars (per D018)

---

## IMPORTANT CONTEXT FOR NEXT CLAUDE

### About the operator
- Bonners Ferry, ID — rural northern Idaho
- No coding background. Relies entirely on Claude Code on the Mac Mini for execution
- Reads briefs on phone (Telegram) primarily, Obsidian on Mac secondarily
- Terse, direct shorthand. `g` at end of message = wants shortest possible answer, hard ceiling 6 sentences
- Default to recommendation, not options. Pick one, defend briefly.
- Stays strictly in declared session mode — don't drift
- Asks "what is this?" / "how do I read this?" questions when format/syntax is unfamiliar. Don't assume markdown/CLI/syntax knowledge. Walk through it.
- Operator's biggest execution risk = research-before-building. Push toward shipping when he drifts into analysis paralysis.

### About sign-off discipline (PROJECT RULE)
- ✅ requires the pass condition was EXECUTED and produced expected output
- Artifact-exists ≠ ✅
- Honest ⚠ beats premature ✅
- CC modeled this beautifully this session — caught its own template overreach in Casper brief, flagged honestly rather than papering over, asked operator to decide accept-as-is vs iterate. Continue crediting CC's pushback.

### About the market-agent build philosophy
- Cowen = wide knowledge harvest (the goal is a personal Cowen-pedia)
- Casper = narrow signal extraction (the goal is timing the markets)
- Treat them differently in code, templates, and storage
- Knowledge accretion happens at ingestion time (curator rewrites synthesis files) — this is intentional, not just RAG with extra steps
- Topic proliferation is FINE early — operator will trim later. Light dupe guardrail (≥0.90) only catches obvious dupes.

### Communication style for session 2
- Format CC prompts in titled code blocks for easy copy/paste
- Time estimates on every CC prompt: ⏱️ CC estimate: X min, bottleneck: Y, your involvement: Z
- Push back honestly when something seems off. Don't validate; integrate corrections and move on.
- Avoid TTWC footer (permanently disabled per project instructions)

### Critical state notes
- Telegram bot is currently running in FOREGROUND. Dies on terminal close / sleep / reboot until D026 launchd resolves.
- The Mac Mini "hardly ever shuts off" per operator preference. Real risk is silent bot death.
- D025 orchestrator perms wall means Telegram operational queries beyond Bash(df)/web search will hit a permission wall. For session 2 this matters only if briefer or curator tries to be queried through orchestrator routing — they shouldn't be; they run from cron.

---

## FIRST MOVES FOR NEXT SESSION

When operator opens session 2:

1. **Confirm session 1 context save loaded** — make sure this file + the auto-harvested CC `/ctx` save are in project knowledge before any build prompts
2. **Ask about the topic seed list** — did operator decide to provide one, or curator starts empty?
3. **Confirm session 2 scope is still curator + briefer + `/important` + Opus side-by-side** — operator may want to add/cut/reorder
4. **Confirm Option A monospace format is still the call** — operator agreed late in session 1; double-check on a fresh read
5. **Write the CC build prompt for session 2 in one block** — operator copy/pastes
6. **Start with curator (the harder of the two)** — get it right before briefer depends on it

### The 80/20 for session 2
- Curator quality matters MORE than briefer. Briefer is glorified template rendering. Curator is actual synthesis work.
- Spend more iteration time on curator's prompt than on briefer's structure.
- Test curator on the existing Cowen brief from session 1 — should produce 1-2 topic files (probably fed-policy-transition and equities-correlation given that video's content).

---

## DON'T FORGET

- **Session 1 audit row** = `session_1_complete` (look up ID in audit_log when needed)
- **Smoke test files** still on disk at:
  - `~/brain/channels/cowen/briefs/2026-05-15_FgxAe_NAh5c.md`
  - `~/brain/channels/casper/briefs/2026-05-15_zyHJRBIEVcs.md`
- **CC context save** from session 1 lands at `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_HHMM.md` after `/ctx` runs
- **Project instructions** (v2 in project knowledge) — read first if anything in this handoff feels stale

---

*End of handoff. Operator: drop this file into ~/Downloads/ then run /ctx on CC to trigger auto-harvest. Also drop into Claude.ai project knowledge for session 2.*
