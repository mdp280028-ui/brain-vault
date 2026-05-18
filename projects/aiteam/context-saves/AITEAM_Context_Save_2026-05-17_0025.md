# AITEAM Context Save — 2026-05-17_0025
**Generated:** 2026-05-17T00:25:40-0700
**Since last save:** 2026-05-17 00:25:21
**Session topic:** Pipeline discovery + SSG content-pipeline scaffold (sibling track to verdict-persistence/deploy-throttle work)

---

## Mechanical record

### Git activity since last save
```
(no commits since last save)
```

### Files changed
```
(no file changes in ~/agents/)
```

### Diary entries since last save
(none since last save)

### Agent activity (audit_log)
(no audit rows since last save)

### Token usage
(no token usage since last save)

### Open mission tasks
(no open mission tasks)

---

## Decisions made this session

- **Use `asbestos-contractors` (not `UST-Contractors-`) as the SSG pipeline template.** Cloned, reset history, and committed as `~/projects/ssg-content/`. Asbestos is the more recently active guide-production pipeline (run-batch v10.13, audit_guide v3.4, CONTENT_SPEC_GUIDE_ASBESTOS v2.7); UST is ahead on AUDIT_SPEC (v1.6.5 with SPC-16) and service spec (v1.7), but those are service-page features not currently needed for SSG. Alternative considered + rejected: merge UST + asbestos best-of-both upfront — deferred as premature optimization.
- **Keep `~/projects/ssg-content/` local-only.** No GitHub remote. The decision to make it the SSG pipeline repo is deferred until the `smartsourceguide` deploy pipeline is understood. ssg-content is a working scaffold, not a committed project.
- **Move asbestos content artifacts to `~/projects/_asbestos-reference/` rather than delete.** Two-pass cleanup: 97 files in first pass + 4 stragglers (named after materials, not the word "asbestos") in second pass. Total: 101 files preserved as reference. Rationale: the keyword research report, approved guide JSONs, and per-slug keyword configs may be useful patterns when the SSG content begins, even though the content itself is not reusable.
- **Defer all asbestos→SSG content rewrites.** The 367-line `CONTENT_SPEC_GUIDE_SSG.md` is still asbestos-flavored (AsbestosHQ.com persona, popcorn-ceiling/transite-pipe/black-mastic examples). The two stub specs (`CONTENT_SPEC_SSG.md`, `CONTENT_SPEC_SERVICE_SSG.md`) still say "STUB — asbestos pipeline placeholder." The 8 `claude -p` invocations in run-batch.sh all hardcode "AsbestosHQ.com" persona strings. Rewriting these is meaningful work, not mechanical, so it's deferred to a session where the SSG voice + target domain are decided.
- **Defer `_template/` agent scaffold work.** Confirmed `~/agents/_template/` is empty (just `.gitkeep`). The "scaffold for new agents" doesn't exist on disk; new agents are currently hand-cloned from `editor/` or `librarian/`. Not blocking this session's goal.

## Lessons learned

- **macOS Desktop is TCC-blocked for terminal apps by default.** `~/Desktop/AITEAM FILES/`, `~/Desktop/CC UST FILES/`, `~/Desktop/CC ASBESTOS FILES/`, `~/Desktop/payroll-detective/` are all referenced in CLAUDE.md files but cannot be read from the shell — every `ls`/`find`/`mdfind` against them returns `Operation not permitted` (or empty). Solutions: (a) System Settings → Privacy & Security → Full Disk Access → enable for Terminal/Claude Code, or (b) move folders out of Desktop via Finder (Finder isn't subject to the same TCC restriction). Any project that documents paths under `~/Desktop/<project>/` as authoritative is inaccessible to CC by default. Future-CC: if a CLAUDE.md references a Desktop path you can't read, that's TCC, not a missing file.
- **Mandy's payroll content pipeline is local-only.** `git@github.com:mdp280028-ui/payroll-detective.git` returns `Repository not found` despite being named as "Source of truth" in `payrolldetective/CLAUDE.md`. The CLAUDE.md is honest about this — it says "the content production system lives in a separate repo (`payroll-detective`, with hyphen)" but doesn't promise it's pushed. It exists only at `~/Desktop/payroll-detective/` (TCC-blocked). To get the pipeline files (run-batch, audit_guide, CONTENT_SPEC_PAYROLL.md), the operator must either grant FDA, push the local repo to GitHub, or move via Finder.
- **UST and asbestos pipelines have forked, not parent-child.** Each repo has features the other lacks: UST has SPC-16 (AUDIT_SPEC v1.6.5) + a real service spec (v1.7) + spc-audit.py; asbestos has run-batch v10.13 + audit_guide v3.4 (G1 SEMANTIC-VARIETY) + guide-spec v2.7. Neither is downstream of the other. A future "merge forward" would need to be cherry-picked, not a simple rebase.
- **`cp -R` + `rm -rf .git` + `git init` is a clean way to fork a repo's structure.** Used for the asbestos→ssg-content port. Initial commit captures the entire state as a single baseline, with no upstream history pollution. Pairs well with `_asbestos-reference/` for keeping the original content artifacts queryable without bloating the new repo.
- **The volume of "X references" in a forked content pipeline comes from content artifacts, not pipeline mechanics.** ssg-content had 2,530 occurrences of "asbestos" — but 2,400+ of those were inside approved guide JSONs, the keyword research report, assignment batches, and per-slug keyword configs. Only ~80 references were in pipeline files (run-batch.sh, audit_guide.py, AUDIT_SPEC.md, CONTENT_SPEC_GUIDE_SSG.md, ctx.sh, auto-harvest-to-library.sh). Distinguishing the two saves time: content artifacts are delete-not-rename; pipeline files need targeted edits. Always count occurrences-by-file before deciding the edit strategy.
- **`smartsourceguide` is a bare site shell — 0 docs/specs/keyword research.** Last commit 2026-04-10 (5+ weeks before this session). The repo has 6 app routes across 3 categories (answering-services, fleet-tracking, it-support) but no CLAUDE.md, no README, no content pipeline references. Unlike asbestos which inherited a 374-mention keyword research report and 27 reference articles from UST, SSG content production starts from **zero domain reference material**.
- **`mdfind` (Spotlight) is a useful TCC-bypass for confirming a file's nonexistence system-wide.** Spotlight indexes everything including Desktop, so `mdfind -name "<filename>"` returning empty means the file truly doesn't exist anywhere indexable. Used to confirm `run-batch.sh` and `audit_guide.py` were not on this Mac before cloning from GitHub. Caveat: only confirms nonexistence, not existence — TCC-blocked content still indexes, so a Spotlight hit doesn't mean the shell can read it.
- **`gh` (GitHub CLI) is not installed on this Mac.** When operator asked for `gh repo list mdp280028-ui --limit 100`, command-not-found. Workaround: open browser to GitHub repositories page, or use `git ls-remote git@github.com:owner/<guess>.git` (exit code 0 = exists). For this session, the operator handed me the four exact repo names — but for future sessions, install via `brew install gh` would unblock `gh repo list`/`gh repo clone`/`gh pr` workflows.

## Operator corrections

None this session. Operator gave precise step-by-step prompts with explicit commands to run; CC executed and reported. Two minor course-corrections were operator-initiated, not CC-corrected:
- After initial find on `~/Desktop/` returned `Operation not permitted`, operator pivoted to `~/agents`, `~/brain`, `~/projects` instead of demanding TCC be granted. CC did not propose a wrong path that needed correcting.
- When CC reported all six UST canonical pipeline files missing locally, operator pivoted to cloning from GitHub rather than retry the Desktop search. Again — CC reported truthfully, operator routed.

## What's next

**Immediate next session priorities (SSG track):**
1. Decide the SSG target domain + persona for the 8 hardcoded `claude -p` prompts in `~/projects/ssg-content/content/run-batch.sh` (currently all say "AsbestosHQ.com"). Without this, the pipeline cannot run for SSG.
2. Rewrite `~/projects/ssg-content/content/ssg/CONTENT_SPEC_GUIDE_SSG.md` from asbestos voice/examples to SSG voice. 367 lines, v2.7 — substantial rewrite, not mechanical.
3. Fill in the two stub specs: `CONTENT_SPEC_SSG.md` + `CONTENT_SPEC_SERVICE_SSG.md`.
4. Rewrite `~/projects/ssg-content/CLAUDE.md` root file from "Asbestos Project Context" to SSG context.
5. Decide whether to rewire the 8 `claude -p` calls to AITEAM's `ai-do.sh`/`ai-think.sh` tier wrappers (budget governance).

**Open questions deferred to operator:**
- Does `~/projects/ssg-content/` become a GitHub repo? Named what? (current default candidate: `smartsourceguide-content`)
- Where does SSG content publish to? Is `smartsourceguide` the deploy target, or is a separate site repo planned (mirroring the `asbestoshq-site` / `UST-Contractors-` split)?
- Should the 6 `ASBESTOS_*.md` strategy docs under `cc-context/05-REFERENCE/ported-from-ust-adapted/` (Strategic_Prompts, Fake_Rotation_Strategy, Data_Processing_Playbook, Directory_Business_Context, State_Database_Research_Skill, Guide_Creation_Workflow) be re-adapted for SSG, or are they asbestos-specific noise?
- Mandy's payroll pipeline: grant FDA to read `~/Desktop/payroll-detective/`, push from there to a new GitHub repo, or skip it from the comparison set?

**Blocked:**
- Cannot inventory `~/Desktop/*` content from CC without Full Disk Access or a Finder-side move.

**Not blocking SSG track but worth tracking:**
- Yesterday's verdict-persistence + deploy-throttle work (other session, documented in `HANDOFF.md` as of 2026-05-16 23:15) is the higher-priority operational track. SSG scaffolding is sibling work that doesn't compete with it for production cron slots.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-17_0023.md
**Active HANDOFF.md state:** will be overwritten by this session's narrative pass. Previous summary covered verdict-persistence + deploy-throttle; new summary adds the SSG scaffolding track.
