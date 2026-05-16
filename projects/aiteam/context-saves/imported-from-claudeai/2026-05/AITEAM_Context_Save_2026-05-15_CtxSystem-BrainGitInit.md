# AITEAM Context Save — 2026-05-15 Ctx System + Brain Git-Init
**Date:** May 15, 2026
**Session topic:** Built the AITEAM context-save system on CC + git-initialized the brain vault with private GitHub backup

---

## TL;DR

Two infrastructure builds completed end-to-end in this session:

1. **AITEAM ctx system** — `/ctx` slash command on CC now produces a structured context save (with mechanical record + narrative pass), updates HANDOFF.md, appends to LESSONS.md, and auto-harvests Claude.ai context saves from `~/Downloads/`.
2. **Brain vault git-init** — `~/brain/` is now a git repo with private GitHub remote (`git@github.com:mdp280028-ui/brain-vault.git`), daily auto-commit+push cron at 23:55, SSH key auth.

Both systems tested end-to-end with the test save firing successfully at session end.

---

## How this session started

Operator asked whether CC could do context saves. I incorrectly assumed yes, then searched and found: HANDOFF.md exists on Mac Mini as a manual pattern, but no `/ctx` slash command was built. Operator remembered we built one for UST and assumed it ported over. Conversation search confirmed: UST has `scripts/ctx.sh` but the auto-harvest chain was never wired correctly. We surveyed it on May 14 and identified the fixes but never built the AITEAM version.

This session built the AITEAM version with the UST bugs fixed.

---

## Decisions made

### Decision 1 — Build the ctx system on CC, mirroring Claude.ai context saves

**What was decided:** Build `~/agents/scripts/ctx.sh` + `~/agents/.claude/commands/ctx.md` + auto-harvest + LESSONS.md aggregation. Operator triggers with `/ctx` (or plain `ctx` / `save context`).

**Why:** Operator wanted end-of-session continuity for CC sessions equivalent to what Claude.ai context saves provide. The two surfaces produce different work (Claude.ai = architecture/strategy, CC = code/tests) and need separate continuity mechanisms.

**Architecture choices made on operator's "what do you recommend":**
- **Disk location:** `~/brain/projects/aiteam/context-saves/` — inside Obsidian vault, durable asset.
- **What's captured:** Hybrid — mechanical sections (git/diary/audit_log/token_usage/mission_tasks) + CC-written narrative on top.
- **Lessons learned:** Both per-save section AND auto-append to growing `~/brain/projects/aiteam/LESSONS.md` so system gets smarter over time.

### Decision 2 — Two-stage save pattern (skeleton + narrative)

**What was decided:** ctx.sh writes the deterministic mechanical skeleton, then prints a handoff block telling CC to fill the four narrative sections (Decisions / Lessons Learned / What's Next / Operator Corrections).

**Why:** Pure mechanical misses the "why." Pure narrative loses fidelity. Hybrid gives both — deterministic record + CC's judgment.

### Decision 3 — Git-init the brain vault (not just track templates separately)

**What was decided:** Run `git init` on `~/brain/`, push to a private GitHub repo (`mdp280028-ui/brain-vault`), daily auto-commit at 23:55.

**Why:** Original deviation from the ctx system build flagged that `handoff-template.md` wasn't git-tracked because brain wasn't a repo. Three options considered: (a) git init brain, (b) move template to ~/agents/, (c) leave it. Picked (a) — asymmetric upside: backs up everything by accident, full audit trail of every change to diary/LESSONS/context saves.

### Decision 4 — SSH over HTTPS for GitHub auth

**What was decided:** Generate SSH key, no passphrase, register with GitHub.

**Why:** One-time setup, then forget about it. HTTPS would require personal access tokens that expire. Passphrase skipped because operator-controlled machine where physical access is the security boundary. Operator can add a passphrase later with `ssh-keygen -p` if needed.

### Decision 5 — Exclude inbox/ and *.log from git

**What was decided:** Add `*.log`, `**/*.log`, and `projects/aiteam/inbox/` to .gitignore before initial commit. Unstage `dashboard.log` and inbox contents.

**Why:** Inbox is staging by design — items flow through, audit_log captures the events (which is the actual signal worth preserving). Logs are regenerable and create noisy diffs.

### Decision 6 — Let the 23:55 cron handle the first /ctx commit (don't manually push)

**What was decided:** After the test `/ctx` run, do not run `brain-autocommit.sh` manually. Let tonight's cron pick it up.

**Why:** If operator manually pushes after every save, the cron is decorative. The whole design is "do work, /ctx, walk away."

---

## Files created/modified on Mac Mini

### ctx system build
| File | Status |
|---|---|
| `~/agents/scripts/ctx.sh` | created, committed cbbd5ba |
| `~/agents/scripts/auto-harvest-to-library.sh` | created, committed cbbd5ba |
| `~/agents/.claude/commands/ctx.md` | created, committed cbbd5ba |
| `~/brain/projects/aiteam/docs/handoff-template.md` | created (now tracked via brain git-init) |
| `~/agents/CLAUDE.md` | updated, committed cbbd5ba |

### Brain git-init
| File / state | Status |
|---|---|
| `~/brain/.git/` | initialized |
| `~/brain/.gitignore` | created (excludes operator/, raw_dropzone/, .obsidian UI state, .DS_Store, *.log, inbox/) |
| `~/.ssh/id_ed25519` + .pub | generated, no passphrase, fingerprint SHA256:O7v8JyVIyjlyHAsd2yhoXQ/Tndd0E4t2MNHFK1wOZCo |
| GitHub repo `mdp280028-ui/brain-vault` | created, private, empty, SSH remote wired |
| `~/agents/scripts/brain-autocommit.sh` | created, committed |
| Crontab entry | `55 23 * * * /Users/mmm2/agents/scripts/brain-autocommit.sh` |
| `~/agents/CLAUDE.md` brain-tracked note | committed 3c73840 |
| Test ctx save | `~/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_1316.md` |
| LESSONS.md | grew from 2.3 KB to 4.1 KB across 8 new lessons |

### Brain commits this session
```
3c73840 docs: brain vault now git-tracked  (in ~/agents)
d8730cf test cleanup
736ce67 brain: auto-commit 2026-05-15
f128663 Initial commit: brain vault — projects, diary, docs
c4325a1 Initial commit: gitignore
```

---

## Lessons learned

1. **The UST ctx system had three bugs we intentionally did not port.** (a) auto-harvest chain never wired; (b) CLAUDE.md line 43 actively blocked the narrative pass; (c) stale SESSION_SAVE_TEMPLATE.md. Fixing-on-port saved a full second cycle of debugging.

2. **Two-stage save (deterministic skeleton + CC narrative pass) is the right pattern.** Pure mechanical misses the "why." Pure narrative loses fidelity. This pattern is reusable — any future "system that captures what happened" should follow it.

3. **`ssh-keyscan -t ed25519 github.com >> ~/.ssh/known_hosts`** before first `ssh -T` is required for non-interactive runs. Not in CC's original spec, but necessary. Worth keeping in the playbook for any future SSH-setup work.

4. **Git identity split is fine, but document it once and forget.** `~/agents/` commits as "AITEAM Builder <builder@aiteam.local>" (global identity). `~/brain/` commits as "mdp280028-ui" (local override). Future-operator will wonder why `git log` looks different in the two repos — answer: different repos, different identities, intentional.

5. **SSH key without passphrase is the right call for operator-controlled machines.** Convenience > marginal security gain. Cron jobs that push would break with a passphrase unless ssh-agent is always loaded, which adds complexity. Physical-access is the real security boundary.

6. **Don't push manually after `/ctx`.** Let the 23:55 cron do its job. Otherwise the cron is decorative. Trust the system.

7. **Inbox + log files should never be in git.** Staging areas and regenerable artifacts create noise. The signal lives in audit_log, not the staging directory.

8. **CC's deviations from spec were all defensible and correct.** Specifically: SQL column name corrections (timestamp → ts, cost_usd → estimated_cost_usd), `set -uo pipefail` instead of `-e`, `case` instead of regex for bash 3.2 portability. The pattern: CC reads the spec, hits reality, makes the right adjustment, flags it transparently. This is the operator's preferred mode and should be encouraged.

9. **LESSONS.md size can drop between saves and that's OK.** CC reads-then-rewrites rather than pure-appending, which normalizes whitespace and trailing blank lines. If LESSONS.md actually loses content between sessions (visible in `git log -p`), that's the real bug. Monitor periodically.

10. **First context save is always retrospective and bloated.** Because there's no "since last save" baseline, the first run captures everything. Second save onward is properly incremental. Don't read too much into the first save's size.

---

## Operator corrections

1. **"I thought we gave CC the ability to do a context save"** — operator was right to push back on my initial wrong answer that nothing existed. Conversation search revealed UST had it. I should have searched before answering definitively.

2. **"ok well when I'm done with cc I want to be able to type ctx and it does the handoff.md AND a context save"** — operator clarified the desired behavior crisply. The "AND" matters: not either/or, both files updated atomically.

3. **"what do you recommend?"** — operator delegated three design decisions to me. I made calls (disk location, hybrid mechanical+narrative, both-per-save-and-aggregated). All accepted without pushback. Good signal that the recommendations were sound. Future pattern: when operator says "what do you recommend," give a confident single recommendation with reasoning, not options.

---

## DEFERRED ITEMS

1. **Add LESSONS.md to PRIME.md's "read at session start" list.** Skipped now because only one session's lessons exist. Revisit when 3-4 sessions of lessons accumulated — at that point LESSONS becomes pre-task reading that prevents repeats. Trigger: ~3-4 ctx runs from now.

2. **Backup verification cadence for brain repo.** Per project instructions open decision #6. Brain is now backed up to GitHub + local SSDs + WD Archive. Monthly test restore? Quarterly? Decide after first time something actually breaks. Trigger: first real recovery scenario, or 90 days from now, whichever comes first.

3. **Update HANDOFF.md currency note in project instructions.** Open decision #7. HANDOFF.md is now current as of this session — but the project-instructions file still references the stale "Chassis build pending" version. Should update the project instructions file when next editing it. Trigger: next project-instructions edit cycle.

4. **Phase 1 step 10c (cost chart) is next.** token_usage now has real data, chart can populate. Operator's call whether to continue Phase 1 infrastructure or pivot to first-site / revenue work. Trigger: next CC session that isn't infrastructure.

5. **Phase 1 open items: 17a (silver platters — may defer), 17b (slash commands — quick win), 18 (war room discuss mode), 19 (failure modes per agent).** All TBD per build plan. Trigger: after 10c decision.

6. **First site target still open** — leaning PLR/Etsy/Gumroad. Does not block chassis. Trigger: when operator wants to stop building infrastructure and start building revenue.

---

## What's next

Immediate next action (operator's call):

**Option A — Continue Phase 1 infrastructure.** Step 10c (cost chart on dashboard) is the natural next step. token_usage table has data now.

**Option B — Pivot to first-site / revenue work.** Phase 1 chassis is infrastructure; it doesn't pay the $200/mo bill. First-site target is the open decision blocking revenue work.

**Option C — Smaller polish items.** 17b (slash commands beyond ctx) is a quick win. Could knock out before deciding A vs B.

Recommendation: Operator should decide A or B explicitly. The research-before-building pattern flagged in operator's profile suggests it's easy to stay in infrastructure-mode indefinitely. Now is a natural seam to pivot.

---

## Where to resume

**Last context save before this one:** `AITEAM_Context_Save_2026-05-15_SitePortfolio-SSGAgentSpec.md` (May 15 earlier session)

**Active state:**
- Phase 1 mid-phase
- ctx system + brain git-init both ✅ this session
- HANDOFF.md current as of 2026-05-15 ~13:16 PDT
- Working tree on ~/brain has uncommitted changes from the /ctx run that will be picked up by tonight's 23:55 cron — do not force-push

**Project files updated this session (Claude.ai project):** none. This context save is the only artifact for project knowledge. Operator should upload this to project knowledge to make it available to future Claude.ai sessions.
