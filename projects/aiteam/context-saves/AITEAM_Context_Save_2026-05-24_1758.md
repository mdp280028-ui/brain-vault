# AITEAM Context Save — 2026-05-24_1758
**Generated:** 2026-05-24T17:58-0700 (hand-authored; see collision note)
**Session topic:** domain-hunter (v0.1 build + cron, then v0.1.5 topical-fit refit + cron), idea-agent synthesize-phase fix + weekly→daily cadence, Issues Queue dashboard view + capture daemon + cron. Four agents touched, three crons installed.

> **ctx.sh collision:** my `ctx.sh` run at 17:56 produced `_1756.md`, but a parallel **sister chat** ran `ctx.sh` in the same minute and overwrote that skeleton with its own session (D066/D072 editor cap + asbestoshq.com internal-link audit/fix saga — commits `36cb356 / 711748b / 81ab725 / c0b6471`, none mine). To avoid trampling the sister's narrative I authored this save as `_1758.md`. The mechanical git/audit record is omitted (the anchor captured nothing); commit SHAs are cited inline below.

---

## Mechanical record
Narrative-only save (anchor empty). My commits this session, in order:

- domain-hunter v0.1: `ee9e34c` scaffold+migration, `cf99967` fetchers, `75aa423` hard-filter+scoring, `5a31df0` wayback, `8fa1f66` digest+scan.sh, `0a4adad` cron staged + first run verified. Cron installed live (06:30).
- domain-hunter v0.1.5: `8c4ec29` sites.yaml+topical columns, `dd52c53` topical_fit.py, `4f554d9` gate+digest, `c2d5a49` backfill, `73ad067` docs, `044d619` step-label fix.
- idea-agent: `ca21007` prompt tighten, `344e34b` (budget cap — later reverted), `5329a09` robust parser, `8722089` retry-once, `6192499` weekly→daily, `77f11a8` docs, `353d1d3` remove budget cap, `ce46867` cost-doc correction. Cron switched live (Sun 09:00 → daily 07:10).
- Issues Queue: `bdc1909` agent_issues table, `eded032` capture.sh, `d4df59d` /issues view+counter, `f133cac` status/notes routes, `40d7f6c` CC-prompt modal, `0d099ab` cron staged. Cron installed live (*/5).

---

## Decisions made this session

- **domain-hunter shipped as v0.1.5 "topical-fit only," not the operator's first v0.2 spec.** The v0.2 prompt referenced columns/concepts that never existed in our build (`weak_301`/`strong_301`/`niche_rebuild` use_case enum, `topical_fit_score`, `backlink_score`, `quality_score`, `time_remaining_hours`, `target_site`, auction price/TF/CF/RDs). v0.1 was a brandability + Wayback scout with business-niche *tags*, no topical scoring, no auction/backlink data (free ExpiredDomains tier exposes none). I stopped at pre-flight, surfaced the mismatch, and the operator re-specced as v0.1.5: Haiku topical-fit scoring against 3 named sites (ust/asbestos/ssg), gate `topical_fit_max >= 0.3`, brandability demoted to secondary. Alternative rejected: fabricating the v0.2 schema to match the prompt.
- **idea-agent synth gets NO cost cap.** Diagnosed the synth failure (Sonnet rambled 34K tokens + narrated multi-fence output; brittle single-regex parser). Fixed with hardened prompt (single JSON object) + 3-tier parser (raw → concat-fences → bracket-extract → exit 4) + retry-once. Then verification killed the planned $0.20 budget cap: healthy synth genuinely costs **$0.35–0.45** (18–24K output tokens), so any cap tight enough to matter sits *below* healthy cost and guillotines real runs (it did — caused an exit-2 empty-response failure). Operator chose "remove the cap; the prompt+parser+retry are the runaway guard." The `AI_DO_MAX_BUDGET_USD` passthrough stays in `ai-do.sh` for other agents.
- **idea-agent kept the `run_weekly.sh` filename + `weekly_run_*` audit-action names** when switching to daily, to avoid rippling into `idea_proposals.week_of`, the `idea_calibration` one-row-per-week contract, and `calibrate.sh`. Low-surgery; documented as legacy identifiers.
- **Issues Queue `/issues` is a separate page**, ungated like `/` (data + mutations gated), not folded into the big `/` SPA — lower risk, matches the operator's `/issues`-namespaced route list. Capture daemon is fresh-start anchored (no historical backfill) and writes a local `new_issues.log` one-liner per issue (Telegram paging deferred to v1.1 until noise rate is known).

## Lessons learned

- **The `claude` CLI (v2.1.150) has NO `--max-tokens` flag.** Output-cost bounding is `--max-budget-usd` only. A budget abort returns `is_error:true, subtype:error_max_budget_usd` with **no `result` field** — callers must read `d.get('result') or ''`, not `d['result']`, or `ai-do.sh` KeyErrors under `set -e`. Also: the budget is a soft ceiling checked between chunks/turns — it can overshoot (a $0.02 cap spent $0.05 in testing).
- **`claude --json-schema` exists but does NOT drop into the `ai-do.sh` `.result` extraction pattern.** It routes output through internal tool-use turns, leaves `.result` empty, and `error_max_turns` at `--max-turns 1` (still empty at 2). Adopting it means rearchitecting `.result` extraction in shared `ai-do.sh` — high blast radius. Avoid unless you own that rework.
- **idea-agent synthesize legitimately costs $0.35–0.45/run** (3 idea bodies + the rejected[] log + notes = 18–24K output tokens). Full chain ~$0.78/run, ~$23/mo at daily cadence (scan + google_search Haiku is ~half). Don't assume a synthesis agent is "~$0.10."
- **Wayback Machine fetch has three traps, all hit this session:** (1) snapshots are wrapped in the Wayback toolbar — insert `id_` after the timestamp (`/web/<ts>id_/<url>`) for the raw archived page; (2) Wayback replays the *original* response bytes verbatim, so gzip slips through `Accept-Encoding: identity` — detect magic bytes `1f 8b` and decompress in code; (3) archived bytes can contain control/null bytes that make `subprocess.run` raise "embedded null byte" — strip control bytes before passing to a subprocess arg.
- **bash `IFS=$'\t' read` collapses consecutive tabs** (tab is a whitespace IFS char), so an empty field shifts every later column left (a blank summary leaked the row's `id` into the summary slot). Use `\x1f` (unit separator, non-whitespace) for SQL row parsing where fields may be empty.
- **GoDaddy auctions, Ahrefs app, and ExpiredDomains/GoDaddy public pages are all Akamai/Cloudflare edge-blocked (403) to unauthenticated UA-agnostic GETs.** Free-tier scraping of GoDaddy auctions is a non-starter; the fetcher detects the edge-block and degrades gracefully (exit 4) rather than failing the run.
- **When a task prompt names columns/functions/concepts that don't exist in the codebase, STOP and surface the mismatch — don't fabricate a schema to fit the prompt.** The domain-hunter v0.2→v0.1.5 pivot turned on this; the operator confirmed the assumed `weak_301` track "never existed" in our build.
- **`INSERT OR IGNORE` on a UNIQUE key keeps the FIRST-seen row's metadata** (e.g. `correlation_id`), so funnel counts keyed by correlation_id zero out on day-2 dedup. Count against the actual input set (the fetched-domains list) instead.
- **Verify cost assumptions with a real run before committing to a cap value.** The $0.20 cap was committed (`344e34b`) then removed (`353d1d3`) one verification later. A live run is the cheapest way to invalidate a wrong premise — do it before, not after, baking the number into code.

## Operator corrections

- **domain-hunter v0.2:** the operator interrupted my clarifying `AskUserQuestion` and re-specced the whole thing as v0.1.5 topical-fit, implicitly confirming the v0.2 schema I flagged as missing was the right thing to flag (not build). Lesson reinforced: pre-flight "STOP and report" is a real gate, and surfacing a mismatch beats guessing.
- **idea-agent budget (verbatim intent):** *"Option 1 — remove the cap. … the hardened prompt + parser + retry are the real runaway guard now. … Keep AI_DO_MAX_BUDGET_USD passthrough in ai-do.sh as a feature but do NOT set SYNTH_BUDGET_USD for idea-agent."*
- **Issues capture v1 scope (verbatim intent):** *"append a one-line summary to ~/agents/issues-capture/log/new_issues.log with timestamp + agent_id + action. Not Telegram-paging it yet; that's a v1.1 decision after we see the noise rate."*
- **Commit-message accuracy is a hard rule:** I renamed the planned "cap synthesize output at 8K tokens" commit to reflect the actual `--max-budget-usd` mechanism, and the cron/cap commits name what truly changed.

## What's next

- **domain-hunter** — first autonomous daily fire **06:30 local** (v0.1.5 topical gate live). Will almost certainly send "0 candidates" until a topically-relevant domain (ust/asbestos/B2B-services) hits the expired listings. Monitor 2–4 weeks; if signal is real + ≥1 domain acquired, evaluate $10/mo ExpiredDomains Pro for backlink + auction data (→ v0.2). Operator should tap-test the Ahrefs deep link in a real digest when one arrives. GoDaddy fetcher is wired but edge-blocked (non-producing).
- **idea-agent** — first daily fire **07:10 local** (synth fix live, no cap). Watch cost (~$0.78/run, ~$23/mo Max-sub value, $0 real money on Max sub). Open follow-up: daily cadence + `week_of`/`idea_calibration` one-row-per-week semantics may want a revisit (calibrate now runs 7×/week).
- **issues-capture** — live `*/5`. Watch `new_issues.log` noise rate; if low, tighten the filter or wire Telegram paging (v1.1). Known limit: log-path/`cd` best-guess assumes top-level `~/agents/<agent>`; nested agents (e.g. `market/scribe`) show the fallback line.
- **Dashboard /issues** — clipboard copy could not be verified headlessly; operator should open `/issues?token=…`, expand an issue, click **Generate CC Prompt**, and confirm the paste works in a browser.
- **Blocked:** none.

---

## Where to resume
**This save:** `/Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-24_1758.md`
**Concurrent sister save (do not trample):** `_1756.md` (internal-link audit/fix saga).
**Active HANDOFF.md:** overwritten this session — see current state there.
