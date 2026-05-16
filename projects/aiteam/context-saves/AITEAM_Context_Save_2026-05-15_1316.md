# AITEAM Context Save — 2026-05-15_1316
**Generated:** 2026-05-15T13:16:31-0700
**Since last save:** 2026-05-15 00:41:36
**Session topic:** Git-init the `~/brain/` Obsidian vault — first offsite backup chain. SSH key gen + GitHub repo wiring + nightly autocommit cron.

---

## Mechanical record

### Git activity since last save
```
3c73840 docs: note brain vault is now git-tracked
cbbd5ba Context save system: ctx.sh + harvest + /ctx + CLAUDE.md
```

### Files changed
```
A	.claude/commands/ctx.md
A	CLAUDE.md
A	scripts/auto-harvest-to-library.sh
A	scripts/ctx.sh
M	CLAUDE.md
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
- **SSH (not HTTPS) for the GitHub remote.** Operator-controlled machine with physical-access as the trust boundary; no passphrase on the key. Alternative considered: HTTPS with personal-access-token — rejected because PAT rotation is operationally noisier than key revocation.
- **Bundle the `gitignore` update (`*.log` + `inbox/`) into the initial commit** instead of a separate commit. Same logical change ("what counts as the initial state of the repo"). Cleaner one-commit history for the bootstrap.
- **Local git identity (`mdp280028-ui`) for `~/brain/` only**, not global. The `~/agents/` repo continues to use the existing `AITEAM Builder` identity. Per-repo identity scoping keeps the two project histories cleanly attributable.
- **Exclude `*.log` and `projects/aiteam/inbox/` from version control** after I flagged them in the staged file list. Reasoning: logs are regenerable and produce noisy diffs; inbox is staging-by-design (items flow in → librarian sorts → items flow out, with `audit_log` capturing the events).
- **Pre-seed `~/.ssh/known_hosts` with GitHub's ed25519 key via `ssh-keyscan`** before the first `ssh -T` test. Non-interactive contexts can't answer the host-fingerprint prompt; this is the standard secure bootstrap pattern.

## Lessons learned
- **Pre-flight checks catch blockers before they're expensive.** Step 1 scoping discovered `~/.ssh/` didn't exist on this machine — flagging it at the top of the build let SSH-key generation happen in parallel with the operator's manual GitHub steps. Catching it at step 6 (push) would have stalled the chain.
- **Verify operator-stated assumptions before relying on them.** The build prompt said "the ust-contractors repo uses SSH on this machine, so the key should already be there." It wasn't. Don't pattern-match to "this should already work" — check.
- **Pause points are leverage.** Reporting the staged file list before the initial commit gave the operator a chance to opt `dashboard.log` and `inbox/` out of permanent history. Skipping the pause would have committed both with no easy undo.
- **`ssh -T git@github.com` exits non-zero on success.** GitHub doesn't provide shell access, so SSH disconnects with exit 1 after authenticating. Look for `Hi <user>! You've successfully authenticated` in stdout, not exit code 0.
- **`ssh-keyscan -t <key-type> <host>` is the right way to pre-populate `known_hosts` for non-interactive SSH.** Avoids the interactive yes/no fingerprint prompt without disabling host verification.
- **Specific token-pattern regex beats loose secret-keyword grep.** `(sk-[a-zA-Z0-9]{20,}|ghp_[a-zA-Z0-9]{20,}|xoxb-[a-zA-Z0-9-]{30,}|AKIA[A-Z0-9]{16})` catches actual credentials. Loose `secret|api_key|password` returns false positives from any doc that mentions secrets management.
- **Per-repo `git config user.name` works without `--global`.** Useful for machines that host repos belonging to multiple identities. Don't pollute global identity for one project.
- **`git rm --cached` (without `-r` for files, with `-r` for dirs) removes from index without touching working tree** — the right unstage pattern when adjusting gitignore mid-build.

## Operator corrections
- **Operator added two `.gitignore` rules I didn't propose** in the staged-file pause: `*.log` (regenerable, noisy diffs) and `projects/aiteam/inbox/` (staging-by-design, `audit_log` captures the events). I caught the candidates and surfaced them; operator made the calls. Worth noting as a future default: any "scratch" or "log" subtree should be excluded unless there's an explicit reason to track it.
- **Operator chose SSH + no-passphrase explicitly** with the rationale: *"fine for an operator-controlled machine where physical access is the security boundary. If operator later wants a passphrase, they can add one with `ssh-keygen -p`."* — captures the threat model cleanly. The key on this machine is the trust anchor; physical compromise is the only meaningful attack.

## What's next
- **Tonight 23:55**: first scheduled `brain-autocommit.sh` run via cron. If anything in `~/brain/` changed during the day, it'll commit + push automatically. Log lands at `~/agents/logs/brain-autocommit.log`.
- **Next `ctx` invocation** will exercise the "incremental since last save" code path (it's been first-save-only until now). The current save acts as that baseline.
- **Phase 2 triggers still apply**: 17a (silver platters) unblocks when first AITEAM site is live with 30 days of GA4; 10b (kanban) unblocks when `mission_tasks` has rows. Neither met yet.
- **No active build threads.** Standing by for the next operator prompt.

---

## Where to resume
**Last context save before this one:** /Users/mmm2/brain/projects/aiteam/context-saves/AITEAM_Context_Save_2026-05-15_0039.md
**Active HANDOFF.md state:** # AITEAM Project Handoff **Last updated:** 2026-05-15 00:42 **Last session summary:** Phase 1 closed; built and verified context-save system (ctx.sh + harvest + slash command + handoff template).  --- 
