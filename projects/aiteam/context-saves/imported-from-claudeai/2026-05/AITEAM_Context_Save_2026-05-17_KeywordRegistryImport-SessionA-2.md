# AITEAM Context Save — Keyword Registry Import Session A + Handoff

**Date:** 2026-05-17 (evening session, ~21:30–23:30 PT)
**Session length:** ~2 hours
**Chat role:** Primary chat continuing from GEO Phase 3 + Backlink Prospector V01 close-out. Sister chat in parallel on Research/Opportunity Agent Session 2 → began Keyword Registry build (shipped earlier in same evening, separate session).
**Session character:** One clean closure (Session A keyword research import). Three pre-flight HALTs caught real decisions. ~700 keyword rows imported across two sites with zero LLM cost. Several high-leverage items surfaced from GSC data + cadence discussion + agent-discipline gap, all captured for next session.

---

## TL;DR

**Session A of keyword registry import complete.** Operator's research files from ~/Downloads (5 files, ~395 KB) imported into the registry built earlier in the day. Asbestos registry grew 162 → 394 rows (+232). SSG registry created fresh at 457 rows (4 live + 453 researched). 38 SSG rows tagged out-of-scope-cluster per Fork C. 19 asbestos rows tagged killed (dead clusters from Batch4 dead-angle analysis).

**Two clean commits.** Agents `86029fd`, brain `00b24a2`. Both idempotence checks drift=False. $0.00 LLM cost.

**Major discovery during session: real GSC data changed the cadence plan.** Asbestos site has 5,170 impressions / 13 clicks over 28 days at avg position 15.3 — actively ranking. Cluster signals visible: vermiculite (80+ impressions across 5 query variants), asbestos siding (~80 impressions across 7 variants), transite (~26 impressions across 6 variants), black mastic (~23), shingles (~40). Five pages getting 250+ impressions with zero clicks (CTR problem, not ranking problem).

**Several Session-B and beyond items surfaced and captured.** Most urgent: agent pre-flight discipline (D-next-A). Title/meta rewrite (D-next-B) is highest-revenue lever visible. Strategy extraction (D-next-D) finishes what registry import started.

**Mission bar:** $0 / $200 mo unchanged. Registry now holds ~850 keywords across two sites — feeds future keyword-picker agent. GSC data confirms autonomous loop is producing rankable content; CTR optimization is the next leverage point.

---

## SESSION ARC

1. Opened in this chat session asking about adapting old keyword files into the new Keyword Registry format (built earlier same evening by sister chat session). Three uploaded files in Downloads: ASBESTOS_Batch4_Keyword_Research, SMART_Master_Research, MASTER_keywords. Plus two more on disk: ASB_Keyword_Data_CONSOLIDATED, ASB_Strategy_Framework.

2. **Pre-flight discussion: scope of preservation.** Operator instinct: "most files have other useful info... maybe include originals so good info doesn't get lost." Three options surfaced (archive only / archive + parse strategy into agent-readable notes / strategy parsing via Sonnet). Operator picked Layer 1 (registry) + Layer 2 (sources archive) for this session, deferred Layer 3 (strategy extraction) to Session B.

3. **Real-content audit of uploaded files.** Sampled the three Downloads files via bash_tool. Findings: MASTER is 100% keyword data. ASB_Batch4 is ~30% keywords / ~70% strategy (dead-angle analyses, per-slug content angles, methodology notes, saturation calls, affiliate notes). SMART is ~25% keywords / ~75% strategy (cluster analysis, 8-article roadmap, affiliate programs, strategic pathways). Confirmed the Layer 3 extraction recommendation is high-leverage.

4. **Operator concern surfaced: "I want agents to read through and learn things and get smarter from all our files."** Realistic stack laid out (retrieval, structured knowledge, pattern extraction, model improvement). Operator clarified the actual problem: "agents don't check what already exists — they make duplicate files, do stupid things, repeat decisions." This is a discipline problem, not a learning problem. Solution sketched: `brain_search.sh` + mandatory pre-flight contract in CLAUDE.md. Operator parked it as D-next-A for after keyword work.

5. **Cadence question raised.** "I don't want a flood of content all at once — what's the right pace for asbestos and SSG?" Initial recommendation: 2-3/week asbestos, 1-2/week SSG. Then operator shared GSC screenshot.

6. **GSC data changed the analysis.** Asbestos at 5,170 impressions / 13 clicks / avg position 15.3 in 28 days. Not "waiting for first impressions" — at Phase 2 (impressions exist, ranking improving). Revised cadence to 4-5/week asbestos. Cluster-completer prioritization beats Batch4 spread because demand signals already visible on vermiculite, siding, transite, black mastic, shingles clusters.

7. **CTR problem identified.** 5 pages with 250+ impressions and 0 clicks. Title/meta rewrite is highest-revenue lever visible — fixable in 30 min, ~$0.05.

8. **GSC-aware keyword scoring added to deferred list.** Future keyword-picker agent should weight by GSC impressions on related queries, not just SV/KD from registry. Demand-proven beats research-estimated.

9. **Session A pre-flight executed by CC.** 5-file scan, 8 forks surfaced (A through H), 2 honest ⚠ gaps surfaced (SSG slug→keyword mapping + dead-keyword detection). 2 additional forks added (I + J). All 10 forks resolved by operator with CC's recommendations accepted.

10. **Session A build executed cleanly.** Phase 1 schema extension (researched status + --site flag + cluster column on SSG only) passed both idempotence tests. Phase 2 importer (3 parsers: ASB-tier, MASTER 9-col, SMART bold-anchor) produced 2 staging files. Phase 3 HALT delivered bucket breakdowns.

11. **Spot-check before merge.** Operator picked spot-check over trust-and-merge. 15 checks across both staging files — all clean. One cosmetic flag: 32 SSG cluster names with ~8 duplicates from MASTER kebab-case vs SMART freeform. Deferred to Session B as cluster normalization pass.

12. **Merge executed.** Both rebuilds clean, both idempotence checks drift=False. Two honest commits. Audit trail captured (2 correlation IDs).

---

## WHAT GOT SHIPPED

### Session A — Keyword Registry Import

**Agents commit:** `86029fd` — feat(keyword-registry): import_research.py + researched status + SSG site support + dead-keyword detection

**Brain commit:** `00b24a2` — feat(keyword-registry): import 206 asbestos researched + 19 asbestos killed + 457 ssg net-new + sources/ archive (5 files)

**Schema extensions:**
- `researched` status added to rebuild_registry.py status detection. Precedence: live > in_pipeline > researched > killed
- `--site` flag accepted (asbestos | ssg). Default behavior preserved when omitted.
- SSG path includes `cluster` column (9-col table); asbestos stays 8-col
- Both idempotence tests passed (drift=False with and without explicit flag)

**Importer (`~/agents/keyword-registry/lib/import_research.py`):**
- 3 parsers: ASB-tier (Keyword | SV | KD | CPC | Cluster/Notes/Purpose), MASTER 9-col canonical, SMART bold-anchor
- Dead-keyword detection: scans for "DEAD" / "trap" / "skip" / "wall" / "legal-wall" / "seller-wall" markers + Batch4 "Dead angles" section → status=killed
- SSG fuzzy match: 17 SSG page.tsx slugs cross-checked against keyword set → status=live for hits
- Strategy_Framework handled per Fork G: scanned, 0 rows contributed, audit log entry `file_scanned_no_rows`
- Outputs staging files (not direct registry writes) for operator review

**Source archive:** all 5 source files copied verbatim to `~/brain/projects/aiteam/keyword_registry/sources/` with README.md describing provenance and import dates.

**Final registry state:**

Asbestos (`keyword_registry.md` — 394 rows):
| status | count | Δ from pre-merge |
|---|---|---|
| live | 155 | +28 |
| researched | 206 | +206 (new status) |
| killed | 19 | +19 (new status) |
| unowned | 14 | −21 (upgraded to researched/live) |
| **total** | **394** | **+232** |

SSG (`keyword_registry_ssg.md` — 457 rows, new file):
| status | count |
|---|---|
| live | 4 (Fork I matches) |
| researched | 453 |
| **total** | **457** |

**Reconciliation notes (precedence working as designed):**
- 27 killed in staging → 19 killed in registry. 8 keywords (e.g. asbestos insulation, respirator, vermiculite) mentioned in shipped guides → live wins per status precedence. Correct.
- 281 researched in staging → 206 researched + 28 new live. ~75 staged researched kws either mentioned in live pages (upgraded) or alias-collapsed. Correct.

**Audit trail:**
- correlation_id `2F67A587-8DB0-4DF4-917B-A69F7497106A` — keyword_registry_imported with full per-status breakdown
- correlation_id `D2114073-8E5B-4F07-8793-884A21FAD537` — sources_archived

**Cost:** $0.00 LLM. All parsing pure regex/string ops. Well under <$0.10 budget.

---

## KEY FINDINGS / DECISIONS

### GSC data changes cadence math (highest-impact session insight)

Asbestos site is at 5,170 impressions / 13 clicks / 0.3% CTR / avg position 15.3 over 28 days. Cluster signals already visible:

| Cluster | Impressions (28d) | Existing pages | Demand signal |
|---|---|---|---|
| Vermiculite | 80+ across 5 query variants | 1 (vermiculite-insulation-guide) | Build 3-4 supporting pages |
| Asbestos siding | ~80 across 7 variants | asbestos-siding-guide (516 imp / 2 clicks) | Build 2-3 supporting pages |
| Transite | ~26 across 6 variants | transite-pipe-guide (270 imp / 0 clicks) | Build Batch4 transite slugs |
| Black mastic | ~23 across 3 variants | black-mastic-guide (512 imp / 1 click) | Build Batch4 black-mastic slug |
| Shingles | ~40 across 5 variants | asbestos-shingles-guide (272 imp / 2 clicks) | Build supporting pages |

**Cadence recommendation revised:**
- Asbestos: 4-5/week (not the conservative 2-3/week initial recommendation)
- SSG: 1-2/week post-launch (unchanged, gated on site rewrite)
- Phase triggers: keep the 4-phase progression (first impressions → first ranking <50 → first traffic → first $)

**Cluster-completer prioritization beats Batch4 spread.** Vermiculite/siding/transite/shingles clusters have proven demand; PPE/HEPA/encapsulation clusters from Batch4 have no demand signal on the site yet. Build the compounders first.

### Five zero-click pages = highest revenue lever visible

These pages have 250+ impressions and 0 clicks over 28 days:
- asbestos-inspection-cost (417 imp)
- asbestos-remediation-cost (330 imp)
- when-was-asbestos-used-in-homes (310 imp)
- transite-pipe-guide (270 imp)
- is-popcorn-ceiling-asbestos (259 imp)

CTR problem, not ranking problem. Title/meta rewrite (D-next-B) is ~30 min CC + $0.05. Likely 2-3x click lift on these pages. Higher revenue leverage than shipping more content.

### Real problem is agent discipline, not agent intelligence

Operator's "agents read files and get smarter" question turned out to mean "agents don't check what already exists, they make duplicate files, repeat decisions." This is a discipline gap, not a learning gap. Solution sketched and parked as D-next-A: `brain_search.sh` (markdown-aware grep across brain + agents + projects) + mandatory pre-flight contract in CLAUDE.md. ~45 min CC, $0, every future agent inherits the discipline.

### Fork decisions pinned (do not re-derive)

| Fork | Decision |
|---|---|
| A | Separate file `keyword_registry_ssg.md`. Asbestos stays as-is. |
| B | Skip UST files entirely (separate project). |
| C | Import all 262 MASTER keywords. cybersecurity/healthcare-software/uncategorized tagged `out-of-scope-cluster` in notes. 38 rows flagged. |
| D | Source-conflict: newest source_date wins, ties alphabetical. Conflicts logged to notes. |
| E | Researched-row primary_slug = empty string + status=researched. No placeholders. |
| F | Notes column: "SV={n} KD={n} CPC=${n} cluster={x}" single-line. Append " src={file}" for MASTER. Cap 100 chars. Max observed: 100 (asbestos) / 96 (SSG). |
| G | Strategy_Framework: skipped for import, archived to sources/. 0 rows contributed. |
| H | SSG cluster column added; asbestos registry stays 8-col. |
| I | SSG fuzzy match: 4 hits identified, applied at rebuild time (not staging). |
| J | Dead-keyword detection: 27 markers caught in staging → 19 became killed in registry (8 upgraded to live per precedence). |

### Pre-flight HALTs continued to pay this session

3 HALTs caught real decisions:
1. CC's recon of source files surfaced format heterogeneity (3 parsers needed, not 1)
2. Pre-flight surfaced 2 additional forks beyond the 8 planned (Fork I SSG slug mapping, Fork J dead-keyword detection)
3. Spot-check before merge caught the 32-cluster duplication issue (cosmetic but real — deferred to Session B normalization pass)

Pattern holds: pre-flight is the cheapest place to catch architecture decisions.

### Parallel-chat coordination clean

Sister chat had built the Keyword Registry foundation earlier in the day (separate session). This session built on top of their work without conflict. File-boundary split natural: sister's work on `~/agents/keyword-registry/` foundation + `~/brain/projects/aiteam/keyword_registry/` initial files; this session added importer script + staging dir + sources archive + 2 registry imports. Zero shared-file conflicts.

---

## FILES CREATED / MODIFIED

### In `~/agents/`
- `keyword-registry/lib/import_research.py` — new (commit `86029fd`)
- `keyword-registry/lib/rebuild_registry.py` — modified (added researched status, --site flag, cluster column logic)
- `keyword-registry/staging/asbestos_import_20260517_222359.md` — new (staging artifact)
- `keyword-registry/staging/ssg_import_20260517_222359.md` — new (staging artifact)

### In `~/brain/projects/aiteam/`
- `keyword_registry/keyword_registry.md` — updated (162 → 394 rows)
- `keyword_registry/keyword_registry_ssg.md` — new file (457 rows)
- `keyword_registry/sources/ASB_Keyword_Data_CONSOLIDATED_2026-05-17.md` — archived verbatim
- `keyword_registry/sources/ASB_Strategy_Framework_CONSOLIDATED_2026-05-17.md` — archived verbatim
- `keyword_registry/sources/ASBESTOS_Batch4_Keyword_Research_Report_CONSOLIDATED_2026-05-17.md` — archived verbatim
- `keyword_registry/sources/MASTER_keywords.md` — archived verbatim
- `keyword_registry/sources/SMART_Master_Research_Report_2026-05-17_consolidated.md` — archived verbatim
- `keyword_registry/sources/README.md` — new (provenance + import dates)

### In `~/store/aiteam.db`
- `audit_log` rows: `keyword_registry_imported` (correlation 2F67A587-…), `sources_archived` (correlation D2114073-…), staging-related rows from import_research.py runs

### In this Claude project
- `AITEAM_Context_Save_2026-05-17_KeywordRegistryImport-SessionA.md` — this file

### Not touched (lanes respected)
- Sister chat lanes (research-opportunity, backlink-prospector)
- POLICY.md, HANDOFF.md, PRIME.md
- DEFERRED.md (D-next items captured in this context save; sync to DEFERRED.md at next brain-touching CC session per project rule)

---

## DEFERRED ITEMS

### New this session — pending sync to DEFERRED.md

**D-next-A — Mandatory pre-flight search discipline**
- Build `~/agents/lib/brain_search.sh`: markdown-aware grep across `~/brain/projects/aiteam/`, `~/agents/*/CLAUDE.md`, `~/projects/*/` filenames, `~/Downloads`, `~/Desktop` filenames. Returns filenames + section excerpts + line numbers.
- Add pre-flight contract to `~/CLAUDE.md` + `~/agents/CLAUDE.md`: agents MUST `brain_search.sh <topic>` before creating files, answering factual project questions, recommending actions. Must report findings before proceeding.
- Audit-logged on every invocation for operator spot-checks.
- ~45 min CC, $0 LLM.
- Goal: stop duplicate files, stop re-derivation, force agents to check existing context.
- Trigger: highest-leverage infrastructure item. Recommend before any other agent build.

**D-next-B — Title/meta rewrite on top 5 zero-click pages**
- 5 pages with 250+ GSC impressions and 0 clicks: asbestos-inspection-cost (417 imp), asbestos-remediation-cost (330 imp), when-was-asbestos-used-in-homes (310 imp), transite-pipe-guide (270 imp), is-popcorn-ceiling-asbestos (259 imp)
- Sonnet rewrite of title tags + meta descriptions, optimized for CTR
- ~30 min CC, ~$0.05 LLM.
- Highest visible revenue lever in current GSC data.
- Trigger: when operator wants revenue signals (CTR doubling on these 5 pages = measurable click lift within days)

**D-next-C — GSC-aware keyword scoring**
- Fetch GSC query data via API (Search Console access already exists)
- Score registry rows by: (GSC impressions on related queries) × (SV from registry) ÷ (KD from registry)
- Cluster gap detection: parent page fielding 5+ query variants with 0 supporting content → flag cluster as "ready to compound"
- Feeds future keyword-picker agent
- ~4-6h CC, $0 LLM, $0 API (GSC free tier)
- Trigger: before keyword-picker agent is built (which is itself trigger-gated on operator wanting auto-content selection)

**D-next-D — Session B: Strategy extraction + research-opportunity hook**
- Extract strategic content from ASB_Batch4 (~1,400 lines) + SMART_Master (~1,000 lines) into structured agent-readable brain notes:
  - `~/brain/projects/aiteam/strategy/<site>/dead_angles.md` — seller-walls, legal-walls, YMYL clusters to skip
  - `~/brain/projects/aiteam/strategy/<site>/slug_intents.md` — per-slug content angles + affiliate notes
  - `~/brain/projects/aiteam/strategy/<site>/monetization_map.md` — cluster → affiliate program mapping
  - `~/brain/projects/aiteam/strategy/<site>/affiliate_programs.md` — confirmed/unverified split (SSG)
  - `~/brain/projects/aiteam/strategy/<site>/saturation_calls.md` — "stop researching X" notes
  - `~/brain/projects/aiteam/strategy/<site>/methodology_notes.md` — research-process learnings
  - `~/brain/projects/aiteam/strategy/_README.md` — agent integration map
- Wire `research-opportunity/extract.sh` to read `dead_angles.md` before Sonnet extraction (skip dead clusters, save ~$0.005 per skip + avoid surfacing un-buildable opportunities)
- ~60-90 min CC, ~$0.40-0.80 LLM (Sonnet extraction)
- Finishes the Layer 1 + 2 + 3 plan we started this session.
- Trigger: when D-next-A discipline lands (so Session B's CC instance has pre-flight discipline from the start)

**D-next-E — Cluster normalization pass on SSG registry**
- 32 distinct cluster names in keyword_registry_ssg.md, ~8 are duplicates from MASTER kebab-case vs SMART freeform formats:
  - answering-services (93) vs answering services (13) → collapse to answering-services
  - fleet-tracking (63) vs fleet tracking (10) → collapse to fleet-tracking
  - it-support (63) vs it support / msp (11) → collapse to it-support
  - +5 more similar
- ~30 min CC, $0
- Trigger: opportunistic next SSG hygiene session

**D-next-F — SSG slug→keyword mapping for 12 stranded live slugs**
- 12 of 17 SSG live page.tsx slugs have no exact-match keyword in registry (Fork I caught 4; remaining 12 unmapped)
- Hand-add slug→primary-kw mappings OR accept 0-coverage display
- ~15-30 min operator review + CC application
- Trigger: when SSG site rewrite ships (gated on D-SSG-rewrite)

**D-next-G — Agent write-through awareness of `researched` status**
- deploy_batch.sh + apply_approval.sh write-through paths don't know about researched status
- Currently: a ship that targets a researched kw doesn't transition researched → in_pipeline → live distinctly
- No regression, but a gap to wire properly
- ~30 min CC, $0
- Trigger: bundle with Session B agent hooks OR next ship-to-site hygiene pass

**D-next-H — Live-row notes don't inherit research metadata (SV/KD/CPC)**
- Imported researched rows have SV/KD/CPC in notes
- Pre-existing live rows have empty notes
- Could backfill SV/KD/CPC from sources/ archive if operator wants registry to double as research DB
- ~30 min CC, $0
- Trigger: optional — only if operator wants enriched live-row metadata for the keyword-picker

### Carried forward, no change
D025, D026, D028, D033, D039, D040, D041, D042, D043, D052, D053, D055, D057, D058, D059, D060, D061, D062, D063, D065, D066 (~/agents/ half closed 05-17), D069, D071, D072, D073, D074, D076, D077, D-SSG-01, D-SSG-02, D-SSG-03, D-SSG-04, D-SSG-06, D-SSG-07, D-SSG-08, D-SSG-09. Plus 3 deferreds from prior session pending close-out: D-next-A Haiku turn cap, D-next-B check_kill_switches -e leak, D046 sighting append (these from the GEO Phase 3 + Backlink Prospector V01 session — numbers may need renumbering if not yet synced to DEFERRED.md).

**⚠ D-number collision risk:** This save uses D-next-A through D-next-H. The prior session's save also used D-next-A and D-next-B (different content). When syncing to DEFERRED.md at next CC session, run `grep -oE "\bD[0-9]+\b" ~/brain/projects/aiteam/DEFERRED.md | sort -u -V | tail -10` to find next available number and reassign. Likely range: D078-D085 or thereabouts.

### Closed this session
- Session A keyword registry import (Sessions B, agent hooks, GSC integration all separately tracked)

---

## HANDOFF — NEXT CHAT

### Read in this order
1. **This context save** (you're reading it)
2. `~/brain/projects/aiteam/PRIME.md` — points at POLICY / HANDOFF / DEFERRED
3. `~/brain/projects/aiteam/POLICY.md` — 7 locked operator policies
4. `~/brain/projects/aiteam/HANDOFF.md` — current state
5. `~/brain/projects/aiteam/DEFERRED.md` — D076, D077 + new D-next items pending sync

### Current state at session close

| Surface | State |
|---|---|
| `~/agents/` | HEAD `86029fd`, working tree clean of this session's work. Sister-chat lanes (research-opportunity, backlink-prospector) untouched. Autocommit handles overnight push. |
| `~/projects/asbestos-contractors/` | HEAD unchanged from prior session (`624d334`). |
| `~/projects/asbestoshq-site/` | HEAD unchanged. |
| `~/brain/` | HEAD `00b24a2` for this session's import. Plus pre-existing dirty state (HANDOFF.md, LESSONS.md likely from sister-chat work, context saves). Brain-autocommit at 23:55 handles. |
| Sister chat | Research/Opportunity Agent Session 2 status unknown at session close — last context save indicated they were mid-flight. Cross-reference operator's most recent statement before acting. |
| Asbestos registry | 394 rows (155 live + 206 researched + 19 killed + 14 unowned) |
| SSG registry | 457 rows (4 live + 453 researched). 38 tagged out-of-scope-cluster. |
| Sources archive | 5 files verbatim at `~/brain/projects/aiteam/keyword_registry/sources/` with README. |
| Mission bar | $0 / $200 mo. Foundation harder. Revenue still gated on operator-side work (cluster-completer slugs into drafter_queue.txt + AdSense enrollment + title/meta optimization on top 5 zero-click pages). |
| GSC data | 5,170 imp / 13 clicks / avg pos 15.3 / 28 days — actively ranking, ready for cluster-completer cadence. |

### Opening moves for next chat

**1. Confirm overnight crons fired.**
- `cd ~/agents && git log origin/main..HEAD --oneline | wc -l` — expected ~0
- `cd ~/projects/asbestos-contractors && git log origin/main..HEAD --oneline | wc -l` — expected ~0
- `cd ~/brain && git log origin/main..HEAD --oneline | wc -l` — expected ~0
- Check 20:00 + 23:00 deploy throttle fired honestly (audit_log for `preview_ping` + `deploy_batch`)
- Check 03:00 keyword-registry rebuild fired (audit_log for `keyword_registry_rebuilt`)

**2. Sync D-next-A through D-next-H to DEFERRED.md.** Run D-number collision check first:
```
grep -oE "\bD[0-9]+\b" ~/brain/projects/aiteam/DEFERRED.md | sort -u -V | tail -10
```
Assign next available numbers (likely D078+) and write entries with triggers + cost estimates.

**3. Decide next track.** Recommendation order (highest leverage first):

| Option | Time | Cost | Why |
|---|---|---|---|
| **D-next-A — pre-flight discipline** | 45 min | $0 | Highest infrastructure leverage. Every future agent inherits. Stops duplicate-file pattern documented across multiple prior sessions. |
| **D-next-B — title/meta rewrite** | 30 min | $0.05 | Highest visible revenue lever in current GSC data. Likely 2-3x click lift on 5 pages within days. |
| **D-next-D — Session B strategy extraction** | 60-90 min | $0.40-0.80 | Finishes what registry import started. Compounds with every future agent that reads strategy notes. |
| **Operator-side: enqueue vermiculite cluster slugs** | operator | $0 | Cluster-completer prioritization per GSC data. 3-4 supporting pages for vermiculite (biggest gap between demand and coverage). |
| **Operator-side: AdSense enrollment** | operator | $0 | Mission-bar lever. Until ads run, $0 doesn't move regardless of agent builds. |

**Top pick if operator has CC time:** D-next-A then D-next-B. Lands ~75 min CC total, both ship same evening, both are high-leverage with low risk.

### Critical context for next Claude

- **Keyword registries are LIVE for both sites.** Don't re-derive import logic — `import_research.py` exists at `~/agents/keyword-registry/lib/import_research.py`. Future imports use the same script with `--target-registry` flag.
- **Status precedence matters:** live > in_pipeline > researched > killed. A keyword mentioned in a live page stays live even if research data says researched.
- **38 SSG rows tagged out-of-scope-cluster** (cybersecurity/healthcare-software/uncategorized). These are visible but flagged for future new-domain decision (per SMART report's Path 3 — parked).
- **Sources archive at `~/brain/projects/aiteam/keyword_registry/sources/`** contains 5 verbatim source files (~395 KB total). Search this directory when looking for original keyword research context. README.md describes provenance.
- **GSC data is ground truth for cadence decisions.** 5,170 impressions / 13 clicks / avg position 15.3 over 28 days = asbestos is ranking. Cluster-completer prioritization (vermiculite, siding, transite, black mastic, shingles) > random Batch4 pickup.
- **5 zero-click pages identified.** 250+ impressions each, 0 clicks. CTR problem. Title/meta rewrite is the fix, not more content.
- **Cadence revised to 4-5/week asbestos, 1-2/week SSG post-launch.** Initial 2-3/week recommendation was too conservative given GSC ranking data.
- **Agent pre-flight discipline is the highest-leverage infrastructure work undone.** Multiple prior sessions documented duplicate-file pattern. D-next-A fixes it permanently.

### Do NOT

- Re-derive any of the 10 Fork decisions (A through J) from this session — all pinned in DEFERRED.md and commit body of `86029fd`.
- Touch sister-chat lanes without confirming closure (research-opportunity, backlink-prospector).
- Run `import_research.py` again without `--target-registry` and explicit source paths — it has no idempotence guard for the same source file twice. (Tracked implicitly under D-next-G alongside write-through awareness.)
- Try to "fix" the 38 out-of-scope-cluster rows — they're intentionally tagged for the future new-domain decision.
- Skip the D-number collision check when syncing D-next items to DEFERRED.md.
- Trust training-data confidence on third-party API pricing (lesson from GEO/Backlink session: Google CSE deprecation, Brave free-tier removal both caught operator off-guard). Always web_search before recommending services.

---

## LESSONS LEARNED

### GSC data should be the cadence anchor, not theoretical SEO advice

Initial cadence recommendation (2-3/week asbestos) was based on "young site, no rankings yet" assumption. GSC screenshot showed asbestos is actually at 5,170 imp / pos 15.3 — already ranking, ready for compound content. Pattern: always ask for GSC data before making cadence recommendations. Don't assume new sites have no rankings.

### Cluster signals are more actionable than keyword counts

The Batch4 report has ~130 keywords across many clusters. GSC shows 5 specific clusters (vermiculite, siding, transite, black mastic, shingles) where existing pages are already pulling impressions. Building supporting content for those 5 compounds rankings within Google's existing consideration set. Building Batch4's PPE/HEPA cluster starts from zero. Cluster gap > keyword count for prioritization.

### Operator's "agents should be smarter" is often "agents should check what exists"

Spent ~10 min explaining retrieval/RAG/fine-tuning before operator clarified the actual problem: duplicate files + ignored prior context. Different problem, much simpler fix (pre-flight discipline contract). Pattern: when operator asks for agent intelligence improvements, first surface specific frustrations to distinguish discipline gaps from capability gaps.

### Spot-check before merge caught a cosmetic issue worth tracking

Spot-check found 32 SSG cluster names with ~8 duplicates from format mismatch. Not blocking but real. Deferred cleanly to Session B normalization pass. Pattern continues: 5-min spot-check on staged data routinely surfaces issues that would be painful to fix post-merge.

### Stale-info on third-party services remains expensive

Reinforced from prior session (Google CSE / Brave / Serper.dev mess). This session avoided that trap entirely by not recommending any new third-party services. Pattern: when adding capabilities, prefer extending existing infrastructure (`brain_search.sh` instead of Algolia, GSC API instead of Ahrefs) over adopting new services.

### Pre-flight HALTs continue to compound value

3 HALTs this session caught real architectural decisions (format heterogeneity → 3 parsers, 2 additional forks beyond planned 8, cluster duplication). Same pattern as prior 4+ sessions. Pre-flight is the cheapest place to catch issues — never relax it to "speed up" a build.

### `g` rule and "explain like I'm 14" requests are real signals

Operator requested "explain like I'm 14 under 200 words" twice this session. Each time, complex technical content compressed cleanly without losing decisional value. Pattern: when operator asks for compression, default to it for the rest of the session unless they explicitly expand. Honors stated communication preferences.

### Mission-bar drift check: still $0 / $200 mo

Build leverage is real and compounding (registries, sources archive, stacked quality gates, internal+backlink+keyword agents all live). But $200/mo doesn't move without revenue, and revenue doesn't move without (1) traffic and (2) monetization. Traffic is now generating (GSC confirms). Monetization is still un-enrolled (no AdSense yet). The next mission-bar-moving operator action is AdSense enrollment + cluster-completer slug queueing, not another agent build. Worth surfacing every session until it lands.

---

## SIGN-OFF NOTE

One clean closure (Session A keyword registry import) end-to-end in ~2 hours. Two clean commits, both idempotence checks drift=False, $0.00 LLM cost. Three pre-flight HALTs caught real decisions. Eight new deferred items captured for next session (D-next-A through D-next-H pending DEFERRED.md sync).

Major mid-session discovery: GSC data confirms asbestos site is ranking (5,170 imp / 13 clicks / avg pos 15.3 / 28 days). Cluster signals visible (vermiculite, siding, transite, black mastic, shingles). Revised cadence to 4-5/week asbestos. Identified 5 zero-click pages as highest-revenue lever via title/meta rewrite.

Operator's "agents should be smarter" turned out to be a discipline gap (duplicate files, ignored context) not a capability gap. Fix sketched as `brain_search.sh` + mandatory pre-flight contract — parked as D-next-A, highest-leverage infrastructure work undone.

Mission bar: $0 / $200 mo. Foundation stronger — registries hold ~850 keywords across two sites, sources archived, status precedence enforced, GSC ranking data informing cadence. Revenue still operator-gated on slug queueing (vermiculite cluster first) + AdSense enrollment + title/meta rewrite on top 5 zero-click pages.

Next session opens by verifying overnight crons, syncing D-next items to DEFERRED.md with collision check, then picking between pre-flight discipline (infrastructure), title/meta rewrite (revenue), or Session B strategy extraction (continuation).

---

*End of context save. Resume next session by reading this file first, then PRIME.md (which points at POLICY/HANDOFF/DEFERRED). Verify overnight crons fired before any new work.*

---

## APPENDIX A — FRONT-LOAD KEYWORDS NEXT SESSION (PRIMARY RECOMMENDATION)

**Purpose:** Skip the keyword-picker agent build entirely for now. Front-load 100-200 prioritized slugs per site into `drafter_queue.txt` in one ~90-minute Claude session. Autonomous pipeline chews through the queue for 4-5 months without operator keyword work.

**Why this beats the agent approach:**

| | Keyword-picker agent | Front-load approach |
|---|---|---|
| Time to first slug auto-queued | 15-20h CC build (4 prerequisites + agent itself) | 90 min this session |
| Quality of picks | Sonnet at runtime, no human review | Claude + operator review, one-shot |
| Ongoing cost | $0.05-0.20/pick × N picks | $0 ongoing |
| Operator review overhead | Per-pick or trust runtime | One-time deep review |
| Failure modes | 4 documented live concerns | None — it's a sorted list |
| Maintenance burden | Real (cron, halt flags, drift detection) | Zero |

The agent makes sense at 1,000+ slugs/site where re-ranking matters or cluster signals shift faster than humans can keep up. At ~600 researched keywords across both sites and 4-5/week shipping cadence, front-loading covers 4-5 months of pipeline.

### Session inputs (what to gather before opening the chat)

1. **Asbestos registry** — `~/brain/projects/aiteam/keyword_registry/keyword_registry.md` (394 rows). Filter to status=researched (206 rows).
2. **SSG registry** — `~/brain/projects/aiteam/keyword_registry/keyword_registry_ssg.md` (457 rows, 453 researched).
3. **GSC export for asbestos** — Last 28 days. CSV or paste of queries + impressions + clicks + avg position. Operator pulls from search.google.com.
4. **Strategy notes if D-next-D has landed** — `~/brain/projects/aiteam/strategy/asbestos/dead_angles.md` + `slug_intents.md` + `saturation_calls.md`. If Session B strategy extraction hasn't run yet, pull the raw Batch4 + SMART files from `~/brain/projects/aiteam/keyword_registry/sources/` instead.
5. **Existing drafter_queue.txt** — what's already in flight (don't double-queue).
6. **Live slug inventory** — `~/agents/internal-link/internal_link_inventory.db` cluster rollup (which clusters have existing content + how many supporting pages each).

### Recommended session structure (~90 min)

**Phase 1 — Asbestos prioritization (~45 min)**

Open a Claude.ai chat. Single prompt outline:

> Rank these ~206 researched asbestos keywords for ship order. Apply this composite:
> - Cluster gap signal: prioritize keywords where a parent page is already ranking (GSC impressions present) but supporting content count is <3. Vermiculite, asbestos siding, transite, black mastic, shingles clusters all qualify per current GSC data.
> - Demand signal: GSC impressions on related queries (28-day) × log(SV). Demand-proven beats research-estimated.
> - Difficulty: 1/(KD+1). All else equal, lower KD wins.
> - Portfolio balance: max 2 slugs per cluster in the first 20-slug batch. After batch 1, can concentrate.
> - Skip: dead clusters from Batch4 (mesothelioma legal, asbestos test kits, training/cert, ban-dates, mobile homes, wood stoves, brake/auto/marine, schools/commercial, real estate, health symptoms).
> 
> Output: 150 slugs in ship order, format `<slug>,<primary_kw>,<cluster>,<one-line reasoning>`. Group into batches of 20-25 with batch-level theme.

**Phase 2 — SSG prioritization (~30 min)**

Same composite, simpler because SSG has no GSC data yet (site rewrite pending):

> Rank these ~453 researched SSG keywords. Apply:
> - Cluster priority: answering-services, fleet-tracking, it-support first (existing SSG live pages = these clusters). Skip the 38 out-of-scope-cluster rows (cybersecurity, healthcare-software, uncategorized) — flagged for future new-domain decision.
> - Difficulty: 1/(KD+1).
> - Volume: log(SV) since no GSC data yet.
> - SMART report Tier rankings: any keyword tagged anchor / Tier 1 in source files = top half.
> - Portfolio balance: max 3 per cluster in first 20-slug batch.
> 
> Output: 100 slugs in ship order, same format as asbestos. Wait — SSG can't ship until site rewrite lands. Queue can be authored but flag clearly "DO NOT WRITE TO drafter_queue.txt until SSG site rewrite ships (D-SSG site rewrite)."

**Phase 3 — Operator review + queue write (~15 min)**

Operator scans the 150 asbestos picks. Flags anything obviously wrong (cluster doesn't make sense, dead-end slug, redundant with existing live). Removes ~10-20 if needed.

Then CC writes the survivors to `~/projects/asbestos-contractors/content/asbestos/drafter_queue.txt` in batches:
- Batch 1 (20 slugs) → drafter_queue.txt
- Batches 2-7 (130 slugs) → `~/agents/keyword-picker/queue_backlog.txt` for operator to graduate as batches 1-6 ship

SSG output goes to a holding file pending site rewrite.

### Pre-flight checks before queue write

For each slug about to land in drafter_queue.txt:
1. `brain_search.sh <slug>` (or grep equivalent if D-next-A hasn't shipped) — verify no existing file
2. Cross-check approved/guides/ — already shipped?
3. Cross-check current drafter_queue.txt — already queued?
4. Cross-check keyword-configs/ — config already exists (means slug was tried before)?

Any positive hit → skip, move to next candidate.

### After this session

| Cadence | Result |
|---|---|
| Asbestos at 4-5/week | 150 slugs = 30-37 weeks of pipeline |
| SSG at 2/week post-launch | 100 slugs = 50 weeks of pipeline |

Operator's next keyword-research session: probably December 2026 or later, when first batch runs out OR GSC data shows new clusters worth pivoting to.

### What about keyword-config generation?

The existing pipeline expects a `keyword-configs/<slug>.json` for each slug in drafter_queue.txt. Two options:

**Option A — Pre-generate all 150 configs in this session.** ~$5-15 LLM cost (Sonnet). Operator + Claude assembly-line through them. Time: ~90 additional minutes. Pipeline runs unattended afterward.

**Option B — Batch as needed.** Generate configs for batch 1 (20 slugs) this session. Subsequent batches need ~30 min per batch to generate configs. Lower upfront cost, more operator touchpoints.

Recommend A if operator has the time tonight; B if not.

### Mission-bar math

Front-loading 150 asbestos slugs at 4-5/week = ~7-9 months of autonomous content production. By end of that run, asbestos site has 32 + 150 = 182 articles. At even 1% AdSense conversion of that traffic, that's the $200/mo bar.

This is the highest-leverage thing operator could do tonight. Higher than D-next-A (infrastructure), higher than D-next-B (CTR optimization on 5 pages), higher than D-next-D (Session B strategy extraction).

### Honest pushback

Front-loading has one real failure mode: **decisions made tonight age poorly.** GSC data in 3 months will look different. New clusters may emerge. Saturation may hit clusters we don't expect.

Mitigation:
- Don't ship all 150 in one batch — keep batches of 20-25 with monthly review
- Monthly check: pull fresh GSC data, see if any clusters are surprisingly weak or strong, re-rank the remaining backlog
- The backlog is a list, not a contract. Reorder freely.

This makes the front-load approach equivalent to "monthly keyword-picker review session" — which is genuinely how content sites are run by operators at scale, not by agents.

### Next-Claude action

When operator says "let's do the keyword front-load session":
1. Verify D-next-A pre-flight discipline has landed (or skip if operator wants to push through)
2. Pull all inputs listed above
3. Run Phase 1 → 2 → 3
4. Generate Option A configs if time allows
5. Operator commits to drafter_queue.txt + queue_backlog.txt
6. Audit row + commit

Session estimate: 90 min Phase 1-3 + 90 min Option A config generation = 3 hours total if all done in one sitting. Or split into two sessions if context budget tight.

---

## APPENDIX B — KEYWORD-PICKER AGENT DESIGN BRIEF (FUTURE REFERENCE, NOT CURRENT BUILD)

⚠ **Build trigger:** ONLY when registry exceeds 1,000+ rows per site OR cluster signals shift faster than monthly human re-ranking can keep up. Today, neither is true. Front-loading (Appendix A) covers 4-9 months of pipeline. Revisit this design only when front-load approach hits its limits.

**Purpose:** Close the autonomous content loop. Today, operator manually picks the next keyword and adds slug to `drafter_queue.txt`. Once registries are full (850+ keywords across asbestos + SSG), the operator is the bottleneck. Keyword-picker agent reads registries + GSC + portfolio rules and queues the next slug autonomously.

**Mission-bar rationale:** Until this exists, $0 → $200/mo growth is operator-gated on weekly keyword research sessions. With it, the loop runs without operator touch except for portfolio-level kill decisions.

### When to build

NOT before:
- D-next-A pre-flight discipline lands (so this agent inherits brain_search.sh)
- D-next-D Session B strategy extraction lands (so this agent reads dead_angles.md before picking)
- D-next-C GSC-aware scoring lands (so picks are weighted by real demand signals)
- D-next-G write-through awareness for `researched` status (so the agent can transition researched → in_pipeline cleanly)

Approximate order: D-next-A → D-next-D → D-next-C → D-next-G → keyword-picker. Total ~15-20 hours CC across 4-5 sessions before keyword-picker should ship.

Build keyword-picker prematurely = it picks keywords from dead clusters because it can't read dead_angles.md, queues content that contradicts saturation calls, generates duplicate work the operator has to manually undo. Sequence matters.

### Build estimate

~6-10h CC across 2 sessions. $0 LLM at build time. Ongoing cost: ~$0.05-0.20 per pick (Sonnet ranking + reasoning).

### Core design — what it actually does

**Trigger:** cron `0 6 * * *` (6 AM PT daily, before deploy throttle's 20:00 preview).

**Input sources (read-only):**
1. `~/brain/projects/aiteam/keyword_registry/keyword_registry.md` — asbestos registry (394 rows)
2. `~/brain/projects/aiteam/keyword_registry/keyword_registry_ssg.md` — SSG registry (457 rows)
3. `~/brain/projects/aiteam/strategy/<site>/dead_angles.md` — skip keywords in dead clusters
4. `~/brain/projects/aiteam/strategy/<site>/saturation_calls.md` — skip clusters operator marked saturated
5. `~/brain/projects/aiteam/strategy/<site>/slug_intents.md` — read existing per-slug content angles for context
6. GSC API — per-query impressions for last 28 days (D-next-C dependency)
7. `~/agents/internal-link/internal_link_inventory.db` — what's already live, what's clustering
8. `~/projects/<site>/content/<site>/drafter_queue.txt` — what's already queued (don't double-queue)
9. `~/projects/<site>/content/<site>/approved/guides/*.json` — what's already shipped
10. `~/agents/keyword-picker/config.yaml` — per-site weekly caps + cluster prioritization rules

**Output (write):**
1. Append slug to `~/projects/<site>/content/<site>/drafter_queue.txt`
2. UPDATE registry row: status `researched` → `in_pipeline`
3. Create `~/projects/<site>/content/<site>/keyword-configs/<slug>.json` (primary + secondary + intent + entities, populated from registry row + Sonnet enrichment)
4. Audit row: `keyword_picker_queued` with payload (site, slug, primary_kw, score, reasoning, registry_row_id)
5. Telegram notification: "Queued <slug> for <site>. Score: X. Reasoning: <one line>"

### Scoring algorithm — the load-bearing decision

This is the part that has to be right. Naive ranking on SV/KD alone produces "asbestos siding" being picked first 50 times in a row.

**Composite score formula (v1):**

```
score = cluster_gap_signal × demand_signal × difficulty_adjustment × portfolio_balance × freshness_bonus

where:
  cluster_gap_signal:    1.0-3.0  (existing parent page + supporting content needed = 3.0, isolated keyword = 1.0)
  demand_signal:         (GSC impressions on related queries, 28d) × log(SV from registry)
  difficulty_adjustment: 1.0 / (KD + 1)
  portfolio_balance:     1.0 / (slugs_published_this_week_in_cluster + 1)  — penalizes over-concentration
  freshness_bonus:       1.5 if cluster has 0 supporting content built in last 30 days, else 1.0
```

**Hard filters applied BEFORE scoring:**
- status != `researched` → exclude (already live, in_pipeline, or killed)
- keyword in `dead_angles.md` for that site → exclude
- cluster in `saturation_calls.md` for that site → exclude
- slug-form already in `drafter_queue.txt` → exclude
- weekly cap reached for that site → exclude entire site this run

**Weekly cap config:**
```yaml
asbestos:
  weekly_cap: 5           # revised from real GSC data, was 3
  min_cluster_diversity: 2  # don't pick from same cluster more than 2x/week
ssg:
  weekly_cap: 2           # post-launch only; pre-launch = 0
  min_cluster_diversity: 1
```

**Cap counting:** count audit_log rows where action='keyword_picker_queued' AND site=<site> in last 7 days. If >= weekly_cap, skip the site this run.

### keyword-config.json generation

The agent needs to write the same JSON shape that existing keyword-configs use:

```json
{
  "_comment": "primary 'X' SV={n} KD={n} CPC=${n}. Generated by keyword-picker <date>.",
  "primary": "<from registry>",
  "primary_target": [4, 10],
  "primary_in_h2s_target": [1, 2],
  "secondary": {
    "<related kw 1>": {"range": [3, 7]},
    "<related kw 2>": {"range": [2, 5]}
  },
  "ngram_ignore": [...],
  "intent_concepts": [...],
  "required_entities": [...]
}
```

Three of these fields (secondary, intent_concepts, required_entities) need Sonnet generation. The agent should:
1. Look up the registry row for the chosen keyword (primary + cluster)
2. Find 5-10 related researched keywords in the same cluster from registry
3. Read 2-3 existing keyword-configs in the same cluster as examples
4. Sonnet prompt: "Generate the secondary/intent_concepts/required_entities fields for primary keyword X, given these cluster siblings and examples"
5. Cost: ~$0.05-0.15 per config

### Operator override mechanism

The agent must respect manual operator picks. Two ways operator overrides:
1. Manually adding a slug to `drafter_queue.txt` — agent skips that slug, lets it process normally
2. `~/agents/keyword-picker/halt.flag` — agent pauses entire automatic picking, only operator-added slugs flow through (kill switch)
3. Per-site flag: `KEYWORD_PICKER_PAUSED_<SITE>=true` in `.env` — pauses just that site

Halt-flag pattern matches existing agents (research-opportunity, backlink-prospector).

### Pre-flight checks before queuing

After scoring picks the top keyword, agent MUST verify before queuing:
1. brain_search.sh `<primary_kw>` — verify no existing slug already covers it (catch missed coverage in registry)
2. internal_link_inventory cross-check — is there already a live slug with similar h1?
3. drafter_queue.txt grep — already queued?
4. approved/guides/<derived-slug>.json existence check
5. If ANY check returns "already exists" → mark registry row as `live` (not `researched`), pick next candidate, log audit row `keyword_picker_skipped_already_exists`

This is the discipline-fix from D-next-A applied to agent-internal logic.

### Telegram approval gate (v1 vs v2)

**v1 (recommended):** No approval gate. Agent picks, queues, fires Telegram notification. Operator can yank from drafter_queue.txt if disagrees. Fast iteration, low friction.

**v2 (later):** Per-pick Telegram approval. Agent picks, sends to operator with reasoning + alternative #2 and #3, waits for /approve or /reject. Higher quality, slower throughput. Adopt only if v1 produces enough wrong picks to justify friction.

Default v1. Decision rests on first 10-20 picks' quality.

### Sister-chat coordination

This agent reads but doesn't write to:
- internal-link's database (read internal_link_inventory)
- research-opportunity's database (no direct interaction; both feed off registry)
- backlink-prospector (no interaction)

Writes only to:
- drafter_queue.txt (single-file boundary — no other agent writes here)
- keyword-configs/ (per-slug; collision risk only if same slug picked twice, prevented by pre-flight check)
- registry status update (idempotent UPDATE)

File-boundary clean. Build in `~/agents/keyword-picker/` greenfield.

### Failure modes to plan for

1. **GSC API down or rate-limited** → score with `demand_signal = log(SV) × 1` (registry-only). Audit row notes `gsc_unavailable`.
2. **No researched keywords pass hard filters** → audit row `keyword_picker_no_candidates`. Telegram notification: "No queue-able keywords for <site> today. Possible causes: weekly cap hit, all clusters saturated, registry empty."
3. **Sonnet config-generation fails** → leave registry row at `researched`, don't write keyword-config, audit row `config_generation_failed`. Don't queue a slug without a config (current pipeline assumption).
4. **Selected slug collides with operator manual queue** → skip, pick next. Audit row.

### Mission-bar dashboard impact

Once keyword-picker is live, the dashboard should surface:
- Slugs queued by keyword-picker (last 7 days) per site
- Weekly cap utilization (3 of 5 used this week, etc.)
- Cluster diversity score (am I over-concentrating?)
- Pick reasoning history (operator can review whether picks are sensible)

This is a separate dashboard build, not part of keyword-picker proper. Track as future deferred when keyword-picker ships.

### Build phase order (when this agent eventually ships)

Phase 1 — Schema + config (~1h)
- `~/agents/keyword-picker/agent.yaml`
- `~/agents/keyword-picker/CLAUDE.md`
- `~/agents/keyword-picker/config.yaml` (weekly caps + cluster rules)
- Migration: `keyword_picker_picks` table for audit trail beyond audit_log

Phase 2 — Scoring engine (~2-3h)
- `~/agents/keyword-picker/lib/score_candidates.py`
- Pure Python, no LLM. Reads all 10 input sources, applies filters, computes composite score, returns top 5.
- Unit-testable with mocked registries.

Phase 3 — Config generator (~1-2h)
- `~/agents/keyword-picker/lib/generate_config.py`
- Sonnet call per pick. Outputs valid keyword-config JSON.
- Schema-validates against existing keyword-configs format.

Phase 4 — Orchestrator + writes (~1-2h)
- `~/agents/keyword-picker/run.sh`
- Calls score → generate_config → write outputs → fire Telegram → registry status update + audit row
- Honors halt.flag + per-site pause env vars

Phase 5 — Cron + dry-run mode (~30 min)
- Cron `0 6 * * *`
- `--dry-run` flag shows top 3 picks without queuing (for first-week observation)

Phase 6 — Smoke test (~30 min)
- Dry-run for 3 consecutive days. Operator reviews picks. Tune scoring weights if any pick is obviously wrong.
- Production-fire on day 4 if dry-runs look sensible.

### Bottom line for next-Claude

This agent is the keystone for autonomous revenue. Without it, every $200/mo of revenue requires operator weekly keyword sessions. With it, the loop runs itself. But it can't ship until 4 prerequisites land. Build the prerequisites in order. Don't shortcut.

When you do build it, the scoring algorithm is where 80% of the quality lives. Don't trust SV/KD alone. GSC demand × cluster gap × difficulty × portfolio balance is the right composite.

---

*End of context save. Resume next session by reading this file first, then PRIME.md (which points at POLICY/HANDOFF/DEFERRED). Verify overnight crons fired before any new work.*
